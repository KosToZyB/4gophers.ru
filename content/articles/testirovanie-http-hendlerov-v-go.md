+++
date = "2016-03-30T17:11:05+03:00"
draft = false
title = "Тестирование HTTP хендлеров в Go"

+++

<p>Перевод статьи "<a href="http://elithrar.github.io/article/testing-http-handlers-go/">Testing Your (HTTP) Handlers in Go</a>".</p>

<p>Вы пишите веб-сервис на Go и, конечно же, вы хотите тестировать ваши хендлеры. У вас используется пакет <a href="https://golang.org/pkg/net/http/">net/http</a>, но вы не знаете как протестировать возврат корректных кодов в ваших хендлерах, правильность заголовков HTTP и насколько верно был сформирован ответ клиенту.</p>

<p>Давайте обсудим эти вопросы и посмотрим как можно писать более тестируемые приложения с использованием внедрения зависимостей и моков.</p>

<h3>Стандартные хендлеры</h3>

<p>Для начала, напишем простой тест, который будет проверят код ответа. В нашем случает это должен быть 200 ответ.</p>

<pre><code>// handlers.go
package handlers

// http.HandleFunc("/health-check", HealthCheckHandler)
func HealthCheckHandler(w http.ResponseWriter, r *http.Request) {
    // Очень простой хендлер проверки состояния.
    w.WriteHeader(http.StatusOK)
    w.Header().Set("Content-Type", "application/json")

    // В будущем мы хотим сообщать сообщать о состоянии
    // базы данных или кеша (например Redis) выполняя 
    // простой PING и отдавать все это в запросе
    io.WriteString(w, `{"alive": true}`)
}
</code></pre>

<p>И вот наш тест:</p>

<pre><code>// handlers_test.go
package handlers

import (
    "net/http"
    "testing"
)

func TestHealthCheckHandler(t *testing.T) {
    // Создаем запрос с указанием нашего хендлера. Нам не нужно
    // указывать параметры, поэтому вторым аргументом передаем nil
    req, err := http.NewRequest("GET", "/health-check", nil)
    if err != nil {
        t.Fatal(err)
    }

    // Мы создаем ResponseRecorder(реализует интерфейс http.ResponseWriter)
    // и используем его для получения ответа
    rr := httptest.NewRecorder()
    handler := http.HandlerFunc(HealthCheckHandler)

    // Наш хендлер соответствует интерфейсу http.Handler, а значит
    // мы можем использовать ServeHTTP и напрямую указать 
    // Request и ResponseRecorder
    handler.ServeHTTP(rr, req)

    // Проверяем код
    if status := rr.Code; status != http.StatusOK {
        t.Errorf("handler returned wrong status code: got %v want %v",
            status, http.StatusOK)
    }

    // Проверяем тело ответа
    expected := `{"alive": true}`
    if rr.Body.String() != expected {
        t.Errorf("handler returned unexpected body: got %v want %v",
            rr.Body.String(), expected)
    }
}
</code></pre>

<p>Как видите, Go'шные пакеты <a href="https://golang.org/pkg/testing/">testing</a> и <a href="https://golang.org/pkg/net/http/httptest/">httptest</a> делают тестирование наших хендлеров очень простым. Мы подготавливаем <code>*http.Request</code>, a <code>*httptest.ResponseRecorder</code> и затем используем их для проверки работы самого хендлера(коды, тело ответа и т.д.).</p>

<p>Если ваш хендлер ожидает определенные параметры, то это не проблема:</p>

<pre><code>    // например GET /api/projects?page=1&amp;per_page=100
    req, err := http.NewRequest("GET", "/api/projects",
        // url.Values это просто map[string][]string
        url.Values{"page": {"1"}, "per_page": {"100"}})

    if err != nil {
        t.Fatal(err)
    }

    // Наш хенлер может ожидать определенный ключ для API 
    req.Header.Set("Authorization", "Bearer abc123")

    // Заем вызываем handler.ServeHTTP(rr, req) как в примере выше.
</code></pre>

<p>Кроме того, если есть необходимость проверить как ваш хендлер или миделваре изменяют и мутируют данные запроса, то можно создать анонимную функцию, у которой будет доступ к внешним переменным которые вы сможете проверить в рамках своего теста.</p>

<pre><code>    // Declare it outside the anonymous function
    var token string
    handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request){
        // ':=' operator so we don't shadow our token variable above.
        // Обратите внимание, что тут нужно обязательно использовать 
        // '=' , а не ':=' чтобы избежать затенения переменной
        token = GetToken(r)
        // Мы устанавливаем заголовок как и в примере выше
        w.Header().Set("Content-Type", "application/json")
    })

    // Запускаем хендлер, проверяем коды, тело ответа и т.д.

    if token != expectedToken {
        t.Errorf("token does not match: got %v want %v", token, expectedToken)
    }

    if ctype := rr.Header().Get("Content-Type"); ctype != "application/json") {
        t.Errorf("content type header does not match: got %v want %v",
            ctype, "application/json")
    }
</code></pre>

<p>Маленький лайфхак: для строк <code>application/json</code> или <code>Content-Type</code> можно использовать константы, тогда вам не прийдется заново набирать(и ошибаться), а пользоваться автоподстановкой. Опечатка в тесте может быть очень фатальна, потому что вы будете уверенны, что все протестировано и работает правильно.</p>

<p>Не забывайте, что нужно тестировать и отказы, и исключительные ситуации в том числе. Проверяйте, возвращает ли ваш хендлер ошибку, когда это необходимо (например HTTP 403, или HTTP 500)</p>

<h3>Заполняем context.Context в тестах</h3>

<p>Как быть с хендлерами, которые ожидают данные из context.Context? Есть ли способ руками создать контекст и заполнить его различными данными, например типом пользователя и токеном?</p>

<blockquote>
  <p>Предпложим, что у вас есть кастомный хендлер который предоставляет метод <code>ServeHTTPC(context.Context, http.ResponseWriter, *http.Request)</code>. В Go 1.7 <a href="https://github.com/golang/go/issues/14660">context.Context добавят в http.Request</a> и это сделает жизнь значительно проще.</p>
</blockquote>

<p>Для примера ниже я буду использовать <a href="https://goji.io/">Goji</a> роутер. Он предоставляет возможность использовать хендлеры с <code>context.Context</code>. Тем не менее, описанный способ подойдет для большинства роутеров/фрейиворков, которые работают с <code>context.Context</code>.</p>

<pre><code>func TestGetProjectsHandler(t *testing.T) {
    req, err := http.NewRequest("GET", "/api/users", nil)
    if err != nil {
        t.Fatal(err)
    }

    rr := httptest.NewRecorder()
    // например func GetUsersHandler(ctx context.Context, 
    //    w http.ResponseWriter, r *http.Request)
     goji.HandlerFunc(GetUsersHandler)

    // Создаем новый context.Context и заполняем его данными
    ctx = context.Background()
    ctx = context.WithValue(ctx, "app.auth.token", "abc123")
    ctx = context.WithValue(ctx, "app.user",
        &amp;YourUser{ID: "qejqjq", Email: "user@example.com"})

    // Указываем на контекст *http.Request и ResponseRecorder.
    handler.ServeHTTPC(ctx, rr, req)

    // Проверяем код, тело ответа и т.д.
    if status := rr.Code; status != http.StatusOK {
        t.Errorf("handler returned wrong status code: got %v want %v",
            status, http.StatusOK)
    }

    // Тут мы можем проверить какие данные изменились/добавились
    // в нашем context.Contex
    if id , ok := ctx.Value("app.req.id").(string); !ok {
        t.Errorf("handler did not populate the request ID: got %v", id)
    }
}
</code></pre>

<h3>Мокаем обращения к базе данных</h3>

<p>Наши хендлеры используют некоторый интерфейс <code>datastore.ProjectStore</code> с тремя методами (<code>Create</code>, <code>Get</code>, <code>Delete</code>). Мы можем замокать этот интерфейс для наших тестов определенным образом и проверить правильно ли отдаются HTTP коды.</p>

<blockquote>
  <p>Если вы хотите больше узнать о использовании интерфейсов для абстракции работы с базой данных, то рекомендую прочитать <a href="https://robots.thoughtbot.com/interface-with-your-database-in-go">статью Thoughtbot</a> и <a href="http://www.alexedwards.net/blog/organising-database-access#using-an-interface">статью от fAlex Edwards</a>.</p>
</blockquote>

<pre><code>// handlers_test.go
package handlers

// Возвращает ошибки в каждом методе
type badProjectStore struct {
    // Это конкретный тип, который реализует datastore.ProjectStore.
    // Мы встроили его сюда, у нас автоматически добавились необходимые 
    // методы и теперь наш тип badProjectStore удовлетворяет
    // интерфейсу datastore.ProjectStore, без необходимости добавлять заглушки 
    // каждый метод (в зависимости от теста у нас будет использоваться
    // только те или иные методы)
    *datastore.Project
}

func (ps *badProjectStore) CreateProject(project *datastore.Project) error {
    return datastore.NetworkError{errors.New("Bad connection"}
}

func (ps *badProjectStore) GetProject(id string) (*datastore.Project, error) {
    return nil, datastore.NetworkError{errors.New("Bad connection"}
}

func TestGetProjectsHandlerError(t *testing.T) {
    var store datastore.ProjectStore = &amp;badProjectStore{}

    // Мы выполняем внедрение нашего окружения в хендлеры.
    // Ref: http://elithrar.github.io/article/http-handler-error-handling-revisited/
    env := handlers.Env{Store: store, Key: "abc"}

    req, err := http.NewRequest("GET", "/api/projects", nil)
    if err != nil {
        t.Fatal(err)
    }

    rr := httptest.Recorder()
    // Handler это кастомный тип, который работает с env and a http.Handler
    // GetProjectsHandler обращается к GetProject и должен вернуть 500
    // в случае ошибки.
    Handler{env, GetProjectsHandler)
    handler.ServeHTTP(rr, req)

    // Тут мы проверяем, что наш хендлер отработал с ошибкой.
    if status := rr.Code; status != http.StatusInternalServeError {
        t.Errorf("handler returned wrong status code: got %v want %v"
            rr.Code, http.StatusOK)
    }

    // Мы должны проверить, что в теле JSON тоже есть упоминание ошибки.
    expected := []byte(`{"status": 500, "error": "Bad connection"}`)
    if !bytes.Equals(rr.Body.Bytes(), expected) {
        t.Errorf("handler returned unexpected body: got %v want %v",
        rr.Body.Bytes(), expected)
    }
</code></pre>

<p>Пример выше несколько переусложнен, но нужно обратить внимание на несколько ключевых моментов:</p>

<ul>
<li>В нашей заглушке нет никакой работы с базой данных. Модульные тесты в пакете <code>handlers</code> не должны ничего знать про базу.</li>
<li>Мы создали заглушку, которая возвращает ошибку и это позволило проверить работу нашего хендлера в исключительной ситуации. Мы проверили какой код возвращается и что пишется в тело ответа.</li>
<li>Как вы могли догадаться, можно добавить "хорошую" заглушку на базе <code>*datastore.Project</code> и протестировать в таком виде, например выполнить кодирование/декодирование в JSON. Таким образом, мы могли бы отловить ситуации, когда внесение изменений может сломать совместимость с  <code>encoding/json</code>.</li>
</ul>

<h3>Что дальше?</h3>

<p>Конечно, это не исчерпывающий туториал, но после прочтения этой статьи вы будете знать с чего начинать. Если вы застряли на более сложном и комплексном примере, <a href="http://4gophers.ru/slack">обратитесь к сообществу</a> или посмотрите исходники пакетов, которые используют пакет <a href="https://godoc.org/net/http/httptest?importers">httptest</a> via GoDoc.</p>

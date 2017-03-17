+++
date = "2015-07-01T13:43:01+03:00"
draft = false
title = "Хендлеры и избавление от глобальных переменных"

+++

<p>Перевод статьи "<a href="https://elithrar.github.io/article/custom-handlers-avoiding-globals/">Custom Handlers and Avoiding Globals in Go Web Applications</a>"</p>

<p>Пакет <a href="http://golang.org/pkg/net/http/"><code>net/http</code></a> невероятно гибкий, благодаря повсеместному использованию интерфейса <a href="http://golang.org/pkg/net/http/#Handler"><code>http.Handler</code></a>. Построение приложения на основе интерфейсов позволяет вам проще расширять его, предоставлять различную реализацию и позволит поддерживать совместимость с другими пакетами в дикой природе. Так как дефолтная реализация довольна проста, мы разберемся как построить наш собственный тип хендлера(для избавления от повторяющихся обработок ошибок) и как его расширить для реализации некоторого "контекста" с пулом базы данных, списком шаблонов и всего такого, что позволит нам избавиться от глобальных переменных.</p>

<h3>Создаем кастомный тип хендела</h3>

<p><code>net/http</code> предоставляет базовый тип <code>HandlerFunc</code>, по сути, это просто функция <code>func(w http.ResponseWriter, r *http.Request)</code>. Это довольно простой для понимания и достаточно распространенный подход, который покрывает большинство простых юзкейсов. Но тут скрывается несколько неприятных проблем:</p>

<ul>
<li>Мы не можем передать дополнительные параметры для <code>http.HandlerFunc</code>.</li>
<li>И нам необходимо выполнять однотипную проверку ошибок в каждом хендлере.</li>
</ul>

<p>Если вы новичек в Go, то вам может показаться, что нет простого и быстрого решения этих проблем, которое еще и будет совместимо с другими HTTP пакетами. Но на самом деле это не так.</p>

<p>Мы создадим собственный тип хендлера, который реализует интерфейс <code>http.Handler</code>(читайте как реализующий метод <code>ServeHTTP(http.ResponseWriter, *http.Request)</code>) и будет совместим с чистым <code>net/http</code> HTTP миделварями(вроде <a href="https://github.com/justinas/nosurf">nosurf</a>) и роутерами/фреймворками(например <a href="http://www.gorillatoolkit.org/pkg/mux">gorilla/mux</a> или <a href="https://goji.io/">Goji</a>).</p>

<p>Для начала, давайте формализируем проблему:</p>

<pre><code class="go">func myHandler(w http.ResponseWriter, r *http.Request) {
    session, err := store.Get(r, "myapp")
    if err != nil {
        http.Error(w, http.StatusText(http.StatusInternalServerError), 
                        http.StatusInternalServerError)
        return 
        // Если забудете про return, то код в хендлере 
        // продолжит выполняться.
    }

    id := // Получаем id из get параметров, 
    // преобразуем с помощью strconv.Atoi, 
    // проверяем ошибки преобразования

    post := Post{ID: id}
    exists, err := db.GetPost(&amp;post)
    if err != nil {
        http.Error(w, http.StatusText(http.StatusInternalServerError), 
                        http.StatusInternalServerError)
        return // Снова повторяем обработку ошибок
    }

    if !exists {
        http.Error(w, http.StatusText(http.StatusNotFound), http.StatusNotFound)
        return // ... и опять.
    }

    err = renderTemplate(w, "post.tmpl", post)
    if err != nil {
        // Ага, опять тоже самое...
    }
}
</code></pre>

<p>Проблема даже не столько в многословности, сколько в трудности отлова багов в таком коде. Если мы не выполним <code>return</code> после ошибочного запроса в базу или после проверки неправильного пароля, то выполнение кода продолжиться и это приведет к непредсказуемым результатам. В лучшем случае мы передадим в шаблон пустую структуру, что немного расстроит пользователя. В худшем - мы пометим запрос как HTTP 401 (неавторизированный), но при этом покажем любому пользователю все, что доступно только для залогиненного.</p>

<p>К счастью, мы можем пофиксить эту проблему очень просто и красиво, добавив тип хендлера, который проверяет и возвращает ошибку:</p>

<pre><code class="go">type appHandler func(http.ResponseWriter, *http.Request) (int, error)

// Наш appHandler соответствует http.Handler 
func (fn appHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    if status, err := fn(w, r); err != nil {
        // Также, мы можем залогировать ошибку
        // Например: log.Printf("HTTP %d: %v", err)
        switch status {
        // Тут мы можем обрабатывать любые ошибки
        // в том числе и кастомные для определенный кодов.
        case http.StatusNotFound:
            notFound(w, r)
        case http.StatusInternalServerError:
            http.Error(w, http.StatusText(http.StatusInternalServerError), 
                            http.StatusInternalServerError)
        default:
        // Перехватываем остальные ошибки, для которых 
        // нет специальных обработчиков
            http.Error(w, http.StatusText(http.StatusInternalServerError), 
                            http.StatusInternalServerError)
        }
}

func myHandler(w http.ResponseWriter, r *http.Request) (int, error) {
    session, err := store.Get(r, "myapp")
    if err != nil {
        // Так намного лучше
        return http.StatusInternalServerError, err
    }

    post := Post{ID: id}
    exists, err := db.GetPost(&amp;post)
    if err != nil {
        return http.StatusInternalServerError, err
    }

    // Мы можем сократить это. Так как renderTemplate возвращает `error`
    // наш метод ServeHTTP вернет HTTP 500, и не будет пытаться
    // отдать сломанный шаблон с кодом HTTP 200(реализацию renderTemplate
    // посмотрите ниже). Если все рендерится хорошо, то все 
    // пойдет по плану.
    return http.StatusOK, renderTemplate(w, "post.tmpl", data)
}

func main() {
    // Кастуем myHandler к appHandler
    http.Handle("/", appHandler(myHandler))
    http.ListenAndServe(":8000", nil)
}
</code></pre>

<p>Конечно, в этом нет ничего нового и фантастического. Andrew Gerrand <a href="http://blog.golang.org/error-handling-and-go">описывал подобный прием</a> в оф. блоге еще в 2011 году. Наша реализация лишь немного адаптированна для небольшого улучшения обработки ошибок. Я предпочитаю возвращать конкретный тип (<code>int</code>, <code>err</code>), так как я считаю это более идиоматическим. Но вы можете создать свой собственный тип ошибок(только давайте пока оставим этот код достаточно простым).</p>

<h3>Расширяем наш кастомный хендлер</h3>

<p>Давайте заранее определимся, что глобальные переменные - это зло воплоти. Вы не можете контролировать что и когда модифицирует их, очень проблематично отследить их текущее состояние, кроме того, глобальные переменные могут быть не защищенными он при конкурентном доступе. Конечно, они могут быть удобными при правильном использовании. И многие достойные проекты построены с использованием глобальных переменных(например <a href="http://golang.org/doc/articles/wiki/#tmp_10">вот</a> и <a href="http://www.gorillatoolkit.org/pkg/schema">вот</a>). Тип <code>*sql.DB</code> из пакета <code>database/sql</code> может безопасно использоваться глобально, так как он представляет собой пул и защищен мютексами, мапы (например мапы для шаблонов) можно читать(но не писать) конкурентно, хранилище для сессий также работает аналогично <code>database/sql</code>.</p>

<p>После прилива вдохновения, полученного от статьи <a href="https://medium.com/@benbjohnson/structuring-applications-in-go-3b04be4ff091">@benbjohnson о структурировании Go приложения</a> и дебатов с товарищем Gopher на редите(<a href="http://www.jerf.org/iri/post/2929">который использует аналогичный подход</a>), я решил пристальнее присмотреться к моему коду(в котором было несколько глобальных объектов) и порефакторить его, добавив структуру для контекста, которая будет использоваться в моих хендлерах. В большинстве случаев, тип который я описал выше будет работать гладко, но вы можете отхватить проблемы, если ваш контекст будет использоваться за рамками хендлеров.</p>

<p>Вот список глобальных переменных, которое у меня были изначально:</p>

<pre><code class="go">var (
    decoder   *schema.Decoder
    bufpool   *bpool.Bufferpool
    templates map[string]*template.Template
    db        *sqlx.DB
    store     *redistore.RediStore
    mandrill  *gochimp.MandrillAPI
    twitter   *anaconda.TwitterApi
    log       *log.Logger
    conf      *config // конфиг приложения: хосты, порты и т.д.
)
</code></pre>

<p>И так, как же нам, используя наш кастомный хендлер, перенести все эти глобальные переменные в контекст, который мы можем передавать в наши хендлеры и использовать в рамках вызова <code>ServeHTTP</code>? Как нам получить доступ к мапе с шаблонами для отрисовки красивой ошибки с кастомным шаблоном? И как нам сделать все это совместимым с типом <code>http.Handler</code>?</p>

<pre><code class="go">package main

import (
    "fmt"
    "log"
    "net/http"

    "html/template"

    "github.com/gorilla/sessions"
    "github.com/jmoiron/sqlx"
    "github.com/zenazn/goji/graceful"
    "github.com/zenazn/goji/web"
)


// appContext содержит наш локальный контекст:
// доступ к базе, хранилище для сессий, список
// шаблонов и все остальное, что необходимо 
// внутри наших хендлеров. Мы создаем все эти
// инстансы в функции main и указываем ссылки на них.
type appContext struct {
    db        *sqlx.DB
    store     *sessions.CookieStore
    templates map[string]*template.Template
    decoder *schema.Decoder
    // ... и все остальные глобальные объекты.
}

// Мы превратили наш первоначальный appHandler 
// в структуру с двумя полями:
// - Функция с типом, аналогичным нашему типу хенделера(только она
 принимает еще и *appContext)
// - Поле типа *appContext
type appHandler struct {
    *appContext
    h func(*appContext, http.ResponseWriter, *http.Request) (int, error)
}

// Наш метод ServeHTTP почти не изменился, за исключением, что 
// в нем теперь есть доступ к полю *appContext(шаблоны, логи и т.д.).
func (ah appHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    // Мы можем передавать ah.appContext как параметр в наш хендлер.
    status, err := ah.h(ah.appContext, w, r)
    if err != nil {
        log.Printf("HTTP %d: %q", status, err)
        switch status {
        case http.StatusNotFound:
            http.NotFound(w, r)
            // И если нам необходимо отобразить пользователю
            // вменяемую страницу ошибки, то мы можем использовать
            // наш контекст, например:
            // err := ah.renderTemplate(w, "http_404.tmpl", nil)
        case http.StatusInternalServerError:
            http.Error(w, http.StatusText(status), status)
        default:
            http.Error(w, http.StatusText(status), status)
        }
    }
}

func main() {
    // В нашем примере указан 'nil', но в реальности
    // нам нужно заасайнить множество различных значений 
    // или использовать функцию конструктор 
    // (NewAppContext(conf config) *appContext) для инициализации
    // приложения, например, из конфигурационного файла.
    context := &amp;appContext{db: nil, store: nil} // Для примера

    r := web.New()
    // Тут мы указываем инстанс нашего контекста и наш хендлер.
    r.Get("/", appHandler{context, IndexHandler})

    graceful.ListenAndServe(":8000", r)
}

func IndexHandler(a *appContext, 
                  w http.ResponseWriter,
                  r *http.Request) (int, error) {

    // В нашем хендлере теперь есть доступ к объектам нашего контекста.
    // Например, мы можем сделать запрос в базу: err := a.db.GetPosts()
    fmt.Fprintf(w, "IndexHandler: db is %q and store is %q", a.db, a.store)
    return 200, nil
}
</code></pre>

<p>Все осталось хорошо читаемым. Мы хорошо разобрались в системе типов и существующих интерфейсов, поэтому, если возникнет такая необходимость, мы также можем использовать обычные <code>http.HandlerFunc</code>. Также, наши хендлеры достаточно легко обернуть во что-то еще, принимающее <code>http.Handler</code>, поэтому мы можем легко использовать их в работе с Goji или gorilla/mux, нам не прийдется переписывать их полностью. Не забывайте про правильное использование и безопасное использование различных типов и применяйте пакет <a href="http://golang.org/pkg/sync/">sync</a>, если это необходимо.</p>

<p>Короче говоря, это работает. Мы сократили повторяющиеся проверки ошибок, мы избавились от глобальных переменны и код по прежнему читабельный.</p>

<h3>Дополнения</h3>

<ul>
<li>Стоит прочитать замечательную <a href="http://justinas.org/best-practices-for-errors-in-go/">статью от Justina про обработку ошибок</a> в Go, особенно секцию про реализацию кастомного <code>httpError</code>.</li>
<li>Пишите миделваре для вашего Go приложения? Используйте функции типа <code>func(http.Handler) http.Handler</code> и вы получите универсальный способ создания различных миделварь. Единственное "исключение" в таком подходе - это проблема передачи состояний между хендлерами(например, CSRF токен), которая возникает когда вам нужно привязываться к контексту запроса(например, как это организовано в <a href="https://godoc.org/github.com/zenazn/goji/web#C">web.C</a> для Goji или <a href="http://gorillatoolkit.org/pkg/context">gorilla/context</a>). Старайтесь избегать большого количества миделварей.</li>
<li><a href="https://gist.github.com/elithrar/fbf3772e6a0a6f997d8a">Вот вам пример</a>, как можно отлавливать ошибки до рендеринга шаблона(в кратце - используйте буферный пул).</li>
<li>И конечно же финальные <a href="https://gist.github.com/elithrar/5aef354a54ba71a32e23">исходники последней реализации</a>, которые вы можете комментировать.</li>
</ul>

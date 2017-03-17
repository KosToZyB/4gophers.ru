+++
date = "2016-05-10T02:45:02+03:00"
draft = false
title = "Авторизация в Go с использованием JWT"

+++

<p>Частичный перевод статьи "<a href="https://auth0.com/blog/2016/04/13/authentication-in-golang/">Authentication in Golang with JWTs</a>".</p>

<p>Go замечательный выбор для создания быстрых и масштабируемых API. Стандартный пакет <code>net/http</code> предоставляет весь необходимый функционал, а при использовании различны дополнительных пакетов, например <a href="http://www.gorillatoolkit.org/">Gorilla Toolkit</a>, вы сможете сократить время написания вашего сервиса до минимума. В этой статье мы разберем как защитить свое Go приложение с использованием JSON веб токенов(JWT). А если нет желания перечитывать все эти тонны букв, то можете сразу читать код в <a href="https://github.com/auth0-blog/auth0-golang-jw">репозитории на GitHub</a>.</p>

<p>Язык программирования Go разработан компанией Google для разработки современного программного обеспечения. Этот язык разрабатывался для более быстрого и быстрого и эффективного решения большинства задач. Основные характеристики этого языка:</p>

<ul>
<li>Строгая типизация и сборщик мусора.</li>
<li>Невероятно быстрая компиляция</li>
<li>Встроенная поддержка конкурентного программирования</li>
<li>Обширная стандартная библиотека</li>
</ul>

<p>Синтаксис Go способствуем минимальному количеству нажатий на клавиши. Переменные можно легко объявлять и сразу инициализировать с помощью инструкции <code>:=</code>, точка с запятой не нужна и не существует сложной системы типов. Go очень "упрямый" язык. Конечный результат получается чистым, очень легко читается и легко понимается другими разработчиками.</p>

<h3>Golang песочница</h3>

<p>В этом туториале мы будем разрабатывать RESTful API на Go и вам, конечно же, понадобится знание основ самого языка. Изучение основ языка выходит за рамки этой статьи. Если вы новичок в Go, то можете пройти "<a href="https://tour.golang.org/welcome/1">Tour of Go</a>" в котором очень хорошо описаны базовые концепции Go такие как конкурентность. А уже после этого возвращайтесь к чтению этой статьи. Если у вас уже есть некоторый опыт использования Go, то давайте приступим к написанию API!</p>

<h3>Строим API на Go</h3>

<p>Go замечательно подходит для написания различных RESTful API. Стандартный пакет <code>net/http</code> предоставляет все ключевые функции для работы с http протоколом. Приложение, которое мы сейчас разрабатываем, называется "We R VR". Приложение обеспечивает обратную связь с разработчиком в тех играх, над которыми он работает.</p>

<p><img src="http://cdn.auth0.com/blog/go-auth/app-page.png" alt="" /></p>

<p>В Go идиоматически правильно использовать небольшие пакеты вместо больших фреймворков и стараться применять стандартную библиотеку везде где это возможно. Мы тоже будем использовать такой подход и наш код можно будет без боязни использовать в экосистеме Go. Для разработки мы будем использовать несколько пакетов, таких как <code>gorilla/mux</code> для роутинга и <code>dgrijalva/jwt-go</code> для работы с JSON веб токенами.</p>

<h3>Golang фреймворки</h3>

<p>Перед тем окунуться в код, хочу заметить, что идиоматическая правильность(стараться не использовать фреймворки) не означает, что на Go нет ни одного фреймворка. <a href="http://beego.me/">Beego</a>, <a href="https://gin-gonic.github.io/gin/">Gin Gionic</a>, <a href="https://labstack.com/echo">Echo</a> и <a href="https://revel.github.io/">Revel</a> предоставляют вполне традиционный функционал для разработки веб-приложений. Но, так как стандартный пакет <code>net/http</code> предоставляет обширные возможности, то все указанные выше фреймворки построены как надстройка над этим пакетом. И работа с любым из этих фреймворков не будет сильно отдалять вас от самого Go, как это бывает в других языках.</p>

<h3>Начинаем</h3>

<p>Начать стоит с файла <code>main.go</code>. Пока у нас будет только один файл, это позволит запускать наше приложение без явной сборки при внесении любого изменения в код. Мы сможем запускать наше приложение простой командой <code>go run</code> в терминале. Теперь мы готовы,чтобы начать кодить:</p>

<pre><code>package main

// Импортируем необходимые зависимости. Мы будем использовать
// пакет из стандартной библиотеки и пакет от gorilla
import (
    "net/http"

    "github.com/gorilla/mux"
)

func main() {
    // Инициализируем gorilla/mux роутер
    r := mux.NewRouter()

    // Страница по умолчанию для нашего сайта это простой html.
    r.Handle("/", http.FileServer(http.Dir("./views/")))

    // Статику (картинки, скрипти, стили) будем раздавать 
    // по определенному роуту /static/{file} 
    r.PathPrefix("/static/").Handler(http.StripPrefix("/static/", 
                                http.FileServer(http.Dir("./static/"))))

    // Наше приложение запускается на 3000 порту. 
    // Для запуска мы указываем порт и наш роутер
    http.ListenAndServe(":3000", r)
}
</code></pre>

<p>Необходимо создать две папки, которые мы указали в файле <code>main.go</code> и назвали их <code>views</code> и <code>static</code>. В папке <code>views</code> сразу можно создать файл с HTML для нашей главной страницы и назвать его <code>index.html</code>. Пока что это очень простая страница:</p>

<pre><code>&lt;!DOCTYPE html&gt;
&lt;head&gt;
  &lt;title&gt;We R VR&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
  &lt;h1&gt;Welcome to We R VR&lt;/h1&gt;
&lt;/body&gt;
</code></pre>

<p>Давайте убедимся, что наш сервер работает. Для этого выполним команду <code>go run main.go</code> из терминала. Если вы раньше не использовали пакет <code>gorilla/mux</code>, то вам придется установить его у себя с помощью команды <code>go get</code>, которая скачает и установит все необходимые зависимости. Как только приложение запуститься, откройте в браузере адрес <code>localhost:3000</code>. Если все сделано правильно
, то вы должны увидеть текст <em>Welcome to We R VR</em>. Дальше можно переходить к определению API.</p>

<h3>Определяем API</h3>

<p>Самое время для определения наших роутов. Для нашего демо приложения мы будем использовать только <code>GET</code> и <code>POST</code> запросы. В дополнение к роутам, мы реализуем хендлер <code>NotImplemented</code>. Это будет хендлер по умолчанию для всех роутов, которые мы пока не добавили функциональность. Продолжим добавлять код в <code>main.go</code>:</p>

<pre><code>func main(){
    //...

    // Наше API состоит из трех роутов
    // /status - нужен для проверки работоспособности нашего API
    // /products - возвращаем набор продуктов, 
    // по которым мы можем оставить отзыв
    // /products/{slug}/feedback - отображает фидбек пользователя по продукту
    r.Handle("/status", NotImplemented).Methods("GET")
    r.Handle("/products", NotImplemented).Methods("GET")
    r.Handle("/products/{slug}/feedback", NotImplemented).Methods("POST")

    //...
}

// Необходимо реализовать хендлер NotImplemented. 
// Этот хендлер просто возвращает сообщение "Not Implemented"
var NotImplemented = http.HandlerFunc(func(w http.ResponseWriter, r *http.Request){
  w.Write([]byte("Not Implemented"))
})
</code></pre>

<p>Неплохое начало реализации API. Давайте запустим сервер снова и попробуем обратиться по каждому роуту. Каждый роут должен вернуть код <code>200 OK</code> и сообщение <code>Not Implemented</code>. Если все хорошо, то дальше можем начинать добавлять функциональность.</p>

<h3>Добавляем функциональность</h3>

<p>У нас есть пачка роутов, но в данный момент они ничего не делают. Будем добавлять необходимую функциональность.</p>

<pre><code>//...

// Создадим новый тип Product. Мы будем использовать
// этот тип для для хранения информации о опытах VR
type Product struct {
    Id int
    Name string
    Slug string
    Description string
}

// Создадим наш каталог VR экспериментов и сохраним его в виде слайса.
var products = []Product{
    Product{Id: 1, Name: "Hover Shooters", Slug: "hover-shooters", 
    Description : "Shoot your way to the top on 14 different hoverboards"},
    Product{Id: 2, Name: "Ocean Explorer", Slug: "ocean-explorer", 
    Description : "Explore the depths of the sea in this one of a kind"},
    Product{Id: 3, Name: "Dinosaur Park", Slug : "dinosaur-park", 
    Description : "Go back 65 million years in the past and ride a T-Rex"},
    Product{Id: 4, Name: "Cars VR", Slug : "cars-vr", 
    Description: "Get behind the wheel of the fastest cars in the world."},
    Product{Id: 5, Name: "Robin Hood", Slug: "robin-hood", 
    Description : "Pick up the bow and arrow and master the art of archery"},
    Product{Id: 6, Name: "Real World VR", Slug: "real-world-vr", 
    Description : "Explore the seven wonders of the world in VR"}
}

// Хендлер StatusHandler будет срабатывать в тот момент момент, когда 
// пользователь обращается по роуту /status. Этот хендлер просто возвращает
// строку "API is up and running".
var StatusHandler = http.HandlerFunc(func(w http.ResponseWriter, r *http.Request){
    w.Write([]byte("API is up and running"))
})

// ProductsHandler срабатывает в момент вызова роута /products
// Этот хендлер возвращает пользователю список продуктов для оценки.
var ProductsHandler = http.HandlerFunc(func(w http.ResponseWriter, r *http.Request){
    // Конвертируем наш слайс продуктов в json
    payload, _ := json.Marshal(products)

    w.Header().Set("Content-Type", "application/json")
    w.Write([]byte(payload))
})

// Этот хендлер позволяет добавить позитивный или негативный отзыв 
// по конкретному продукту. Правильно было бы сохранить результат в базу 
// данных, но для демо нам это не нужно.
var AddFeedbackHandler = http.HandlerFunc(func(w http.ResponseWriter, r *http.Request){
  var product Product
  vars := mux.Vars(r)
  slug := vars["slug"]

  for _, p := range products {
    if p.Slug == slug {
        product = p
    }
  }

  w.Header().Set("Content-Type", "application/json")
  if product.Slug != "" {
    payload, _ := json.Marshal(product)
    w.Write([]byte(payload))
  } else {
    w.Write([]byte("Product Not Found"))
  }
})

//...
</code></pre>

<p>Теперь у нас есть хендлеры. Можем добавить их к определению наших роутов.</p>

<pre><code>func main(){
  //...
  r.Handle("/status", StatusHandler).Methods("GET")
  r.Handle("/products", ProductsHandler).Methods("GET")
  r.Handle("/products/{slug}/feedback", AddFeedbackHandler).Methods("POST")
  //...
}
</code></pre>

<h3>Хендлеры и прослойки</h3>

<p>В Go прослойки работают аналогично хендлерам. Если вы раньше не сталкивались с прослойками, то представляйте их как абстрактный код, который выполняется до срабатывания основной бизнес логики. К примеру, у вас может быть прослойка, которая выполняет логирование каждого запроса. Скорее всего, у вас нет желания добавлять одинаковый код во все ваши хендлеры. Вместо этого вы можете написать прослойку, которая добавляется перед основной функцией при определении роута.</p>

<p>Чуть позже мы будем использовать кастомный хендлер для защиты нашего API, но сейчас просто добавим глобальный хендлер, который реализует логирование информации по всем запросам. Мы будем использовать готовый код из пакета <code>gorilla/handlers</code>. Выглядеть это будет примерно так:</p>

<pre><code>package main

import(
    //...
    // Пакет с набором готовых хендлеров
    "github.com/gorilla/handlers"
    //...
)

func main(){
    //...
    // Обернем наш роутер функцией LoggingHandler. 
    // Таким образом нам будет доступен каждый запрос.
    http.ListenAndServe(":3000", handlers.LoggingHandler(os.Stdout, r))
}
</code></pre>

<p>Всего две строчки кода, а мы уже добавили функциональность логирования ко всему нашему приложению. Теперь, при каждом обращении к приложению, в терминале будет отображаться определенная информация, тип запроса, код ответа и время ответа на запрос. Если заинтересовались и хотите побольше узнать о этой прослойке/хендлере и о других возможностях пакета <code>gorilla/handlers</code>, то можете почитать <a href="http://www.gorillatoolkit.org/pkg/handlers">документацию</a>.</p>

<h4>Библиотеки прослоек</h4>

<p>Мы привязываемся к пакету <code>net/http</code> насколько это возможно в рамках нашей реализации. Но нельзя не упомянуть о большом числе различных пакетов, которые предоставляют альтернативные способы реализации прослоек и хендлеров, (например для Auth0). <a href="https://github.com/codegangsta/negroni">Negroni</a> и <a href="https://github.com/justinas/alice">Alice</a> - это две прекрасные альтернативы для реализации прослоек в Go.</p>

<h3>Защищаем наше API с помощью JSON веб токенов</h3>

<p>Давайте уже защитим наше приложение с помощью JWT. Мы сделаем это в два подхода. Для начала, реализуем простой пример, как в Go приложении работать с JSON веб токенами. В дальнейшем мы полностью реализуем все приложение, которое использует Auth0 аутентификацию.</p>

<p>Для базового примера у нас есть роутер, который генерирует новый JWT. И добавим прослойку, которая выполняет проверку токена.</p>

<pre><code>//...
func main(){
    //...
    r.Handle("/get-token", GetTokenHandler).Methods("GET")
    //...
}
    // Глобальный секретный ключ
    var mySigningKey = []byte("secret")

    var GetTokenHandler = http.HandlerFunc(func(w http.ResponseWriter, 
                                            r *http.Request){
        // Создаем новый токен
        token := jwt.New(jwt.SigningMethodHS256)

        // Устанавливаем набор параметров для токена
        token.Claims["admin"] = true
        token.Claims["name"] = "Ado Kukic"
        token.Claims["exp"] = time.Now().Add(time.Hour * 24).Unix()

        // Подписываем токен нашим секретным ключем
        tokenString, _ := token.SignedString(mySigningKey)

        // Отдаем токен клиенту
        w.Write([]byte(tokenString))
    })
</code></pre>

<p>Этот пример позволяет генерировать новые токены и устанавливать необходимые параметры для токенов. Это очень простой пример, то мы просто хардкодим все параметры и не требуем аутентификации для получения этого токена. Теперь можно доработать API. Прежде всего нам необходимо создать прослойку, которая будет выполнять проверку нашего токена.</p>

<pre><code>//...
var jwtMiddleware = jwtmiddleware.New(jwtmiddleware.Options{
    ValidationKeyGetter: func(token *jwt.Token) (interface{}, error) {
        return mySigningKey, nil
    },
    SigningMethod: jwt.SigningMethodHS256,
})
</code></pre>

<p>Также, необходимо чтобы все наши роуты использовали эту прослойку.</p>

<pre><code>//...
func main(){
    //...
    r.Handle("/status", NotImplemented).Methods("GET")
    // Добавляем прослойку к products и feedback роутам, все остальные
    // роуты у нас публичные
    r.Handle("/products", 
        jwtMiddleware.Handler(ProductsHandler)).Methods("GET")
    r.Handle("/products/{slug}/feedback", 
        jwtMiddleware.Handler(AddFeedbackHandler)).Methods("POST")
    //...
}
</code></pre>

<p>Теперь можно собирать наше приложение, запустить его и попытаться перейти по URL <code>localhost:3000/products</code>. Вы увидите сообщение <em>Required Authorization token not found</em>. Это значит, что наша прослойка работает. Если перейти по URL <code>localhost:3000/get-token</code>, то вы получите токен. Скопируйте этот токен. Теперь вы можете отправить запрос на <code>localhost:3000/products</code> с указанием этого JWT в заголовке вида <code>Bearer {TOKEN-YOU-COPIED}</code> (для отправки запроса можно использовать, например, <a href="https://www.getpostman.com/">Postman</a>). Все хорошо, вы можете увидеть список продуктов. Прослойка <code>jwtMiddleware</code> проверяет токен перед хендлером <code>ProductsHandler</code> и вам отдаеются продукты.</p>

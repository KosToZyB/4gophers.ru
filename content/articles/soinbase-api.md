+++
date = "2015-01-28T18:11:06+03:00"
draft = false
title = "Coinbase API"

+++

<p>Перевод статьи "<a href="http://fabioberger.com/blog/2014/11/06/building-a-coinbase-app-in-go/">Building a Coinbase App in Go</a>"</p>

<p>Coinbase это потрясающий сервис, предоставляющий биткоин веб-кошельки для упрощения покупок и хранения биткоинов. Этот сервис предоставляет обширное API для третьесторонних девелоперов, которые могут создавать свои приложения поверх этой платформы. К сожалению, нет никаких Go-врапер над API, поэтому я взялся написать свой. В этой статье я расскажу вам, как написать простое приложение с использованием API и OAuth аутентификацией.</p>

<p>Готовая либа размещена на <a href="https://github.com/fabioberger/Coinbase-go">Github</a>. В комплекте идет исчерпывающий <a href="https://github.com/fabioberger/coinbase-go/#examples">список примеров</a>, с описанием множества API вызовов.</p>

<h3>Получаем API ключ для приложения</h3>

<p>Любой пользователь coinbase может получить API ключ и секретный ключ, привязанные к его аккаунту. Используя эти доступы, можно программно использовать возможности аккаунта(это автоматизация покупки и продажи биткоинов, доступ к последним транзакциям, заказам и получателям, т.д.). В первой части этого мануала мы рассмотрим как узнать ваш текущий баланс используя пакет coinbase-go.</p>

<h4>Добавляем пару ключей как переменные окружения</h4>

<p>Прежде всего, нам необходимо сгенерировать пару ключей. Это можно сделать на <a href="https://coinbase.com/settings/api">странице настроек coinbase API</a>. Убедитесь, что ключи включены, скопировав проверочный код из письма от coinbase.</p>

<p>Как закончите с этим, установите ключи как переменные окружения в вашем файле конфигурации для шела:</p>

<pre><code class="sh">echo "export COINBASE_KEY='YOUR_KEY' " &gt;&gt; ~/.bash_profile
echo "export COINBASE_SECRET='YOUR_SECRET' " &gt;&gt; ~/.bash_profile
</code></pre>

<p>Обратите внимание, что у вас вместо .bash_profile может быть другой файл, например .bashrc. Все зависит от используемого шела(<a href="http://www.tldp.org/LDP/Bash-Beginners-Guide/html/sect_03_01.html">подробности тут</a>).</p>

<p>После этого, нам необходимо перегрузить конфиг:</p>

<pre><code class="sh">source ~/.bash_profile #  or .bashrc, etc.
</code></pre>

<p>Чтобы убедится, что все работает нормально, запустите в терминале:</p>

<pre><code class="sh">echo $COINBASE_KEY
</code></pre>

<p>Если эта команда выведет наш coinbase ключ - мы на верном пути! Установка секретных данных как переменных окружения - хорошая практика в плане безопасности. Нам не нужно вписывать значения ключей в наших исходниках. Это важно, потому что если кто-то получить доступ к вашим API ключам, то он сможет сделать все что угодно с вашим аккаунтом.</p>

<h4>Качаем пакет</h4>

<p>Исходим из того что у нас уже есть установленный Go на машине. Убедитесь, что у вас правильно настроен $GOPATH. Если это не так, запустите команду ниже, указав путь к папке с установленным Go:</p>

<pre><code class="sh">echo "export GOPATH='path/to/your/go/folder'" &gt;&gt; ~/.bash_profile
</code></pre>

<p>Теперь мы готовы скачивать пакет coinbase-go:</p>

<pre><code class="sh">go get github.com/fabioberger/coinbase-go
</code></pre>

<h4>Пишем приложение</h4>

<p>Для демонстрации  попробуем написать простое консольное приложение, которое будет показывать текущий баланс в coinbase. Давайте создадим новый проект в нашей рабочей директории, назовем его coinbase-app и создадим файл main.go</p>

<pre><code class="sh">mkdir $GOPATH/src/coinbase-app
cd $GOPATH/src/coinbase-app
touch main.go
</code></pre>

<p>В файле main.go начнем писать наше приложение:</p>

<pre><code class="go">package main

import (
    "fmt"
    "log"
    "os"
    "github.com/fabioberger/coinbase-go"
)

func main() {

    c := coinbase.ApiKeyClient(os.Getenv("COINBASE_KEY"), os.Getenv("COINBASE_SECRET"))

    balance, err := c.GetBalance()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Balance is %f BTC", balance)
}
</code></pre>

<p>Это простое приложение импортирует пакет <code>сoinbase-go</code> совместно с <code>fmt</code>, <code>log</code> и <code>os</code>, которые входят в стандартную поставку Go. Внутри функции <code>main</code> мы получаем инстанс <code>APIKeyClient</code>, указывая пару ключей, которые мы указали как переменные окружения. После этого мы можем использовать переменную <code>c</code> для вызова методов и отправки запросов к Coinbases API. В этом примере мы вызываем метод <code>GetBalance()</code>, чтобы запросить наш текущий баланс с coinbase.</p>

<p>Для проверки этого примера запустите следующее:</p>

<pre><code class="sh">go run main.go
</code></pre>

<p>Полный список доступных методов можно <a href="https://godoc.org/github.com/fabioberger/coinbase-go">посмотреть на GoDoc</a>. Так же, в <a href="https://github.com/fabioberger/coinbase-go/blob/master/README.md#examples">README есть примеры</a> как пользоваться этими методами.</p>

<h3>OAuth аутентификация</h3>

<p>Это здорово, что есть возможность взаимодействовать с нашей учетной записью на coinbase через API, особенно для автоматизации сделок и наблюдения за состоянием нашего аккаунта. Однако, еще круче если мы сможем создать приложение для управления любимы аккаунтами coinbase. Для этого нам нужно добавить поддержку OAuth в наше приложение.</p>

<h4>Установка</h4>

<p>Для начала нам нужно создать OAuth приложение на странице <a href="https://coinbase.com/oauth/applications">https://coinbase.com/oauth/applications</a>. Нам обязательно нужно указать URI для редиректа(в нашем случае https://localhost:8443/tokens). Coinbase предоставляет нам ID клиента и секретный ключ клиента для нашего OAuth приложения, давайте добавим это в наш шел конфиг как переменные окружения:</p>

<pre><code class="sh">echo "export COINBASE_CLIENT_ID='YOUR_CLIENT_ID' " &gt;&gt; ~/.bash_profile
echo "export COINBASE_CLIENT_SECRET='YOUR_CLIENT_SECRET' " &gt;&gt; ~/.bash_profile
source ~/.bash_profile # .bashrc, etc.
</code></pre>

<p>Вы помните, что при необходимости вам нужно заменить .bash_profile на ваш файл конфига для конкретного шела.</p>

<p>Мы уже скачали пакет coinbase-go, но если вы пропустили этот шаг, вернитесь к пункту "Качаем пакет" и выполните описанные инструкции. Чтобы написать пример веб-приложения мы будем использовать <a href="http://martini.codegangsta.io/">фреймворк Martini</a>, для этого нам нужно его скачать:</p>

<pre><code class="sh">go get github.com/go-martini/martini
</code></pre>

<p>Martini это мощный пакет для быстрого создания модульных веб-приложений/сервисов на Go. Мы будем использовать его для простого OAuth приложения, которое будет возвращать баланс для каждого coinbase пользователя, который будет аутентифицироваться через наше приложения.</p>

<h4>Пишем приложение</h4>

<p>Создадим новое приложение в нашей рабочей директории и назовем его coinbase-oauth и создадим файл main.go:</p>

<pre><code class="sh">mkdir $GOPATH/src/coinbase-oauth
cd $GOPATH/src/coinbase-oauth
touch main.go
</code></pre>

<p>В файле main.go пишем код:</p>

<pre><code class="go">package main

import (
    "log"
    "net/http"
    "os"
    "strconv"
    "github.com/fabioberger/coinbase-go. "
    "github.com/go-martini/martini"
)

var o *coinbase.OAuth

func main() {
    // Настройка Martini
    m := martini.New()
    m.Use(martini.Logger())
    m.Use(martini.Recovery())
    m.Use(martini.Static("public"))
    r := martini.NewRouter()
    m.MapTo(r, (*martini.Routes)(nil))
    m.Action(r.Handle)
    // Инстанс OAuthService с указанными ID клиента и 
    // секретным ключем клиента
    o, err := coinbase.OAuthService(os.Getenv("COINBASE_CLIENT_ID"), 
              os.Getenv("COINBASE_CLIENT_SECRET"), 
              "https://localhost:8443/tokens")
    if err != nil {
        panic(err)
        return
    }

    // На странице http://localhosts:8443/ мы отображаем
    // ссылку на "authorize"
    r.Get("/", func() string {
        authorizeUrl := o.CreateAuthorizeUrl([]string{
            "all",
        })
        link := "&lt;a href='" + authorizeUrl + "'&gt;authorize&lt;/a&gt;"
        return link
    })

    // После аутентификации, AuthorizeUrl редиректит 
    // на https://localhost:8443/tokens с указанным
    // параметром 'code'. Если все хорошо, отображается
    // пользователя
    r.Get("/tokens", func(res http.ResponseWriter, req *http.Request) string {

        // Получаем токены учитывая параметр 'code'
        tokens, err := o.NewTokensFromRequest(req) 
        // Мы используем 'code' параметр из req
        if err != nil {
            return err.Error()
        }

        // Инстанс OAuthClient с токенами
        c := coinbase.OAuthClient(tokens)

        // Запрашиваем баланс пользователя
        amount, err := c.GetBalance()
        if err != nil {
            return err.Error()
        }
        return strconv.FormatFloat(amount, 'f', 6, 64)
    })

    // HTTP
    go func() {
        if err := http.ListenAndServe(":8080", m); err != nil {
            log.Fatal(err)
        }
    }()

    // HTTPS
    // для генерации cert and key запустите указанный скрипт
    // в *nix терминале 
    // go run $(go env GOROOT)/src/pkg/crypto/tls/generate_cert.go 
    // --host="localhost"
    if err := http.ListenAndServeTLS(":8443", "cert.pem", "key.pem", m); err != nil {
        log.Fatal(err)
    }
}
</code></pre>

<p>Перед тем как мы сможем запустить написанный код, необходимо позаботиться о SSL, потому что сoinbase требует шифрованного соединения для передачи данных на наш редирект. Для использования HTTPS в рамках нашего сервера, необходимо сгенерировать девелоперский cert и key. К счастью, это делается довольно просто:</p>

<pre><code class="sh">go run $(go env GOROOT)/src/pkg/crypto/tls/generate_cert.go --host="localhost"
</code></pre>

<p>Эта команда создаст cert.pem и key.pem файлы в нашей директории. Так как это самоподписанный сертификат, то браузер будет ругаться, что этому сертификату нельзя доверять. Однако, так как мы сами запустили этот сервер, то мы можем просто попросить браузер игнорировать это предупреждение и перейти к странице нашего приложения. Когда мы будем выкладывать нашу поделку в продакшен, нам нужно будет получить SSl сертификат из доверенного центра сертификации. Можем запускать наше приложение:</p>

<pre><code class="sh">go run main.go
</code></pre>

<p>Чтобы посмотреть как это работает, перейдите в браузере на страницу https://localhost:8443/.</p>

<p>Окей, давайте немного поговорим о коде, который мы только что написали.
Мы начали с импорта Martini, coinbase-go и нескольких других пакетов из стандартной библиотеки Go. Затем, в функции <code>main</code> мы создаем инстанс <code>OAuthService</code> с указанием ID клиента и секретным ключем, которые мы определили как переменные окружения. Последним параметром указываем адрес для редиректа. Теперь мы можем использовать <code>o</code> как экземпляр <code>OAuthService</code> по всему приложению.</p>

<p>Затем мы определили обработчик для запросов к http://localhost:8443/. Когда кто либо заходит по этому url, в приложении вызывается <code>o.CreateAuthorizeUrl()</code> для генерации ссылки на аутентификацию. Пользователю необходимо пройти по этой ссылки и авторизировать наше приложение, что бы мы смогли работать с его аккаунтом. Мы передаем в этот метод желаемые уровни доступа, которые нам необходимо запросить у пользователя(<a href="https://www.coinbase.com/docs/api/permissions">полный список уровней доступа</a>). После этого сгенерированная ссылка на авторизацию в coinbase отображается на странице.</p>

<p>Когда пользователь кликает по этой ссылке и авторизирует наше приложение в coinbase, он отправляется на указанный редирект, который мы определили в вызове <code>OAuthService</code>(в нашем случае это https://localhost:8443/tokens). Срабатывает обработчик <code>o.NewTokensFromRequest()</code> который будет извлекать параметр <code>code</code> отправленный coinbase и запрашивать ассоциированные с ним токены.</p>

<p>Как только мы получим эти токены, будем использовать инстанс <code>OAuthClient</code> с указанием этих токенов. Этот клиент аналогичен <code>ApiKeyClient</code> и имеет доступ к тем же методам для отправки запросов к coinbase API. Единственное отличие в том, что каждый запрос будет от имени авторизированного пользователя. Следующий шаг - это вызов <code>GetBalance()</code> и отображение результата.</p>

<p>Токены, которые мы получили от coinbase, имеют свой срок жизни и узнать его мы можем обратившись к полю <code>tokens.ExpireTime</code>. После того как токены прогорели, нам нужно их обновить, вызвав функцию <code>o.RefreshTokens(tokens)</code>.</p>

<h3>Заключение</h3>

<p>Я надеюсь, что рассмотренные примеры по созданию приложений для работы с coinbase API оказались полезными. Не забывайте поглядывать на <a href="https://github.com/fabioberger/coinbase-go/blob/master/README.md#examples">примеры  в README</a> проекта на Github. Там описано большое количество различных запросов, которые можно совершать используя coinbase-go. Если у вас есть вопросы или комментарии, не стесняйтесь, пишите.</p>

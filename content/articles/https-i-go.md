+++
date = "2015-04-06T02:08:02+03:00"
draft = false
title = "HTTPS и Go"
tags = ["http", "https"]

+++

<p>Это перевод статьи "<a href="https://www.kaihag.com/https-and-go">HTTPS and Go</a>". Статья больше для новичков, чем для матерых гоферов, но есть полезная информация для всех программистов.</p>

<p>Работа с HTTP сервером - это одна из первых задач, с которой сталкивается начинающий Go программист.</p>

<p>Реализовать простенький HTTP сервер на Go легко. Необходимо написать всего пару строк кода и у вас готов и работает сервер на 8080 порту:</p>

<pre><code class="go">package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Привет!")
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
</code></pre>

<p>и у вас запущен и работает сервер на 8080 порту. Откройте страничку https://127.0.0.1:8080 в вашем браузере и вы увидите сообщение "Привет!".</p>

<p>Но что если вам нужно работаться с защищенным HTTPS соединением? В первом приближении, это достаточно просто. Для этого можно использовать метод <a href="http://golang.org/pkg/net/http/#ListenAndServeTLS">ListenAndServeTLS</a>, вместо <code>http.ListenAndServe(":8080", nil)</code>.</p>

<pre><code class="go">http.ListenAndServeTLS(":8081", "cert.pem", "key.pem", nil)
</code></pre>

<p>И все готово. Ну, почти. Эта функция получает на два аргумента больше: "cert.pem" - ваш серверный сертификат в PEM формате, "key.pem" - приватный ключ в PEM формате.</p>

<h2>Получение сертификата для сервера и приватного ключа</h2>

<h3>Использование OpenSSL</h3>

<p>Вы можете легко сгенерировать оба файла с помощью OpenSSL. OpenSSL поставляется в Mac OS X и Linux. Если вы используете Windows, то вам нужно установить бинарники отдельно.</p>

<p>К счастью, для генерирование сертификата и приватного ключа с помощью OpenSSL достаточно одной команды:</p>

<pre><code class="sh">openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout key.pem -out cert.pem
</code></pre>

<p>Вам нужно будет ответить на пару вопросов в момент генерации. Самая важная часть, это поле "Common Name (e.g. server FQDN or YOUR name)". Тут вы должны указать имя вашего сервера (например myblog.com, или 127.0.0.1:8081 если вам нужен доступ к вашей локальной машине на 8081 порту).</p>

<p>После этого, вы обнаружите два файла "cert.pem" и "key.pem" в той папке, где вы запускали OpenSSL команду. Учтите, что эти файлы называются самоподписанным сертификатом. Это значит, что вы можете использовать эти файлы, но браузер будет определять соединение как не безопасное.</p>

<p>Вы можете сами проверить это, как только запустите сервер</p>

<pre><code class="go">http.ListenAndServeTLS(":8081", "cert.pem", "key.pem", nil)
</code></pre>

<p>с указанием сгенерированного файла. Перейдя на страничку https://127.0.0.1:8081 в браузере вы увидите предупреждение безопасности.</p>

<p>Это означает, что сертификат на сервере не подписан доверенным центром сертификации. В Firefox мне нужно кликнуть "I Understand the Risks" и после этого браузер перейдет на сайт. В хроме нужно кликнуть "Advanced" и затем так же последует переход на страницу.</p>

<h3>Используем Go</h3>

<p>Есть другой способ генерации файлов сертификата и ключа - вы можете сделать это непосредственно с помощью Go кода. В стандартной поставке Go есть пример программы, которая демонстрирует как это делается. Она называется <a href="http://golang.org/src/crypto/tls/generate_cert.go">generate_cert.go</a>.</p>

<p>Для удобства, я собрал все это в отдельную библиотеку, названную <a href="https://github.com/kabukky/httpscerts">httpscerts</a>. Мы можем модифицировать нашу программу для использования httpscerts и автоматической генерации необходимых сертификатов:</p>

<pre><code class="go">package main

import (
    "fmt"
    "github.com/kabukky/httpscerts"
    "log"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Привет")
}

func main() {
    // Проверяем, доступен ли cert файл.
    err := httpscerts.Check("cert.pem", "key.pem")
    // Если он недоступен, то генерируем новый.
    if err != nil {
        err = httpscerts.Generate("cert.pem", "key.pem", "127.0.0.1:8081")
        if err != nil {
            log.Fatal("Ошибка: Не можем сгенерировать https сертификат.")
        }
    }
    http.HandleFunc("/", handler)
    http.ListenAndServeTLS(":8081", "cert.pem", "key.pem", nil)
}
</code></pre>

<p>Конечно, ваш браузер все также отобразит предупреждение о самоподписанном сертификате.</p>

<h3>Использование StartSSL</h3>

<p>Самодписанные сертификаты это удобно для тестирования. Но как только вы запустите сервер в продакшене, вам станет нужен сертификат подписанный в доверенном центре, который будет нормально принимать браузер и операционные системы.</p>

<p>К сожалению, это платная услуга. Для примера, Comodo сдерет с вас $100 за сертификат на 1 год. Internet Security Research Group работает <a href="https://letsencrypt.org/">над этой проблемой</a>, но пока нет возможности получить бесплатный сертификат.</p>

<p>Единственная альтернатива, это использовать сервис <a href="https://www.startssl.com/">StartSSL</a>. StartSSL выдает сертификат для одного домена, за который не нужно будет платить в течении первого года. Конечно, вам придется им заплатить за отзыв сертификата, в случае <a href="http://heartbleed.com/">Heartbleed</a> например, но сейчас это единственный вариант получить бесплатный сертификат, хоть и на ограниченное время.</p>

<p>Зарегистрируйтесь на StartSSL и сгенерируйте сертификат и приватный ключ для своего домена. Внимательно прочитайте инструкцию или найдите пару туториалов по использованию StartSSL.</p>

<p>В дальнейшем, будем считать, что вы сохранили сертификат как "cert.pem" и приватный ключ как "key.pem".</p>

<p>Ваш сертификат может быть защищен паролем. Для этого откройте "key.pem" в  обычном текстовом редакторе. Если он действительно зашифрован паролем, то вы увидите что-то вроде:</p>

<pre><code class="go">Proc-Type: 4,ENCRYPTED
</code></pre>

<p>Чтобы удалить пароли из приватного ключа, используйте OpenSSL команду:</p>

<pre><code class="go">openssl rsa -in key.pem -out key_unencrypted.pem
</code></pre>

<p>В конце концов, вам нужно добавить StartSSL Intermediate CA и StartSSL Root CA в "cert.pem"</p>

<p>Скачайте "<a href="https://www.startssl.com/certs/sub.class1.server.ca.pem">Class 1 Intermediate Server CA</a>" и "<a href="https://www.startssl.com/certs/ca.pem">StartCom Root CA (PEM encoded)</a>" из StartSSL Tool Box (Log In > Tool Box > StartCom CA Certificates) и положите файлы рядом с вашим "cert.pem". Используя Linux и Mac OS X, запустите:</p>

<pre><code class="go">cat cert.pem sub.class1.server.ca.pem ca.pem &gt; cert_combined.pem
</code></pre>

<p>Используя Windows, запустите:</p>

<pre><code class="go">type cert.pem sub.class1.server.ca.pem ca.pem &gt; cert_combined.pem
</code></pre>

<p>Теперь вы можете использовать "cert_combined.pem" и "key_unencrypted.pem" в вашей Go программе. Если хотите, можете переименовать их в "cert.pem" и "key.pem".</p>

<h2>Обработка HTTP соединения</h2>

<p>Сейчас ваш HTTPS сервер работает отлично. Используя StartSSL сертификат вы не будете видеть предупреждения, заходя на ваш сайт https://yourdomain.com.</p>

<p>Но как быть http://yourdomain.com? Так как HTTP сервер теперь не запущен, то страничка не загрузится. Есть два способа решить эту проблему.</p>

<h3>Раздача одинакового контента по HTTP и HTTPS</h3>

<p>Это очень просто реализовать:</p>

<pre><code class="go">package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there!")
}

func main() {
    http.HandleFunc("/", handler)
    // Запуск HTTPS сервера в отдельной go-рутине
    go http.ListenAndServeTLS(":8081", "cert.pem", "key.pem", nil)
    // Запуск HTTP сервера
    http.ListenAndServe(":8080", nil)
}
</code></pre>

<h3>Редирект с HTTP на HTTPS</h3>

<p>Это наиболее верный подход, если вы хотите заинкриптить весь ваш трафик. Для достижения этого, вам нужна функция, которая будет выполнять редирект с HTTP на HTTPS:</p>

<pre><code class="go">package main

import (
    "fmt"
    "net/http"
)

func redirectToHttps(w http.ResponseWriter, r *http.Request) {
    // Перенаправляем входящий HTTP запрос. Учтите, 
    // что "127.0.0.1:8081" работает только для вашей локальной машина
    http.Redirect(w, r, "https://127.0.0.1:8081"+r.RequestURI, 
                            http.StatusMovedPermanently)
}

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there!")
}

func main() {
    http.HandleFunc("/", handler)
    // Запуск HTTPS сервера в отдельной go-рутине
    go http.ListenAndServeTLS(":8081", "cert.pem", "key.pem", nil)
    // Запуск HTTP сервера и редирект всех входящих запросов на HTTPS
    http.ListenAndServe(":8080", http.HandlerFunc(redirectToHttps))
}
</code></pre>

<p>Или, используя два разных <code>ServeMux</code> для HTTP и HTTPS серверов, вы можете редиректить на HTTPS только по специфическим путям (например /admin/):</p>

<pre><code class="go">package main

import (
    "fmt"
    "net/http"
)

func redirectToHttps(w http.ResponseWriter, r *http.Request) {
    // Перенаправляем входящий HTTP запрос. Учтите, 
    // что "127.0.0.1:8081" работает только для вашей локальной машина
    http.Redirect(w, r, "https://127.0.0.1:8081"+r.RequestURI, 
                                http.StatusMovedPermanently)
}

func homeHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there!")
}

func adminHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi admin!")
}

func main() {
    // Создаем новый ServeMux для HTTP соединений
    httpMux := http.NewServeMux()
    // Создаем новый ServeMux для HTTPS соединений
    httpsMux := http.NewServeMux()
    // Перенаправляем /admin/ на HTTPS
    httpMux.Handle("/admin/", http.HandlerFunc(redirectToHttps))
    // Обрабатываем все остальное
    httpMux.Handle("/", http.HandlerFunc(homeHandler))
    // Так же, обрабатываем все по HTTPS
    httpsMux.Handle("/", http.HandlerFunc(homeHandler))
    httpsMux.Handle("/admin/", http.HandlerFunc(adminHandler))
    // Запуск HTTPS сервера в отдельной go-рутине
    go http.ListenAndServeTLS(":8081", "cert.pem", "key.pem", httpsMux)
    // Запуск HTTPS сервера
    http.ListenAndServe(":8080", httpMux)
}
</code></pre>

<p>И на этом все. Экспериментируйте с HTTPS и Go в удовольствие!</p>

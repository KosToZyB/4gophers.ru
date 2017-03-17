+++
date = "2016-09-29T21:43:09+03:00"
draft = false
title = "DNS и Go"

+++

<p>Перевод статьи "<a href="https://miek.nl/2014/August/16/go-dns-package/">Go DNS package</a>".</p>

<p>Пакет Go DNS реализует интерфейс для управления DNS с помощью Go. По сути, это либа которая позволяет делать и принимать DNS запросы. Весь код лицензируется так же как и код самого Go.</p>

<p>Основная цель создания этого инструмента - предоставить простой и мощный инструмент.</p>

<p>Что поддерживает эта библиотека:</p>

<ul>
<li>Все типы RR;</li>
<li>Синхронные и асинхронные запросы и ответы:</li>
<li>DNSSEC - валидация, подписи, генерация ключей, чтение файлов .private:</li>
<li>(Скоро) отправка/получение/отображение пакетов и RR;</li>
<li>Полный контроль в буквальном смысле:</li>
<li>Перенос зон, EDNS0, TSIG, NSID;</li>
<li>Возможности полноценного сервера имен:</li>
<li>(Скоро) чтение зон/RR из файлов и строк:</li>
</ul>

<h3>Код</h3>

<p>Сам код пакета находится на <a href="http://github.com/miekg/dns">github</a>.</p>

<p>Примеры использования пакета можно найти в специальном пакете, <a href="http://github.com/miekg/exdns">все на том же github</a>.</p>

<h3>Чуть больше о использовании</h3>

<ul>
<li><a href="http://www.bortzmeyer.org/go-dns-icinga.html">Использование Go DNS в Nagios</a>.</li>
<li><a href="http://news.ntppool.org/2012/10/new-dns-server.html">Сервер имен GeoDNS для pool.ntp.org</a>.</li>
</ul>

<h3>Вывод MX записей</h3>

<p>Маленький лайвхак, как выводить MX записи используя Go DNS.</p>

<p>Попробуем написать простую программу, которая отображает MX записи для домена. Это будет выглядеть вот так:</p>

<pre><code>% mx miek.nl
miek.nl.        86400   IN      MX      10 elektron.atoom.net.
</code></pre>

<p>Или так:</p>

<pre><code>% mx microsoft.com
microsoft.com.  3600    IN      MX      10 mail.messaging.microsoft.com.
</code></pre>

<p>Начнем со списка используемых пакетов:</p>

<pre><code>package main

import (
    "os"
    "net"
    "fmt"
    "log"

    "github.com/miekg/dns"
)
</code></pre>

<p>Теперь мы можем создать небольшой локальный сервер имен:</p>

<pre><code>config, _ := dns.ClientConfigFromFile("/etc/resolv.conf")
</code></pre>

<p>Теперь создадим клиент <code>dns.Client</code> для выполнения наших запросов:</p>

<pre><code>c := new(dns.Client)
</code></pre>

<p>В этом примере мы пропустили обработку ошибок и считаем, что зона уже указана. Для продолжения нам нужно еще несколько вещей:</p>

<ul>
<li>создать пакет экземпляр пакета(<code>(dns.Msg)</code>);</li>
<li>установить некоторые заголовочные биты;</li>
<li>указать секцию запроса;</li>
<li>заполнить секцию запроса: считаем, что в <code>os.Args[1]</code> содержится имя зоны:</li>
</ul>

<p>В итоге получится вот так:</p>

<pre><code>m := new(dns.Msg)
m.SetQuestion(dns.Fqdn(os.Args[1]), dns.TypeMX)
m.RecursionDesired = true
</code></pre>

<p>В конце всего мы должны выполнить наш запрос. Для этого нужно вызвать метод <code>Exchange()</code>. Пропущенное возвращенное значение - это rtt (round trip time).</p>

<pre><code>r, _, err := c.Exchange(m, net.JoinHostPort(config.Servers[0], config.Port))
</code></pre>

<p>Ниже представлен кусок кода, который выводит в консоль ответ от сервера. В случае ошибки просто прерываем выполнение:</p>

<pre><code>if r == nil {
    log.Fatalf("*** error: %s\n", err.Error())
}

if r.Rcode != dns.RcodeSuccess {
        log.Fatalf(" *** invalid answer name %s after MX query for %s\n", os.Args[1], os.Args[1])
}

// Весь результат должен быть в поле Answer
for _, a := range r.Answer {
        fmt.Printf("%v\n", a)
}
</code></pre>

<p>И на этом, собственно, все. Полный код примера:</p>

<pre><code>package main

import (
    "net"
    "os"
    "log"
    "fmt"

    "github.com/miekg/dns"
)

func main() {
    config, _ := dns.ClientConfigFromFile("/etc/resolv.conf")
    c := new(dns.Client)

    m := new(dns.Msg)
    m.SetQuestion(dns.Fqdn(os.Args[1]), dns.TypeMX)
    m.RecursionDesired = true

    r, _, err := c.Exchange(m, 
            net.JoinHostPort(config.Servers[0], config.Port))
    if r == nil {
        log.Fatalf("*** error: %s\n", err.Error())
    }

    if r.Rcode != dns.RcodeSuccess {
            log.Fatalf(" *** invalid answer name %s after MX query for %s\n",
            os.Args[1], os.Args[1])
    }
    // Весь результат должен быть в поле Answer
    for _, a := range r.Answer {
            fmt.Printf("%v\n", a)
    }
}
</code></pre>

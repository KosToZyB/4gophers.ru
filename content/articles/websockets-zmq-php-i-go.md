+++
date = "2015-04-19T20:21:08+03:00"
draft = false
title = "Websockets, ZMQ и Go"

+++

<p>Риалтайм - это стильно, модно, молодежно. Самый риалтаймистый риалтайм - это вебсокеты. Для работы с ними уже много всего написанного на PHP, Nodejs и Python. Тот же <a href="http://socketo.me/">Ratchet</a>, к примеру. Однако, нам нужны серьезные RPS и тут PHP нам не подходит. Кроме того, приложение на Go будет с уже встроенным вебсервером, а, например, с PHP нам придется что-то <a href="http://nginx.org/ru/docs/http/websocket.html">выдумывать с nginx</a>, или (не дай Бог) запускать PHP как демона. Сами понимаете - PHP не тру вей для написания долгоработающих приложений.</p>

<p>Таким образом, выходит что Go один из лучших кандидатов для написания вебсокет-сервиса. К тому же, есть уже много готовых пакетов, реализующих поддержку вебсокетов в Go.</p>

<p>Но как написать стабильное приложение, при условии что часть проекта уже написана на PHP? Например, нам нужно реализовать чат для пользователей, зарегистрированных на сайте, или в реальном времени отображать сообщения на странице. Для этого нам нужен некоторый механизм взаимодействия между PHP кодом и Go сервисом. В этом случае нам нужно продумать надежный механизм взаимодействия между частями нашего приложения.</p>

<h3>Бэкенд</h3>

<p>Начинаем с создания простого http сервера. Все что нам нужно есть в коробочной поставке самого Go.</p>

<pre><code class="go">http.HandleFunc("/ws", handler)
err = http.ListenAndServe(wsaddr, nil)
if err != nil {
    log.Fatal("ListenAndServe: ", err)
}
</code></pre>

<p><strong>handler</strong> - Это хендлер, который нам нужно реализовать для соединения с вебсокетом.
<strong>wsaddr</strong> - url, на который будут вешаться соединения. Этот url, в нашем случае, берется из конфига.</p>

<p>Примеров и статей по написанию веб сервера на Go великое множество, не политесь нагуглить, если у вас что-то не получилось.</p>

<h4>ZMQ</h4>

<p>Есть множество вариантов взаимодействия между Go сервисом и PHP кодом. Например, мы можем писать сообщения в базу, а затем читать от туда в Go через определенные промежутки времени. Не самый быстрый способ. Еще один вариант - написать tcp клиент/серверное приложение, однако, написать действительно надежное приложение тот еще челендж. В конце концов, можем взаимодействовать через REST API, но опят же, большой скорости тут не стоит ожидать.</p>

<p>К счастью, все уже написано до нас. Нам достаточно воспользоваться одним из вариантов очередей, например ZMQ. Для своих нужд вы можете выбрать что ни будь другое, например RabitMQ. В моем случае ZMQ удовлетворяет всем требованиям и есть один большой плюс - для PHP реализованна нативная поддержка в виде сишного расширения.</p>

<p>Работа с ZMQ в Go не сложнее чем написание http сервера. Создаем новый сокет, подключаемся по адресу из конфига(<code>zmqaddr</code>) и подписываемся на сообщения от транспорта. <code>zmqsubject</code> - это название канала, по которому будут приходить сообщения, берется из конфига.</p>

<pre><code class="go">responder, _ = zmq.NewSocket(zmq.SUB)
defer responder.Close()
responder.Connect(zmqaddr)
responder.SetSubscribe(zmqsubject)
</code></pre>

<p>Устанавливаем соединение и ждем собщения от брокера. Для работы с вебсокетами используем пакет <a href="https://github.com/gorilla/websocket">websocket</a> из <a href="https://github.com/gorilla/">gorillatoolkit</a>.</p>

<p>Хендлер, который обслуживает вебсокет-соединение будет выглядеть так:</p>

<pre><code class="go">func handler(w http.ResponseWriter, r *http.Request) {

    conn, err := websocket.Upgrade(w, r, nil, 1024, 1024)
    if _, ok := err.(websocket.HandshakeError); ok {
        http.Error(w, "Not a websocket handshake", 400)
        return
    } else if err != nil {
        log.Println(err)
        return
    }

    log.Println("Start...")
    for {
        msg, _ := responder.RecvMessage(0)
        log.Println("Received ", msg)    
        if err := conn.WriteMessage(websocket.TextMessage, []byte(msg[1])); err != nil {
            log.Println(err)
            return
        }
    }
}
</code></pre>

<p>Начинаем с создания веб-сокета. Вызов <code>websocket.Upgrade</code> апгрейдит текущее соединение, переводит его на уровень работы с вебсокетами:</p>

<pre><code class="go">conn, err := websocket.Upgrade(w, r, nil, 1024, 1024)
</code></pre>

<p>Это как бы рукопожатие между клиентом и сервером. Более подробно о протоколе можно почитать в <a href="https://ru.wikipedia.org/wiki/WebSocket">той же википедии</a>.</p>

<p>Далее создается бесконечный цикл, который блокируется в ожидании получения сообщения из ZMQ очереди.</p>

<pre><code class="go">msg, _ := responder.RecvMessage(0)
</code></pre>

<p>Как только в очередь приходит новое сообщение, тут же отдаем его в вебсокет.</p>

<pre><code class="go">if err := conn.WriteMessage(websocket.TextMessage, []byte(msg[1])); err != nil {
    log.Println(err)
    return
}
</code></pre>

<p>Кроме все этого, рядом с приложением лежит файлик config.yml, который парситься приложением. Настройки очень банальные и понятные</p>

<p>Кстати, для тестирования нашего приложения можно использовать инструмент <a href="http://telsocket.org/">telsocket.org</a>. Это как телнет, только для вебсокетов.</p>

<h4>Конфиг</h4>

<p>Как вы успели заметить, много параметров в коде задается с помощью конфига. Это обычный YAML файл, который лежит в корне нашего приложения <code>config.yaml</code>. Для работы с ним используется пакет <code>github.com/kylelemons/go-gypsy/yaml</code></p>

<pre><code class="go">file := "config.yaml"
config, err := yaml.ReadFile(file)
if err != nil {
    log.Fatal("Error load config", err)
}

wsaddr, _ = config.Get("ws.address")
zmqaddr, _ = config.Get("zmq.address")
zmqsubject, _ = config.Get("zmq.subject")  
</code></pre>

<p>В этом примере ошибки игнорируются. Не забывайте их проверять в своих реальных приложениях.</p>

<h3>JS Клиент</h3>

<p>Окей, с бекендом мы справились. Давайте теперь напишем клиентский код, который будет получать сообщения и отображать их на HTML страничке.</p>

<p>В качестве примера будем использовать API браузера, в моем случае хрома. Конечно, работать это будет только в нормальных браузерах. Для поддержки более старых версий используйте <a href="http://socket.io/docs/client-api/">дополнительные библиотеки и инструменты</a>.</p>

<p>Создадим файл index.html:</p>

<pre><code class="js">try {
    var sock = new WebSocket("ws://localhost:8080/ws");
    console.log("Websocket - status: " + sock.readyState);
    sock.onopen = function(m) { 
        console.log("CONNECTION opened..." + this.readyState);
    }

    sock.onmessage = function(m) { 
        console.log("message: " + m.data);
    }

    sock.onerror = function(m) {
        console.log("Error occured sending..." + m.data);
    }

    sock.onclose = function(m) { 
        console.log("Disconnected - status " + this.readyState);
    }
} catch(exception) {
    console.log(exception);
}
</code></pre>

<p>Создаем вебсокет-соединение, подключаемся к серверу.</p>

<pre><code class="js">var sock = new WebSocket("ws://localhost:8080/ws");
</code></pre>

<p>Все приходящие соединения будут в консоле:</p>

<pre><code class="js">sock.onmessage = function(m) { 
    console.log("message: " + m.data);
}
</code></pre>

<p>Для тестирования можно просто запустить встроенный PHP сервер и посмотреть на наш файлик index.html</p>

<pre><code class="sh">$ php -S localhost:12345
</code></pre>

<h3>PHP Приложение.</h3>

<p>В конце концов, перейдем к тому ради чего все это затевалось. Давайте напишем пример минимального PHP приложения, которое отправляет сообщения в очередь.</p>

<p>Расширение для ZMQ нативное и его можно установить из <a href="https://pecl.php.net/package/zmq">pecl репозиториев</a>.</p>

<p>Инструкция по установке <a href="http://zeromq.org/bindings:php">есть на cайте ZMQ</a>.</p>

<p>Максимально простое PHP приложение, с использование очереди выглядит как то так:</p>

<pre><code>&lt;?php

if (count($argv) &lt; 2) {
    echo "usage: php ./zmq.php &lt;message&gt;\n";
}

$message = $argv[1];

$context = new ZMQContext();
$socket = $context-&gt;getSocket(ZMQ::SOCKET_PUB, 'message');
$socket-&gt;bind("tcp://127.0.0.1:5563");

echo "Sending: ".$message."\n";

$socket-&gt;sendmulti(array("message", $message));
?&gt;
</code></pre>

<p>Обратите внимание, как мы отправляем сообщение:</p>

<pre><code class="php">$socket-&gt;sendmulti(array("message", $message));
</code></pre>

<p>Первый элемент массива строка <code>message</code> - тема сообщения. Когда мы в Go подписываемся на получение сообщений, то указываем <code>zmqsubject</code>, значение которой и есть строка <code>message</code>. Таким образом, мы будем получать только сообщения с правильно указанной темой.</p>

<p>Использовать режим SOCKET_PUB не самый хороший вариант, но для начала сойдет.</p>

<p>На этом можно закончить этот небольшой обзор вебсокетов и очередей. Экспериментируйте с очередями и вебсокетами в своих приложениях и делитесь опытом.</p>

<p>Исходники к <a href="https://github.com/4gophers/websockets">статье можно найти на github</a>.</p>

<h3>Почитать</h3>

<ul>
<li>Сайт <a href="http://zeromq.org/">zeromq.org</a> и <a href="http://zeromq.org/intro:read-the-manual">их мануал</a></li>
<li>Хорошая <a href="http://habrahabr.ru/post/198578/">статья про ZMQ на хабре</a></li>
<li>Мануал по <a href="http://php.net/manual/ru/class.zmq.php">расширению для PHP</a></li>
<li>Статья <a href="https://ru.wikipedia.org/wiki/WebSocket">в википедии про вебсокеты</a>.</li>
</ul>

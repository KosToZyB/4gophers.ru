+++
date = "2016-07-31T20:42:08+03:00"
draft = false
title = "Делаем своего Slack бота на Go"

+++

<p>Перевод статьи "<a href="https://www.opsdash.com/blog/slack-bot-in-golang.html">Build your own slack bot in Go</a>"</p>

<p>Боты для Slack это довольно веселая и простая в написании штука. В этой статье мы посмотрим, как написать одного из таких ботов.</p>

<p>Вот что сможет делать наш свеженаписанный бот:</p>

<p><img src="https://www.opsdash.com/blog/images/mybot.png" alt="" /></p>

<p>Для создания и использования ботов вам нужен Slack аккаунт с высокими привилегиями(для включения ботов) и немного знаний по Go.</p>

<p>Чтобы добавить нового бота, вам нужно пройти по ссылке <code>https://YOURORG.slack.com/services/new/bot</code>(вместо YOURORG нужно подставить ваш реальный домен), затем выбрать имя. Не забудьте сохранить токен для доступа к API, позже он нам очень пригодится.</p>

<p>После всех этих, манипуляций вы увидите имя бота в списке пользователей(в состоянии залогинен). По умолчанию бот будет добавлен в канал #general.</p>

<p>Теперь нужно скачать код с github. Мы будем использовать его для понимания основных принципов.</p>

<pre><code>go get github.com/rapidloop/mybot
</code></pre>

<p>После выполнения команды, у вас появится бинарный файл в папке <code>$GOPATH/bin</code>. Запускаем и смотрим, что получится:</p>

<pre><code>$ mybot
usage: mybot slack-bot-token
$ mybot xxxx-99999999999-USE.THE.TOKEN.FROM.ABOVE
mybot ready, ^C exits
</code></pre>

<p>теперь <code>mybot</code> готов показывать вам котировки акций, как показано на скриншоте выше. Когда наиграетесь с ботом, достаточно нажать комбинацию клавиш CTR+C для выхода из программы.</p>

<p>Теперь можем погружаться в глубины кода.</p>

<h3>Код</h3>

<p><em>Предупреждение:</em> Код приложения <code>mybot</code> используется только для иллюстрации возможностей, в нем нет всех проверок ошибок и многого другого. Не используйте его в реальных проектах!</p>

<p>Для начала, нам нужно использовать токен для запуска сессии <a href="https://api.slack.com/rtm">Real Time Message API</a>. Для этого используем <a href="https://api.slack.com/methods/rtm.start">rtm.start</a> API. Как сказано в документации, нам нужно передать токен как параметр при вызове <code>rtm.start</code> и в ответ мы получим целую кучу всякой всячины. Из этого всего нам нужна только пара вещей.</p>

<p>Пример того, как мы делаем вызов (файл <code>slack.go</code>, метод <code>slackStart</code>)</p>

<pre><code>url := fmt.Sprintf("https://slack.com/api/rtm.start?token=%s", token)
resp, err := http.Get(url)
//...
body, err := ioutil.ReadAll(resp.Body)
//...
var respObj responseRtmStart
err = json.Unmarshal(body, &amp;respObj)
</code></pre>

<p>В этом куске кода мы делаем HTTP GET запрос, читаем тело ответа и декодируем его как JSON в отдельный объект <code>responseRtmStart</code>. Как было сказано выше, нам нужно всего несколько вещей. Метод <code>Unmarshal</code> из пакета <code>encoding/json</code> позволяет игнорировать ненужные поля, что позволяет определить структуру, состоящую только из необходимых полей:</p>

<pre><code>type responseRtmStart struct {
    Ok    bool         `json:"ok"`
    Error string       `json:"error"`
    Url   string       `json:"url"`
    Self  responseSelf `json:"self"`
}

type responseSelf struct {
    Id string `json:"id"`
}
</code></pre>

<p>Поле <code>Url</code> это специальный Websocket URL к которому мы можем подключится для запуска сессии RTM. Также, нам нужен параметр <code>Id</code>, который является уникальным идентификатором нашего бота. Он должен выглядеть примерно так: U1A2B3C4D. В будущем, мы будем использовать этот идентификатор для получения всех упоминаний.</p>

<h3>Используем API для работы с сообщениями в реальном времени</h3>

<p>Специальные Go библиотеки <code>golang.org/x</code> включают в себя пакет для работы с веб-сокетами: <code>golang.org/x/net/websocket</code>. В этом пакете есть JSON кодек, который предоставляет обертку для отправки и получения JSON объектов через веб-сокеты.</p>

<p>Запускаем websocket соединение с использованием этого пакета (файл slack.go, метод slackConnect):</p>

<pre><code>ws, err := websocket.Dial(wsurl, "", "https://api.slack.com/")
</code></pre>

<p>Через это соединение, Slack сообщает нам [почти обовсем]https://api.slack.com/rtm), что происходит в чатах. Эти нотификации включают в себя и сообщения, которые пользователи отправляют в каналы, и сообщения, которые пользователи отправляют лично нашему боту. Личные сообщения нам особенно интересны.</p>

<p>Каждая нотификация - это JSON объект, котрый содержит поле <code>type</code>. Нам нужны нотификации, у которых в этом поле указан тип <code>message</code>. Как и раньше, мы можем получить значения из нужных полей JSON объекта с помощью правильной структуры (файл slack.go):</p>

<pre><code>type Message struct {
  Id      uint64 `json:"id"`
  Type    string `json:"type"`
  Channel string `json:"channel"`
  Text    string `json:"text"`
}
</code></pre>

<p>Для работы с веб-сокет соединениями, нам нужен вот такой код(файл slack.go):</p>

<pre><code>func getMessage(ws *websocket.Conn) (m Message, err error) {
    err = websocket.JSON.Receive(ws, &amp;m)
    return
}
</code></pre>

<p>Для отправки сообщений в канал нам нужно записать JSON объект обратно в веб-сокет. Обратите внимание, нет никакой возможности отвечать на сообщения. Все что мы можем - это отправлять сообщения в канал. Для каждого сообщения нам нужен уникальный идентификатор. Для этого будем использовать атомарный инкремент счетчика.</p>

<pre><code>var counter uint64

func postMessage(ws *websocket.Conn, m Message) error {
  m.Id = atomic.AddUint64(&amp;counter, 1)
  return websocket.JSON.Send(ws, m)
}
</code></pre>

<h3>Главный цикл</h3>

<p>Давайте теперь соберем все вместе. Напишем главный цикл, который будет выглядеть как-то так (файл main.go):</p>

<pre><code>ws, id := slackConnect(token)

for {
    msg, err := getMessage(ws)
    // ...
    // обрабатываем сообщение и отвечаем при необходимости
    // ...
    postMessage(ws, reply)
}
</code></pre>

<p>Мы читаем все сообщения, но нам нужны только те, в которых упомянули нашего бота. Slack хранить все упоминания пользователя как строки вида &lt;@U1A2B3C4D>(U1A2B3C4D это идентификатор пользователя, которого упомянули) в тексте сообщения. Таким образом, сообщение вида "@mybot: stock goog" превратиться в "&lt;@U1A2B3C4D>: stock goog". Мы можем использовать эту информацию для распознавания упоминаний нашего бота.</p>

<pre><code>if m.Type == "message" &amp;&amp; strings.HasPrefix(m.Text, "&lt;@"+id+"&gt;") {
</code></pre>

<p>И пример обработки самого сообщения:</p>

<pre><code>parts := strings.Fields(m.Text)
if len(parts) == 3 &amp;&amp; parts[1] == "stock" {
    // получаем содержимое второй части сообщения через parts[2]
} else {
    m.Text = fmt.Sprintf("sorry, i didn't get that\n")
    postMessage(ws, m)
}
</code></pre>

<p>Для обработки сообщения и формирования ответа используем go-рутину:</p>

<pre><code>go func(m Message) {
    m.Text = getQuote(parts[2])
    postMessage(ws, m)
}(m)
</code></pre>

<p>Метод <code>getQuote</code> получает данные котировок по указанному тиккеру используя Yahoo API. Конечно же, вы можете заменить этот функционал.</p>

<p><a href="https://api.slack.com/bot-users">Вот несколько вещей</a>, которые может делать бот и неописанных в этой статье.</p>

<p>Удачи вам в создании новых Slack ботов. Вот еще чутка ссылок, которыебудут вам интересны:</p>

<ul>
<li><a href="https://github.com/rapidloop/mybot">Код бота на github</a></li>
<li><a href="https://api.slack.com/methods/rtm.start">Slack rtm.start API</a></li>
<li><a href="https://api.slack.com/rtm">Slack RTM API</a></li>
<li><a href="https://godoc.org/golang.org/x/net/websocket">Документация по websocket API</a></li>
</ul>

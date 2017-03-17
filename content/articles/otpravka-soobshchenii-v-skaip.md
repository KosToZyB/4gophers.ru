+++
date = "2014-02-26T00:57:12+03:00"
draft = false
title = "Cкайп, D-Bus и Go"

+++

<p>Мелкософт уже давно грозят прикрыть АПИ в skype для сторонних программ. Но видать,там что-то такое накуралешено, что они уже сами не разберутся. Поэтому апи пока работает и добраться до него можно. Но не факт что будет работать в следующих версиях. Если что, то у меня убунту 12.04 и версия скайпа 4.2.0.11</p>

<h3>D-Bus</h3>

<p><a href="http://ru.wikipedia.org/wiki/D-Bus">D-Bus</a> - это система для взаимодействия программ. Одно приложение может выполнять запросы к другому.</p>

<p>В Go есть несколько либ для работы с D-Bus. Самая нормальная это <a href="https://github.com/norisatir/go-dbus">github.com/norisatir/go-dbus</a>. В принципе, кроме этой либы нам больше ничего не нужно, можем подключаться к скайпу:</p>

<pre>
<code>package skype

import (
    &quot;log&quot;
    &quot;github.com/norisatir/go-dbus&quot;
)

var conn *dbus.Connection

func SkypeConnect() {
    var (
        err error
        conn *dbus.Connection
        method *dbus.Method
        out []interface{}
    )
    if conn, err = dbus.Connect(dbus.SessionBus); err != nil {
        log.Fatal(&quot;Connection error:&quot;, err)
    }

    //...
}
</code></pre>

<p>Замечательно. Теперь нужно аутентифицироваться и разрешить наше приложение в настройках скайпа</p>

<pre>
<code>if err = conn.Authenticate(); err != nil {
    log.Fatal(&quot;Authentication error:&quot;, err)
}
</code></pre>

<p>Теперь можем получить экземпляр API и начать работу непосредственно с скайповским D-Bus протоколом. Из самого API нам нужен метод Invoke</p>

<pre>
<code>obj := conn.Object(&quot;com.Skype.API&quot;, &quot;/com/Skype&quot;)

method, err = obj.Interface(&quot;com.Skype.API&quot;).Method(&quot;Invoke&quot;)
if err != nil {
    log.Fatal(err)
}
</code></pre>

<p>Используем API выполняя запросы через D-Bus. При выполнении первого вызова в скайпе появится окошко, которое попросит разрешить стороннему приложению подключиться к скайпу.</p>

<pre>
<code>out, err = conn.Call(method, &quot;NAME Go&quot;) // устанавливаем имя приложения
fmt.Println(out)
out, err = conn.Call(method, &quot;PROTOCOL 8&quot;) // это версия которая будет использоваться
                                           // при обращении к API
fmt.Println(out)
out, err = conn.Call(method, &quot;GET SKYPEVERSION&quot;)
fmt.Println(out)
out, err = conn.Call(method, &quot;SEARCH RECENTCHATS&quot;) // выведет список последних чатов
fmt.Println(out)
out, err = conn.Call(method, &quot;CHAT CREATE myfriend&quot;) // создаем новый чат с skype-пользователем
fmt.Println(out)
// отправляем сообщение в определенный чат
out, err = conn.Call(method, &quot;CHATMESSAGE #artemkovardin/$myfriend;8cbc8f1ec76c5a61 message&quot;)
</code></pre>

<p>Собственно, все. Если интересно разбираться дальше, то вы хрен найдете вменяемую документацию. Я нашел <a href="http://4gopgers.com/skype/ApiDoc-DesktopEdition.html">только это</a></p>

<p>Код к статье <a href="https://github.com/4gophers/skype-example">лежит на github</a></p>

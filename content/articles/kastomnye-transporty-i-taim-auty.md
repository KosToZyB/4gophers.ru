+++
date = "2015-06-03T02:32:02+03:00"
draft = false
title = "Кастомные транспорты и тайм-ауты"

+++

<p>Перевод "<a href="http://biasedbit.com/blog/golang-custom-transports/">Golang custom transports and timeouts</a>".</p>

<p>Этот пост - результат препарирования лога закрешеной Go ситемы, работающей в продакшене. Мне пришлось перешерстить логи, чтобы добраться до самых внутренностей кода, работающего с сетью, и разобраться с причинами утечек памяти в основной части нашего стека в Timehop. Проблемы возникли в результате использования кастомных транспортов для выполнения запросов и DNS-лукапа, а также использования старой версии glibc на Linux.</p>

<h3>TL;DR</h3>

<ul>
<li>Всегда явно задавайте тайм-ауты, особенно если вы используете кастомные транспорты.</li>
<li>Если вы работаете на Linux, обновите glibc на версию 2.20</li>
</ul>

<h3>"Ваш Timehop день готов"</h3>

<p>Один из элементов работы Timehop - это утренние пуш-нотификации на телефон:</p>

<p><img src="http://biasedbit.com/images/posts/golang-custom-transports-push.png" alt="The morning push" /></p>

<p>За этим уведомлением скрывается целый ряд серверных задач, так называемых подготовок. Они гарантируют, что все будет готово, когда вы проверите приложение утром.</p>

<p>Чтобы не углубляться в суть всех этих подготовок, в рамках этого поста я остановлюсь только на некоторых деталях. Самое важное:</p>

<ol>
<li>На момент написания этого поста, мы запускаем 11 миллионов подготовок каждый день</li>
<li>Каждая подготовка, в среднем, занимает 300 миллисекунд</li>
<li>Каждая подготовка конкурентна и интенсивно использует сетевой ввод/вывод(об этом писал <a href="http://twitter.com/kevrone">@kevrone</a> в <a href="https://medium.com/building-timehop/one-year-of-dynamodb-at-timehop-f761d9fe5fa1">этой статье</a>, так что вы можете оценить объем данных, с которым приходится работать)</li>
<li>Каждая машина конкурентно запускает 30 подготовок</li>
<li>У нас имеется 4 амазоновские <a href="http://aws.amazon.com/ec2/instance-types/#c3">c3.large</a> машины</li>
<li>Мы используем DataDog как главную мониторилку для нашего AWS CloudFormation стека.</li>
</ol>

<h3>Ничего не предвещало беды(пока)</h3>

<p>Когда мы перевалили за 10 миллионов проверок в день, мы начали наблюдать падения в метриках DataDog, хотя подготовки все равно продолжаются - мы наблюдаем за размером не-так-резервированной очереди альтернативным методом.</p>

<p>Поначалу, это случалось очень редко и мы не особо обращали на это внимание. Казалось вполне вероятным, что проблема была в рецепте для chef, который рестартовал DD агента или в самом агенте. Нам это было не очень интересно, мы просто гасили эту машину(хотя она работала, просто не общалась с DD) и CloudFormation поднимал новую. Вуаля, "пофикшено".</p>

<p>Но такие ситуации возникали все чаще:</p>

<p><img src="http://biasedbit.com/images/posts/golang-custom-transports-machine-goes-sip.png" alt="Очень хорошо видны ежедневные провалы по доступной памяти. Это время, когда DataDog полностью отмирал." /></p>

<p>Очень хорошо видны ежедневные провалы по доступной памяти. Это время, когда DataDog полностью отмирал.</p>

<p>Внезапная и очень сильная утечка памяти. И в тоже время, подготовки продолжают выполняться.</p>

<p>Я полез смотреть логи DD:</p>

<pre><code>2015-03-14 08:23:33 UTC | ERROR | dd.collector | collector(agent.py:354)
  | Uncaught error running the Agent
error: [Errno 12] Cannot allocate memory
</code></pre>

<p>И наши Go логи:</p>

<pre><code>fatal error: out of memory (stackcacherefill)
runtime stack:
runtime.throw(0xbb7bc0)
  /usr/local/go/src/pkg/runtime/panic.c:520 +0x69
stackcacherefill()
  /usr/local/go/src/pkg/runtime/stack.c:52 +0x94
</code></pre>

<p>Упс.</p>

<h3>Поиски утечки</h3>

<p>Когда программа закрешилась, то весь рантайм стек сдампился в лог, а это 4.5 миллиона строк.</p>

<p>После нескольких часов разбора и трассировки для каждой написанной нами go-рутины и функции, я обратил внимание на этот участок:</p>

<pre><code>goroutine 281550311 [semacquire, 27 minutes]:
sync.runtime_Semacquire(0xc2267cadc0)
  /usr/local/go/src/pkg/runtime/sema.goc:199 +0x30
sync.(*WaitGroup).Wait(0xc2337c8f80)
  /usr/local/go/src/pkg/sync/waitgroup.go:129 +0x14b
net.(*singleflight).Do(0xbbdc50, 0xc22f602e40, 0x20, ...)
  /usr/local/go/src/pkg/net/singleflight.go:37 +0x127
net.lookupIPMerge(0xc22f602e40, 0x20, 0x0, 0x0, 0x0, 0x0, 0x0)
  /usr/local/go/src/pkg/net/lookup.go:42 +0xae
net.func·025()
  /usr/local/go/src/pkg/net/lookup.go:80 +0x3e
created by net.lookupIPDeadline
  /usr/local/go/src/pkg/net/lookup.go:82 +0x2d8
</code></pre>

<p>Количество аналогичных блоков в файле, а также время, на которой go-рутина залипала (около 30 минут) мне показалось достаточно странным, я решил подсчитать количество аналогичных записей в логе...</p>

<pre><code class="sh">$ grep 'net.lookupIPDeadline' crash.log | wc -l
420563
</code></pre>

<p>Ого. Очень много зависших лукапов.</p>

<p><em>Но они все работают до определенного дедлайна и чистят за собой, правильно?</em></p>

<p>Ничуть. Вот код <a href="https://github.com/golang/go/blob/master/src/net/lookup.go#L69-L103"><code>lookupIPDeadline</code></a>:</p>

<pre><code class="go">// lookupIPDeadline ищет имя хоста с учетом определенного дедлайна.
func lookupIPDeadline(host string, deadline time.Time)
                     (addrs []IPAddr, err error) {
    if deadline.IsZero() {
        return lookupIPMerge(host)
    }

    // Мы могли бы передать дедлайн ниже, передав его в 
    // функцию поиска. Но наиболее часто используется 
    // реализация, вызывающая getaddrinfo без всяких тайм-аутов.
    timeout := deadline.Sub(time.Now())
    if timeout &lt;= 0 {
        return nil, errTimeout
    }
    t := time.NewTimer(timeout)
    defer t.Stop()

    ch := lookupGroup.DoChan(host, func() (interface{}, error) {
        return lookupIP(host)
    })

    select {
    case &lt;-t.C:
        // Время работы определения DNS истекло по некоторым причинам.
        // Следующий запрос  запустил определение DNS не дожидаясь
        // завершения текущего определения. Смотрите issue 8602.
        lookupGroup.Forget(host)

        return nil, errTimeout

    case r := &lt;-ch:
        return lookupIPReturn(r.v, r.err, r.shared)
    }
}
</code></pre>

<p>Несколько интересных замечаний:</p>

<ul>
<li>Если параметр deadline не определен, то нет гарантий, что функция что-то вернет.</li>
<li>Когда параметр deadline определен, то это только тайм-аут в Go и нет никаких более низкоуровневых тайм-аутов(на уровне ОС).</li>
<li>Обратите внимание на упомянутую <a href="https://github.com/golang/go/issues/8602">проблему #8602</a>, к которой мы вернемся позже.</li>
</ul>

<p>Я был удивлен отсутствием какого-либо низкоуровневого тайм-аута, при выполнении поиска DNS... Давайте заглянем внутрь <a href="https://github.com/golang/go/blob/78082dfa801d484848ac47c04ce3aa9805d2b0c9/src/net/lookup.go#L36-L53"><code>lookupIPMerge</code></a>:</p>

<pre><code class="go">func lookupIPMerge(host string) (addrs []IP, err error) {
    addrsi, err, shared :=
        lookupGroup.Do(host, func() (interface{}, error) {
            return lookupIP(host)
        })
    // ...
</code></pre>

<p>В этом месте функция <code>lookupIP</code> платформо-зависимая. В моем случае она определена в файле <a href="https://github.com/golang/go/blob/78082dfa801d484848ac47c04ce3aa9805d2b0c9/src/net/lookup_unix.go#L63-L69">lookup_unix.go</a> (бинарник запускается на Linux машине).</p>

<pre><code class="go">func lookupIP(host string) (addrs []IP, err error) {
  addrs, err, ok := cgoLookupIP(host)
  if !ok {
    addrs, err = goLookupIP(host)
  }
  return
}
</code></pre>

<p>Функция <code>cgoLookupIP</code> (которая использует <code>cgoLookupIPCNAME</code>) определена в файле <a href="https://github.com/golang/go/blob/78082dfa801d484848ac47c04ce3aa9805d2b0c9/src/net/cgo_unix.go#L84">cgo_unix.go</a></p>

<p>На всем нашем пути в недра кода никакие тайм-ауты не встречаются. Посмотрите на <a href="http://www.manpages.info/freebsd/getaddrinfo.3.html">документацию <code>getaddrinfo</code></a>, никакого упоминания про тайм-ауты.</p>

<p>В принципе, если вызывается <a href="https://github.com/golang/go/blob/master/src/net/lookup.go#L86-L88"><code>lookupIP(host)</code></a> из <a href="https://github.com/golang/go/blob/master/src/net/singleflight.go#L67-L100"><code>DoChan()</code></a> этой <a href="https://github.com/golang/go/blob/master/src/net/lookup.go#L42"><code>lookupGroup</code></a>(что, судя по названию, и происходит), то такое поведение заблокирует go-рутину навсегда. Все, до свидания.</p>

<p>Единственный способ исправить ситуацию, это пробрасывать тайм-аут ниже по стеку. На этом месте вернемся к проблеме #8602 и в коммите почитаем <a href="https://github.com/golang/go/commit/77595e462be07b8229f88cbdf947e320bfc7e639#diff-a020b66d0c34e7f7fda17e33ecd9abf9L60">вот это туду</a>:</p>

<pre><code class="go">// TODO(bradfitz): рассмотреть пробрасывание дедлайна ниже,
// в функцию поиска имени. Но это предполагает переписывание
// для нативного Go, cgo, Windows и так далее.
</code></pre>

<p>И в результате это было заменено на:</p>

<pre><code class="go">// Мы могли бы пробросить дедлайн ниже, в функцию поиска имени.
// Но чаще всего используются реализации на базе getaddrinfo, 
// у которой нет никаких тайм-аутов.
</code></pre>

<p>Да, вы прочитали все правильно:</p>

<blockquote>
  <p><code>getaddrinfo</code> у которой нет никаких тайм-аутов.</p>
</blockquote>

<p>В конце концов, единственный вариант пофиксить утечку, это реализация <code>getaddrinfo</code> с каким-то жестким лимитом на время работы DNS лукапа и выходом по истечению этого лимита.</p>

<p>Меня поразил столь большой недосмотр и понимание того, что нет никакого универсального и кроссплатформенного решения для пробрасывания тайм-аутов ниже по стеку. Это натолкнуло меня на мысль, что я тоже поступаю безответственно, полностью доверяя сторонней реализации.</p>

<p>В нашем случае это был баг <code>getaddrinfo</code> из glibc, который <a href="https://github.com/golang/go/issues/8602#issuecomment-66098142">пофиксили в версии v2.20</a>, поэтому единственным решением было обновить версию glibc.</p>

<p>Так можно проверить какая версия glibc используется в вашей системе(в случае с Linux):</p>

<pre><code class="sh">$ ldd --version
ldd (Ubuntu EGLIBC 2.19-0ubuntu6) 2.19
</code></pre>

<p>Теперь, когда проблема решена в экстренном режиме, у нас есть возможность оглянуться и посмотреть, как стоило поступать с самого начала.</p>

<h3>Используйте явные тайм-ауты везде</h3>

<p>Когда вы кодите что-то вроде такого...</p>

<pre><code class="go">foo := &amp;http.Transport{}
</code></pre>

<p>... то использование этого транспорта может завесить на неопределенный срок выполнение лукапа и/или TLS рукопожатия.</p>

<p>В случае незашифрованных соединений, инстанс <code>Transport</code> для установления соединения будет использовать функцию, указанную в <a href="https://github.com/golang/go/blob/77595e462be07b8229f88cbdf947e320bfc7e639/src/net/http/transport.go#L67-L70">поле <code>Dial</code></a>. Если <a href="https://github.com/golang/go/blob/77595e462be07b8229f88cbdf947e320bfc7e639/src/net/http/transport.go#L477-L482">функция не указана</a>, то по умолчанию будет использоваться <a href="https://github.com/golang/go/blob/77595e462be07b8229f88cbdf947e320bfc7e639/src/net/dial.go#L113-L144"><code>net.Dial</code></a>, которая создает временный <a href="https://github.com/golang/go/blob/77595e462be07b8229f88cbdf947e320bfc7e639/src/net/dial.go#L12-L27"><code>Dialer</code></a>, который может иметь или не иметь тайм-аут(ключевая мысль, что все таки может).</p>

<p>И так как вы, скорее всего, будете использовать этот транспорт совместно с <a href="https://github.com/golang/go/blob/77595e462be07b8229f88cbdf947e320bfc7e639/src/net/http/client.go#L25-L35"><code>http.Client</code></a>, то рекомендую устанавливать <a href="https://github.com/golang/go/blob/77595e462be07b8229f88cbdf947e320bfc7e639/src/net/http/client.go#L71">значение для поля <code>Timeout</code></a>. Имейте ввиду, что достаточно сложно реализовать глобальный тайм-аут для всего цикла запроса(установка соединение, отправка запрос и разбор ответа).</p>

<p>Я поднялся вверх по стеку вызовов(уже к нашему коду) и обнаружил вот это:</p>

<pre><code class="go">client := &amp;http.Client{
    Transport: &amp;http.Transport{},
}
</code></pre>

<p>Этот код был прям на второй строчкое. Быстрое и не очень красивое исправление проблемы - просто заменить <code>&amp;http.Transport{}</code> на <code>&amp;http.DefaultTransport</code>. Но можно пойти более правильным(для продакшен систем) путем:</p>

<pre><code class="go">secs := 3 // довольно агрессивно
client := &amp;http.Client{
    Transport: &amp;http.Transport{
        Proxy: http.ProxyFromEnvironment,
        Dial: (&amp;net.Dialer{
            Timeout:   secs * time.Second,
            KeepAlive: 30 * time.Second,
        }).Dial,
        TLSHandshakeTimeout: secs * time.Second,
    },
}
</code></pre>

<h3>Заключение</h3>

<ul>
<li>Используйте явные тайм-ауты везде, где это возможно.</li>
<li><a href="https://github.com/golang/go/blob/77595e462be07b8229f88cbdf947e320bfc7e639/src/net/http/client.go#L75"><code>http.DefaultClient</code></a> относительно безопасный, так как использует <a href="https://github.com/golang/go/blob/77595e462be07b8229f88cbdf947e320bfc7e639/src/net/http/transport.go#L28-L40"><code>http.DefaultTransport</code></a>, у которого предустановленны тайм-ауты на соединение(хотя и нет тайм-аутов на запросы, поэтому используйте осторожно).</li>
<li>Убедитесь, что ваша <code>Dial</code> функция в <code>http.Client</code> имеет определенные тайм-ауты и устанавливает <code>TLSHandshakeTimeout</code>.</li>
<li>Обновите glibc до версии 2.20 или выше.</li>
</ul>

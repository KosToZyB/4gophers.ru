+++
date = "2015-12-07T01:35:01+03:00"
draft = false
title = "Конкурентность в Go -'гонки'"

+++

<p>Перевод статьи "<a href="http://wysocki.in/golang-concurrency-data-races/">Golang concurrency - data races</a>"</p>

<p>Конкурентное программирование может оказаться очень непонятными сложным, если вы недостаточно внимательны. Когда у вас есть несколько go-рутин, которые читают или пишут в одну и ту же структуру данных, то всегда может наступить такой момент, когда эти потоки попытаются одновременно получить доступ к данным, что приведет к битым значениям.</p>

<h3>Начальные условия</h3>

<p>Чтобы убедиться, что у вас все правильно работает, вам нужно будет запустить примеры кода на машине с несколькими ядрами и значением <code>GOMAXPROCS</code> больше 1 (иначе не будет двух или более одновременно работающих go-рутин). Стоит отметить, что в Go версии выше 1.5, значение <code>GOMAXPROCS</code> автоматически равно количеству ядер.</p>

<h3>Пример 1 - "гонки"</h3>

<p>В примере, приведенном ниже, мы реализуем простой счетчик в основе которого инкремент целого значения.</p>

<p>Затем добавим 100 go-рутин, каждая из которых будет инкрементировать счетчик 10 000 раз, что в результате должно дать нам значение в 1 000 000.</p>

<pre><code>package main

import (
    "fmt"
    "time"
)

type intCounter int64

func (c *intCounter) Add(x int64) {
    *c++
}

func (c *intCounter) Value() (x int64) {
    return int64(*c)
}

func main() {
    counter := intCounter(0)

    for i := 0; i &lt; 100; i++ {
        go func(no int) {
            for i := 0; i &lt; 10000; i++ {
                counter.Add(1)
            }
        }(i)
    }

    time.Sleep(time.Second)
    fmt.Println(counter.Value())

}
</code></pre>

<p><a href="http://play.golang.org/p/_iZzudgmc5">Запустить на play.golang.org</a></p>

<p>Давайте запустим наш пример (запускайте пример на своей машина, в песочнице play.golang.org все будет хорошо, так как там <code>GOMAXPROCS</code> установлен в <code>1</code>)</p>

<pre><code>❯ go run counter.go
248863
</code></pre>

<p>Что произошло? Ведь мы должны были получить результат равный <code>1 000 000</code>. Вот это и называется "гонками"(data race).</p>

<p>Чтобы выловить такие моменты до релиза вашего приложения, периодически запускайте ваш код с флагом <code>-race</code>.</p>

<pre><code>go run -race app.go
</code></pre>

<p>И в результате вы увидите что-то такое:</p>

<pre><code>❯ go run -race app.go &gt;&gt; out.txt
==================
WARNING: DATA RACE
Read by goroutine 7:
  main.main.func1()
      /home/exu/src/github.com/exu/go-workshops/101-concurrency-other/app.go:24 +0x42

Previous write by goroutine 6:
  main.main.func1()
      /home/exu/src/github.com/exu/go-workshops/101-concurrency-other/app.go:24 +0x58

Goroutine 7 (running) created at:
  main.main()
      /home/exu/src/github.com/exu/go-workshops/101-concurrency-other/app.go:26 +0x92

Goroutine 6 (running) created at:
  main.main()
      /home/exu/src/github.com/exu/go-workshops/101-concurrency-other/app.go:26 +0x92
==================
Found 1 data race(s)
exit status 66
</code></pre>

<p>Да-да! Go может обнаруживать "гонки" автоматически, не забывайте пользоваться детектором гонок когда вы работаете с множеством go-рутин. Такие ошибки достаточно сложно воспроизвести и они могут проскакивать на продакшен, поэтому почаще пишите тесты.</p>

<p>И так, мы обнаружили "гонки". Что дальше? Давайте попробуем исправить их. Есть несколько подходов к этому, но основное правило очень простое - синхронизируйте ваши данные.</p>

<h3>Пример 2 - Атомарные счетчики</h3>

<p>Для начала попробуем исправить наше приложение с помощью атомарных счетчиков из пакета <code>sync/atomic</code>, который включен в стандартную библиотеку Go.</p>

<pre><code>package main

import (
    "fmt"
    "runtime"
    "sync/atomic"
    "time"
)

type atomicCounter struct {
    val int64
}

func (c *atomicCounter) Add(x int64) {
    atomic.AddInt64(&amp;c.val, x)
    runtime.Gosched()
}

func (c *atomicCounter) Value() int64 {
    return atomic.LoadInt64(&amp;c.val)
}

func main() {
    counter := atomicCounter{}

    for i := 0; i &lt; 100; i++ {
        go func(no int) {
            for i := 0; i &lt; 10000; i++ {
                counter.Add(1)
            }
        }(i)
    }

    time.Sleep(time.Second)
    fmt.Println(counter.Value())
}
</code></pre>

<p><a href="http://play.golang.org/p/6Qrd3j-zvs">Запустить на play.golang.org</a></p>

<p>Чтобы быть уверенным, в том что go-рутина не простаивает, мы используем явное "выталкивание"" с помощью <code>runtime.Gosched()</code> после каждой операции. Такое "выталкивание"" происходит автоматически при работе с <code>channel</code> или при блокирующих вызовах, таких как <code>time.Sleep</code>. Но в нашем конкретном случае мы сами должны позаботиться об этом сами.</p>

<p>Теперь наш счетчик потокобезопасный. Сейчас вы можете проверить, остались ли у вас "гонки":</p>

<pre><code>$ go run -race atomic.go
1000000
</code></pre>

<p>Ура! Мы победи "гонки"!</p>

<h3>Пример 3 - Мьютексы</h3>

<p>Теперь мы можем попробовать исправить наш пример с помощью мьютексов, которые предоставляются вместе со стандартной библиотекой в пакете <code>sync</code>. Использование атомарных счетчиков и выталкивание с помощью <code>runtime.Gosched</code> выглядит не очень красиво. Использование <code>mutex</code> - это более правильный подход.</p>

<p>Нам нужно будет немного изменить код:</p>

<pre><code>package main

import (
    "fmt"
    "sync"
    "time"
)

type mutexCounter struct {
    mu sync.Mutex
    x  int64
}

func (c *mutexCounter) Add(x int64) {
    c.mu.Lock()
    c.x += x
    c.mu.Unlock()
}

func (c *mutexCounter) Value() (x int64) {
    c.mu.Lock()
    x = c.x
    c.mu.Unlock()
    return
}

func main() {
    counter := mutexCounter{}

    for i := 0; i &lt; 100; i++ {
        go func(no int) {
            for i := 0; i &lt; 10000; i++ {
                counter.Add(1)
            }
        }(i)
    }

    time.Sleep(time.Second)
    fmt.Println(counter.Value())

}
</code></pre>

<p><a href="http://play.golang.org/p/zzGE5yByPo">Запустить на play.golang.org</a></p>

<p>И снова проверяем, остались ли у нас "гонки":</p>

<pre><code>$ go run -race mutex.go
1000000
</code></pre>

<p>Все отлично работает, "гонок" нет.</p>

<h3>Заключение</h3>

<p>Когда мы пишем многопоточное приложение:</p>

<ul>
<li>Нужно помнить, что теперь программа работает не последовательно</li>
<li>Нужно быть осторожным при синхронизации данных между go-рутинами</li>
<li>Необходимо использовать каналы, мьютексы и атомарные счетчики</li>
<li>Для поиска гонок необходимо использовать встроенные инструменты языка, <code>-race</code> ваш друг</li>
<li>Не помешало бы реализовать приведенные выше пример счетчика с помощью каналов</li>
</ul>

<h3>Что дальше?</h3>

<p>Если вам понравился это материал, то вы можете почитать мою серию <a href="http://wysocki.in/series/go-basics/">статей для начинающих</a></p>

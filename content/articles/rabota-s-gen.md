+++
date = "2014-03-15T17:15:05+03:00"
draft = false
title = "Работа с gen"

+++

<p><a href="http://clipperhouse.github.io/gen/">Gen</a> - сборная солянка идей из C#&rsquo;s Linq, JavaScript&rsquo;s Array методов и библиотеки underscore. И все это оформлено в некоторое подобие дженериков.</p>

<p>На самом деле, это генератор кода, который может ускорить работу с множествами.</p>

<p>Например, если у вас есть некоторый тип <code>SomeType</code>, то теперь не нужно таскать из проекта в проект куски кода для работы с срезами, типа <code>[]*SomeType</code>. Можно одной командой сгенерировать целую пачку методов, таких как Count, Where и т.д.</p>

<h3>Использование</h3>

<p>Ставится как большинство пакетов:</p>

<pre>
<code>$ go get github.com/clipperhouse/gen
</code></pre>

<p>Чтобы начать работу, нужно определить новый тип, и приписать магический комментарий.</p>

<pre>
<code>// +gen
type SomeType struct {
    Id string
}
</code></pre>

<p>Теперь запускаем gen в папке с файлом, в котором описан наш тип и получаем еще один файл, в котором будут описаны все методы для этого типа.</p>

<p>У меня новый тип определен в файле это <em>main.go</em>. Соответственно, рядом появился файл <em>sometype_gen.go</em></p>

<p>Генерацией методов можно управлять через тот же магический комментарий. Например, можно генерировать методы для указателя на структуру:</p>

<pre>
<code>// +gen *
type SomeType struct {
    //...
}
</code></pre>

<p>Сгенерирует методы для:</p>

<pre>
<code>type SomeType []*SomeType
</code></pre>

<p>Можно указать, какие именно методы должны генерироваться:</p>

<pre>
<code>// +gen * methods:&quot;Any,Where,Count&quot;
type SomeType struct {
    //...
}
</code></pre>

<p>Или наоборот, не должны генерироваться:</p>

<pre>
<code>// -gen * methods:&quot;Any,Where,Count&quot;
type SomeType struct {
    //...
}
</code></pre>

<p>Этот магический комментарий работает аналогично тегам в структурах, поэтому стоит придерживаться схожего синтаксиса.</p>

<p>Кроме тега <em>methods</em> можно указать тег <em>projections</em>, который позволит сгенерировать дополнительные полезные методы для определенных типов в структуре.</p>

<pre>
<code>// -gen * projections:&quot;int&quot;
type SomeType struct {
    Id int
}
</code></pre>

<p>И последний тег - <em>containers</em>. Поможет сгенерировать дополнительные структуры данных, которые можно юзать для своих нужд.</p>

<pre>
<code>// +gen containers:&quot;Set,List,Ring&quot;
type SomeType struct {
    //...
}
</code></pre>

<h3>Непонятности</h3>

<p>Мне не нравится идея одного <em>GOPATH</em> для всех проектов. Мне приятней и удобней структура проекта, описанная в <a href="http://4gophers.com/article/pristupaya-k-rabote">этой статье</a>. Но тут возникают проблемы с gen. Он не понимает дополнительные пути в <em>GOPATH</em>.</p>

<p>Проще говоря, если ваш проект будет иметь такую структуру:</p>

<pre>
<code>gentest/
    |- bin/
    |- pkg/
    |- src/
    |   |-4gophers.com/
    |        |-/somepackage/
    |            |-file.go
    |            |-somefile.go
    |- main.go
    |- activate.sh
    |- Gomfile
</code></pre>

<p>И путь к <em>gentest</em> будет дополнительно добавлен в <em>GOPATH</em></p>

<pre>
<code>$GOPATH=/home/user/go1.2/global:/home/user/Projects/gentest/vendor:/home/kovardin/Projects/gentest
</code></pre>

<p>То при использовании gen в корне вылезет такая ошибка:</p>

<pre>
<code>error: main.go:5:5: could not import 4gophers.com/somepackage (can&#39;t find import: 4gophers.com/
somepackage)
error: failed to evaluate type SomeType (-: undeclared name: SomeType)
operation canceled
use the -f flag if you wish to force generation (i.e., ignore errors)
</code></pre>

<p>И даже <em>gen -f</em> не поможет.</p>

<h3>Свои шаблоны</h3>

<p>Их нет. А я хочу)</p>

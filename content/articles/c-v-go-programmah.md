+++
date = "2014-05-26T12:44:12+03:00"
draft = false
title = "C в Go программах"

+++

<p><span style="line-height: 1.6em;">Статья, которая послужит памяткой о использовании сишного кода в программах на Go. Для профи тут мало чего интересного, но новичкам будет полезно.</span></p>

<h3 id="cgo">Cgo</h3>

<p>Go настолько замечательный язык, что его можно использовать не только вместо языка Си но и вместе с Си. Для этого есть специальная <a href="http://golang.org/cmd/cgo/">тулза cgo</a>. И есть <a href="http://blog.golang.org/c-go-cgo">замечательная статья</a>, которая объясняет основы использования cgo. Но там, как пример, приводится использование только стандартных библиотечных функций. В этой же статье я попытался показать маленький пример, как использовать cgo для работы со своими кусками сишного кода.</p>

<h3 id="c-go_1">C в go</h3>

<p>Для использования Си кода проще всего создать библиотеку, которую можно будет линковать с нашей программой.</p>

<p>Сперва напишем два файла: example.c</p>

<pre>
<code class="c">
int x( int y ) {
    return y+1;
}
</code></pre>

<p>и example.h:</p>

<pre>
<code class="c">
int x(int);
</code></pre>

<p>Теперь компилируем и создаем нашу библиотеку, которую потом будем использовать.</p>

<pre>
<code class="bash">
$ gcc -O2 -c example.c
$ ar q libexample.a example.o
</code></pre>

<p>Название библиотеки должно начинаться с lib иначе линкер не сможет ее найти.</p>

<pre>
<code class="go">
package main

// #cgo CFLAGS: -I.
// #cgo LDFLAGS: -L. -lexample
// #include &lt;example.h&gt;
import &quot;C&quot;

import &quot;fmt&quot;

func main() {
  fmt.Printf(&quot;Invoking c library...\n&quot;)
  fmt.Println(&quot;Done &quot;, C.x(10) )
}
</code></pre>

<p><code>#cgo CFLAGS: -I.</code> - Добавляет каталог &#39;директория&#39; в начало списка каталогов, используемых для поиска заголовочных файлов. <code>#cgo LDFLAGS: -L. -lexample</code> - Флаги линковщика, которые указывают в какой папке следует искать библиотеки(<code>-L.</code>) и какую библиотеку нужно подключать(<code>-lexample</code>).</p>

<p>Следует обратить внимание, что импорт виртуального пакета &quot;С&quot; (<code>import &quot;C&quot;</code>) проходит отдельно от импорта всех других пакетов. Кроме того, все комментарии с сишным кодом должны идти сразу над инструкцией импорта виртуального пакета &quot;С&quot;, без пропуска строк.</p>

<p>Теперь попробуем собрать все вместе</p>

<pre>
<code class="bash">
$ go build test.go
$ ./test
Invoking c library...
Done  11
</code></pre>

<p>Вуаля! все прекрасно работает.</p>

<h3 id="go-c">Go в C</h3>

<p>Кроме использования Си в программах на Go, так же можно использовать Go в программах на Си.</p>

<pre>
<code class="go">
package main

// extern void Use_In_C();
import &quot;C&quot;

import &quot;fmt&quot;

func main() {
    C.Use_In_C()
}

//export UseInC
func UseInC() {
    fmt.Println(&quot;UseInC()&quot;)
}
</code></pre>

<p><code>//export UseInC</code> - это наш хендлер, который мы будем использовать в Си коде.</p>

<p>Используем нашу Go функцию:</p>

<pre>
<code class="c">
#include &lt;stdio.h&gt;
#include &quot;_cgo_export.h&quot;
void Use_In_C() {
    UseInC();
}
</code></pre>

<p>Для того, чтобы собрать все это добро, проект должен обязательно находится в <code>$GOPATH</code> или подпапке <code>$GOPATH/src</code>. Если исходники лежать прям в корне <code>$GOPATH</code>, то собираем все одной командой</p>

<pre>
<code>
$ go build .
</code></pre>

<p>Если в подпапке <code>$GOPATH/src/projectname</code>, тогда собираем так:</p>

<pre>
<code>
$ go build projectname
</code></pre>

<h3 id="_1">Почитать по теме:</h3>

<ul>
	<li><a href="http://blog.golang.org/c-go-cgo">C? Go? Cgo!</a></li>
	<li><a href="https://code.google.com/p/go-wiki/wiki/cgo">Примеры использования cgo</a></li>
	<li><a href="http://golang.org/cmd/cgo/">Документация по cgo</a></li>
	<li><a href="http://www.goinggo.net/2013/08/using-c-dynamic-libraries-in-go-programs.html">Using C Dynamic Libraries In Go Programs</a></li>
</ul>

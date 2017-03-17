+++
date = "2014-03-29T19:02:07+03:00"
draft = false
title = "Публикация пакетов"

+++

<p>Стандартный <em>go get</em> прекрасно работает с популярными системами контроля версий. Но иногда хочется, чтоб пакеты были доступны для установки по кастомному урл. Например, если эти пакеты публикует какая-нибудь компания.</p>

<h3>Простой пример</h3>

<p>Есть некоторый пакет <a href="https://github.com/4gophers/hello">hello</a>, который хоститься на гитхабе. Этот пакет можно установить так:</p>

<pre>
<code>$ go get github.com/4gophers/hello
</code></pre>

<p>Теперь сделаем этот пакет доступным для установки как <em>4gophers.com/hello</em></p>

<p>Если команде go get указать путь не к одному из известных кодахостингов и без классификатора контроля версий, тогда программа попытается найти <code>&lt;meta&gt;</code> тег в ответе от сервера.</p>

<pre>
<code>&lt;meta name=&quot;go-import&quot; content=&quot;import-prefix vcs repo-root&quot;&gt;
</code></pre>

<ul>
	<li><em>import-prefix</em> - это путь, который будет соответствовать корню хранилища. Именно это нужно будет указывать в импорте и при использовании go get</li>
	<li><em>vcs</em> - это указание, какая система контроля версий используется.</li>
	<li><em>repo-root</em> - это реальный корень нашего репозитория.</li>
</ul>

<p>В нашем случае этот метатег будет выглядеть вот так:</p>

<pre>
<code>&lt;meta name=&quot;go-import&quot; content=&quot;4gophers.com/hello git https://github.com/4gophers/hello&quot;&gt;
</code></pre>

<p>Если нужно из репозитория тянуть больше одного пакета, то в <em>import-prefix</em> можно указать только <em>4gophers.com</em></p>

<pre>
<code>&lt;meta name=&quot;go-import&quot; content=&quot;4gophers.com git https://github.com/4gophers/hello&quot;&gt;
</code></pre>

<p>Таким образом, при обращении <code>go get 4gophers.com/somepackage</code> будет получен пакет <code>github.com/4gophers/hello/somepackage</code></p>

<h3>Немного динамики</h3>

<p>Что происходит, когда мы запускаем <code>go get 4gophers.com/hello</code>? К нашему серверу приходят запросы вида:</p>

<pre>
<code>https://4gophers.com/hello?go-get=1 (preferred)
http://4gophers.com/hello?go-get=1  (fallback)
</code></pre>

<p>Проверяется наличие метатега. Если есть тег, тогда приходит еще одни запрос к корню сайта <code>http://4gophers.com?go-get=1</code> и тоже проверяется метатег.</p>

<p>Наличие в параметра <code>go-get=1</code> дает нам возможность показывать метатег, только когда это действительно необходимо.</p>

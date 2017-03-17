+++
date = "2014-02-26T01:18:01+03:00"
draft = false
title = "Приступая к работе"

+++

<p>Для начала нужно настроить workflow с блекджеком и шлюхами. Для качественной работы нам нужно:</p>

<ul>
	<li>Менеджер версий Go</li>
	<li>Менеджер сторонних пакетов</li>
	<li>Как бонус - локальная документация</li>
</ul>

<h3>Менеджер версий</h3>

<p>Тут, вроде как, не очень много вариантов для выбора. <a href="https://github.com/moovweb/gvm">gvm</a> - самое отличное, что я нашел.</p>

<p>Ставится просто:</p>

<pre>
<code>$ bash &lt; &lt;(curl -s https://raw.github.com/moovweb/gvm/master/binscripts/gvm-installer)
</code></pre>

<p>Зависимости:</p>

<pre>
<code>$ sudo apt-get install curl git mercurial make binutils bison gcc
</code></pre>

<p>Устанавливаем новую версию Go и используем ее:</p>

<pre>
<code>$ gvm install go1.2
$ gvm use go1.2
</code></pre>

<p>Если gvm надоел, то удаляется тоже просто:</p>

<pre>
<code>$ rm -rf ~/.gvm
</code></pre>

<h3>Структура проекта</h3>

<p><a href="http://golang.org/doc/code.html">Вот тут</a> очень хорошо описано про базовую структуру проекта. Описанный подход не совсем мне нравиться. Хочется такую структуру для каждого проекта, а не для всех сразу. И это вполне достижимо.</p>

<pre>
<code>project/
    |- bin/
    |- pkg/
    |- src/
    |   |-4gophers.com/
    |        |-/somepackage/
    |            |-file.go
    |            |-somefile.go
    |- main.go
    |- Godeps
</code></pre>

<pre>
<code>$ source activate.sh
</code></pre>

<p>Теперь, весь код можно убирать в папку <em>src</em>, спокойно разделить на пакеты и использовать:</p>

<pre>
<code>import &quot;4gophers.com/somepackage&quot;
</code></pre>

<p>Папка <em>4gophers.com</em> в <em>src</em> нужна, чтобы разруливался конфликт имен при использовании пакетов.</p>

<h3>Управляем зависимостями</h3>

<p>Менеджеров пакетов для golang тьма тьмущая. Как-то не могут определиться с одним и использовать его. Это тема для отдельной статьи.</p>

<p>В принципе, можно юзать стандартный <em>go get</em>, но тут проблема с переключением между определенными версиями. И неудобно если в проекте много зависимостей.</p>

<p>Мне больше всего нравится <a href="https://github.com/pote/gpm">gpm</a> - никаких излишеств. Устанавливаем:</p>

<pre>
<code>$ git clone https://github.com/pote/gpm.git &amp;&amp; cd gpm
$ ./configure
$ make install
</code></pre>

<p>Использовать gpm лучше всего совместно с <a href="https://github.com/pote/gvp">gvp</a>. Это позволит устанавливать пакеты определенных версий в отдельную папку <em>.godeps</em>. Устанавливаем:</p>

<pre>
<code>$ git clone https://github.com/pote/gvp.git &amp;&amp; cd gvp
$ ./configure
$ make install
</code></pre>

<p>Используем:</p>

<pre>
<code>$ gvp init
$ source gvp in
$ source gvp out
</code></pre>

<p>Теперь можно устанавливать наши зависимости. Для этого в корне нужно создать файл <em>Godeps</em> с примерно таким содержанием:</p>

<pre>
<code>github.com/nu7hatch/gotrail               v0.0.2
github.com/replicon/fast-archiver         v1.02
launchpad.net/gocheck                     r2013.03.03   # Bazaar repositories are supported
code.google.com/p/go.example/hello/...    ae081cd1d6cc  # And so are Mercurial ones
</code></pre>

<p>И запустить:</p>

<pre>
<code>$ gpm install
</code></pre>

<p>Все необходимые пакеты скачаются и установятся в папку <em>.godeps</em>.</p>

<h3>Документация</h3>

<p>В Go очень удобная офлайн документация, которую можно посмотреть выполнив:</p>

<pre>
<code>$ godoc -http=:6060
</code></pre>

<p>Если все хорошо, в браузере можно будет посмотреть документацию <a href="http://localhost:6060">http://localhost:6060</a>. А если все как у меня, то <a href="http://godoc.org/code.google.com/p/go.tools">нужно заморочиться</a></p>

<pre>
<code>$ go get code.google.com/p/go.tools/godoc
$ go get code.google.com/p/go.tools/cmd/godoc
</code></pre>

<p>И еще раз попробовать запустить локальный сервер документации.</p>

<h3>Что дальше</h3>

<p>На самом деле, еще можно рассказать про настройку тулзовин для тестирования, сборки и т.д. Но на это одной статьи не хватит :)</p>

<p>Шаблон проекта <a href="https://github.com/4gophers/template">на гитхабе</a></p>

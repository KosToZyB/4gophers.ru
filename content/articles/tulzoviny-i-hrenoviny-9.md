+++
date = "2014-06-19T07:56:07+03:00"
draft = false
title = "Тулзовины и хреновины #9"

+++

<p>Победа над прокрастинацией! Задержавшийся выпуск &quot;тулзовин&quot; завершен. В этот раз необычно много всяких пакетов и проектов, прям хоть свой катологизатор делай. Рекомендую взглянуть на <a href="http://naoina.github.io/kocha/">Kocha</a>.</p>

<p>И большое объявление для сообщества:</p>

<p><a href="http://www.meetup.com/Golang-Moscow/events/186845472/">Встреча гоферов</a> в Москве. Уже совсем скоро - 21 июня 2014</p>

<p>Очередная встреча нашего сообщества, при поддержке <a href="http://undev.ru/">Undev</a>, <a href="http://nptv.com/">NPTV</a> и <a href="http://www.digitaloctober.ru/">Digital October</a>. Трансляция будет на сайте последнего (<a href="http://www.digitaloctober.ru/ru/events/golang_moscow">на странице мероприятия</a> и на главной).</p>

<p>Андреенко Артём из Openstat (Spylog) расскажет про опыт использования Go в своей компании в режиме реального времени.</p>

<p>Вячеслав Бахмутов из Яндекса расскажет про опыт использования Go в своей компании в облачной платформе Cocaine.</p>

<p>Александр Орловский из sports.ru расскажет про NSQ, очередь сообщений, написанную компанией Bitly на Go, и про её опыт использования в своей компании.</p>

<h3 id="_1">Проекты</h3>

<ul>
	<li><a href="https://github.com/jakecoffman/go-command-running-thing">go-command-running-thing</a> - Веб запускалка команд на удаленной машине. Это как shellinabox, только больше багов и менее юзабельна. Но на Go.</li>
	<li><a href="http://mholt.github.io/binding/">Binding</a> - Связывает данные из net/http реквеста с кастомными структурами и валидирует.</li>
	<li><a href="https://github.com/cjtoolkit/lexy">CJToolkit Lexy</a> - Необычная система шаблонов для Go. С смайликами:)</li>
	<li><a href="https://github.com/spf13/cobra">cobra</a> - Удобный пакет для создания консольных приложений.</li>
	<li><a href="https://github.com/benmanns/goworker">goworker</a> - Resque-совместимый воркер, который выполняет задачи из очереди. Написан на Go.</li>
	<li><a href="https://github.com/robertkrimen/otto">otto</a> - JavaScript парсер и интерпретатор написанный на нативном Go.</li>
	<li><a href="https://github.com/jwilder/docker-gen">docker-gen</a> - Генератор файлов, который рендерит шаблоны используя метаинформацию docker-контейнеров</li>
	<li><a href="https://github.com/skycoin/skycoin">skycoin</a> - Еще одна криптовалюта.</li>
	<li><a href="https://github.com/chewxy/nanjingtaxi">nanjingtaxi</a> - Безопасная, P2P распределенная система для чата на Go.</li>
	<li><a href="https://github.com/extemporalgenome/lwa">lwa</a> - Проект, который облегчает запуск Go веб приложений как десктоп программ. То еще извращение.</li>
	<li><a href="https://github.com/beefsack/go-astar">go-astar</a> - Алгоритм нахождения пути A*</li>
	<li><a href="https://github.com/dimfeld/httptreemux">httptreemux</a> - HTTP роутинг для Go. Простой, расширяемый.</li>
	<li><a href="https://github.com/abhishekkr/goshare">goshare</a> - Инструмент, для расшаривания любых данных между нодами. Работает поверх HTTP и ZeroMQ.</li>
	<li><a href="https://github.com/dimfeld/unwebhook">unwebhook</a> - Сервер, который дает возможность запускать разные команды по событиям для Gitlab или Github</li>
	<li><a href="https://github.com/vinceprignano/gochatapp">gochatapp</a> - Чат на Golang, AngularJs и Socket.io. Все модно и вебсокетно.</li>
	<li><a href="https://github.com/progrium/execd">execd</a> - Легковесный SSH фронт сервер написанный на Go.</li>
	<li><a href="https://github.com/eknkc/amber">amber</a> - Еще одни шаблонизатор, который &quot;как HAML и Jade&quot;, но на Go.</li>
	<li><a href="https://github.com/koding/websocketproxy">websocketproxy</a> - реверс прокси для вебсокетов. Работает поверх gorilla/websocket.</li>
	<li><a href="https://github.com/michaelmaltese/golang-distributed-filesystem">golang distributed filesystem</a> - Клон HDFS на языке программирования Go.</li>
	<li><a href="https://github.com/andlabs/ui">ui</a> - Либа для создания нативных графических интерфейсов на языке Go.</li>
	<li><a href="https://hockeypuck.github.io/">hockeypuck</a> - Сервер для OpenPGP публичных ключей.</li>
	<li><a href="https://github.com/chrhlnd/dynjson">dynjson</a> - Маленькая библиотека для динамического доступа к JSON на языке Go.</li>
	<li><a href="https://github.com/azer/boxcars">boxcars</a> - Простой, легконастраиваемый проксисервер написанный на языке Go.</li>
	<li><a href="https://github.com/lestrrat/peco">peco</a> - Инструмент похожий <a href="https://github.com/mooz/percol">percol</a> и взявший за основу его концепцию фильтрации для unix пайпов.</li>
	<li><a href="https://github.com/guilhermebr/gowf">gowf</a> - Еще один микрофреймворк на языке Go. Горшочек, не вари!</li>
	<li><a href="https://github.com/james4k/fault">fault</a> - Пакет, который реализует полезный паттерн ожидания нескольких go-рутин с ранним отказом.</li>
	<li><a href="https://github.com/microcosm-cc/bluemonday">bluemonday</a> - Санитайзер написанный на Go.</li>
	<li><a href="https://github.com/conformal/gotk3/">gotk3</a> - Go биндинг для Gtk3 и сопутствующих проектов. С документацией печаль.</li>
	<li><a href="https://github.com/pims/spark">spark</a> - Обертка над spark.io API и консольный клиент в придачу.</li>
	<li><a href="https://github.com/bmizerany/pat">pat</a> - Паттерн мультиплексор для net/http библиотеки. Очень в стиле Sinatra.</li>
	<li><a href="https://github.com/martensson/vaban">Vaban</a> - Пакет для контроля над Varnish Cache хостами с помощью REST Api</li>
	<li><a href="https://github.com/davegardnerisme/phonegeocode">phonegeocode</a> - Пакет для определения страны по номеру телефона.</li>
	<li><a href="http://reshnesh.github.io/pixlserv/">pixlserv</a> - Сервер для ресайза картинок на лету. Удобная и быстрая(?) штука написанная на Go.</li>
	<li><a href="https://github.com/yosssi/boltstore">BoltStore</a> - Фронтенд для сессий gorilla/sessions который использует Bolt.</li>
	<li><a href="https://github.com/natefinch/lumberjack">lumberjack</a> - Пакет для ведения логов с ротацией файлов.</li>
	<li><a href="https://github.com/outbrain/orchestrator">orchestrator</a> - Инструмент для управления структурой MySQL базы и ее визуализации.</li>
	<li><a href="http://naoina.github.io/kocha/">Kocha</a> - полноразмерный веб-фреймворк, аналогичный beego или revel, но с исправленным фатальным недостатком.</li>
	<li><a href="https://github.com/DavidHuie/quartz">quartz</a> - Инструмент, который позволяет вызывать Go код из Ruby приложения.</li>
	<li><a href="http://parnurzeal.github.io/gorequest/">gorequest</a> - Удобный HTTP клиент на Go. Написан по аналогии SuperAgent lib в Node.js.</li>
	<li><a href="https://github.com/docker/libchan">libchan</a> - Это как каналы в Go, только для сервисов в сети.</li>
	<li><a href="https://github.com/nexneo/samay">samay</a> - Консольные таймтрекер на Golang. Инструмент мечты.</li>
	<li><a href="https://github.com/VividCortex/trace">trace</a> - Весьма удобная тулза для отладки программ. Выводит номер горутины, номер строки, имя файла и имя вызывающей функции.</li>
	<li><a href="https://github.com/gwaldo/nag">nag</a> - Забавный консольный таймер-напоминатель написанный на Go.</li>
	<li><a href="https://github.com/takashi/erk">erk</a> - Консольный менеджер задач, аналогичный watson-ruby.</li>
</ul>

<h3 id="_2">Новости</h3>

<ul>
	<li>Вышел Go версии <a href="http://golang.org/dl/#go1.3rc1">1.3 rc1</a> и <a href="http://golang.org/dl/#go1.3rc2">rc2</a></li>
	<li>Выпуск <a href="http://www.golangweekly.com/archive/go-newsletter-issue-27/">golangweekly #27</a> и <a href="http://www.golangweekly.com/archive/go-newsletter-issue-28/">golangweekly #28</a>. Дайджест новостей из мира Go.</li>
	<li>Стабильный релиз интересного инструмента <a href="http://gopkg.in/clipperhouse/gen.v1">gen v1</a>, который добавляет немного дженериков в Go.</li>
	<li>Убунтовцы выложили на гитхаб свою <a href="https://github.com/juju/juju">juju</a>.</li>
	<li>Прошел <a href="http://dockercon.com/">DocerCon 2014</a> и зарелизился <a href="http://blog.docker.com/2014/06/its-here-docker-1-0/">Docker v1.0</a>.</li>
	<li><a href="http://gopkg.in/igm/sockjs-go.v2/sockjs">sockjs-go</a> дожил до версии 2. Это sockjs сервер, написанный на Go.</li>
	<li>Релиз моего любимого менеджера зависимостей <a href="https://github.com/pote/gpm/releases/tag/v1.2.3">gpm v1.2.3</a></li>
</ul>

<h3 id="_3">Статьи и статейки</h3>

<ul>
	<li><a href="https://willnorris.com/2014/05/go-rest-apis-and-pointers">Go, REST APIs и указатели</a>. И еще разные пакеты.</li>
	<li><a href="http://bcarrell.me/posts/why_go/">Причины</a>, по которым функциональным программистам стоит посмотреть в сторону Go.</li>
	<li><a href="http://vluxe.io/golang-archive.html">Работа с архивами</a> в языке программирования Go.</li>
	<li><a href="http://austingwalters.com/templating-in-go/">Шаблоны</a>(не generic) в языке программирования Go.</li>
	<li>Очень простое <a href="http://metakeule.github.io/article/wrap-go-middlware-framework.html">веб-мидлваре на Go</a> в 13 строчек кода.</li>
	<li><a href="http://blog.yhathq.com/posts/yhat-meets-go.html">&quot;Yhat meets Go&quot;</a>. Статья в блоге Yhat о том, как они используют Go.</li>
	<li><a href="http://elithrar.github.io/article/generating-secure-random-numbers-crypto-rand/">Секурная генерация</a> случайных чисел с использованием crypto/rand.</li>
	<li><a href="http://blog.mozo.jp/2014/06/synchronization-patterns-in-go.html">Паттерн синхронизации</a> в языке программирования Go.</li>
	<li><a href="http://getprismatic.com/story/1401430056464">Одна неделя</a> с языком программирования Go.</li>
	<li><a href="http://stevieholdway.tumblr.com/post/87438353774/from-martini-to-golang-stdlib-and-gorilla-mux">Переход с Martini</a> на стандартную библиотеку Go и Gorilla MUX.</li>
	<li><a href="http://vanmaasakkers.net/2014/05/28/url-request-post-request-struct/">Пример преобразования данных</a> из POST в структуру без использования сторонних фреймворков на чистом Go.</li>
	<li><a href="http://golang-examples.tumblr.com/post/87553422434/template-and-associated-templates">&quot;Template and Associated templates&quot;</a> - в деталях описываются ассоциативные шаблоны в Go.</li>
	<li><a href="http://abdullin.com/long/happypancake/">Очень большая, подробная и интересная статья</a> про опыт создания сложного сайта знакомств на Go.</li>
	<li>Еще одна статья из цикла &quot;Building a Web Server in Go&quot;. <a href="http://austingwalters.com/building-a-web-server-in-go-salting-passwords/">Теперь учимся хранить пароли</a>.</li>
	<li><a href="http://www.webupd8.org/2014/06/syncthing-open-source-bittorrent-sync.html">Хорошая обзорная статья</a> про P2P дропбоксозаменитель <a href="http://syncthing.net/">Syncthing</a>.</li>
	<li><a href="http://dave.cheney.net/2014/06/07/five-things-that-make-go-fast">&quot;5 моментов, которые сделают Go быстрее&quot;</a>. Статья в блоге Dave Cheney.</li>
	<li>Замечательная <a href="http://spf13.com/post/is-go-object-oriented">статья о объектно ориентированности</a> языка Go.</li>
	<li>Шаг за шагом учимся использовать <a href="http://golang-basic.blogspot.com/2014/06/step-by-step-guide-to-ssh-using-go.html">Go для работы с SSH</a>.</li>
	<li><a href="http://golang-basic.blogspot.com/2014/06/curl-in-golang-go-curl.html">Работаем с cURL</a> на языке программирования Go.</li>
	<li><a href="http://habrahabr.ru/post/225907/">&quot;Является ли Go языком ООП?&quot;</a> - перевод статьи <a href="http://spf13.com/post/is-go-object-oriented">&quot;Is Go An Object Oriented Language?&quot;</a> сделанный <a href="https://twitter.com/kouprianov">@kouprianov</a></li>
	<li>&quot;<a href="http://lk4d4.darth.io/posts/streak/">30 days of hacking Docker&quot;</a> - <a href="https://twitter.com/LK4D4math">@LK4D4math</a> делится опытом использования Docker.</li>
	<li><a href="http://vluxe.io/golang-bufio.html">&quot;Gopher Go! - Bufio&quot;</a> - статья про работу с пакетом bufio в языке программирования Go.</li>
	<li><a href="http://vluxe.io/os-syscall.html">&quot;Gopher Go! - OS &amp; Syscall&quot;</a>. Использование пакета os и системные вызовы в Go.</li>
	<li><a href="http://www.sitepoint.com/go-building-web-applications-beego/">Создаем веб-приложение</a> на Go c помощью фреймворка beego.</li>
</ul>

<h3 id="_4">Видео</h3>

<p><a href="http://channel9.msdn.com/Events/Lang-NEXT/Lang-NEXT-2014/From-Parallel-to-Concurrent">&quot;From Parallel to Concurrent&quot;</a> - обязательный к просмотру доклад от Роба Пайка на конференции Lang.NEXT 2014</p>

<h3 id="_5">Всякое</h3>

<ul>
	<li><a href="http://hidskes.com/blog/2014/05/21/ethereum-dapp-development-for-web-developers/">Ethereum &ETH;Apps</a> для разработчиков - пишем свою криптовалюту.</li>
	<li><a href="https://github.com/mkaz/working-with-go">Простые примеры</a> для новичков, которые помогают учиться читать Go код.</li>
	<li>Новая книга: <a href="https://dl.dropboxusercontent.com/u/750049/4gophers.com/books/golang-tdd.zip">&quot;Test-driven development with Go&quot;</a></li>
	<li>Сказ о том, как <a href="http://thenewstack.io/from-node-js-to-go-why-one-startup-made-the-switch/">Kelsey Falter с Node.js на Go</a> перелазила.</li>
	<li>Презентация <a href="http://slides.yoss.si/gocon/martini.html#/">&quot;Martini. Web framework for Go&quot;</a> c GopherCon 2014.</li>
	<li><a href="http://play.golang.org/p/XnPglbkLW4">Иногда</a> 2 + 2 равно 5.</li>
	<li><a href="http://talks.golang.org/2014/gocon-tokyo.slide#1">&quot;Go: 90% Perfect, 100% of the time&quot;</a> - немного оффтопная презентация о том, насколько хорош Go.</li>
	<li><a href="https://code.ohloh.net/search?s=%22if%20err%20!%3D%20nil%22&amp;pp=0&amp;fe=go&amp;mp=1&amp;ml=0&amp;me=1&amp;md=1&amp;ff=1&amp;filterChecked=true">Забавное сравнение</a> кто как обрабатывает ошибки.</li>
	<li><a href="https://jobs.github.com/positions/af511ae4-eb52-11e3-815e-7758c19a2ab8">В Apple ищут человека</a> со знанием Go. Это успех.</li>
	<li>Небольшая <a href="https://leanpub.com/howdoiusesourcegraph">бесплатная книга</a> про использование Sourcegraph от Satish Talim.</li>
	<li>Хороший <a href="https://github.com/gedex/yet-another-simple-note">пример todo</a> приложения на Go и JS.</li>
	<li><a href="https://github.com/julienschmidt/go-http-routing-benchmark">Сравнительные бенчмарки</a> web-фреймворков на Go.</li>
	<li><a href="http://talks.golang.org/2012/10things.slide#1">10 вещей</a>, которые вы (возможно) не знаете про Go.</li>
	<li>Немного сумасшествия. <a href="https://github.com/cryptix/GopherAngularTutorial">Gopher Angular Tutorial</a> - порт <a href="https://docs.angularjs.org/tutorial">AngularJS.org tutorial</a> на <a href="https://github.com/gopherjs/gopherjs">GopherJS</a> с использованием <a href="https://github.com/gopherjs/go-angularjs">go-angularjs</a>.</li>
	<li>Совсем недавно прошла Docker конфа. Там показали много интересных проектов, найти которые можно в <a href="https://github.com/docker">Docker сообществе на github</a>.</li>
	<li><a href="http://www.slideshare.net/SergeyLerg/go-google-app-engine">&quot;Go на Google App Engine&quot;</a> - Доклад с DevConf от Сергея Лалова.</li>
	<li><a href="http://blog.natefinch.com/2014/06/autogenerate-docs-with-this-one-dumb.html">Одни простой трюк</a> для автоматической генерации документации от Nate Finch.</li>
	<li><a href="https://github.com/mmcgrana/pgpin">pgpin</a> - Пример database-backend сервиса на Go, клон dataclips.heroku.com.</li>
	<li><a href="http://www.sporcle.com/games/MatrixFrog/go-keywords">Небольшой тест</a> на знание зарезервированных слов в Go.</li>
</ul>

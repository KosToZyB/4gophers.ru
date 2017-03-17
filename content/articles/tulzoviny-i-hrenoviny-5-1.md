+++
date = "2014-05-13T00:05:12+03:00"
draft = false
title = "Тулзовины и хреновины #7"

+++

<p><span style="line-height: 1.6em;">С днем победы, мои Go-товарищи! Ура!</span></p>

<p>Пару недель назад прошла самая ожидаемая golang конфа и теперь есть много-много материалов для переваривания, обсуждения и размышления. Мой пост,&nbsp;со всеми нарытыми презентациями и видосами,&nbsp;<a href="http://4gophers.com/article/gophercon-2014">можно найти тут</a>.</p>

<p>И, кто еще не в куре, подписываемся на рассылку <a href="http://4gophers.com/subscribe">&quot;тулзовин и хреновин&quot;</a>.</p>

<h3 id="_1">Проекты</h3>

<ul>
	<li><a href="https://github.com/inconshreveable/go-update">go-update</a> - Автоматическое обновление go-программ.</li>
	<li><a href="https://goless.readthedocs.org/en/latest/index.html">goless</a> - Забавная шутка, которая делает python слегка похожим на Go. Можно юзать горутины, каналы, select.</li>
	<li><a href="https://github.com/joeshaw/gengen">gengen</a> - Опять дженерики в Golang. Неймется людям.</li>
	<li><a href="http://jmoiron.github.io/sqlx/">SQLX</a> - Пакет, расширяющий возможности стандартного database/sql.</li>
	<li><a href="https://github.com/rcrowley/go-metrics">go-metrics</a> - Порт java проекта <a href="https://github.com/dropwizard/metrics">metrics</a> на golang.</li>
	<li><a href="http://brace.r358.org/">Brace</a> - Несколько конкурентных утилит для golang, которые делают жизнь проще.</li>
	<li><a href="https://github.com/martini-contrib/binding">martini-contrib/binding</a> - Пакет для связывания данных в запросе и валидации.</li>
	<li><a href="https://github.com/rcarmo/raspi-cluster">raspi-cluster</a> - Raspberry Pi кластер с управлением на golang.</li>
	<li><a href="https://github.com/qur/withmock">withmock</a> - Инструмент для создания заглушек при тестировании.</li>
	<li><a href="https://github.com/apg/wipes">wipes</a> - Сервер, который получает текстовый вывод программы через *nix канал и отдает его в вебсокет соединение.</li>
	<li><a href="https://github.com/mattn/webapp-vim">webapp-vim</a> - Сервер, который позволяет программировать веб на Vim скрипте. Знатное извращение.</li>
	<li><a href="http://robteix.com/blog/2014/04/29/package-validate/">Валидация</a> в Go для данных в структурах</li>
	<li><a href="https://github.com/pengux/check">check</a> - Еще один пакет для валидации данных.</li>
	<li><a href="https://github.com/xlvector/hector">hector</a> - Либа для машинного обучения и решения проблем бинарной классификации.</li>
	<li><a href="https://github.com/zachlatta/postman">postman</a> - Консольная утилита для пакетной отправки электронной почты.</li>
	<li><a href="http://spf13.com/project/viper">Viper</a> - Управление конфигурацией с клыками для Go приложений.</li>
	<li><a href="https://github.com/dgryski/hokusai">hokusai</a> - Реализация Hokusai paper.</li>
	<li><a href="http://tech.gilt.com/post/65724312210/goreq-a-simple-sane-http-request-library-for-go">GoReq</a> - Простая и красивая либа для реализации HTTP запросов в Go</li>
	<li><a href="https://github.com/ChimeraCoder/anaconda">anaconda</a> - Пакет для работы с твиттер апи в Golang.</li>
	<li><a href="https://github.com/mitchellh/mapstructure">mapstructure</a> - Пакет для превращения мапов в структуры.</li>
	<li><a href="https://github.com/larzconwell/ar">ar</a> - Пакет для доступа к ar архивам</li>
	<li><a href="https://github.com/koyachi/go-nude">go-nude</a> - Позволяет определить есть ли на фотографии голые тела.</li>
	<li><a href="https://github.com/pascalj/disgo">disgo</a> - Свой собственный сервис комментариев на Go</li>
	<li><a href="https://github.com/aokoli/goutils">GoUtils</a> - Инструмент для манипулирования строками в Golang.</li>
	<li><a href="https://github.com/agonopol/gosplat">gosplat</a> - Пакет для рисования графиков, который генерирует готовый HTML файл.</li>
	<li><a href="https://github.com/mesosphere/mesos-go">mesos-go</a> - Golang биндинг к Apache Mesos.</li>
	<li><a href="https://github.com/nsf/termbox-go">termbox-go</a> - Библиотека для построения простых и минималистичных текстовых интерфейсов.</li>
	<li><a href="https://kidoman.io/framework/embd.html">EMBD</a> - Фреймворк для встраиваемых систем, который обеспечивает взаимодействие с многими аппаратными датчиками (например, гироскоп, магнитометр, барометр и т.д.)</li>
	<li><a href="https://code.google.com/p/biogo/">Biogo</a> - Биоинформационная библиотека для Go.</li>
	<li><a href="https://github.com/mkaz/lanyon">lanyon</a> - Простой вебсервер, который читает markdown файлы.</li>
	<li><a href="https://github.com/jmervine/GoT">GoT</a> - Набор хелперов, которые упрощают написание тестов для Golang проектов.</li>
</ul>

<h3 id="_2">Новости</h3>

<ul>
	<li>Минорный релиз <a href="https://groups.google.com/forum/#!msg/golang-nuts/WDX2hJfqfck/tIqtMwvFvK0J">Go v1.2.2</a>.</li>
	<li>Релиз системы дистрибьютинга сообщений <a href="https://github.com/bitly/nsq/releases/tag/v0.2.28">nsq v0.2.28</a></li>
	<li>Новый релиз пакета для работы с геопозиционированием, расчетом координат, расчетом расстояния, etc - <a href="https://github.com/kellydunn/golang-geo">golang-geo v0.3.0</a>.</li>
	<li>Новый выпуск <a href="http://www.golangweekly.com/archive/go-newsletter-issue-25/">Go Newsletter #25</a>.</li>
	<li>Mailgun зарелизили официальное <a href="http://blog.mailgun.com/post/the-official-go-sdk-available-now/">SDK для golang</a>.</li>
	<li>После 150 релизов и 9 месяцев альфы, <a href="https://coreos.com/blog/coreos-beta-release/">мир увидел бета релиз coreos</a>.</li>
	<li>Свежий релиз <a href="https://github.com/ant0ine/go-json-rest/releases/tag/v2.0.3">go-json-rest v2.0.3</a>.</li>
	<li><a href="http://www.goinggotraining.net/">goinggotraining</a> - Вроде как, скоро будут курсы по языку программирования Go.</li>
</ul>

<h3 id="_3">Статьи и статейки</h3>

<ul>
	<li><a href="http://justinas.org/best-practices-for-errors-in-go/">Лучшие практики</a> при работе с ошибками в Go.</li>
	<li><a href="http://peter.bourgon.org/go-in-production/">Примеры использования</a> Go в реальном продакшене. И <a href="http://4gophers.com/2014-talks/best-practices-for-production-environments.pdf">презентация</a> к статье.</li>
	<li><a href="http://milocast.com/golang01.html">Первая</a> и <a href="http://milocast.com/golang02.html">вторая</a> статья от начинающего гофера Shawn Milochik, в которых он делится своим опытом. Будет интересно для новичков.</li>
	<li><a href="http://nathanleclaire.com/blog/2014/04/27/a-surprising-feature-of-golang-that-colored-me-impressed/">Статья от Nathan Leclaire</a> про работу с хештаблицами в Go.</li>
	<li>Интересная <a href="http://www.onebigfluke.com/2014/04/gos-power-is-in-emergent-behavior.html">статья про хитрости</a> работы с интерфейсами в Go. Возможно, вы не все знаете про http.HandlerFunc.</li>
	<li><a href="https://boomstarter.ru/projects/120343/tekstovaya_rasshifrovka_2-go_sezona_it-podkasta_eaxcast">Поможем</a> хорошему подкасту быть еще лучше. Дай немного денег для <a href="http://eax.me/tag/podcast/">EaxCast</a>.</li>
	<li><a href="http://jmoiron.net/blog/go-performance-tales/">Сказки</a> про производительность Go. <a href="http://matt.aimonetti.net/posts/2014/04/28/refactoring-go-code/">Go, роботы и рефакторинг кода</a> - статья о прошедшей GopherConf от Matt Aimonetti.</li>
	<li><a href="http://engineering.viki.com/blog/2014/thread-safe-bytepool-for-go/">Многопоточно-безопасный</a> байтпул на Go. И <a href="http://openmymind.net/Understanding-Pools/">пост автора</a> пакета, который проясняет несколько моментов.</li>
	<li><a href="http://nelhagedebugsshit.tumblr.com/post/84342207533/things-i-learned-writing-a-jit-in-go">Вещи</a>, которые понимаешь при написании JIT на Go.</li>
	<li><a href="http://www.jerf.org/iri/post/2931">&quot;Go: More UNIX than UNIX&quot;</a> - статья про Go и развитие идеи UNIX.</li>
	<li><a href="http://www.sjwhitworth.com/machine-learning-in-go-using-golearn/">Машинное обучение</a> с использованием GoLearn.</li>
	<li><a href="http://influxdb.org/blog/2014/04/30/java-is-the-cobol-of-my-generation-and-go-is-its-successor.html">Впечатления и размышления</a> после GopherCon в блоге influxDB.</li>
	<li><a href="http://sendgrid.com/blog/send-email-go-google-app-engine/">Отправка email</a> сообщений с помощью Go в GAE.</li>
	<li><a href="http://supermar.in/go-for-a-rubyist/">Golang</a> для рубистов.</li>
	<li><a href="http://herman.asia/efficient-string-concatenation-in-go">Эффективная конкатенация</a> строк в Go.</li>
	<li>Новая статья от William Kennedy <a href="http://www.goinggo.net/2014/05/methods-interfaces-and-embedded-types.html">&quot;Методы, интерфейсы и встроенные типы в Go&quot;</a></li>
	<li>Реализация алгоритма <a href="http://blog.bekijkhet.com/2014/05/golang-trie-implementation.html">префиксного дерева</a> на Go.</li>
	<li><a href="http://blog.joshsoftware.com/2014/05/05/my-experience-at-the-awesome-first-ever-gophercon-2014/">Впечатления</a> после GopherCon 2014 от Satish Talim</li>
	<li><a href="http://www.darkcoding.net/software/gophercon-2014-best-talks-and-notes/">Еще одни обзор</a> GopherCon 2014 от Graham King.</li>
	<li><a href="http://blog.leanstack.io/how-docker-was-born">Отличное интервью</a> с J&eacute;r&ocirc;me Petazzoni, который рассказывает о идеи создания Docker.</li>
	<li><a href="http://highscalability.com/blog/2014/5/7/update-on-disqus-its-still-about-realtime-but-go-demolishes.html">Go постепенно заменяет</a> python в популяной системе комментариев Disqus. Особенно, в риалтайм части.</li>
	<li><a href="https://software.intel.com/en-us/blogs/2014/05/10/debugging-performance-issues-in-go-programs">Отладка проблем</a> с производительностью в Go программах.</li>
</ul>

<h3 id="_4">Видео</h3>

<ul>
	<li><a href="http://4gophers.com/video/zeromq">&quot;Лучшие практики&quot; и &quot;Работа с ZeroMQ&quot;</a></li>
	<li><a href="http://4gophers.com/video/prilozhenie-dlya-gae-1">Первая часть</a> видео туториала по созданию веб приложения для Google App Engine.</li>
	<li>Про настройки, использование SDK, администрирование, деплой. Все, что нужно <a href="http://4gophers.com/video/gae-i-go-nachalo-ispolzovaniya">для начала работы с Google App Engine</a>.</li>
	<li><a href="http://4gophers.com/video/prilozhenie-dlya-gae-2">Продолжение серии</a> про создание Light Grid на Google App Engine</li>
	<li><a href="http://4gophers.com/video/prilozhenie-dlya-gae-3">Последнее видео</a> про создание Light Grid на Google App Engine и Go.</li>
	<li><a href="http://4gophers.com/video/gophercon-2014-rob-pike">Видео с GopherCon</a> 2014 с Rob Pike в главной роли.</li>
	<li><a href="http://4gophers.com/video/gophercon-2014-data-snarfing-with-go">Доклад от Rob Miller</a> про бигдата и крутой инструмент от Mozilla - Heka.</li>
	<li><a href="http://4gophers.com/video/gophercon-2014-gophers-on-a-plane">Доклад от David Symonds</a> про Google App Engine и Go в этом самом GAE.</li>
	<li>Kelsey Hightower на GopherCon 2014 <a href="http://4gophers.com/video/gophercon-2014-go-for-sysadmins">рассказывает как использовать Go</a>.</li>
	<li><a href="http://4gophers.com/video/intro-to-go">Еще одно введение</a> в Golang, но теперь с разными плюшками типо Ginkgo и прочим TDD.</li>
	<li>Derek Collison на <a href="http://4gophers.com/video/high-performance-systems-in-go">GopherCon рассказывает</a> о том, как строить высокопроизводительные системы с Go</li>
</ul>

<h3 id="_5">Всякое</h3>

<ul>
	<li>Презентация <a href="http://www.slideshare.net/albertstrasheim/serialization-in-go">&quot;Serialization in Go&quot;</a> от Albert Strasheim</li>
	<li>Некоторые <a href="http://blog.erlware.org/2014/04/27/some-thoughts-on-go-and-erlang/">размышления</a> о Go и Erlang.</li>
	<li><a href="http://sawyerhood.com/blog/?p=24">Сравнение Python и Go</a> на примере простого UDP клиента и сервера.</li>
	<li>Презентация <a href="http://talks.golang.org/2014/go4gophers.slide#1">&quot;Go for gophers&quot;</a> от Andrew Gerrand.</li>
	<li><a href="http://gobot.io/blog/2014/04/29/taking-the-stage-at-gophercon/">Роботы</a> на GopherConf 2014.</li>
	<li><a href="http://talks.golang.org/2014/c2go.slide#1">&quot;Go, from C to Go&quot;</a> - презентация с GopherCon 2014 от Russ Cox</li>
	<li><a href="https://blog.conformal.com/btcd-getwork-cgminer-profit/">Майнинг биткоинов</a> с помощью <a href="https://github.com/conformal/btcd">btcd</a>, getwork и cgminer.</li>
	<li><a href="http://go-talks.appspot.com/github.com/davecheney/gosyd/gccgo.slide#1">&quot;Experiments with gccgo&quot;</a> - презентация от Dave Cheney из Canonical.</li>
	<li><a href="http://labs.strava.com/slide/">Slide</a> - Интересный проект, который работает с GPS данными и использует Go на серверной стороне.</li>
	<li>Еще одна презентация с GoherCon 2014 <a href="http://rcrowley.org/talks/gophercon-2014.html#1">&quot;Building web services in Go&quot;</a> от Richard Crowley.</li>
	<li>Та самая игра <a href="https://github.com/peterhellberg/life">&quot;Жизнь&quot;</a> теперь на Go.</li>
	<li>Исчерпывающий <a href="https://docs.google.com/file/d/0B2GBHFyTK2N8TzM4dEtIWjBJdEk/edit">мануал</a> по созданию вебприложений на Go.</li>
	<li><a href="https://github.com/wmbest2/rats_server">RATS</a> - сервис для удаленного запуска тестов на андроид устройствах.</li>
	<li><a href="http://packetbeat.com/">packetbeat</a> - Крутейшие Golang проект для мониторинга состояния вашего приложения.</li>
	<li>Попытка сделать <a href="http://kidoman.io/programming/god-save-the-js.html">gofmt для JavaScript</a>.</li>
	<li><a href="http://go-tour-ru.appspot.com/#1">Перевод на русский</a> язык сайта tour.golang.org.</li>
	<li><a href="http://docs.deis.io/en/latest/gettingstarted/concepts/">Deis</a> - это простой, расширяемый и мощный PaaS, который позволяет деплоить Twelve-Factor приложения как Docker контейнеры на кластеры CoreOS машин.</li>
</ul>

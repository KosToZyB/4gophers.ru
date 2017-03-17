+++
date = "2014-03-23T16:57:04+03:00"
draft = false
title = "Тулзовины и хреновины #4"

+++

<p><span style="line-height: 1.6em;">Парам-пам-пам! Тема выпуска - муки выбора. Какой </span><a href="https://docs.google.com/forms/d/1_yqks892m3e5zK1d-6iziRIt_1LdZvrvDOysjvVRpOU/viewanalytics" style="line-height: 1.6em;">ORM пользоваться</a><span style="line-height: 1.6em;">? </span><a href="http://dave.cheney.net/2014/03/22/thoughts-on-go-package-management-six-months-on" style="line-height: 1.6em;">Каким пакетным</a><span style="line-height: 1.6em;"> менеджером? Что использовать </span><a href="http://www.reddit.com/r/golang/comments/20umt3/how_do_you_handle_sql_migrations_with_go_web_apps/" style="line-height: 1.6em;">для миграций</a><span style="line-height: 1.6em;">?</span></p>

<p><span style="line-height: 1.6em;">Есть кое-какие </span><a href="https://docs.google.com/forms/d/1_yqks892m3e5zK1d-6iziRIt_1LdZvrvDOysjvVRpOU/viewanalytics">результаты</a><span style="line-height: 1.6em;"> опроса о самой крутой ORM. Для себя выбрал </span><a href="https://github.com/jinzhu/gorm" style="line-height: 1.6em;">gorm</a><span style="line-height: 1.6em;">.</span></p>

<p>Немного троллинга: делаем <a href="http://play.golang.org/p/DDab4dhju5">try...catch</a> в Golang и пишем на Go <a href="https://igo.herokuapp.com/">аки на coffeescript</a>.</p>

<h3 id="_1">Проекты</h3>

<ul>
	<li><a href="https://github.com/apexskier/httpauth">httpauth</a> - Авторизация пользователей.</li>
	<li><a href="https://github.com/skybox/skybox">skybox</a> - Инструмент для анализа поведений пользователя.</li>
	<li><a href="https://github.com/tcolar/authorize">authorize</a> - Пакет для работы с authorize API.</li>
	<li><a href="https://github.com/slyrz/mango">mango</a> - Генератор хелп страниц для консольных утилит.</li>
	<li><a href="https://github.com/franela/goblin">goblin</a> - Милый минималистический фреймворк для BDD тестирования.</li>
	<li><a href="https://github.com/nfnt/resize">resize</a> - Отличная либа для ресайза картинок с помощью Go.</li>
	<li><a href="https://github.com/ant0ine/go-json-rest">go-json-rest</a> - Go-Json-Rest инструмент для просттого и быстрого создания REST API.</li>
	<li><a href="https://github.com/gigablah/dashing-go">dashing-go</a> - Аналог <a href="http://shopify.github.io/dashing/">shopify/dashing</a> только на Go</li>
	<li><a href="https://github.com/franela/go-supertest">go-supertest</a> - Тулза для BDD тестирования. Упрощая работу с HTTP.</li>
	<li><a href="https://github.com/mitchellh/go-mruby">go-mruby</a> - Биндинг Go к руби. И наоборот.</li>
	<li><a href="https://github.com/alsm/hrotti">hrotti</a> - MQTT брокер, написанный на Go.</li>
	<li><a href="https://github.com/leeview/godsl">godsl</a> - Попытка добавить дженерики в Go.</li>
</ul>

<h3 id="_2">Новости</h3>

<ul>
	<li><a href="http://blog.labix.org/2014/03/13/go-qml-contest">Конкурс для Go QML</a> разработчиков от Gustavo Niemeyer. Все серьезно, призы реальные. Продлится до 21 апреля.</li>
	<li><a href="http://brainwashing.pro/go">Интенсив по Go</a> в Москва. Будет проходить 17-18 мая в штаб-квартире злых марсиан.</li>
	<li><a href="https://codereview.appspot.com/74250043">Пофикшен магический баг</a> связанный с сборщиком мусора.</li>
	<li>MongoDB зарелизили <a href="http://blog.mms.mongodb.com/post/79274617846/march-11-release-notes-expanded-user-roles-and-a-go">монитор агент</a> для MongoDB Monitoring Service.</li>
	<li><a href="https://github.com/kavu/go_reuseport">go_reuseport</a> - Для тех, кому надоели ошибки &quot;Address already in use&quot;. От <a href="https://twitter.com/HornedKavu">@HornedKavu</a>.</li>
	<li><a href="https://github.com/pote/gpm/releases/tag/v1.1.1">Новый релиз</a> моего любимого менеджера пакетов - gpm v1.1.1. Теперь есть плагины!</li>
	<li>17 марта прошла <a href="http://blog.anynines.com/gophers-meet-in-berlin/">тусовка гоферов</a> в Берлине.</li>
	<li>Вышла новая версия <a href="http://sourceforge.net/projects/liteide/files/">liteide</a>. William Kennedy <a href="http://www.goinggo.net/2013/06/installing-go-gocode-gdb-and-liteide.html">обновил инструкцию</a> по настройке девокружения.</li>
	<li><a href="https://groups.google.com/forum/#!msg/golang-nuts/_rbVuzl-OqA/N_xoNaD4kAoJ">godoc.org переходит к Go projects</a>. Gary Burd решил отдохнуть от его поддержки. С самим сервисом все ок.</li>
</ul>

<h3 id="_3">Статьи и статейки</h3>

<ul>
	<li><a href="http://albertptobey.blogspot.com/2014/03/open-source-video-production.html">Go и Работа с видео</a> - Linux, консоль, трувей.</li>
	<li>Пишем веб сервис на Go. <a href="http://habrahabr.ru/post/208680/">Часть первая</a> и <a href="http://habrahabr.ru/post/214425/">часть вторая</a>. Офигенский здоровенский туториал на habrahabr.</li>
	<li><a href="https://blog.mozilla.org/services/2014/03/12/sane-concurrency-with-go/">Sane Concurrency with Go</a>. Замечательная статья в блоге мозилы.</li>
	<li><a href="http://blog.joshsoftware.com/2014/03/12/learn-to-build-and-deploy-simple-go-web-apps-part-one/">Учимся создавать и деплоить</a> простые веб приложения. Часть I</li>
	<li><a href="http://blog.joshsoftware.com/2014/03/17/learn-to-build-and-deploy-simple-go-web-apps-part-four/">Деплой веб приложений</a> на Go. Часть II</li>
	<li><a href="http://blog.natefinch.com/2014/03/go-tips-for-newbie-gophers.html">Хитрости</a> для начинающих разработчиков Go.</li>
	<li>Про <a href="http://www.goinggo.net/2014/03/exportedunexported-identifiers-in-go.html">экспортируемые и не экспортируемые идентефикаторы</a> в Go. Статья от небезызвестного William Kennedy.</li>
	<li><a href="http://burke.libbey.me/conserving-file-descriptors-in-go/">С толком используем файловые дескрипторы</a> в многопоточных алгоритмах.</li>
	<li><a href="http://copyninja.info/blog/workaround-gotypesystems.html">Работа с системой типов</a> golang ипользуя unsafe.</li>
	<li><a href="http://words.volant.is/articles/authentication-golang-web-applications/">Аутентификация</a> конечного пользователя в приложениях на Go.</li>
	<li><a href="http://dave.cheney.net/2014/03/17/pointers-in-go">Указатели в Go</a>. Особенности и отличии от указателей в C.</li>
	<li><a href="http://blog.gamingrobot.net/go/2014/03/16/floating-point-hell/">Floating Point Hell</a>. Про декод/енкод больших чисел в Go и JavaScript.</li>
	<li><a href="http://technosophos.com/2014/03/19/generating-stack-traces-in-go.html">Генерация стектрейтов в Go</a>. Этого, порой, очень не хватает.</li>
	<li>Про <a href="http://www.tech-foo.net/the-problems-with-errors.html">обработку ошибок</a> в Go и Python. Достаточно холиварная тема.</li>
	<li><a href="http://blog.labix.org/2014/03/21/arbitrary-qt-extensions-with-go-qml">Пишем Qt расширение</a> на Go QML. Статья от Gustavo Niemeyer.</li>
	<li><a href="http://blog.campoy.cat/2014/03/github-and-go-forking-pull-requests-and.html">GitHub и Go</a>. Статья про работу с Go проектами на гитхабе и небольшое <a href="file:///home/artem/Projects/articles/4gophers.com/Небольшое%20дополнение%20к%20недавней%20статье]">дополнение</a> к этой статье</li>
	<li>Обновляемые <a href="http://nyeggen.com/blog/2014/03/19/upgradable-locks-in-go/">блокировки</a> в golang.</li>
	<li><a href="http://dave.cheney.net/2014/03/19/channel-axioms">Отличная памятка</a> по работе с каналами в golang.</li>
	<li><a href="http://clipperhouse.com/2014/03/21/what-stack-overflow-and-go-have-in-common/">Что общего между</a> Go и Stack Overflow?</li>
	<li>Использование <a href="https://medium.com/p/528af8ee1a58">Go в продакшене</a> без проблем.</li>
	<li>Гугловский язык программирования Go <a href="http://readwrite.com/2014/03/21/google-go-golang-programming-language-cloud-development">как буря для развивающихся облачных платформ</a>.</li>
</ul>

<h3 id="_4">Видео</h3>

<ul>
	<li>Замечательное, полуторачасовое видео &quot;<a href="http://vimeo.com/88887786">Flow, Idiomatic Go Bindings, and Layers with GolangDC</a>&quot;.</li>
	<li><a href="https://skillsmatter.com/skillscasts/5110-building-a-soa-network-of-daemons-with-go-ruby-and-zmq">Построение SOA сети</a> с Go, Ruby и ZMQ. Рассказывает Ismael Celis - разработчик из New Bamboo</li>
	<li><a href="http://4gophers.com/video/vvedenie-v-go">Введение в Go</a> - Mark Smith рассказывает о Go на конференции linux.conf.au 2014. А тут <a href="http://www.slideshare.net/dreamwidth/lca2014-introduction-to-go">еще слайды есть</a>.</li>
	<li><a href="http://4gophers.com/video/docker-v-google-compute-engine">Docker в Google Compute Engine</a> - Видео от Marc Cohen</li>
	<li><a href="http://4gophers.com/video/vebmorda-dlya-gdb">Вебморда для gdb</a> - Небольшое видео про начало работы с <a href="https://github.com/sirnewton01/godbg">godbg</a>. Это такая вебморда для gdb. Автор Chris McGee.</li>
	<li><a href="http://4gophers.com/video/stream-multiplexing-v-go">Stream Multiplexing в Go</a> - Видео от Alan Shreve.</li>
	<li><a href="http://4gophers.com/video/portirovanie-go">Портирование Go приложений</a> - Доклад &quot;Портирование Go на новые системы&quot; от Aram Hăvărnanu. Основная мысль в том, что портировать довольно просто, но есть несколько нюансов. <a href="https://bitbucket.org/4ad/gofosdem2014/overview">Исходники презентации на битбакете</a>.</li>
	<li><a href="http://4gophers.com/video/upravlenie-zavisimostyami">Управление зависимостями</a> - Еще одно видео с hakkalabs. Больше информации по <a href="http://www.hakkalabs.co/articles/go-dependency-by-keith-rarick">ссылке</a>.</li>
</ul>

<h3 id="_5">Всякое</h3>

<ul>
	<li><a href="http://go-talks.appspot.com/github.com/dlsniper/fosdem-golang-idea/idea-golang2.slide#1">Презентация от Florin Patan</a>. IDE для Go это безумие(или нет).</li>
	<li><a href="http://engineering.oysterbooks.com/post/79458380259/resizing-images-on-the-fly-with-go">Ресайз</a> картинок на лету</li>
	<li>Небольшой <a href="http://mattjibson.com/blog/2014/03/13/goread-one-year-with-money-and-app-engine/">финансовый отчет</a> проекта <a href="https://www.goread.io/">www.goread.io</a>. Кто не знает, это клон гуглридера на Go с <a href="https://github.com/mjibson/goread">открытыми исходниками</a>.</li>
	<li>Хорошее, объяснение <a href="http://groups.google.com/forum/#!msg/golang-nuts/Ayx-BMNdMFo/4rL8FFHr8v4J">почему в Go нет</a> удобной функции path.Exists(path string) bool</li>
	<li>Удобное расширение для chrome <a href="https://chrome.google.com/webstore/detail/go2doc/mnpdpppgidppdhingkmlcmmgdjknecif">Go2Doc</a>, которое дает ссылку на godoc.</li>
	<li>Реализация MapReduce на golang: <a href="http://nakul02.blogspot.in/2013/12/gomapreduce-mapreduce-implementation-in.html">GoMapReduce</a>.</li>
	<li>Почему стоит <a href="http://www.cbinsights.com/team-blog/why-we-chose-golang/">выбрать Go</a>.</li>
	<li>Презентация из солнечной Индии от Satish Talim: &quot;<a href="http://go-talks.appspot.com/github.com/SatishTalim/slides/whygo.slide#1">Почему Go?</a>&quot;</li>
	<li><a href="http://redmonk.com/dberkholz/2014/03/18/go-the-emerging-language-of-cloud-infrastructure/">Популярность Go растет</a>. Особенно, в области разработки облачных технологий.</li>
	<li><a href="https://gist.github.com/esimov/9622710">Вычисляем факториал</a> классически и через замыкание.</li>
	<li>Презентация &quot;<a href="http://go-talks.appspot.com/github.com/motain/why-golang/why-golang2.slide#1">Golang at THE Football App</a>&quot; от Dirk Pahl.</li>
</ul>

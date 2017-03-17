+++
date = "2015-05-17T19:36:07+03:00"
draft = false
title = "Тулзовины и хреновины 22"

+++

<p>Поздравляю всех с прошедшими праздниками. Потихоньку отходим от празднований и возвращаемся в рабочий ритм.</p>

<p>Очень значимых событий в мире Go за это время не происходило, но у нас всегда есть чего почитать и что попробовать в деле.</p>

<p>Не забывайте, что вы можете <a href="http://4gophers.com/shop">поддержать проект и получить наклейки с милыми гоферами</a>.</p>

<h3>Новости</h3>

<ul>
<li>Красивая <a href="https://medium.com/matryer/stopping-goroutines-golang-1bf28799c1cb">остановка go-рутин и использование пакета runner</a>.</li>
<li>Тут <a href="http://www.arduino.cc/en/Careers/BackendDeveloper">ищут бекенд разработчика со знанием Go</a> в Arduino LLC.</li>
<li><a href="https://about.gitlab.com/2015/05/03/unofficial-runner-becomes-official/">GitLab runner переписан на полностью на Go</a>. Ждем когда перепишут весь GitLab.</li>
<li><a href="https://blog.pusher.com/pusher-golang-library/">Анонсирована официальная Golang либа для Pusher</a>. Радуйтесь.</li>
<li><a href="http://blog.docker.com/2015/04/docker-networking-takes-a-step-in-the-right-direction-2/">Статья в блоге Docker про работу с сетью</a> и как все будет хорошо.</li>
<li>В свежем релизе Tor браузера 4.5 <a href="https://blog.torproject.org/blog/tor-browser-45-released">транспорты obfs2, obfs3 и ScrambleSuit переписаны на Go</a>.</li>
<li><a href="http://coderfactory.co/posts/top-sites-built-with-go">Топ 15 веб-проектов, которые используют Go</a>. Есть довольно значимые игроки.</li>
<li><a href="https://chimeracoder.github.io/docker-without-docker/#1">Пользуемся докером без докера</a>. Но с сsystemd. Интересная презентация про то, как все устроено.</li>
<li>На <a href="http://devcraft.tv/">devcraft.tv</a> появилось несколько скринкастов по Go. Ждем, что из этого получится.</li>
</ul>

<h3>Проекты</h3>

<ul>
<li><a href="http://sse.getgin.io/room/hn">Реализация Server-Sent Events</a> на Go и фреймворке Gin.</li>
<li><a href="https://github.com/dadamssg/starterapp">starterapp</a> - Шаблон для быстрого старта веб-проекта.</li>
<li><a href="https://github.com/Tapjoy/dynamiq">Dynamiq</a> - Простая реализация очереди на базе Riak 2.0.</li>
<li><a href="https://github.com/beefsack/git-mirror">git-mirror</a> - Тулза на Go для простой реализации зеркал для гитхабовских реп.</li>
<li><a href="http://go-bootstrap.io/">go-bootstrap.io</a> - Шаблон для быстрого старта Go веб-приложения.</li>
<li><a href="https://github.com/adampresley/cordovareloader">cordovareloader</a> - Небольшое приложение, которое следит за изменениями файлов в указанной директории.</li>
<li><a href="https://github.com/alexjlockwood/gcm">gcm</a> - Google Cloud Messaging для приложений с серверами на Go.</li>
<li><a href="https://github.com/dutchcoders/xmlgen">XMLGen</a> - Тулза для генерации нативных Go типов по XML.</li>
<li><a href="https://github.com/arussellsaw/telemetry">telemetry</a> - Еще один способ посчитать метрики вашего Go приложения.</li>
<li><a href="https://github.com/jingweno/ccat">ccat</a> - Аналог программы cat, только с подсветкой синтаксиса.</li>
<li><a href="https://github.com/EverythingMe/go-disque">go-disque</a> - Простой Go клиент для раоты с <a href="https://github.com/antirez/disque">Disque</a> (распределенная in-memory очередь).</li>
<li><a href="https://github.com/slowmail-io/popart">popart</a> - Библиотека для реализации POP3 сервера на Go.</li>
<li><a href="https://github.com/bradfitz/http2/tree/master/h2i">h2i</a> - Консоль для отладки HTTP/2 протокола.</li>
<li><a href="https://github.com/stianeikeland/go-rpio">go-rpio</a> - Нативная либа для GPIO-pins на Raspberry Pi.</li>
<li><a href="http://donnachess.github.io/">Donna Chess Engine</a> - Свободный шахматный движок.</li>
<li><a href="https://github.com/dineshappavoo/basex">basex</a> - Пакет, реализующий basex шифрование. Можете получать ID как у ютуб видео роликов.</li>
<li><a href="https://github.com/divan/expvarmon">expvarmon</a> - Монитор для expvars на базе TermUI.</li>
<li><a href="https://github.com/coocood/freecache">freecache</a> - Пакет, реализующий кеширование без лишнего оверхеда GC.</li>
<li><a href="https://github.com/LyricalSecurity/gigo">gigo</a> - Еще один менеджер зависимостей для Go, только теперь в стиле питоновского pip.</li>
<li><a href="https://www.vaultproject.io/">vault</a> - Приложение для хранения секретов. Зашифрованное хранилище.</li>
<li><a href="https://github.com/codingsince1985/couchcache">couchcache</a> - Cервис для кеширования. REST API в комплекте. Построен на базе Couchbase.</li>
<li><a href="https://github.com/muja/go-exit">go-exit</a> - Пакет, с помощью которого можно модифицировать поведение при завершении программы.</li>
<li><a href="https://github.com/wyc/utwil">utwil</a> - Набор утилит для работы с сервисом Twilio.</li>
<li><a href="https://github.com/bfontaine/vanish">vanish</a> - Пакет для супер простой работы с временными папками и файлами.</li>
<li><a href="https://github.com/jonhoo/cucache">cucache</a> - Клон мемкешид, только на Go и круче.</li>
<li><a href="https://caddyserver.com/">caddyserver</a> - Веб сервер для людей с поддержкой HTTP/2, IPv6, Markdown, WebSockets, FastCGI, шаблонов и еще много разных страшных слов.</li>
<li><a href="https://github.com/dullgiulio/pingo">pingo</a> - Пакет, который дает возможность расширять приложение с помощью плагинов. Работают через RPC.</li>
<li><a href="https://github.com/dvyukov/go-fuzz">go-fuzz</a> - Тулза для тестирования Go пакетов с помощью <a href="http://en.wikipedia.org/wiki/Fuzz_testing">фаззинга</a>.</li>
<li><a href="https://kabukky.github.io/journey/">journey</a> - Блоговый движок на Go, совместимый с Ghost темами.</li>
<li><a href="https://github.com/tejo/boxed">boxed</a> - Движок для блога, который можно хостить в дропбоксе. На Go, конечно.</li>
<li><a href="https://github.com/0intro/pcap">pcap</a> - Пакет для работы с pcap файлами на языке программирования Go.</li>
<li><a href="https://github.com/conejoninja/goblocktrail">GoBlocktrail</a> - Предоставляющий доступ к Blocktrail API. Это такой продвинутый сервис для работы с биткоинами.</li>
<li><a href="https://github.com/shiguredo/fuji">fuji</a> - A MQTT gateway is a sensor-MQTT gateway which receives data from sensors and sends that data to a MQTT broker.</li>
<li><a href="https://github.com/glycerine/offheap">offheap</a> - an off-heap hash-table in Go. Used to be called go-offheap-hashtable, but we shortened it.</li>
<li><a href="https://elithrar.github.io/article/simple-scrypt/">simple-scrypt</a> - Удобный врапер вокруг существующей Go либы scrypt.</li>
<li><a href="https://github.com/hundt/git-time-travel">git-time-travel</a> - Расширение для Git, которое позволяет делать ссылки на будущие коммиты.</li>
<li><a href="https://github.com/vrischmann/envconfig">envconfig</a> - Пакет для конфигурации приложения с помощью переменных окружения.</li>
<li><a href="https://github.com/lxc/lxd">lxd</a> - REST API, консольная тулза и OpenStack плагин для интеграции LXC.</li>
<li><a href="http://patrol.name/">Patrol</a> - Реализация sentry сервера на языке Go.</li>
<li><a href="https://github.com/goware/httpmock">httpmock</a> - Инструмент для быстрого создания заглушек сервисов при тестировании Go кода.</li>
</ul>

<h3>Статьи</h3>

<ul>
<li><a href="https://deferpanic.com/blog/manual-memory-management-in-go/">Ручное управление памятью в Go</a>. Невероятное рядом.</li>
<li><a href="https://medium.com/davidbyttow/scaling-secret-real-time-chat-d8589f8f0c9b">Реализация чата на базе Secret</a> с помощью языка Go.</li>
<li><a href="https://gist.github.com/twotwotwo/2eb69d8b30ac8e08d37a">Улучшаем стандартную Go сортировку в 5 раз</a>. И как это вообще возможно.</li>
<li><a href="http://adampresley.com/2015/05/12/writing-a-lexer-and-parser-in-go-part-2.html">Пишем лексер и парсер на Go</a>. Часть 2.</li>
<li>Хабрастатья "<a href="http://habrahabr.ru/company/ivi/blog/257431/">Реализуем ещё более безопасный VPN-протокол</a>".</li>
<li>"<a href="http://eax.me/go-databases/">Go и работа с базами данных, в частности с PostgreSQL</a>". Новая статья в блоге <a href="https://twitter.com/afiskon">@afiskon</a>.</li>
<li>Новая статья в официальном блоге Go "<a href="http://blog.golang.org/examples">Testable Examples in Go</a>".</li>
<li>Отличная статья от cloudflare "<a href="https://blog.cloudflare.com/go-crypto-bridging-the-performance-gap/">Go crypto: bridging the performance gap</a>"</li>
<li><a href="http://mwholt.blogspot.nl/2015/05/handling-errors-in-http-handlers-in-go.html">Работа с ошибками в HTTP хендлерах</a>. Подходы и практики.</li>
<li>Памятка по <a href="http://davidjpeacock.ca/2015/05/05/cross-compiling-go/">кросс-компилированию Go программ</a>.</li>
<li><a href="http://openmymind.net/RESTful-routing-in-Go/">RESTful роутинг</a> на языке программирования Go.</li>
<li>Пишем <a href="http://laicos.com/writing-handsome-golang-middleware/">красивый и юзабельный миделваре на Go</a>.</li>
<li>Сборка Go проектов с помощью <a href="https://walledcity.com/supermighty/building-go-projects-with-gb">новой модной, стильной, молодежной тулзы gb</a>.</li>
<li>"<a href="https://medium.com/matryer/retrying-in-golang-quicktip-f688d00e650a">Retrying in Go</a>" Небольшая хитрость для повторения некоторых действий(например, в случае нетсплита)</li>
<li><a href="https://medium.com/siddontang/use-hashicorp-raft-to-build-a-redis-sentinel-f3aa2e84c91e">Использование алгоритма "Hashicorp Raft</a>" для построения"Redis sentinel".</li>
<li><a href="https://medium.com/matryer/sync-once-with-reset-in-golang-quicktip-6ac44b015256">Использование sync.Once</a> совместно с Reset()</li>
<li><a href="https://gist.github.com/jonhoo/05774c1e47dbe4d57169">Сравниваем распределенный RWMutex</a> с классическим.</li>
<li><a href="https://medium.com/matryer/5-simple-tips-and-tricks-for-writing-unit-tests-in-golang-619653f90742">Пять простых секретов и приемов</a> для написания юнит тестов для Go приложений.</li>
<li><a href="https://medium.com/sent0hil/consistent-hashing-a-guide-go-implementation-fe3421ac3e8f">Руководство по консистентному хешированию</a> в Go и библиотекам для его использования.</li>
<li><a href="http://www.polyglotweekly.com/2015/04/24/thoughts-of-a-rustacean-learning-go.html">Мнение о Go одного из контрибьютеров Rust</a>. Что понравилось, а что нет.</li>
<li><a href="http://fastah.blackbuck.mobi/blog/securing-https-in-go/">Небольшая памятка по настройке TLS</a> в приложениях на Go.</li>
<li>Статья про <a href="http://www.philipotoole.com/increasing-bleve-performance-sharding/">использование движка для полнотекстового поиска bleve</a>.</li>
<li>Видео и презентация про правильное <a href="http://engineering.clever.com/2015/04/17/using-gos-interfaces-at-clever---more-than-just-easy-collaboration/">использование интерфейсов в Go</a>.</li>
<li>Учимся писать <a href="http://schier.co/blog/2015/04/26/a-simple-web-scraper-in-go.html">простой веб-скрейпер на языке Go</a>. Примеры прилагаются.</li>
<li>Делаем <a href="https://medium.com/matryer/the-http-handlerfunc-wrapper-technique-in-golang-c60bf76e6124">кастомные обертки над http.HandlerFunc</a> и какой в этом профит.</li>
<li><a href="http://madebymany.com/blog/replacing-rails-part-1-lets-go">Замена для Rails. Часть 1</a>. Хипстеры бегут на Go.</li>
<li>Хабрастатья "<a href="http://habrahabr.ru/company/ivi/blog/256365/">Реализуем безопасный VPN-протокол</a>". Все что вы хотели знать о GoVPN.</li>
<li><a href="http://www.philipotoole.com/replicating-sqlite-using-raft-consensus/">Репликация SQLite с помощью Raft</a> алгоритма. И, как результат, проект <a href="https://github.com/otoolep/rqlite">rqlite</a>.</li>
<li><a href="http://insights.ubuntu.com/2015/04/22/package-management-at-scale-with-landscape/">Управление пакетами с помощью Landscape</a> и как с этим связаны микросервисы на Go.</li>
<li><a href="http://txt.fliglio.com/2015/04/di-duck-typing-and-clean-code-in-go/">Внедрение зависимостей</a>, утиная типизация и чистый Go код.</li>
<li><a href="http://pravj.github.io/blog/development-story-of-puzzl/">Лайв-стори о создании мультиплеерной игры puzzl</a> на языке программирования Go.</li>
<li><a href="http://openmymind.net/Golangs-Error-Handling-Good-And-Bad/">Обработка ошибок в Go</a>. Хорошие и плохие стороны.</li>
<li>Пишем <a href="http://blog.codeship.com/using-object-oriented-web-servers-go/">web-серверы на Go объектно ориентированно</a>. И какие в этом есть плюсы.</li>
<li><a href="http://www.evanmiller.org/four-days-of-go.html">Go за 4 дня.</a> Автор делится ощущениями от работы с Go.</li>
<li>Простой <a href="https://medium.com/freeformz/hello-world-with-go-heroku-38295332f07b">пример создания Go приложения на Heroku</a>.</li>
</ul>

<h3>Видео</h3>

<ul>
<li>Не самое свежее, но все еще актуальное видео с DevConf 2014 от Дмитрия Вьюкова. <a href="http://4gophers.com/video/veb-prilozhenie-na-go#.VVizT1Ttmko">Пишем функциональное, надежное и быстрое веб-приложение на Go</a>.</li>
<li><a href="http://4gophers.com/video/vosproizvodimye-sborki">Воспроизводимые сборки</a>. Dave Cheney рассказывает про управление зависимостями и свой новый супер инструмент для этого.</li>
<li><a href="http://4gophers.com/video/kak-ispolzuyut-go-v-the-new-york-times">Как используют Go в "The New York Times"</a>. JP Robinson, сеньер разработчик из  "The New York Times", рассказывает как они используют Go последние два года. </li>
</ul>

<h3>Инструменты</h3>

<ul>
<li><a href="https://github.com/rapidloop/rtop">rtop</a> - Удаленный системный монитор. Можете подключаться к машине по SSH и следить за CPU, диском, памятью, сетью.</li>
<li><a href="https://github.com/maruel/panicparse">panicparse</a> - Парсер для стектрейсов, который превращает их в удобочитаемый лог.</li>
<li><a href="https://github.com/c9s/c6">c6</a> - Быстрый SASS компилятор на Go. Супер быстрый.</li>
<li>На <a href="https://sourcegraph.com/blog/117580140734">sourcegraph.com появился анонс Appdash</a>, инструмента для трассировки.</li>
<li><a href="https://github.com/dkulchenko/bunch">bunch</a> - Тулза для управления зависимостями в стиле npm, только для Go.</li>
<li><a href="https://github.com/constabulary/gb">gb</a> - Тулза для вендоринга зависимостей в вашем проекте и сборки его.</li>
<li><a href="https://github.com/namshi/godo">godo</a> - Простой и удобный инструмент для запуска команд сразу на нескольких машинах.</li>
</ul>

<h3>Всякое</h3>

<ul>
<li><a href="https://github.com/tamnd/spambot">spambot</a> - Программа на Go для генерации случайных предложений.</li>
<li><a href="https://github.com/dariubs/GoBooks">Подборка книжек по языку Go</a>. Тут и платные, и бесплатные. Кстати, у нас есть <a href="http://4gophers.com/books#.VUoVeyftlBc">своя подборка бесплатных книг</a>.</li>
<li>С помощью <a href="http://www.ajostrow.me/articles/put-go-wherever-you-like">симлинков обходим ограничения на структуру Go проекта</a>.</li>
<li><a href="https://docs.google.com/document/d/1y4mYe8Sk9jCPze6AyygxrC7j1sqEgwAycezpaE6JnC8/edit#heading=h.xgr9eiceu2ca">Интересный документ для разработчиков,</a> которые хотят написать еще одни инструмент для управления зависимостями в Go.</li>
<li><a href="http://go-talks.appspot.com/github.com/davecheney/presentations/reproducible-builds.slide#1">Презентация "Reproducible Builds"</a> от Dave Cheney. Поговорим о менеджменте зависимостей.</li>
<li><a href="https://buildkite.com/">buildkite.com</a> - Сервис для автоматизации вашего рабочего процесса.</li>
<li>Доклад от Francesc Campoy "<a href="https://medium.com/francesc/go-for-c-developers-video-slides-51b93c99429">Go для C++ разработчиков</a>".</li>
</ul>

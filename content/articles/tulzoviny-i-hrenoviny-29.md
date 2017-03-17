+++
date = "2015-11-30T15:53:03+03:00"
draft = false
title = "Тулзовины и хреновины 29"

+++

<p>Последний осенний выпуск подборки с новостями из мира Go. В этом месяце мы праздновали <a href="https://blog.golang.org/6years">день рождения</a> нашего любимого языка программирования и узнали много нового на прошедшей конференции dotGo 2015.</p>

<h3>Новости</h3>

<ul>
<li>Начали появляться <a href="http://www.thedotpost.com/conference/dotgo-2015">первые видео с dotGo 2015</a>. Ждем еще.</li>
<li><a href="http://www.levigross.com/2015/11/21/mutual-tls-authentication-in-go/">Взаимная TLS авторизация на Go</a>. Способы и инструменты.</li>
<li><a href="https://golangshow.com/episode/2015/11-26-031/">Golangshow Выпуск 031</a>. Веб-фреймворки, танцы, поговорки. И обязательный к <a href="https://golangshow.com/episode/2015/11-19-028/">прослушиванию выпуск 028</a> - интервью с Пайком и интеграционные тесты.</li>
<li><a href="https://blog.golang.org/6years">6 лет Go! Празднуем!</a> Скоро в школу пойдет.</li>
<li><a href="http://getgb.io/news/gb-version-0.3.2-released/">Релиз менеджера зависимостей gb версии 0.3.2</a>. Фиксы и улучшения.</li>
<li>Cообщество Go ширится и растет. И вот возникла необходимость в кодексе: "<a href="https://github.com/golang/proposal/blob/master/design/13073-code-of-conduct.md">Proposal: A Code of Conduct for the Go community</a>".</li>
<li>Самый последний <a href="http://golangweekly.com/issues/86">Go Newsletter</a> №86.</li>
</ul>

<h3>Проекты</h3>

<ul>
<li><a href="https://github.com/CoinStorage/gocoin">gocoin</a> - Либа для работы с bitcoin. Основана на hellobitcoin, но с пачкой новых фич.</li>
<li><a href="https://github.com/pressly/sup">Stack Up</a> - Простая тулза для деплоя приложений. Может работать сразу с несколькими хостами.</li>
<li><a href="https://github.com/therecipe/qt">qt</a> - Новый биндинг к Qt для Go. Работает под Windows, Mac OS X, Linux и Android.</li>
<li><a href="https://github.com/kkdai/trigram">trigram</a> - Реализация триграмного индекса(поиска с опечатками) на Go.</li>
<li><a href="https://github.com/ejholmes/cloudwatch">cloudwatch</a> - Либа которая позволяет работать с CloudWatch стримами логов как с io.Writers и io.Readers.</li>
<li><a href="https://github.com/chrisledet/rebasebot">rebasebot</a> - Бот который может ребазить ваш пулреквест в GitHub'е по запросу.</li>
<li><a href="https://github.com/remogatto/sms">sms</a> - Конкурентный Sega Master System эмулятор на Go.</li>
<li><a href="https://github.com/michaelmacinnis/oh">oh</a> - Еще один linux shell, только написанный на Go.</li>
<li><a href="https://github.com/devork/twkb">twkb</a> - Пакет для работы с TWKB(Tiny Well-known Binary) форматом.</li>
<li><a href="https://github.com/valyala/fasthttp">fasthttp</a> - Более быстрая реализация HTTP на Go. Уже протестирована в продакшене.</li>
<li><a href="https://github.com/lestrrat/go-jwx">go-jwx</a> - Реализация различных JWx технологий на Go.</li>
<li><a href="https://github.com/gosuri/uitable">uitable</a> - Простой пакет для отображения текста в консоли с красивыми отступами.</li>
<li><a href="https://github.com/ahmetalpbalkan/wagl">wagl</a> - DNS сервис-дискавери для Docker Swarm.</li>
<li><a href="https://github.com/nanopack/yoke">yoke</a> - Высокодоступный кластер Postgres c авто-феловером и автоматическим восстановлением.</li>
<li><a href="https://github.com/joyent/containerbuddy">containerbuddy</a> - Сервис облегчающий конфигурацию и управление приложениями работающими в контейнерах.</li>
<li><a href="https://github.com/nanopack/mist">mist</a> - Распределенный pub-sub сервис с возможностью добавления тегов к сообщению.</li>
<li><a href="https://github.com/wawandco/fako">fako</a> - Пакет для создания фейковых структур. Удобно использовать для тестов.</li>
<li><a href="https://github.com/nebulouslabs/sia">sia</a> - Основанное на blockchain технологиях облачное хранилище данных.</li>
<li><a href="https://github.com/johnbeil/BuoyBot">BuoyBot</a> - Пример написания бота для твиттера. В частности, бота для NBDC Station 46026 (the San Francisco Buoy).</li>
<li><a href="https://github.com/holys/initials-avatar">initials-avatar</a> - Сервис для генерации аватарок по инициалам пользователя.</li>
<li>"<a href="https://github.com/brocaar/l3dsr-hash-balancer">Layer-3 Direct Server Return</a> " - Прототип балансировщика нагрузки с преферансом и танцовщицами.</li>
<li><a href="https://github.com/jcuga/golongpoll">golongpoll</a> - Либа для HTTP лонгпуллинга. Упрощает реализацию pub-sub приложений.</li>
<li><a href="https://github.com/minio/minio-go">minio-go</a> - Пакет для работы с Amazon S3 совместимым хранилищем minio.</li>
<li><a href="https://github.com/graphql-go/graphql">graphql</a> - Реализация GraphQL на языке программирования Go.</li>
<li>Очень простая <a href="https://github.com/Repo2/y">ORM под названием y</a>. Даже название простое.</li>
<li><a href="https://github.com/ChrisTrenkamp/goxpath">goxpath</a> - Реализация XPath на языке программирования Go.</li>
<li><a href="https://github.com/mvader/go-itergen">go-itergen</a> - Тулза для генерации кода работающего с итерируемыми типами.</li>
<li><a href="https://github.com/tucnak/climax">climax</a> - Тулза для быстрого построения консольных приложений на Go.</li>
<li><a href="https://github.com/42wim/matterircd">matterircd</a> - Минималистический IRC сервер, который позволяет интегрироваться с mattermost.</li>
<li><a href="https://github.com/gernest/hero">hero</a> - Пакет для реализации OAuth авторизации в своем приложении.</li>
<li><a href="https://github.com/Francesco149/go-hachi">go-hachi</a> - эмулятор CHIP-8, написанный на Go.</li>
<li><a href="https://github.com/wicast/xj2s">xj2s</a> - Простая утилита для генерирования Go структур из xml и json файлов.</li>
<li><a href="https://github.com/adnanh/webscrot">webscrot</a> - Инструмент для снятия скриншотов с виртуальных экранов. Удобно использовать для тестирования в нескольких браузерах.</li>
<li><a href="https://github.com/jcuga/proxyblock">proxyblock</a> - Инструмент для блокирования нежелательного контента. </li>
<li><a href="https://github.com/KSubedi/gomove">gomove</a> - Инструмент для перемещения пакета из по разным путям.</li>
<li><a href="https://github.com/WnP/go-sfmt">go-sfmt</a> - Набор методов для форматирования строк.</li>
<li><a href="https://github.com/gchaincl/chanson">chanson</a> - Пакет для реализации JSON стриминга.</li>
<li><a href="https://github.com/colegion/goal">goal</a> - Набор инструментов для продуктивной работы на Go проектами.</li>
</ul>

<h3>Статьи</h3>

<ul>
<li>Перевод статьи <a href="http://4gophers.ru/article/vstraivanie-resursov-v-prilozhenie#.Vlw30CftlBc">"Embedding Assets in Go"</a>.</li>
<li><a href="http://wysocki.in/golang-concurrency-data-races/">Разбираемся с "гонками"</a> в конкурентном Go приложении.</li>
<li>Несколько советов, которые <a href="http://www.cockroachlabs.com/blog/how-to-optimize-garbage-collection-in-go/">помогут вам оптимизировать сборку мусора</a> в вашем приложении.</li>
<li><a href="http://dave.cheney.net/2015/11/29/a-whirlwind-tour-of-gos-runtime-environment-variables">Увлекательнейший тур по переменным окружения</a>, которые используются в Go.</li>
<li><a href="http://bit.ly/1NfGRU8">Несколько вещей</a>, которые вы должны сделать до того как заопенсорсите свой проект.</li>
<li><a href="http://bit.ly/1HpDYQv">Танцы с мьютексами</a>. Статья о превратностях многопоточного программирования.</li>
<li><a href="http://www.codepool.biz/swig-link-windows-dll-golang.html">Используем SWIG</a> для линковки виндовой DLL к Go приложению.</li>
<li><a href="http://maciekmm.net/html-golang-stream-processing/">Парсим HTML с помощью Go</a>, используя обработку стримов.</li>
<li>Статья от Dave Cheney: "<a href="http://dave.cheney.net/2013/01/09/go-the-language-for-emulators">Go, the language for emulators</a> "</li>
<li>Хабрастатья "<a href="http://habrahabr.ru/post/271239/">Тестирование веб-сервиса на Go</a>".</li>
<li>Отличная статья про проект spectro и язык программирования Go: "<a href="http://markcrossfield.co.uk/2015-08-22-spectro-adventures-in-go.html">Spectro... adventures in go (lang)</a>".</li>
<li>Есть мнение, что <a href="https://davidnix.io/post/error-handling-in-go/">обработка ошибок в стиле Go</a> весьма элегантна.</li>
<li>Хабрастатья "<a href="http://habrahabr.ru/post/271303/">Интервью: Брайан Керниган и Алан Донован</a>".</li>
<li><a href="http://blog.codeship.com/utilizing-simplicity-go-easy-development/">Подходы в Go, которые упрощают разработку</a> программного обеспечения.</li>
<li>Запускаем <a href="http://www.viaspringboard.com/blog/2015/11/18/running-go-on-via-arm-devices/">Go программы на VIA ARM девайсах</a>.</li>
<li>Хабрастатья "<a href="http://habrahabr.ru/post/271157/">Чистая архитектура в Go-приложении. Часть 3</a>".</li>
<li><a href="http://blog.ralch.com/tutorial/golang-object-relation-mapping-with-gorm/">Учимся мапить объекты с использованием gorm</a>. ORM в массы! </li>
<li>Очень-очень <a href="http://bit.ly/1MSiQ5w">вводная информация о конкурентности</a> в Go для начинающих.</li>
<li>Еще раз о Go и микросервисах: "<a href="http://peter.bourgon.org/a-case-for-microservices/">A case for microservices</a>".</li>
<li><a href="https://dev.acquia.com/blog/open-sourcing-statsgod-a-statsd-implementation-in-go/16/11/2015/8171">Вводная статья по Statsgod</a>. Это такая реализация StatsD на Go.</li>
<li>Почему сейчас самое <a href="http://engineeredweb.com/blog/2015/go-packages-need-release-versions/">время для версионирования пакетов</a>.</li>
<li>Отличная статья "<a href="http://jmoiron.net/blog/for-better-or-for-worse">For Better or For Worse</a>". Мысли про дизайн Go и почему он такой какой есть.</li>
<li><a href="https://wehavefaces.net/learn-golang-graphql-relay-2-a56cbcc3e341">Изучаем Go, GraphQL и Relay</a>. Часть 2.</li>
<li><a href="http://willowtreeapps.com/blog/go-generate-your-database-code/">Использование gogenerate для генерации кода</a> работающего с базой данных.</li>
<li>Разрабатываем <a href="http://www.aychedee.com/2015/09/18/writing-a-real-time-bidder-with-go/">риалтаймовый биддер на Go</a>.</li>
<li>Хабрастатья "<a href="http://habrahabr.ru/post/270849/">Пример решения типичной ООП задачи на языке Go</a>".</li>
<li>Статья от Dave Cheney "<a href="http://dave.cheney.net/2015/11/15/the-legacy-of-go">The Legacy of Go</a>".</li>
<li><a href="https://blog.codeship.com/embedding-assets-in-go/">Встраиваем ассеты в ваше приложение на Go</a>. Различные способы и подходы.</li>
<li><a href="http://dev.tulu.la/post/dmitry-vyukov-part1/">Текстовая расшифровка интервью с Дмитрием Вьюковым</a>. Для тех кто не любит слушать.</li>
<li>Три <a href="http://bryce.is/writing/code/jekyll/update/2015/11/01/3-go-gotchas.html">типичные ошибки Go программистов</a>. Запоминаем и избегаем.</li>
<li>Используем свой собственный <a href="http://www.hydrogen18.com/blog/your-own-pki-tls-golang.html">PKI для TLS соединений в Go</a>.</li>
<li>Делаем <a href="https://joshrendek.com/2015/11/building-a-distributed-waitgroup-with-go-and-redis/">распределенный WaitGroup с помощью Go и Redis</a>.</li>
<li>Реализация сравнительно <a href="http://meros.io/blog/cheap-mapreduce-in-go/">дешевого MapReduce на Go</a>.</li>
<li>Хабрастатья "<a href="http://habrahabr.ru/post/270351/">Чистая архитектура в Go-приложении. Часть 2</a>".</li>
<li>Почему Go это <a href="http://www.cockroachlabs.com/blog/why-go-was-the-right-choice-for-cockroachdb/">правильный выбор для CockroachDB</a> </li>
<li>Настраиваем окружение для <a href="http://blog.codeship.com/implementing-a-bdd-workflow-in-go/">разработки Go приложений с BDD</a>.</li>
<li><a href="https://blog.klauspost.com/travisappveyor-ci-script-for-go/">Скрипт для сервисов Travis CI и AppVeyor</a>. Деплоим Go приложение по умному.</li>
<li><a href="http://spaces-vs-tabs.com/4-weeks-of-golang-the-good-the-bad-and-the-ugly/">Еще одна замечательная история</a> о использовании Go и что из этого вышло.</li>
<li>Статья от Dave Cheney: "<a href="http://dave.cheney.net/2015/11/05/lets-talk-about-logging">Поговорим о логировании в Go</a> "</li>
<li>Хабрастатья "<a href="http://habrahabr.ru/post/270081/">Где находиться типу: справа или слева?</a>".</li>
<li>Делаем гибкую и легко<a href="http://jamesadam.me/index.php/2015/11/03/making-an-extensible-wiki-system-with-go/">расширяемую вики-систему на Go</a>.</li>
</ul>

<h3>Видео</h3>

<ul>
<li><a href="http://4gophers.ru/video/analiz-izobrazhenii">Анализ изображений</a>. Пример использования Go для анализа изображения. Видео от Todd McLeod у которого есть еще много всякого интересного на его канале.</li>
<li><a href="http://4gophers.ru/video/the-legacy-of-go">The Legacy of Go</a>. Отличный доклад от Dave Cheney на GothamGo 2015.</li>
</ul>

<h3>Инструменты</h3>

<ul>
<li><a href="https://github.com/maruel/panicparse">panicparse</a> - Простая утилита, которая превращает вывод паники в более читаемый вид.</li>
<li><a href="https://github.com/miconda/wsctl">wsctl</a> - Консольная тулза для работы с веб-сокетами. Удобно использовать для отладки.</li>
<li><a href="https://github.com/mozilla/mig">mig</a> - Инструмент от Mozilla для одновременного инвестигирования и инспекции на удаленных хостах.</li>
<li>Плагин для разработки на <a href="https://github.com/Microsoft/vscode-go">Go под Visual Studio Code</a>.</li>
<li><a href="https://github.com/arduino/arduino-builder">arduino-builder</a> - Консольная тулза для компилирования arduino скетчей.</li>
<li><a href="https://github.com/shopify/toxiproxy">toxiproxy</a> - Прокси которая может эмулировать различные условия и проблемы в сети.</li>
<li><a href="https://github.com/alecthomas/gometalinter">gometalinter</a> - Инструмент для проверки вашего кода на куче линтеров одновременно. </li>
</ul>

<h3>Всякое</h3>

<ul>
<li>Жизненная история о <a href="http://justinfx.com/2015/11/08/switching-from-wordpress-to-hugo/">переходе с Wordpress на Hugo</a>. С хепи ендом.</li>
<li><a href="https://github.com/benjojo/upsetsysadmins">upsetsysadmins</a> - Тулза для переименовывания процессов и нервного срыва сисадминов.</li>
<li><a href="http://goa.design/">Набор инструментов для создания REST</a> сервисов на языке Go.</li>
<li>Разминка для мозгов от Dave Cheney: "<a href="http://dave.cheney.net/2015/11/18/wednesday-pop-quiz-spot-the-race">Wednesday pop quiz: spot the race</a>".</li>
<li><a href="https://github.com/dariubs/GoBooks">Репозиторий со списком всех книг о Go</a>. Некоторые есть в свободном доступе.</li>
<li>"<a href="http://www.goin5minutes.com/">Go In 5 Minutes</a> " это еженедельный скринкаст для Go программистов. Выпуски короткие, но весьма информативные.</li>
<li>Отличная презентация от Fatih Arslan <a href="https://speakerdeck.com/farslan/tools-for-working-with-go-code">про различные инструменты, которые упрощают работу с Go кодом</a>.</li>
<li>Ждем <a href="http://golangschool.com/">курсов по языку программирования Go</a> для гоферов. </li>
</ul>

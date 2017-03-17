+++
date = "2016-02-29T17:58:05+03:00"
draft = false
title = "Тулзовины и хреновины 32"

+++

<p>Февральский выпуск всего самого интересного из мира Go. В этом месяце зарелизился Go 1.6. Можете посмотреть <a href="https://talks.golang.org/2016/state-of-go.slide#1">презентацию от Francesc Campoy</a> о текущем положении Go и планах на будущее.</p>

<p>Еще у нас появились <a href="http://4gophers.ru/wall">обои с милым гофером</a>. Пользуйтесь на здоровье.</p>

<p>Не забывайте, что у нас есть <a href="https://vk.com/golang">VK группа</a>, <a href="https://twitter.com/4gophers">твиттер</a> и возможность <a href="http://4gophers.ru/subscribe">подписаться на рассылку</a> новых выпусков тулзовин.</p>

<h3>Новости</h3>

<ul>
<li><a href="https://golangshow.com/episode/2016/02-25-045/">GolangShow. Выпуск 045</a>. Производительность Go со всех сторон.</li>
<li><a href="http://golangweekly.com/issues/98">Свежий Go Newsletter номер 98</a> и <a href="http://golangweekly.com/issues/97">номер 97</a>. Много ссылок и новостей.</li>
<li><a href="https://blog.golang.org/go1.6">Релиз Go 1.6!</a> Праздник у всех гоферов мира!</li>
<li><a href="https://github.com/go-ozzo">go-ozzo</a> - Товарищ Qiang Xue(автор Yii) активно пилит проекты на Go. </li>
<li><a href="http://gophergala.com/blog/gopher/gala/2016/02/05/winners-2016/">Победители GopherGala 2016</a>. Ура и поздравления!</li>
<li><a href="https://talks.golang.org/2016/state-of-go.slide#1">Презентация от Francesc Campoy</a> о текущем положении Go и планах на будущее.</li>
<li><a href="http://4gophers.ru/wall">Новые нескучные обои с гофером!</a> Налетайте, украшайте свой стол.</li>
<li>Обсуждаем <a href="https://groups.google.com/forum/#!topic/golang-dev/TerfinvPffw">планы на Go 1.7</a>.</li>
</ul>

<h3>Проекты</h3>

<ul>
<li><a href="https://github.com/google/subcommands">subcommands</a> - Пакет для удобной организации логики в консольных приложениях.</li>
<li><a href="https://github.com/reddec/go-pack">go-pack</a> - Инструмент для сборки deb пакетов из Go бинарников.</li>
<li><a href="https://github.com/fogleman/gg">gg</a> - Пакет для 2D графики на чистом Go.</li>
<li><a href="https://github.com/edmund-huber/ergonomadic">ergonomadic</a> - IRC сервер полностью написанный на Go.</li>
<li><a href="https://github.com/jubalh/gontributions">gontributions</a> - Программа, которая следит за обновлениями в проектах.</li>
<li><a href="https://github.com/stabbycutyou/moldova">moldova</a> - Пакет для генерации фейковых данных по шаблону.</li>
<li><a href="https://github.com/lukasmartinelli/pgclimb">pgclimb</a> - Инструмент для экспортирования данных из PostgreSQL в различные форматы.</li>
<li><a href="https://github.com/cpuguy83/kvfs">kvfs</a> - Основанная на FUSE файловая система для K/V хранилища.</li>
<li><a href="https://github.com/jrozner/sonar">sonar</a> - Инструмент для поиска поддоменов. Проще чем Knock и DNSRecon.</li>
<li><a href="https://github.com/influxdata/telegraf">telegraf</a> - Утилита на Go, которая позволяет писать метрики в influxdb.</li>
<li><a href="https://github.com/alecthomas/kingpin">kingpin</a> - Еще один парсер для флагов в Go программах.</li>
<li><a href="https://github.com/goburrow/netforward">netforward</a> - Программа на Go, которая позволяет форвардить данные даже между разными протоколами.</li>
<li><a href="https://github.com/msoap/html2data">html2data</a> - Пакет для выгребания данных из HTML.</li>
<li><a href="https://github.com/djherbis/stow">stow</a> - Пакет для работы с персистентными данными в boltdb.</li>
<li><a href="https://github.com/dpw/vendetta">vendetta</a> - Менеджер зависимостей который использует сабмодули.</li>
<li><a href="https://github.com/puerkitobio/rehttp">rehttp</a> - Пакет реализует HTTP транспорт с возможностью переподключения.</li>
<li><a href="https://github.com/kardianos/govendor">govendor</a> - Минималистический инструмент для работы с зависимостями.</li>
<li><a href="https://github.com/chrismoos/hpack">hpack</a> - Пакет для компрессии HPACK(Header Compression HTTP/2).</li>
<li><a href="https://github.com/jpillora/overseer">overseer</a> - Инструмент для плавного рестарта и обновления вашего приложения.</li>
<li><a href="https://github.com/mercari/gaurun">gaurun</a> - Сервер для пуш-нотификаций. Поддерживает APNS и GCM.</li>
<li><a href="https://github.com/pagerduty/nut">nut</a> - Инструмент для собки LXC контейнеров. Использует Dockerfile.</li>
<li><a href="https://github.com/goadapp/goad">goad</a> - Инструмент для нагрузочного распределенного тестирования.</li>
<li><a href="https://github.com/maxzerbini/ovo">ovo</a> - K/V хранилище и распределенный кеш на Go,</li>
<li><a href="https://github.com/calavera/docker-volume-keywhiz">docker-volume-keywhiz</a> - Расширение для использования keywhiz-fs в Docker.</li>
<li><a href="https://github.com/atemerev/skynet">Skynet 1M</a> - Бенчмарк для тестирования асинхронщины на разных языках программирования.</li>
<li><a href="https://github.com/micro/go-platform">go-platform</a> - Платформа для реализации микросервисной архитектуры.</li>
<li><a href="https://github.com/gliderlabs/resolvable">resolvable</a> - DNS ресолвер для локальных Docker контейнеров.</li>
<li><a href="https://github.com/majestrate/libi2ptorrent">libi2ptorrent</a> - Библиотека для работы bittorrent на чистом Go.</li>
<li><a href="https://github.com/facebookgo/grace">grace</a> - Плавный рестарт для Go сервера. Реализация с примерами.</li>
<li><a href="https://github.com/mdp/sodiumbox">sodiumbox</a> - Библиотека на Go для коммуникации с libsodiums crypto_box_seal.</li>
<li><a href="https://github.com/fogleman/mol">mol</a> - Go приложение, которое рисует молекулы.</li>
<li><a href="https://github.com/esell/deb-simple">deb-simple</a> - Очень простой сервер для deb репозиториев.</li>
<li><a href="https://github.com/ekanite/ekanite">ekanite</a> - Syslog сервер с встроенным поиском. Про разработку можно почитать в цикле статей.</li>
<li><a href="https://github.com/martensson/nixy">nixy</a> - Nginx автоконфигуратор и сервис-дискавери для Mesos/Marathon.</li>
<li><a href="https://github.com/drone/sqlgen">sqlgen</a> - Пакет для генерации sql сканеров и других полезных вещей.</li>
<li><a href="https://github.com/cavaliercoder/go-rpm">go-rpm</a> - Пакет для работы с RPM файлами. Реализация на чистом Go.</li>
<li><a href="https://github.com/jen20/riviera">riviera</a> - Небольшое SDK для работы с Azure Resource Manager.</li>
<li><a href="https://github.com/tealeg/xlsx">xlsx</a> - Пакет для работы с XLSX файлами в Go программах.</li>
<li><a href="https://sendto.click/">sendto</a> - Тулза для отправки шифрованных файлов.</li>
<li><a href="https://github.com/bobziuchkovski/writ">writ</a> - Гибкий и многофункциональный парсер для консольных флагов.</li>
<li><a href="https://github.com/praetorian-inc/trudy">trudy</a> - Прокси с возможностью модификации и блокирования трафика.</li>
<li><a href="https://github.com/zeromq/gomq">gomq</a> - Реализация ZeroMQ Message Transport Protocol на Go.</li>
<li><a href="https://github.com/jtolds/lecat">lecat</a> - socat-lite с поддержкой letsencrypt.</li>
<li><a href="https://github.com/hashicorp/hil">hil</a> - HashiCorp Interpolation Language. Простой язык для использования в конфигах.</li>
<li><a href="https://github.com/skippbox/kompose">kompose</a> - Библиотеки compose с интегрированным k8s.</li>
<li><a href="https://github.com/ortuman/sirius">sirius</a> - Ультра легкая версия либа для серверов чатов на Go.</li>
<li><a href="https://github.com/cortesi/modd">modd</a> - Инструмент для запуска событий при изменении файлов.</li>
<li><a href="https://github.com/UnnoTed/fileb0x">fileb0x</a> - Простой и гибкий инструмент для встраивания файлов в Go приложение.</li>
<li><a href="https://github.com/tcnksm/gotests">gotests</a> - Пакет для генерации тестов по исходным функциям.</li>
<li><a href="https://github.com/MaxHalford/gago">gago</a> - Реализация параллельных генетических алгоритмов на Go.</li>
<li><a href="https://github.com/purpleidea/mgmt">mgmt</a> - Новое поколение инструментов для работы с конфигами приложения.</li>
<li><a href="https://github.com/miolini/bankgo">bankgo</a> - Пример двух простых приложений на Go, такой себе импровизированный банк.</li>
<li><a href="https://github.com/microsoft/go-winio">go-winio</a> - Утилиты для работы с вводом/выводом в Win32.</li>
<li><a href="https://github.com/wercker/wercker">wercker</a> - Консольная утилита для работы с сервисами wercker.com.</li>
<li><a href="https://github.com/eleme/banshee">banshee</a> - Детектор аномалий в периодических метриках.</li>
<li><a href="https://github.com/slapresta/dit">dit</a> - Пакет для рисования в терминале. Работает поверх termbox-go.</li>
<li><a href="https://github.com/majestrate/go-i2p">go-i2p</a> - Роутер и библиотеки на Go для I2P. Пока еще в очень глубокой альфе.</li>
<li><a href="https://github.com/google/seesaw">seesaw</a> - Виртуальный линуксовский сервер(LVS) для балансировки нагрузки.</li>
<li><a href="https://github.com/icholy/rip">rip</a> - Тулза аналогичная grep, но использует regex выражения.</li>
<li><a href="https://github.com/gophergala2016/festivus">festivus</a> - Приложение для слака которое позволяет проще планировать различные мероприятия.</li>
<li><a href="https://github.com/CodingBerg/benchgraph">benchgraph</a> - Еще один инструмент для визуализации результатов бенчмарков.</li>
<li><a href="https://github.com/nleof/goyesql">goyesql</a> - Пакет, который парсит SQL файл и позволяет использовать в своем приложении.</li>
</ul>

<h3>Статьи</h3>

<ul>
<li>Перевод статьи "<a href="http://4gophers.ru/article/standartnoe-kolco-basic-hash-ring">Basic Hash Ring</a>".</li>
<li>Готовим <a href="http://4gophers.ru/article/gotovim-deb-iz-nashih-binarnikov">deb из наших бинарников</a>. С плюшкой в комментариях.</li>
<li><a href="http://4gophers.ru/article/smid-optimizaciya-v-go">SMID оптимизация в Go</a>. Это ассемблер, детка!.</li>
<li>Хабраперевод "<a href="https://habrahabr.ru/post/277987/">Go с точки зрения PHP программиста</a>".</li>
<li>Книга по <a href="https://www.gitbook.com/book/jannewmarch/network-programming-with-go-golang-/details">сетевому программированию на Go</a>. Много примеров и хорошие описание.</li>
<li><a href="http://elliot.land/making-switch-cool-again">Делаем switch снова крутым</a>. Разбираемся с Go синтаксисом.</li>
<li><a href="http://bit.ly/1Qin20e">Диета для ваших бинарников</a>. Худеем правильно.</li>
<li><a href="http://sobit.me/2016/02/25/go-from-php-engineers-perspective/">Go глазами PHP разработчика</a>. Неожиданности и ожиданности.</li>
<li><a href="http://bit.ly/1XPNPCu">Консистентные билды</a>, или зачем пользоваться вендорингом.</li>
<li>Свежий выпуск <a href="http://golangweekly.com/issues/98">Go Newsletter номер 98</a>.</li>
<li>Несколько <a href="http://www.integralist.co.uk/posts/golang-rpc.html">простых примеров реализации RPC</a> на Go.</li>
<li>Учимся работать с <a href="https://rockfloat.com/post/learning-golang-templates.html">текстовыми и HTML'ными шаблонами в Go</a>.</li>
<li><a href="http://www.gmarik.info/blog/2016/go-testing-package-side-effects/">Почему нельзя импортировать пакет "testing"</a> вне файлов с тестами.</li>
<li>Пример использования <a href="http://nathanleclaire.com/blog/2016/02/23/using-golang-1.6-templates/">новых текстовых шаблонов в Go 1.6</a>.</li>
<li><a href="http://ewanvalentine.io/why-go-solves-so-many-problems-for-web-developers/">Почему Go решает</a> многие проблемы веб-разработчиков.</li>
<li>Хабрастатья "<a href="https://habrahabr.ru/post/277705/">Разработка библиотеки для IOS/Android на Golang</a>".</li>
<li><a href="http://bit.ly/1mOwX1x">Ежедневная оптимизация Go кода</a> с использованием профайлинга и бенчмарков.</li>
<li><a href="http://coussej.github.io/2016/02/16/Handling-JSONB-in-Go-Structs/">Работаем с постгресовским JSONB</a> в Go. Небольшой туториал.</li>
<li><a href="https://blog.heroku.com/archives/2016/2/19/microservices_in_go_using_go_kit">Статья от ребят из Heroku</a> про использование микросервисов и go-kit.</li>
<li><a href="http://gobot.io/documentation/platforms/pebble/">Работаем с умными часами Pebble</a> с помощью Go.</li>
<li><a href="https://habrahabr.ru/post/277137/">Хабрастять "Go sync.Pool"</a>. Вольный пересказ документации к sync.Pool.</li>
<li>На хабрахабре продолжение про Катю "<a href="https://habrahabr.ru/post/277099/">Катя, Go, Dcoin и Android</a>".</li>
<li>Хабрастатья "<a href="https://habrahabr.ru/post/276981/">Краш-курс по интерфейсам в Go</a>".</li>
<li>Пишем <a href="http://www.mustafaak.in/2016/02/08/writing-my-own-init-with-go.html">свой собственный init</a>. С шахматами и театром.</li>
<li><a href="https://getgophish.com/blog/post/database-migrations-in-go/">Используем goose</a> для работы с миграциями базы данных.</li>
<li><a href="http://blog.dbalan.in/blog/2016/01/14/golang-shorthand-operator-allows-accidental-shadowing-of-variable/">Затемнение и скрытая угроза</a>. Разбираемся с оператором := в Go.</li>
<li><a href="http://www.fodop.com/ar-1002">Библиотеки для машинного обучения</a> собранные по категориям.</li>
<li><a href="https://goroutines.com/asm">Статья о использовании ассемблерных</a> вставок прям в Go программе.</li>
<li><a href="https://habrahabr.ru/company/skbkontur/blog/276403/">Хабрастатья "Moira: Realtime Alerting"</a> про систему алертинга на Go.</li>
<li><a href="http://morsmachine.dk/causalprof">Казуальное профилирование</a> Go программ.</li>
<li>Пример, когда стоит <a href="https://dhdersch.github.io/golang/2016/01/23/golang-when-to-use-string-pointers.html">использовать указатели на строки</a>.</li>
</ul>

<h3>Инструменты</h3>

<ul>
<li><a href="https://github.com/mailhog/mailhog">MailHog</a> - Инструмент для тестирования работы с почтой.</li>
<li><a href="https://github.com/samuell/devbox-golang">devbox-golang</a> - Vagrant бокс для Go разработчиков с предустановленным инструментарием.</li>
<li><a href="https://github.com/rancher/trash">trash</a> - Еще один пакет для управления зависимостями и вендоринга.</li>
<li><a href="https://github.com/matryer/silk">silk</a> - Тестирование API на основе markdown документации.</li>
<li><a href="https://rootpd.com/2016/02/04/setting-up-intellij-idea-for-your-first-golang-project/">Настраиваем Intellij IDEA</a> для работы с Go проектами.</li>
<li><a href="https://github.com/restic/restic">restic</a> - Идеологически правильная программа для бекапов.</li>
<li><a href="https://github.com/achiku/wbs">wbs</a> - Инструмент для сборки и перезапуска http сервера при изменении файлов.</li>
<li><a href="https://github.com/frou/GoFeather">GoFeather</a> - Альтернативный Sublime Text плагин для работы с Go.</li>
<li><a href="https://github.com/yusukebe/revealgo">revealgo</a> - Инструмент для создания презентаций только с markdown и танцовщицами.</li>
</ul>

<h3>Всякое</h3>

<ul>
<li>Презентация от Brad Fitzpatrick. <a href="https://docs.google.com/presentation/d/1JsCKdK_AvDdn8EkummMNvpo7ntqteWQfynq9hFTCkhQ/preview?pref=2&amp;pli=1&amp;slide=id.p">Что нового в Go 1.6 и как с этим жить</a>.</li>
<li><a href="https://github.com/irismq/book">Книжка по IrisMQ</a> с примерами и сравнением с NSQ.</li>
<li><a href="https://github.com/fogleman/ln">ln</a> - Пакет для 3D векторного рендеринга на Go.</li>
<li><a href="https://github.com/matcornic/subify">subify</a> - Качалка субтитров для любимых телешоу на Go.</li>
</ul>

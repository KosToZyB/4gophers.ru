+++
date = "2014-12-21T23:56:11+03:00"
draft = false
title = "Тулзовины и хреновины 16"

+++

<p>Новость месяца - релиз Go v1.4. Поддержка андроида и большие планы на версию 1.5.</p>

<p>Кроме того, в этом месяце есть из чего выбирать. Много новых пакетов и статей.</p>

<p>И обязательно обратите внимание на перевод книги &quot;<a href="http://golang-book.ru/">Введение в программирование на Go</a> &quot; Калеба Докси.</p>

<h3>Новости</h3>

<ul>
	<li><a href="https://coreos.com/blog/etcd-2-0-release-candidate/">Выпуск новой версии etcd v2.0.0 RC1</a>. Пробуем, любуемся.</li>
	<li><a href="https://docs.google.com/document/d/1QXzI9I1pOfZPujQzxhyRy6EeHYTQitKKjHfpq0zpxZs/edit#heading=h.5nqylt8x3hka">Предложения по изменению</a> в пакете системных вызовов в Go.</li>
	<li>Теперь есть возможность <a href="http://www.thedotpost.com/conference/dotgo-2014">посмотреть все видосы с dotGo 2014</a> в удобном формате.</li>
	<li>Возрадуйтесь! <a href="http://blog.golang.org/go1.4">Go v1.4 зарелизился</a>. Расчехляем свои <a href="https://github.com/moovweb/gvm">gvm</a> и устанавливаем новую версию.</li>
	<li>Ну и, собственно, вот: <a href="https://github.com/golang/go">исходники Go на Github</a>.</li>
	<li><a href="http://tour.golang.org/">Новая версия тура по языку Go</a> для тех, кто хочет быстро ознакомиться с основными возможностями.</li>
	<li>В блог gopheracademy теперь <a href="https://github.com/gopheracademy/gopheracademy-web">можно написать свои статьи</a> с помощью репозитория на гитхаб.</li>
</ul>

<h3>Проекты</h3>

<ul>
	<li><a href="https://github.com/jroimartin/orujo">orujo</a> - Да, это еще один микро веб-фремворк для Go.</li>
	<li><a href="https://github.com/tgulacsi/gocilib">gocilib</a> - Драйвер для базы данных oracle с OCILIB.</li>
	<li><a href="https://github.com/oniony/TMSU">TMSU</a> - Тулза для тегирования ваших файлов. Идея старая, реализация новая.</li>
	<li><a href="https://github.com/ziyasal/socket.io-go-emitter">socket.io-go-emitter</a> - Реализация socket.io-emitter на языке Go.</li>
	<li><a href="https://github.com/alecthomas/kingpin">kingpin</a> - Еще один пакет для парсинга консольных флагов и аргументов.</li>
	<li><a href="https://github.com/Jeffail/leaps">leaps</a> - веб-сервис для совместного редактирования текстов. Написан на Go.</li>
	<li><a href="https://github.com/muratsplat/webStat">webStat</a> - Простой веб-сервис для получения статистики по CPU и памяти.</li>
	<li><a href="https://github.com/mijia/modelq">ModelQ</a> - Генератор Go кода для доступа к RDBMS базам. Пока только MySQL.</li>
	<li><a href="https://github.com/oleiade/lane">lane</a> - Библиотека для создания очередей, стеков и двусвязных очередей.</li>
	<li><a href="https://github.com/briandowns/spinner">spinner</a> - Простая библиотека для рисования индикаторов(спинеров) в консоле.</li>
	<li><a href="https://github.com/shazow/ssh-chat">ssh-chat</a> - Это как IRC чат, только ходит по SSH.</li>
	<li><a href="https://github.com/mitchellh/ioprogress">ioprogress</a> - Простой пакет, который показывает прогресс копирования файлов. Работает как обертка над io.Reader.</li>
	<li><a href="https://github.com/ivpusic/go-hotreload">go-hotreload</a> - Тулза, которая перекомпилирует и перезапускает Go приложение.</li>
	<li><a href="https://github.com/codingsince1985/geo-golang">GeoService</a> - Принципиально новый и нескучный гео-сервис на Go.</li>
	<li><a href="https://github.com/imdario/nunc">nunc</a> - Простой таск менеджер, призванный победить прокрастинацию.</li>
	<li><a href="https://github.com/takama/assert">assert</a> - Простой пакет для сравнения неопределенных типов (interface{}). Можно использовать при разборе json.</li>
	<li><a href="https://github.com/msoap/yandex-weather-cli">yandex-weather-cli</a> - Консольный клиент для яндекс погоды на Go.</li>
	<li><a href="https://github.com/tomazk/envcfg">envcfg</a> - Работа с переменными окружения в Go.</li>
	<li><a href="https://github.com/auroratechnologies/vangoh">VanGoH</a> - Реализация аутентификации по схеме HMAC</li>
	<li><a href="https://github.com/k0kubun/pp">pp</a> - Пакет для красивого отображения информации в консоли.</li>
	<li><a href="https://github.com/yosuke-furukawa/json5">json5</a> - JSON5 реализация на языке Go.</li>
	<li><a href="https://github.com/elazarl/go-bindata-assetfs">go-bindata-assetfs</a> - Раздаем встроенные данные с помощью net/http.</li>
	<li><a href="https://github.com/mcuadros/go-jsonschema-generator">go-jsonschema-generator</a> - Генератор JSON схемы по Go структурам.</li>
	<li><a href="https://github.com/coreos/rocket">Rocket</a> - Команда coreos написали свой контейнеровоз. Контейнеризации мало не бывает.</li>
	<li><a href="https://github.com/antonholmquist/jason/">jason</a> - Пакет для работы с JSON в Go. Упрощает жизнь, скрывает кастования типов.</li>
	<li><a href="https://github.com/mkevac/debugcharts">debugcharts</a> - Go-программа, которая рисует графики с отладочной информацией.</li>
	<li><a href="https://github.com/sethgrid/multibar">multibar</a> - Рисуем сколько угодно прогрессбаров в консоли.</li>
	<li><a href="https://github.com/keighl/barkup">barkup</a> - пакет для написания программ-бекаперов. Можем бекапить данные из MySQL на S3.</li>
</ul>

<h3>Статьи</h3>

<ul>
	<li><a href="http://blog.travis-ci.com/2014-12-17-faster-builds-with-container-based-infrastructure/">Новая инфраструктура на travis-ci</a> с использованием Docker.</li>
	<li>Пост из <a href="http://blog.gopheracademy.com/advent-2014/atlas/">блога gopheracademy про сервис Atlas</a> , построенный на Rails и Go Microservices.</li>
	<li><a href="https://medium.com/@richardeng/a-word-from-the-beegoist-d562ff8589d7">Статья для тех</a>, кто хочет быстро ознакомится с основными возможностями beego.</li>
	<li>Soy - <a href="http://blog.gopheracademy.com/advent-2014/soy-programmable-templates/">Движок шаблонов для Go</a> приложений. Он же Closure Templates.</li>
	<li><a href="https://www.mikeperham.com/2014/12/17/expvar-metrics-for-golang/">Метрики для вашего Go</a> приложения с помощью пакета <a href="http://golang.org/pkg/expvar/">expvar</a>.</li>
	<li><a href="http://www.weberc2.com/posts/2014/12/12/living-without-generics.txt">Жизнь без дженериков</a>. Выкручиваемся как можем.</li>
	<li><a href="http://dead10ck.github.io/2014/12/15/go-vs-c.html">Сравнение конкурентных фич у Go и С</a>. Немного странный выбор для сравнения И я знаю, кто победит.</li>
	<li><a href="http://www.goinggo.net/2014/12/using-pointers-in-go.html">Работа с указателями в языке Go</a>. Хорошая статья от William Kennedy.</li>
	<li>Реализуем <a href="http://nicolasmerouze.com/how-to-render-json-api-golang-mongodb/">стандартное JSON-API</a> в Go и MongoDB.</li>
	<li><a href="http://blog.gopheracademy.com/advent-2014/case-against-3pl/">Про использование сторонних библиотек</a> или как я перестал волноваться и использовать версионирование библиотек.</li>
	<li><a href="http://adampresley.com/2014/12/15/handling-ctrl-c-in-go-command-line-applications.html">Перехват сочетания клавиш Ctr+C</a> в консольных Go программах.</li>
	<li>Вероятностные структуры данных для Go: <a href="http://blog.gopheracademy.com/advent-2014/bloom-filters/">Блум Фильтры</a>.</li>
	<li>Безопасное <a href="http://blog.gopheracademy.com/advent-2014/safe-json-file-db-in-go/">использование JSON файлов как базы данных</a>. Новая статья в блоге gopheracademy.</li>
	<li>Пишем <a href="http://blog.gopheracademy.com/advent-2014/fuse-zipfs/">свою файловую систему</a> с помощью Go и FUSE.</li>
	<li><a href="http://blog.appsdeck.eu/post/105010314493/writing-a-replacement-to-openssh-using-go-2-2">Пишем замену для SSH</a> на Go. Часть 2.</li>
	<li>Хорошая <a href="http://blog.gopheracademy.com/advent-2014/goquery/">статья про пакет goquery</a> в блоге gopheracademy.</li>
	<li><a href="https://medium.com/@shijuvar/deploying-go-web-apps-with-docker-1b7561b36f53">Деплой Go веб-приложения</a> с помощью docker. Еще один пример.</li>
	<li><a href="http://blog.gopheracademy.com/advent-2014/nigels-webdav-package/">Новый пакет для работы с WebDAV</a> от Nigel Tao и Nick Cooper.</li>
	<li><a href="http://blog.gopheracademy.com/advent-2014/easy-deployment/">Простой деплой для Docker</a> с помощью хуков. Статья из блога gopheracademy.</li>
	<li><a href="http://blog.nytlabs.com/2014/12/09/hive-open-source-crowdsourcing-framework/">Hive. Опенсорсный фреймворк</a> для краудсорсинга.</li>
	<li><a href="http://blog.gopheracademy.com/advent-2014/patchwork/">Patchwork Toolkit</a> - Легковесная платформа для создания интернета вещей(Network of Things).</li>
	<li>Еще немного <a href="http://andlabs.lostsig.com/blog/2014/12/09/189/thoughts-on-generics">рассуждений про дженерики</a> в Go.</li>
	<li><a href="http://avtok.com/2014/11/05/interface-upgrades.html">Еще одна годная статья</a> про интерфейсы в Go.</li>
	<li><a href="http://blog.gopheracademy.com/advent-2014/reading-config-files-the-go-way/">Чтение конфигурационных файлов</a> в Go стиле.</li>
	<li><a href="http://blog.gopheracademy.com/advent-2014/macaron/">Macaron</a>. Фреймворк в стиле Martini, но быстрее и производительнее.</li>
	<li><a href="http://blog.appsdeck.eu/post/104426674353/writing-a-replacement-to-openssh-using-go-1-2">Пишем замену для OpenSSH</a> на языке Go с преферансом и танцовщицами.</li>
	<li><a href="http://blog.gopheracademy.com/advent-2014/string-matching/">Поиск подстрок на Go</a>. Статья из блога gopheracademy.</li>
	<li><a href="http://nicolasmerouze.com/how-to-render-json-api-golang-mongodb/">Реализация JSON-API</a> средствами Go и MongoDB</li>
	<li><a href="https://blog.docker.com/2014/12/announcing-docker-machine-swarm-and-compose-for-orchestrating-distributed-apps/">Анонс Docker машины</a> , сварм и управление распределенными приложениями.</li>
	<li><a href="http://vluxe.io/golang-template.html">Разбираемся с шаблонами</a> в Go(не дженерики!). Статья от Austin Cherry.</li>
	<li>Обработка терабайтов git данных, отслеживание нагрузки на приложение, кеширование HTTP рессурсов. <a href="https://sourcegraph.com/blog/go-at-sourcegraph">И вссе это в блоге Sourcegraph</a>.</li>
	<li><a href="http://blog.gopheracademy.com/advent-2014/git2go-tutorial/">Начинаем работу с Git2go</a>. Новая статья в блоге gopheracademy.</li>
	<li>Code of Ages. <a href="https://medium.com/backchannel/my-computer-language-is-better-than-yours-58d9c9523644">Кое-кто считает, что Go и Swift стоят на шаг выше</a> относительно всех остальных языков программирования.</li>
	<li>Новая статья в боле gopheracademy &quot;<a href="http://blog.gopheracademy.com/advent-2014/go-probably/">Probabilistic Data Structures for Go</a>&quot;.</li>
	<li><a href="http://www.alexedwards.net/blog/handler-chains-using-stack">Контекстно-зависимые обработчики цепочек в Go</a> с использованием пакета <a href="https://github.com/alexedwards/stack">stack</a>.</li>
	<li><a href="http://blog.gopheracademy.com/advent-2014/parsers-lexers/">Самодельные парсеры и лексеры</a> на Go. Еще одна статья из блога gopheracademy.</li>
	<li>Пример <a href="http://dahernan.github.io/2014/12/03/scrap-the-web-with-go/">веб-скрепинга при помощи Go</a>.</li>
	<li>Начинаем работы с <a href="http://tleyden.github.io/blog/2014/12/02/getting-started-with-go-and-protocol-buffers/">Protocol Buffers на языке Go</a>.</li>
	<li>&quot;<a href="http://blog.gopheracademy.com/birthday-bash-2014/using-go-for-anomaly-detection/">Using Go for Anomaly Detection</a>&quot;. Свежая статья в блоге gopheracademy</li>
	<li><a href="http://dave.cheney.net/2014/12/01/five-suggestions-for-setting-up-a-go-project">Пять советов</a> по созданию Go проекта от Dave Cheney.</li>
	<li><a href="http://openmymind.net/basic-micro-optimizations-part-2/">Оптимизация программ на Go</a>. Часть II</li>
</ul>

<h3>Видео</h3>

<ul>
	<li><a href="http://4gophers.com/video/optimizaciya">Оптимизация</a>. Ashish Gandhi из CloudFlare рассказывает как можно оптимизировать свои приложения на Go. Слайды и дополнительный материалы можно <a href="https://www.hakkalabs.co/articles/optimizing-go-3k-requestssec-480k-requestssec">посмотреть в блоге</a>.</li>
	<li><a href="http://4gophers.com/video/rocket-tutorial">Rocket Tutorial</a>. Недавно, ребята из CoreOS написали свой менеджер контейнеров - Rocket. И вот небольшой туториал по его использованию.</li>
	<li><a href="http://4gophers.com/video/kalkulyator-dlya-bignum">Калькулятор для BigNum</a>. Rob Pike описывает интерпретатор для языка вычислений в стиле APL. <a href="http://go-talks.appspot.com/github.com/robpike/ivy/talks/ivy.slide#1">Слайды можно глянуть тут</a>.</li>
</ul>

<h3>Инструменты</h3>

<ul>
	<li><a href="http://samuel.lampa.co/posts/smallest-pipeable-go-program/">Небольшой пример программы</a> , которую можно запускать в nix канале</li>
	<li><a href="http://fsnotify.org/">Новая версия fsnotify</a>. Приложения, которое следит за изменениями файлов на диске.</li>
	<li><a href="https://github.com/visualfc/liteide">Новая версия LiteIde</a> X25.1. Хорошая и простая IDE для Go.</li>
	<li><a href="https://medium.com/@olebedev/live-code-reloading-for-golang-web-projects-in-19-lines-8b2e8777b1ea">Live reload</a> для web проекта на Go в 19 строчек кода.</li>
	<li><a href="http://eefret.me/making-sublime-your-golang-ide/">Делаем полноценную IDE</a> для Go из SublimeText</li>
	<li>Отличный сервис для <a href="http://some2go.herokuapp.com/">конвертирования JSON или YAML в Go структуры</a> от <a href="https://twitter.com/">@saratovsource</a>.</li>
	<li>Веб-сервис для <a href="https://mholt.github.io/json-to-go/">конвертирования JSON в Go</a> структуры.</li>
	<li><a href="http://dominik.honnef.co/posts/2014/12/an_incomplete_list_of_go_tools/">Полезные инструменты</a> , про которые часто забываешь. И это еще не полный список.</li>
	<li><a href="https://github.com/ravenac95/sudolikeaboss">sudolikeaboss</a> - Отличная тулза, если вас задалбывает каждый раз вводить пароли для sudo. Но пока только под маки.</li>
</ul>

<h3>Всякое</h3>

<ul>
	<li>Крутые ребята собрались и перевели &quot;<a href="http://golang-book.ru/">Введение в программирование на Go</a> &quot; от Калеба Докси. Спасибо <a href="http://zenwalker.ru/">Максим Полетаев</a> , <a href="http://rozaev.ru/">Виктор Розаев</a>.</li>
	<li><a href="https://github.com/PuerkitoBio/gocostmodel">gocostmodel</a> - Пакет, в котором собраны бенчмарки по всем базовым операциям в Go.</li>
	<li><a href="https://github.com/MarinX/godroid">godroid</a> - Пишем приложение на Go для андроид.</li>
	<li><a href="https://github.com/michaeldv/donna">donna</a> - Экспериментальный шахматный движок полностью на Go.</li>
</ul>

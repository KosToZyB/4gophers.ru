+++
date = "2014-04-27T19:12:07+03:00"
draft = false
title = "GopherCon 2014"

+++

<p>Закончилась самая ожидаемая Go-конфа этого года. Ооочень много всего интересного было рассказано и показано.</p>

<p>В этом посте постараюсь выкладывать найденные материалы для всех желающих.</p>

<p>К сожалению, я не знаю как встраивать презентации с talks.golang.org к себе на страничку. Если кто подскажет, буду очень благодарен.</p>

<h3 id="hello-gophers">Hello, Gophers!</h3>

<p>И так, первым выступал сам командор - Роб Пайк с докладом <a href="http://talks.golang.org/2014/hellogophers.slide#1">&quot;Hello, Gophers!&quot;</a>. Роб рассказывает о совершенствовании языка от идеи и до сегодняшнего дня.</p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/ZtoZeVadnYA" width="720"></iframe></p>

<h3 id="building-high-performance-systems-in-go-whats-new-and-best-practices">Building High-Performance Systems in Go - What&rsquo;s New and Best Practices</h3>

<p><a href="https://twitter.com/derekcollison">Derek Collison</a> рассказывает о том, как строить высокопроизваодительные системы на Go</p>

<p><iframe allowfullscreen="" frameborder="0" height="600px" marginheight="0" marginwidth="0" scrolling="no" src="http://www.slideshare.net/slideshow/embed_code/33907343" style="border:1px solid #CCC; border-width:1px 1px 0; margin-bottom:5px; max-width: 100%;" width="100%"></iframe></p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/ylRKac5kSOk" width="720"></iframe></p>

<h3 id="go-circuit">Go Circuit</h3>

<p><a href="https://twitter.com/maymounkov">Petar Maymounkov</a> <a href="https://docs.google.com/presentation/d/1ooedstHs8_ow-eHY7z8MCV_1m65gSaiB6Q1ruz3j7Hk/edit#slide=id.g26f183bd0_00">рассказывает про Go Circuit</a> инструмент синхронизации UNIX процессов между целыми кластерами с помощью интерфейса файловой системы. <a href="https://github.com/gocircuit/circuit">И исходники</a>.</p>

<p><iframe allowfullscreen="" frameborder="0" height="420" src="//www.youtube.com/embed/i2VaXnRhob0?list=PLE7tQUdRKcyb-k4TMNm2K59-sVlUJumw7" width="720"></iframe></p>

<h3 id="a-channel-compendium">A Channel Compendium</h3>

<p><a href="https://twitter.com/jgrahamc">John Graham-Cumming</a> с докладом про совместную работу каналов. Есть небольшие <a href="https://github.com/cloudflare/jgc-talks/tree/master/GopherCon/2014">примеры на github</a>.</p>

<p><iframe allowfullscreen="" frameborder="0" height="600px" marginheight="0" marginwidth="0" scrolling="no" src="http://www.slideshare.net/slideshow/embed_code/33908924" style="border:1px solid #CCC; border-width:1px 1px 0; margin-bottom:5px; max-width: 100%;" width="100%"></iframe></p>

<h3 id="embedded-go-and-bluetooth-low-energy-hardware">Embedded Go and Bluetooth Low Energy Hardware</h3>

<p><a href="http://go-talks.appspot.com/github.com/gophercon/2014-talks/offbymany/ble_embedded.slide#1">Доклад</a> <a href="https://twitter.com/offbymany">Josh Bleecher Snyder</a> о использовании Go в встраиваемых системах на нестандартных архитектурах и с ограниченым питанием. В частности, для <a href="https://www.paypal.com/us/webapps/mpp/beacon">PayPal Beacon</a></p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/a4BQRUpQoe8" width="720"></iframe></p>

<h3 id="go-for-sysadmins">Go for Sysadmins</h3>

<p><a href="https://twitter.com/kelseyhightower">Kelsey Hightower</a> <a href="http://go-talks.appspot.com/github.com/gophercon/2014-talks/kelseyhightower/go_for_sysadmins.slide#1">рассказывает</a> о том, как можно использовать golang для решения административных задач. Особенно актуально, когда проект исключительно на Go и не хочется тянуть сторонние пакеты.</p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/41GffiXhN6I" width="720"></iframe></p>

<h3 id="data-snarfing-with-go-a-heka-good-time">Data Snarfing with Go: A Heka Good Time</h3>

<p><a href="http://4gophers.com/2014-talks/rob_miller_heka/#/">Доклад</a> от <a href="https://twitter.com/n0nsequitarian">Rob Miller</a> про замечательный инструмент - <a href="https://github.com/mozilla-services/heka">heka</a>, разработанный в недрах Moozilla. Сбор и обработка огромного количества данных - это то, чем занимается heka.</p>

<p><iframe allowfullscreen="" frameborder="0" height="420" src="//www.youtube.com/embed/RhLIblr_YXs" width="720"></iframe></p>

<h3 id="gophers-on-a-plane-the-story-of-go-on-app-engine">Gophers on a Plane: The Story of Go on App Engine</h3>

<p>David Symonds рассказал про любовь между Go и GAE. Про преимущества, недостатки, отличительные особенности рантайма. К сожалению, пока не нашел его презентации. Будем ждать.</p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/wzSX35qttjg" width="720"></iframe></p>

<h3 id="writing-a-high-performance-database-in-go">Writing a High Performance Database in Go</h3>

<p>Почему бы не использовать Go для написания производительной базы данных? О том, как это лучше всего сделать в докладе <a href="https://twitter.com/benbjohnson">Ben Johnson</a></p>

<p><script async class="speakerdeck-embed" data-id="1c8cdc00ad9901311d303e814ed3dd4c" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script></p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/4xB46Xl9O9Q" width="720"></iframe></p>

<h3 id="from-nodejs-to-go">From Node.js to Go</h3>

<p><a href="https://twitter.com/kfalter">Kelsey Falter</a> рассказывала, почему их контора переходит с Node.js на Go. Доклада я так и не нашел. Но и не факт, что он интересный.</p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/mBy20FgB68Q" width="720"></iframe></p>

<h3 id="spray-some-nsq-on-it">Spray Some NSQ On It</h3>

<p><a href="https://twitter.com/imsnakes">Matt Reiferson</a> читал доклад про весьма интересный инструмент <a href="http://bitly.github.io/nsq/">NSQ</a> - это риалтаймовая система доставки сообщений, pubsub фреймворк.</p>

<p><script async class="speakerdeck-embed" data-id="61b44010aea5013156984eab6e1d3af5" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script></p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/CL_SUzXIUuI" width="720"></iframe></p>

<h3 id="taking-back-the-command-line-with-go">Taking Back the Command Line with Go</h3>

<p><a href="https://twitter.com/mikegehard">Mike Gehard</a> о использовании Go как языка для написания маленьких консольных утилит, которые очень быстро работают. Презентации пока не нашел.</p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/k6W1KS_EGI0" width="720"></iframe></p>

<h3 id="camlistore-the-standard-library">Camlistore &amp; The Standard Library</h3>

<p><a href="https://twitter.com/bradfitz">Brad Fitzpatrick</a> рассказывает про разработку стандартной библиотеки Go и про возникающие трудности. Презентацию можно <a href="http://go-talks.appspot.com/github.com/gophercon/2014-talks/bradfitz/2014-04-Gophercon.slide#1">посмотреть тут</a>, но она как-то хреново отображается, можно <a href="https://github.com/gophercon/2014-talks/tree/master/bradfitz">глянуть исходники</a>.</p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/D6okO8Qzusk" width="720"></iframe></p>

<h3 id="go-from-c-to-go">Go from C to Go</h3>

<p><a href="https://twitter.com/_rsc">Russ Cox</a> говорит, что пора писать компилятор Go на самом Go, а не на C. Рассказывает, что нового в Go 1.3 и что будет еще. <a href="http://talks.golang.org/2014/c2go.slide#1">Появилась презентация</a>.</p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/QIE5nV5fDwA" width="720"></iframe></p>

<h3 id="painless-data-storage-with-mongodb-and-go">Painless Data Storage with MongoDB and Go</h3>

<p>Совместный доклад <a href="https://twitter.com/gniemeyer">Gustavo Niemeyer</a> и <a href="https://twitter.com/gniemeyer">Steve Francia</a> рассказывают про <a href="http://labix.org/mgo">работу с монгой</a>. Презентации пока нет.</p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/9OkclcLgR0U" width="720"></iframe></p>

<h3 id="best-practices-for-production-environments">Best Practices for Production Environments</h3>

<p><a href="https://twitter.com/peterbourgon">Peter Bourgon</a> с отличным докладом про самые правильные практики для серьезного продакшена. Кроме презентации есть даже <a href="http://peter.bourgon.org/go-in-production/">статья в его блоге</a>.</p>

<p><iframe allowfullscreen="" frameborder="0" height="600px" marginheight="0" marginwidth="0" scrolling="no" src="http://www.slideshare.net/slideshow/embed_code/33996165" style="border:1px solid #CCC; border-width:1px 1px 0; margin-bottom:5px; max-width: 100%;" width="100%"></iframe></p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/Y1-RLAl7iOI" width="720"></iframe></p>

<h3 id="building-an-analytics-engine-using-mongodb-and-go">Building an Analytics Engine using MongoDB and Go</h3>

<p><a href="https://twitter.com/GoingGoDotNet">William Kennedy</a> рассказывает про монгу, биг дата и Go.</p>

<p><iframe allowfullscreen="" frameborder="0" height="600px" marginheight="0" marginwidth="0" scrolling="no" src="http://www.slideshare.net/slideshow/embed_code/31052639" style="border:1px solid #CCC; border-width:1px 1px 0; margin-bottom:5px; max-width: 100%;" width="100%"></iframe></p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/EfJRQ1lGkUk" width="720"></iframe></p>

<h3 id="making-docker-go-why-one-of-the-fastest-growing-open-source-projects">Making Docker GO: Why One of the Fastest Growing Open Source Projects</h3>

<p>Хороший доклад от <a href="https://twitter.com/vieux">Victor Vieux</a> про Go и один из самых знаменитых golang проектов - docker.</p>

<p><script async class="speakerdeck-embed" data-id="596f53d0ae760131d02a36b29a007025" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script></p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/i26SYvVu1nw" width="720"></iframe></p>

<h3 id="writing-and-debugging-a-web-based-multi-player-game">Writing and Debugging a Web-Based Multi-Player Game</h3>

<p><a href="https://twitter.com/smcquay">Stephen McQuay</a> и <a href="https://twitter.com/twisted_weasel">Fraser Graham</a> о сложностях написания игр на Go.&nbsp;</p>

<p><iframe frameborder="0" height="500px" marginheight="0" marginwidth="0" scrolling="no" src="http://www.slideshare.net/slideshow/embed_code/34069905" width="100%"></iframe></p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/PJlp1YacstQ" width="720"></iframe></p>

<h3 id="gobot-go-powered-robotics-and-physical-computing">Gobot: Go Powered Robotics and Physical Computing</h3>

<p><a href="https://twitter.com/deadprogram">Ron Evans</a> и <a href="https://twitter.com/adzankich">Adrian Zankich</a> рассказывают про очень интересный проект - <a href="http://gobot.io/">Gobot</a>. Ждем презентации.</p>

<p><iframe allowfullscreen="" frameborder="0" height="470" src="//www.youtube.com/embed/Va-NE55AqBs" width="720"></iframe></p>

<h3 id="building-database-applications-with-databasesql">Building Database Applications with database/sql</h3>

<p><a href="https://twitter.com/xaprb">Baron Schwartz</a> на <a href="http://4gophers.com/2014-talks/baron-schwartz/database-sql.pdf">примерах объясняет</a> как лучше всего делать приложения с <a href="http://go-database-sql.org/">использованием пакета database/sql</a></p>

<p><iframe allowfullscreen="" frameborder="0" height="600px" marginheight="0" marginwidth="0" scrolling="no" src="http://www.slideshare.net/slideshow/embed_code/33996167" style="border:1px solid #CCC; border-width:1px 1px 0; margin-bottom:5px; max-width: 100%;" width="100%"></iframe></p>

<p><iframe width="720" height="420" src="//www.youtube.com/embed/m879N2rzn2g" frameborder="0" allowfullscreen></iframe></p>

<h3 id="build-your-developer-tools-in-go">Build Your Developer Tools in Go</h3>

<p><a href="https://twitter.com/inconshreveable">Alan Shreve</a> автор <a href="https://ngrok.com/">ngrok</a>. Он рассказывает про написание собственных инструментов разработки. Презентации не нашел.</p>

<p><iframe width="720" height="420" src="//www.youtube.com/embed/stFw0f3WEsk" frameborder="0" allowfullscreen></iframe></p>

<h3 id="building-web-services-in-go">Building Web Services in Go</h3>

<p><a href="https://twitter.com/rcrowley">Richard Crowley</a> читает доклад о построении веб сервисов с использованием Go. Презентации пока нет.</p>

<h3>Inside the Gophers Studio with Blake Mizerany</h3>

<p>Общение с метрами и корифеями языка программирования Go </p>

<p><iframe width="720" height="420" src="//www.youtube.com/embed/u-kkf76TDHE" frameborder="0" allowfullscreen></iframe></p>

<h3>Closing Day Keynote by Andrew Gerrand</h3>

<p><iframe width="720" height="420" src="//www.youtube.com/embed/dKGmK_Z1Zl0" frameborder="0" allowfullscreen></iframe></p>

<h3>Lightningtalks<!--3--></h3>

<p><iframe frameborder="0" height="500" marginheight="0" marginwidth="0" scrolling="no" src="http://www.slideshare.net/slideshow/embed_code/34070176" width="100%"></iframe></p>

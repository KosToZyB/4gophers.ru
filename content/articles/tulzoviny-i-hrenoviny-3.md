+++
date = "2014-03-10T20:57:08+03:00"
draft = false
title = "Тулзовины и хреновины #3"

+++

<p>Очередная порция ссылок и интересностей из мира golang. Это уже 3й выпуск и хорошая традиция. Надеюсь, войдет в привычку.</p>

<p>Самая важная новость всего выпуска - встреча гоферов в <a href="http://gopherway.com/">Киеве 1.06.2014 </a> и релиз <a href="https://code.google.com/p/go/downloads/list">Go 1.2.1</a></p>

<p>Выпуски&nbsp;<a href="http://tvorzasp.com/blog/golang-tools-and-hreniviny/">раз</a>&nbsp;и <a href="http://4gophers.com/article/tulzoviny-i-hrenoviny-2">два</a>.</p>

<h3 id="_1">Проекты</h3>

<ul>
	<li><a href="http://clipperhouse.github.io/gen/">gen</a> - людям не хватает дженериков и они извращаются. Мне понравилось.</li>
	<li><a href="https://github.com/mikespook/goemphp">goemphp</a> - фантастика. Выполняем PHP из Go.</li>
	<li><a href="https://github.com/gopherjs/gopherjs">gopherjs</a> - и можем компилировать Go в js.</li>
	<li><a href="https://github.com/johnsto/speedtest">speedtest</a> - консольный клиент для speedtest.net. Никакого флеша.</li>
	<li><a href="https://github.com/russross/blackfriday">blackfriday</a> - шикарный пакет для работы с markdown.</li>
	<li><a href="https://github.com/codegangsta/cli">cli</a> - набор инструментов для создания консольных утилит.</li>
	<li><a href="http://godoc.org/gopkg.in/v1/docs">package mapper</a> - теперь можно ясно видеть какую версию пакета ты импортируешь.</li>
	<li><a href="https://github.com/go-yaml/yaml">yaml</a> - поддержка YAML в golang.</li>
	<li><a href="https://github.com/drone/drone">drone</a> - как бы CI система, построенная на Docker.</li>
	<li><a href="http://heka-docs.readthedocs.org/en/latest/">Heka</a> - совсем не новость, но я узнал про эту штуку совсем не давно. Проект от мозилы по сбору аналитики с ваших серверов.</li>
	<li><a href="http://docs.btcplex.com/">BTCplex</a> - просмотр биткоин блоков.</li>
</ul>

<h3 id="_2">Новости</h3>

<ul>
	<li>Релиз <a href="https://code.google.com/p/go/downloads/list">Go 1.2.1</a></li>
	<li>Зарелизилась новая версия <a href="https://code.google.com/p/googleappengine/wiki/SdkForGoReleaseNotes">Go SDK 1.9.0</a> для GAE.</li>
	<li>Прошла <a href="https://plus.google.com/communities/103337146295481792015">встреча бостонских гоферов</a>. Ждем видео и презентаций.</li>
	<li><a href="https://github.com/spf13/hugo/tree/v0.10">Свежая версия</a> движка для статичных сайтов <a href="http://hugo.spf13.com/">Hugo</a></li>
	<li>Свежий <a href="https://github.com/revel/revel/releases/tag/v0.9.0">Revel 0.9.0</a> и сразу фиксы в версии <a href="https://github.com/revel/revel/releases/tag/v0.9.1">Revel 0.9.1</a>.</li>
</ul>

<h3 id="_3">Статьи и статейки</h3>

<ul>
	<li><a href="http://blog.joshsoftware.com/2014/03/10/how-do-i-create-a-presentation-using-go/">How Do I Create A Presentation Using Go?</a>. Оказывается, есть замечательный пакет <a href="https://godoc.org/code.google.com/p/go.talks/present">present</a> с помощью которого можно фигачить презентации.</li>
	<li><a href="http://habrahabr.ru/post/215111/">Исполнение SSH-команд на сотнях серверов с помощью Go</a>. Отличная статья про использование Go в администрировании. И в каментах отличный пример на <a href="http://habrahabr.ru/post/215111/#comment_7388383">erlamg</a>.</li>
	<li><a href="http://www.drdobbs.com/cloud/restful-web-service-in-go-powered-by-the/240006401">RESTful Web Service in Go Powered by the Google App Engine</a>. Еще одна статья про REST и GAE. Вот еще интересный пример реализации <a href="https://gist.github.com/peterhellberg/9450839">простого апи на Martini И Redigo</a>.</li>
	<li><a href="http://ayende.com/blog/165857/reviewing-etcd">Reviewing etcd</a>. Обзор одного из самых больших и известных проектов на Go. Рекомендую почитать всем гоферам.</li>
	<li><a href="http://www.goinggo.net/2014/03/web-form-validation-and-localization-in.html">Web Form Validation And Localization In Go</a>. Статья от William Kennedy про валидацию форм и интернационализацию.</li>
	<li><a href="http://blog.vladimirvivien.com/2014/03/hacking-go-filter-values-from-multi.html">Интересный хак</a> для использования возвращаемых из функции значений. Уменьшает количество присваиваний и, вроде как, делает код чище.</li>
	<li><a href="http://blog.cloudflare.com/its-go-time-on-linux">It&#39;s Go Time on Linux</a> - работа с временем в golang</li>
	<li><a href="http://blog.stretchr.com/2014/03/05/test-driven-development-specifically-in-golang/">Разработка через тестирование</a> с учетом специфики Go-мира.</li>
	<li><a href="http://smira.ru/en/posts/aptly-memory-usage-optimization.html">Memory Usage Optimization</a>. Про оптимизацию памяти на примере <a href="http://www.aptly.info/">aptly</a></li>
	<li>Большая редкость - статья на русском языке про <a href="http://dafter.ru/duf/golang/133.html">начало работы с Go</a></li>
	<li><a href="http://spf13.com/post/cross-compiling-go">Cross Compiling With Go</a>. Кросс компилирование - это крутая штука.</li>
	<li>Пишем свой <a href="http://dig.floatingsun.net/gaestebin-2.0/">pastebin</a>, но на Go. И хостим на GAE.</li>
	<li><a href="http://nathany.com/go-packages/?2.0">Go Package Management</a>. Новая версия старой статьи. Оставлю тут список <a href="https://code.google.com/p/go-wiki/wiki/PackageManagementTools">всех менеджеров зависимостей</a>.</li>
	<li><a href="http://www.goinggo.net/2014/02/the-nature-of-channels-in-go.html">The Nature Of Channels In Go</a>. Мега крутая статья, объясняющая каналы в Go.</li>
	<li><a href="http://blog.joshsoftware.com/2014/02/26/go-interfaces-and-quacking/">Go Interfaces and Quacking</a>. Про систему типов в Go от нашего индийского друга Gautam Rege.</li>
	<li><a href="http://nesv.github.io/golang/2014/02/25/worker-queues-in-go.html">Writing worker queues, in Go</a>. Про канкаренси и как с этим жить.</li>
</ul>

<h3 id="_4">Видео</h3>

<ul>
	<li>Свежие gophercasts. Авторизация <a href="https://gophercasts.io/lessons/7-auth-part-1">часть I</a> и <a href="https://gophercasts.io/lessons/8-auth-part-2">часть II</a></li>
	<li><a href="http://gophervids.appspot.com/">gophervids</a> - свежие видосы про golang</li>
	<li>Отчет про <a href="http://blog.golang.org/fosdem14">FOSDEM 2014</a> на официальном блоге Go.</li>
	<li>Пишем веб-приложение на Go. <a href="http://4gophers.com/video/pishem-veb-prilozhenie-na-go-chast-1-nethttp">Часть I</a> и <a href="http://4gophers.com/video/pishem-veb-prilozhenie-na-go-chast-2-martini-markdown">Часть II</a></li>
	<li><a href="http://4gophers.com/video/pishem-svoi-kompilyator">Пишем свой компилятор</a></li>
	<li><a href="http://4gophers.com/video/pekursiya-kanaly-paralelnye-otobrazheniya">Pекурсия, каналы, параллельные отображения</a></li>
	<li><a href="http://4gophers.com/video/richard-crowley-iz-betable-rasskazyvaet-kak-delat-servisy-na-go">Как делать веб сервисы на Go</a> от Richard Crowley.</li>
	<li><a href="http://4gophers.com/video/sozdanie-sistemy-distribyutinga">Построение распределенных систем</a></li>
	<li><a href="http://4gophers.com/video/portirovanie-go">Портирование Gо</a> на новые системы</li>
</ul>

<h3 id="_5">Всякое</h3>

<ul>
	<li><a href="http://www.golangprojects.com/">golangprojects</a> - сайт с предложениями по работе для гоферов. Хочу такое, только у нас.</li>
	<li><a href="http://www.golangbootcamp.com/book/">golangbootcamp</a> - прибавление к книгам про golang от Matt Aimonetti.</li>
	<li><a href="https://docs.google.com/file/d/0B0MdpwoO1vCZSHEwd2xxaEdCd3c/edit">Getting to Go</a> - доклад с бостонской встречи гоферов</li>
	<li><a href="http://www.joshlf.com/wp-content/uploads/Reflection-Talk.pdf">Reflection in Go</a> - еще один доклад из Бостона</li>
	<li><a href="http://sendgrid.com/blog/convince-company-go-golang/">Статья</a> о том, как внедрить Gо в вашей конторе.</li>
	<li>Управление зависимостями с помощью gpm <a href="https://github.com/pote/gpm/blob/master/gpm_install.gif">в одной гифке</a>.</li>
	<li><a href="https://groups.google.com/forum/#!topic/golang-nuts/lLJh4ehIi2k">Обсуждение gamedev тулзовин</a> в англоязычной гуглгруппе.</li>
</ul>

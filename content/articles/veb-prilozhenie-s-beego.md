+++
date = "2014-07-13T00:56:12+03:00"
draft = false
title = "Веб приложение с Beego"

+++

<p>Это перевод статьи от <a href="http://www.sitepoint.com/author/msetter/">Matthew Setter</a>. Оригинал на <a href="http://www.sitepoint.com/go-building-web-applications-beego/">sitepoint.com</a></p>

<h3>Введение</h3>

<p>Вы веб-девелопер, пришедший к Go с динамических языков программирования, таких как: PHP, Python, Ruby и хотите узнать, как использовать этот самый Go для разработки веба? Вам интересно узнать, как разрабатывать на Go в привычной для вас манере ваших любимых фреймворков, используя свой накопленный опыт?</p>

<p>Если это так, значит вы уже искали в гугле, на StackOverflow и других сайтах фреймворки, которые могут вам помочь. И вы наверняка видели такие названия как <a href="http://beego.me/">Beego</a>, <a href="http://martini.codegangsta.io/">Martini</a> и <a href="http://www.gorillatoolkit.org/">Gorilla</a>. А также, пакет <a href="http://golang.org/pkg/net/http/">net/http</a>.</p>

<p>Из этих четырех фреймворков я больше всего экспериментировал с Beego. Я обнаружил, что это многофункциональный и притом не очень сложный инструмент, который позволяет очень быстро начать разрабатывать веб-сайты.</p>

<p>Beego не даст вам сразу таких же результатов, как ваши используемые фреймворки. Да, в beego много готовых батареек и различных пакетов, таких как:</p>

<ul>
    <li>Полноценная ORM</li>
    <li>Кеш</li>
    <li>Работа с сессиями</li>
    <li>Интернационализация (i18n)</li>
    <li>Мониторинг и перегрузка приложения при разработке</li>
    <li>Инструменты для деплоя</li>
</ul>

<p>Но, несмотря на существующие сходства, этот фрейморк очень сильно отличается от всех уже известных вам, написанных на динамических языках. И вам прийдётся потратить некоторое временя, чтобы Beego стал действительно продуктивным инструментом.</p>

<p>Кроме того, хоть документация по фреймворку достаточно полная, создается ощущения пропущенных пунктов то там, то тут. Поэтому, я решил написать серию из двух статей, которые помогут вам понять эти нюансы и научиться основам Beego.</p>

<p>Из этой серии вы узнаете, насколько это чудесный фреймворк, и как он сможет облегчить нам работу. Конкретно, в первой части я раскрою такие темы:</p>

<ul>
    <li>Установка и использование консольной утилиты</li>
    <li>Создание проекта</li>
    <li>Экшены(Действия, Actions)</li>
    <li>Представления и Шаблоны(Views)</li>
    <li>Роутинг</li>
    <li>Параметры запроса</li>
</ul>

<p>Если вы хотите почитать полный код всех примеров из этих статей, то <a href="https://github.com/settermjd/Learning-Beego">он доступен на GitHub</a>. Вас никто не ограничивает в копировании и экспериментировании с этим кодом. Let&rsquo;s Go!</p>

<p>Перед тем, как мы начнем, убедитесь что у вас правильно настроено Go окружение. Если это не так или вы не уверены, посмотрите статьи &quot;<a href="http://www.sitepoint.com/getting-started-go/">Getting Started with Go</a>&quot; или <a href="http://www.goinggo.net/2013/06/installing-go-gocode-gdb-and-liteide.html">этот пост от Bill Kennedy</a>, после этого мы сможем продолжить.</p>

<h3>Установка Beego</h3>

<p>Окей, давайте начнем с установки Beego. Как и многие другие фреймворки, beego имеет встроенные инструменты для скафолдинга, к которым можно добраться используя консольную команду <code>bee</code>. Эта команда умеет:</p>

<ul>
    <li>Создание нового приложения</li>
    <li>Запуск приложения</li>
    <li>Тестирование приложения</li>
    <li>Создание роутов и многое другое</li>
</ul>

<p>Bee это не единственный путь для запуска приложения, но мы будем использовать только его в этих двух статьях. Для установки запустите команду <code>go get github.com/beego/bee</code></p>

<h3>Создание основного проекта</h3>

<p>После установки выполните команду из <code>$GOPATH</code>, которая создаст скелет приложения с названием sitepointgoapp:</p>

<pre>
<code>
bee new sitepointgoapp
</code></pre>

<p>На экране должно быть что-то похожее на это:</p>

<pre>
<code>
[INFO] Creating application...
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/conf/
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/controllers/
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/models/
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/routers/
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/tests/
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/static/
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/static/js/
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/static/css/
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/static/img/
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/views/
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/conf/app.conf
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/controllers/default.go
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/views/index.tpl
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/routers/router.go
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/tests/default_test.go
/Users/matthewsetter/Documents/workspace/Golang/src/sitepointgoapp/main.go
14-05-14 06:02:59 [SUCC] New application successfully created!
</code></pre>

<p>Можете посмотреть на созданную структуру проекта</p>

<pre>
<code>
sitepointgoapp
├── conf
│   └── app.conf
├── controllers
│   └── default.go
├── main.go
├── models
├── routers
│   └── router.go
├── static
│   ├── css
│   ├── img
│   └── js
├── tests
│   └── default_test.go
└── views
    └── index.tpl
</code></pre>

<p>Давайте посмотрим, какие файлы у нас получились</p>

<ul>
    <li>Наш стартовый файл <code>main.go</code></li>
    <li>Основной конфигурационный файл <code>conf/app.conf</code></li>
    <li>Дефольный контроллер <code>controllers/default.go</code></li>
    <li>Тесты к нашему контролеру <code>tests/default_test.go</code></li>
    <li>Дефолтный темплейт <code>views/index.tpl</code></li>
</ul>

<p>Базовое приложение готово. Давайте его запустим. Из директории проекта <code>$GOPATH/src/sitepointgoapp/</code> выполним следующую команду:</p>

<pre>
<code>
bee run
</code></pre>

<p>Теперь, наше приложение запушено. Одно из преимуществ bee - это мониторинг изменения файлов. Если есть какие-нибудь изменения, bee перезагружает приложение автоматически. После выполнения команды, вы должны увидеть примерно такой вывод.</p>

<pre>
<code>
14-05-05 11:34:17 [INFO] Start building...
14-05-05 11:34:20 [SUCC] Build was successful
14-05-05 11:34:20 [INFO] Restarting sitepointgoapp ...
14-05-05 11:34:20 [INFO] ./sitepointgoapp is running...
2014/05/05 11:34:20 [I] Running on :8080
</code></pre>

<p>Как видно, приложение запущенно на 8080 порту. Перейдем по адресу <code>http://localhost:8080/</code> в браузере и увидим такую страничку:</p>

<p><img alt="" src="https://dl.dropboxusercontent.com/u/750049/4gophers.com/1400568288welcome-to-beego.png" style="height:530px; width:750px" /></p>

<p>Ничего вызывающего, но это работает. И так, засучим рукава и расширим возможности дефолтного контроллера, добавив новый экшен с каким-нибудь кастомным роутом.</p>

<h3>Добавление нового экшена</h3>

<p>Откройте файл <code>controllers/default.go</code> вы увидите хорошо структурированный контроллер. Все, что связано с рендерингом убрано в шаблоны представления. Давайте немного изменим код и посмотрим, как добавлять новые переменные в шаблоны и работать с этими шаблонами. В <code>default.go</code> добавим еще одни метод:</p>

<pre>
<code>
func (main *MainController) HelloSitepoint() {
    main.Data[&quot;Website&quot;] = &quot;My Website&quot;
    main.Data[&quot;Email&quot;] = &quot;your.email.address@example.com&quot;
    main.Data[&quot;EmailName&quot;] = &quot;Your Name&quot;
    main.TplNames = &quot;default/hello-sitepoint.tpl&quot;
}
</code></pre>

<p>Давайте разберем, что это все обозначает. Мы добавили новый <code>GET</code> метод (экшен) в контроллере указав <code>main *MainController</code>.</p>

<p>У нас есть три инициализированных переменных представления <code>Website</code>, <code>Email</code> и <code>EmailName</code>, записанные в поле контроллера <code>Data</code>, которое является мапом и хранит все переменные представления.</p>

<p>После этого, я указал имя шаблона <code>default/hello-sitepoint.tpl</code> в поле <code>main.TplNames</code>. Нужно заметить, что по умолчанию Beego ищет файлы шаблонов в папке <code>views</code></p>

<p>И так, когда будет запрошен наш роут, Beego найдет указанный файл шаблона и отрендерит его.</p>

<p>Представления</p>

<p>Давайте создадим соответствующий шаблон представления. В директории <code>views</code> создайте подкаталог <code>default</code> и в нем файл <code>hello-sitepoint.tpl</code> с таким кодом:</p>

<pre>
<code>
&lt;header class=&quot;hero-unit&quot;&gt;
    &lt;div class=&quot;container&quot;&gt;
        &lt;div class=&quot;row&quot;&gt;
            &lt;div class=&quot;hero-text&quot;&gt;
                &lt;h1&gt;Welcome to the Sitepoint / Beego App!&lt;/h1&gt;
                &lt;h2&gt;This is My Test Version&lt;/h2&gt;
                &lt;p&gt;{{.Website}} {{.Email}} {{.EmailName}}&lt;/p&gt;
            &lt;/div&gt;
        &lt;/div&gt;
    &lt;/div&gt;
&lt;/header&gt;
</code></pre>

<p>Если это ваше первое знакомство с шаблонами в Go, имейте в виду, что Beego расширяет стандартный <a href="http://golang.org/pkg/html/template/">пакет html/template</a>. Можете посмотреть <a href="http://golang.org/pkg/text/template/#hdr-Arguments">хорошие примеры</a> использования переменных в шаблонах из пакета text/template.</p>

<p>Как и многие другие Go пакеты html/template достаточно обширен, поэтому я расскажу только про интересующие нас возможности. Все переменные в шаблонах доступны при использовании оператора точки и фигурных скобок <code>{{}}</code></p>

<p>Значит, чтобы добраться до наших переменных, установленных ранее в контроллере, нам нужно использовать конструкции <code>{{.Website}}</code>, <code>{{.Email}}</code>, и <code>{{.EmailName}}</code></p>

<p>Роутинг</p>

<p>Окей, у нас есть экшн и привязанный к нему шаблон представления. Но мы все еще не можем зайти на страничку. Если мы попробуем добраться до чего-угодно, кроме дефолтного экшена, то увидим 404 ошибку.</p>

<p><img alt="" src="https://dl.dropboxusercontent.com/u/750049/4gophers.com/1400568282beego-404.png" style="height:559px; width:750px" /></p>

<p>Это значит, что нам нужно добавить роут в файл <code>routers/router.go</code>. Добавляем код в метод <code>init()</code></p>

<pre>
<code>
func init() {
    beego.Router(&quot;/&quot;, &amp;controllers.MainController{})
    beego.Router(&quot;/hello-world&quot;, &amp;controllers.MainController{}, &quot;get:HelloSitepoint&quot;)
}
</code></pre>

<p>Обратите внимание на последнюю строчку этого файла. <code>HelloSitepoint</code> экшен, который мы только что добавили контроллеру <code>MainController</code>, привязывается к /hello-world. Сохраняем все это, ждем минуту, пока проект перекомпилируется, и открываем <code>http://localhost:8080/hello-world</code> в браузеер. Если все правильно сделано, то страничка должна быть такой:</p>

<p><img alt="" src="https://dl.dropboxusercontent.com/u/750049/4gophers.com/1400568285beego-hello-world.png" style="height:559px; width:750px" /></p>

<p>Параметры запроса</p>

<p>К этому моменту у нас есть готовый простой экшен. Но в реальном мире приложение должно взаимодействовать с пользователем данных из POST и/или строки запроса(GET) и возвращать соответствующий результат. Как нам получить доступ к этой информации в Beego?</p>

<p>Давайте начнем с строки запроса. Одно из предустановленных значений в Beego это <a href="http://beego.me/docs/module/context.md">модуль контекста(Context Module)</a>, который инкапсулирует запрос содержащий входящие значения.</p>

<p>Этот модуль дает нам доступ к таким свойствам запроса:</p>

<ul>
    <li>Метод/hello-world/213</li>
    <li>Протокол</li>
    <li>Юзер агент</li>
    <li>Запрос(данные из GET/POST)</li>
    <li>Сессии и многое другое</li>
</ul>

<p>Давайте сделаем наше приложение более интересным и добавим в роут id параметр, которые мы сможем получить в нашем экшене. Для этого в файле <code>router.go</code> нужно изменить наш код.</p>

<pre>
<code>
beego.Router(&quot;/hello-world/:id([0-9]+)&quot;, &amp;controllers.MainController{}, &quot;get:HelloSitepoint&quot;)
</code></pre>

<p>При обращении по этому роуту мы должны передать GET значание, которое может быть только числом, так как в определении роута используется регулярное выражение <code>([0-9]+)</code>. Попробуйте перейти по адресу <code>/hello-world</code> без указания id. Видите ошибку?</p>

<p>А если перейти по <code>/hello-world/213</code>, то наша страница отобразится правильно, без всяких ошибок. Теперь можем получить id в нашем экшене. Для этого, перед <code>main.TplNames</code> добавим:</p>

<pre>
<code>
main.Data[&quot;Id&quot;] = main.Ctx.Input.Param(&quot;:id&quot;)
</code></pre>

<p>И в шаблоне <code>hello-world.tpl</code> добавим <code>{{.Id}}</code> после уже существующих переменных. Обновим страницу и увидим значение <em>213</em> после Email Name.</p>

<p>Разделение экшенов по методу запроса</p>

<p>Мы почти закончили на сегодня, но я хочу раскрыть одну вещь, прежде чем закончить. Довольно часто бывает необходимым ограничить доступ к действию в зависимости от метода запроса. Например, логично, чтобы экшен добавления сущности работал только при POST запросе, а экшен получения списка сущностей только при GET запросе.</p>

<p>Beego позволяет очень просто разделять доступы для экшенов в зависимости от метода запроса. В файле с роутами <code>router.go</code> снова измените роут <code>/hello-world</code> как показано ниже:</p>

<pre>
<code>
beego.Router(&quot;/hello-world/:id([0-9]+)&quot;, &amp;controllers.MainController{}, &quot;get,post:HelloSitepoint&quot;)
</code></pre>

<p>Строчка <code>get,post:HelloSitepoint</code> означает, что экшен <code>HelloSitepoint</code> будет отрабатывать только при GET или POST запросе. Давайте попробуем обратиться по <code>/hello-world/</code> используя DELETE или PUT.</p>

<pre>
<code>
# PUT request
curl -X PUT http://localhost:8080/hello-world/213

# DELETE request
curl -X DELETE http://localhost:8080/hello-world/213
</code></pre>

<h3>Заключение</h3>

<p>Надеюсь, вам понравилось это введение в разработку на Beego фреймворке. Во второй части мы рассмотрим использование баз данных(SQLite3), модели, формы и валидацию.</p>

<p>К концу второй части у нас будет готово приложение со всеми основными функциями, которые будут использоваться изо дня в день.</p>

<p>Несмотря на то, что Beego и Go сильно отличаются от динамических языков программирования, которые вы использовали ранее, потратив немного времени, вы сможете использовать всю мощь фреймворка.</p>

<p>Не забывайте, весь код <a href="https://github.com/settermjd/Learning-Beego">доступен на GitHub</a>. Можете склонировать репозиторий и экспериментировать с ним.</p>

<p>Что вы думаете об этом всем? Достаточно ли этого материала для старта использования Go, или еще нет? Поделитесь своими мыслями в комментариях.</p>

<h3>Ссылки</h3>

<ul>
    <li><a href="http://www.goinggo.net/2013/06/installing-go-gocode-gdb-and-liteide.html">http://www.goinggo.net/2013/06/installing-go-gocode-gdb-and-liteide.html</a></li>
    <li><a href="https://groups.google.com/forum/#!topic/golang-nuts/hbNCHMIA05g">https://groups.google.com/forum/#!topic/golang-nuts/hbNCHMIA05g</a></li>
    <li><a href="http://stackoverflow.com/questions/10105935/how-to-convert-a-int-value-to-string-in-go">http://stackoverflow.com/questions/10105935/how-to-convert-a-int-value-to-string-in-go</a></li>
    <li><a href="http://beego.me/">http://beego.me/</a></li>
    <li><a href="https://github.com/astaxie/beego">https://github.com/astaxie/beego</a></li>
    <li><a href="http://golang.org/pkg/text/template/">http://golang.org/pkg/text/template/</a></li>
    <li><a href="http://golang.org/doc/effective_go.html">http://golang.org/doc/effective_go.html</a></li>
    <li><a href="http://blog.golang.org/go-maps-in-action">http://blog.golang.org/go-maps-in-action</a></li>
</ul>

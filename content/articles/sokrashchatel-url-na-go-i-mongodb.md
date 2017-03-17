+++
date = "2015-09-28T14:59:02+03:00"
draft = false
title = "Сокращатель URL на Go и Mongodb"

+++

<p>Перевод статьи "<a href="http://www.minaandrawos.com/2015/09/05/link-shortener-golang-web-service-tutorial-mongodb/">URL Shortener Golang Web Service tutorial with Mongodb</a>"</p>

<p>Веб технологии, на сегодняшний день, это сердце ИТ индустрии. И одни из самых современных подходов при создании веб сервисов - это реализация <a href="https://en.wikipedia.org/wiki/Representational_state_transfer">HTTP RESTful API</a>, что позволяет проще интегрироваться с другими сервисами, при этом иметь широкий простор для расширения. В тоже время, все большую популярность приобретают различные NoSQL решения, которые завоевывают все большие доли рынка и становятся выбором профессионалов. В свою очередь, Go замечательный современный язык, который позволяет объединить эти технологии в одно целое. Из этой статьи вы узнаете, как с помощью Go и <a href="https://en.wikipedia.org/wiki/NoSQL#Document_store">документо-ориентированной</a> NoSQL базы данных <a href="https://www.mongodb.org/">MongoDB</a> создать веб-сервис. Этот веб-сервис будет сокращать страшные и длинные URL, делая их милыми и симпатичными(аналогично http://tinyurl.com).</p>

<p>Весь код можно найти на <a href="https://github.com/minaandrawos/GoURLLinkShortnerAPI">GitHub</a>. Это небольшое оглавление нашего туториала:</p>

<ul>
<li>Go и HTTP REST</li>
<li>Go и Mongodb</li>
<li>Дизайн сервиса для сокращения ссылок</li>
<li>Добавляем REST слой</li>
<li>Добавляем слой данных</li>
</ul>

<h3>Go и HTTP REST</h3>

<p>Go из коробки имеет достаточно средств, с помощью которых можно создать мощные веб-сервисы, тем не менее, значительно продуктивней использовать пакет <a href="http://www.gorillatoolkit.org/">Gorilla</a>, который предоставляет тонкую надстройку над нативными пакетами Go. Применение Gorilla делает написание сервисов более комфортным и менее затратным по времени. В этой статья мы будем использовать компонент <a href="http://www.gorillatoolkit.org/pkg/mux">Gorillсa Mux</a> для создания нашего сервиса, если вы хотите узнать как делать веб-сервисы с использованием только стандартных пакетов, то вам поможет это <a href="https://golang.org/doc/articles/wiki/">статья</a>. Конечно, существует множество средств для написания веб-приложений и сервисов, например <a href="https://github.com/go-martini/martini">Martini</a> и <a href="http://revel.github.io/">Revel</a>.</p>

<h3>Go и Mongodb</h3>

<p>Mongodb это популярная документо-ориентированная NoSQL база данных. А это означает отсутствие кучи таблиц и связей между ними, как в обычных SQL базах данных. Данные в Mongo хранятся в "коллекциях(collections)" и "документах(documents)". Документ это аналог строки в SQL базе данных, он содержит некую информацию. Коллекция это собрание множества документов, аналогично таблице в SQL базе. Несколько коллекций объединяются в "базу данных(databse)", по аналогии с SQL базами данных.</p>

<p>Из-за своей практичности NoSQL базы данных набирают все большую популярность. Они очень хорошо подходят для тех случаев, когда вам нужно сохранить очень много данных без сложных реляционных отношений между ними или когда вам приходится работать с ветвистыми структурами данных.</p>

<p>Больше информации по коллекциям и документам в Mongodb можно узнать из <a href="http://docs.mongodb.org/manual/reference/glossary/#term-collection">документации</a>. Также, неплохо было бы разобраться с основными <a href="http://docs.mongodb.org/manual/core/data-modeling-introduction/">концепциями</a>. Для работы с Mongo в Go мы будем использовать замечательный пакет <a href="https://labix.org/mgo">mgo</a>, который позволяет использовать всю мощь Mongodb в наших приложениях.</p>

<h3>Проектироавание сервиса для сокращения ссылок</h3>

<p>Пришло время определиться с нашем API. Давайте начнем разработку с проектироавание, как это делается при разработку любых качественных приложений.</p>

<p>Я рассчитываю, что наше API будет работать так:</p>

<ul>
<li>Go сервер стартует и обрабатывает HTTP запросы.</li>
<li>Клиент отправляет POST запрос на URL <code>&lt;http path&gt;/create</code>. В теле этого запроса JSON вида:</li>
</ul>

<pre><code class="json">{ 
    shorturl: "myshorturl", 
    longurl: "http://path/to/a/long/url!"
}
</code></pre>

<ul>
<li>Веб-сервис создает сокращенный URL сохраняет мапинг между коротким и полным URL'ами в Mongodb документе.</li>
<li>Клиент может оправить GET запрос на URL вида <code>&lt;http path&gt;/shorturl</code>.</li>
<li>Если такой сокращенный URL существует в базе, то веб-сервер делает редирект на полный URL, указанный в том же документе.</li>
<li>Если такого короткого URL не существует, то возвращается сообщение об ошибке.</li>
</ul>

<p>Звучит неплохо. Теперь давайте разберемся с тем, какие компоненты нам понадобятся для реализации нашего API. Для реализации базового функционала нам необходимы:</p>

<ul>
<li>Веб-сервер, который будет обрабатывать HTTP запросы (REST слой).</li>
<li>Слой для работы с базой данных.</li>
</ul>

<p><img src="http://i2.wp.com/www.minaandrawos.com/wp-content/uploads/2015/09/Architecture.jpg?resize=604%2C521" alt="Golang web service Design" /></p>

<h3>Добавляем REST слой</h3>

<p>Для создания правильного веб-сервиса с использованием Gorilla toolkit нам нужно:</p>

<ul>
<li>Добавить <em>роуты</em>(routes), которые описывают какие URL'ы поддерживает наш веб-сервис.</li>
<li>Добавить <em>функции обработчики</em>(handler functions) которые определяют, какие действия будет выполнять наше API в зависимости от URL.</li>
<li>Создать роутер для обработки входящих запросов. При инициализации роутера мы указываем роуты и обработчики.</li>
<li>Запустить прослушивание входящего трафика и обработку запросов с помощью роутера.</li>
</ul>

<p>Теперь более подробно разберем все эти моменты.</p>

<h4>Функции обработчики</h4>

<p>Как правило, в Go для обработки запросов используются функции типа <a href="http://golang.org/pkg/net/http/#HandlerFunc"><code>http.HandlerFunc</code></a>. Проще говоря, если ваща функция соответствует указанной сигнатуре, то вы можете использовать эту функцию для реализации логики обработки HTTP запроса:</p>

<pre><code class="go">func myHandlerFunction(w http.ResponseWriter, r *http.Request) {
    // читаем из `r` и пишем в `w`!!    
}
</code></pre>

<p>Простой пример функции обработчика, которая срабатывает, когда пользователь переходит на рутовый URL веб-сервиса</p>

<pre><code class="go">func UrlRoot(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Hello and welcome to the Go link shortner API \n"+
        "Do a Get request with the short Link to get the long Link \n"+
        "Do a POST request with long Link to get a short Link \n")
}
</code></pre>

<p>Функция обработчик имеет два параметра:</p>

<ul>
<li><code>http.ResponseWriter</code> - используется для отправки ответа на HTTP запрос.</li>
<li><code>*http.Request</code> - используется для доступа к информации переданной в запросе.</li>
</ul>

<h4>Каким образом я могу привязать функцию обработчик к определенному URL?</h4>

<p>Небольшое введение в понятие роутов и роутеров. Для начала вам необходимо создать список роутов, которые задают отношения между URL и функциями обработчиками. Затем необходимо скормить получившуюся таблицу роутеру, который сам позаботиться об остальном.</p>

<pre><code class="go">type Route struct {
    Name        string
    Method      string
    Pattern     string
    HandlerFunc http.HandlerFunc
}
type Routes []Route
/*
Создаеи роуты для API. API поддерживает такие URL:
    1. GET "/" =&gt; Показываем описание API
    2. GET "/{shorturl}" =&gt; Если сокращенный URL есть в базе, то выполняем
    редиеркт на полный URL
    3. POST "/Create" =&gt; Получаем пост запрос, в теле которого
    указан JSON вида: 
        {
            shorturl: "short Link"
            longurl:  "original long link"
        }
    И записываем в базу отношение между сокращенным и полным URL
*/
func CreateRoutes()  Routes {
    return Routes{
        Route{
            "UrlRoot",
            "GET",
            "/",
            UrlRoot,
        },
        Route{
            "UrlShow",
            "GET",
            "/{shorturl}",
            UrlShow,
        },
        Route{
            "UrlCreate",
            "POST",
            "/Create",
            UrlCreate,
        },
    }
}
</code></pre>

<p>Вероятно, вас смущает запись <code>{shorturl}</code>. Фигурные скобки необходимы, чтобы указать роутеру, что тут мы ждем переменную. В случае <code>UrlShow</code> в этой переменной будет указан сокращенный URL.</p>

<p>Вы можете добраться к этой переменной из функции обработчика с помощью <code>mux.Vars["variable"]</code>. Пример такой функции обработчика выглядит так:</p>

<pre><code class="go">func (Ls *LinkShortnerAPI) UrlShow(w http.ResponseWriter, r *http.Request) {
    // получаем переменные из запроса
    vars := mux.Vars(r)
    sUrl := vars["shorturl"]
    if len(sUrl) &gt; 0 {
        // Ls.myconnection это указатель на наш слой работы с базой
        // находим полный URL который соответствует сокращенному URL
        lUrl, err := Ls.myconnection.FindlongUrl(sUrl)
        if err != nil {
            fmt.Fprintf(w, "Could not find saved long url that corresponds to the short url %s \n", sUrl)
            return
        }
        // необходимо убедится что у нас абсолютный путь
        // выполняем редирект
        http.Redirect(w, r, lUrl, http.StatusFound)
    }
}
</code></pre>

<p>Теперь скормим все эти роуты самому роутеру:</p>

<pre><code class="go">func NewLinkShortenerRouter(routes Routes) *mux.Router {
    // если  StrictSlash установлен а true, то при роуте "/path/"
    // автоматически будет редиректить на "/path".
    router := mux.NewRouter().StrictSlash(true)
    // указываем всю необходимую информацию для правильной
    // работы роутера
    for _, route := range routes {
        router.
            Methods(route.Method).
            Path(route.Pattern).
            Name(route.Name).
            Handler(route.HandlerFunc)
    }
    return router
}
</code></pre>

<p><code>mux</code> это название <a href="http://www.gorillatoolkit.org/pkg/mux#overview">пакета Gorilla</a> из которого мы используем роутер.</p>

<h4>Парсинг JSON</h4>

<p>Теперь у нас есть роутеры привязанные к нашим функциям обработчикам. Самое время рассказать, как обрабатывать POST запросы. Также как и любой другой REST API, Go веб-сервис получает POST запрос, читает JSON из тела этого запроса и парсит его в необходимые структуры. В Go есть пакет <a href="http://golang.org/pkg/encoding/json/"><code>encoding/json</code></a>, который используется для кодирования и декодирования JSON. Это работает вот так:</p>

<ul>
<li>Вы создаете структуру и указывается какие поля будут соответствовать полям JSON. Для этого используются специальные теги:</li>
</ul>

<pre><code class="go">type UrlMapping struct {
    ShortUrl string `json:shorturl`
    LongUrl  string `json:longurl`
}
</code></pre>

<ul>
<li>Если вы хотите быстро распарсить JSON данные из HTTP запроса в структуру, то нужно подготовить декодер. Этот декодер работает с ридером(параметр <code>Body</code>  у реквеста) и указателем на структуру, в которую будут записываться данные из JSON. Для нашего сервиса этот код будет выглядеть так:</li>
</ul>

<pre><code class="go">func (Ls *LinkShortnerAPI)UrlCreate(w http.ResponseWriter, r *http.Request) {
    // создаем указатель на структуру UrlMapping
    // urlMapping мапится на JSON
    reqBodyStruct := new(UrlMapping)
    // создаем новый декодер, использующий ридер r.Body
    // и читаем данные в структуру
    if err := json.NewDecoder(r.Body).Decode(&amp;reqBodyStruct); err != nil {
        w.WriteHeader(http.StatusBadRequest)
        return
    }
    // используем слой базы данных для добавления URL
    Ls.myconnection.AddUrls(reqBodyStruct.LongUrl, reqBodyStruct.ShortUrl)
    return
}
</code></pre>

<ul>
<li><p>Если вам нужно сгенерировать JSON на основании имеющейся структуры и отдать его как ответ, то вам нужно создать новый енкодер, который будет использовать <code>http.ResponseWriter</code> и вашу структуру с данными. Например, для нашего сервиса нужно иметь возможность отправлять статусные сообщения после получения POST запроса.</p>

<ul>
<li>Структура, которая представляет наше сообщение выглядит так:</li>
</ul>

<pre><code class="go">type APIResponse struct {
    StatusMessage string `json:statusmessage`
}
</code></pre>

<ul>
<li>Создаем новый енкодер:</li>
</ul>

<pre><code class="go">// w это http.ResponseWriter
responseEncoder := json.NewEncoder(w)
// LS.myconnection указатель на слой работы с базой
err := Ls.myconnection.AddUrls(reqBodyStruct.LongUrl, reqBodyStruct.ShortUrl)
if err != nil {
    w.WriteHeader(http.StatusConflict)
    if err := responseEncoder.Encode(&amp;APIResponse{StatusMessage: err.Error()}); err != nil {
        fmt.Fprintf(w, "Error %s occured while trying to add the url \n", err.Error())
    }
    return
}
responseEncoder.Encode(&amp;APIResponse{StatusMessage: "Ok"})
</code></pre></li>
<li><p>Как правило, у вас в одном обработчике будет использоваться декодер и енкодер, так как вам необходимо прочитать данные из запроса и отправить какие-то результаты клиенту. Поэтому финальная версия функции, которая сохраняет отношение между сокращенным и полным URL, будет выглядеть вот так:</p></li>
</ul>

<pre><code class="go">func (Ls *LinkShortnerAPI) UrlCreate(w http.ResponseWriter, r *http.Request) {
    reqBodyStruct := new(UrlMapping)
    responseEncoder := json.NewEncoder(w)
    if err := json.NewDecoder(r.Body).Decode(&amp;reqBodyStruct); err != nil {
        w.WriteHeader(http.StatusBadRequest)
        if err := responseEncoder.Encode(&amp;APIResponse{StatusMessage: err.Error()}); err != nil {
            fmt.Fprintf(w, "Error occured while processing post request %v \n", err.Error())
        }
        return
    }
    err := Ls.myconnection.AddUrls(reqBodyStruct.LongUrl, reqBodyStruct.ShortUrl)
    if err != nil {
        w.WriteHeader(http.StatusConflict)
        if err := responseEncoder.Encode(&amp;APIResponse{StatusMessage: err.Error()}); err != nil {
            fmt.Fprintf(w, "Error %s occured while trying to add the url \n", err.Error())
        }
        return
    }
    responseEncoder.Encode(&amp;APIResponse{StatusMessage: "Ok"})
}
</code></pre>

<h4>На этом мы закончили с сервером?</h4>

<p>Нет, остался еще один маленький шаг для того, чтобы наш сервер начал принимать данные. Мы настраиваем наш сервер так, чтоб работал по определенному адресу, который будет корневым для нашего сервиса. В Go это делается с помощью функции <a href="http://golang.org/pkg/net/http/#ListenAndServe"><code>http.ListenAndServe()</code></a>. Мы будем использовать порт 5100:</p>

<pre><code class="go">// запускаем сервис на локальном порту 5100
http.ListenAndServe(":5100", router)
</code></pre>

<p>На этом мы заканчиваем с нашим веб-сервером и переходим к слою работы с базой данных.</p>

<h3>Слой для работы с базой данных</h3>

<p>Теперь поговорим о базе данных. В хорошо спроектированных приложениях, слой работы с базой данных это часть программы, которая отвечает за любое взаимодействие с базой данных, сохранение или получение данных. Такой подход обеспечивает разделение логики работы с веб-сервером и логику работы с данными, что уменьшает количество кода, улучшает его структурированность, расширяемость, упрощает командную работу.</p>

<p>Я люблю Mongodb за ее простоту, мощь и большое сообщество, которое может всегда помочь. Она бесплатна, исходники открыты и я могу быстро починить проблемы возникающие в приложении. Mongodb для хранения данных в документах использует бинарную разновидность JSON, которая называется <a href="http://bsonspec.org/">BSON</a>, а это означает, что данными в Mongodb можно манипулировать также просто как и обычным JSON. Для работы с Mongo в Go я использую пакет <code>mgo</code>.</p>

<h4>Как использовать mgo</h4>

<p>mgo использует концепцию сессий для соединения с базой данных. Под сессией понимается соединение из пула сокетов. Для получения рабочей сессии необходимо:</p>

<ul>
<li>Соединиться с базой данных, аналогично tcp соединению в Go, и получить от mgo новую сессию.</li>
<li>Вы используете сессию для выполнения запросов к одной баз данных  в Mongo.</li>
<li>Из базы данных вы извлекаете коллекцию.</li>
<li>Теперь вы можете выполнять различные операции(чтения, записи) над документами в этой коллекции.</li>
</ul>

<p>В наш код мы добавим <a href="https://gobyexample.com/structs">структуру</a>, которая будет играть роль модели в слое данных и будет использоваться в нашем веб-сервисе. В Go структуры лучше всего использовать в те моменты, когда в других языках вам понадобились бы классы. Наша структура будет хранить экземпляр сессии для удобного использования этой сессии в других кусках кода</p>

<pre><code class="go">type MongoConnection struct {
    originalSession *mgo.Session
}
</code></pre>

<h4>Уникальные индексы</h4>

<p>В нашем API не должно быть ситуации, когда у нас сохраняется несколько совершенно одинаковых сокращенных URL. Это сделает мапинг более простым и у нас не будет возникать коллизий, когда несколько пользователей захотят создать одинаковые сокращенные URL. Мы можем обеспечить это добавив уникальный индекс для поля shorturl.</p>

<pre><code class="go">index := mgo.Index{
    Key:      []string{"$text:shorturl"},
    Unique:   true,
    DropDups: true,
}

urlcollection.EnsureIndex(index)
</code></pre>

<p>Теперь нам необходимо добавить функцию, которая будет создавать соединение с базой для нашего веб-сервиса</p>

<pre><code class="go">func (c *MongoConnection) createLocalConnection() (err error) {
    fmt.Println("Connecting to local mongo server....")
    c.originalSession, err = mgo.Dial(CONNECTIONSTRING)
    if err == nil {
        fmt.Println("Connection established to mongo server")
        urlcollection := c.originalSession.DB("LinkShortnerDB").C("UrlCollection")
        if urlcollection == nil {
            err = errors.New("Collection could not be created, maybe need to create it manually")
        }
        // этот код нужен для добавления уникального индексаdatabase.
        index := mgo.Index{
            Key:      []string{"$text:shorturl"},
            Unique:   true,
            DropDups: true,
        }
        urlcollection.EnsureIndex(index)
    } else {
        fmt.Printf("Error occured while creating mongodb connection: %s", err.Error())
    }
    return
}
</code></pre>

<h4>Несколько конкурентных сессий</h4>

<p>Теперь у нас есть соединение с базой, но нам нужно предусмотреть что будет при очень нагруженном использовании нашего сервиса. Предположим, что вы создали очень популярный сервис сокращения ссылок и все ломанулись к вам, теперь у вас куча посетителей и тонны запросов. В этот момент вы поймете что использование одной сессии для большого количества запросов это не самый лучший подход. Что же делать?</p>

<p>mgo позволяет создавать несколько конкурентных соединений основываясь на    пуле соектов. Вы можете получить новую сессию из текущей просто вызвав <code>originalSession.Copy()</code>. Этот вызов создаст новую параллельную сессию с аутентификацией из оригинальной сессии и вы сможете использовать ее сразу же. После того как вы закроете сессию, сокет вернется в пул.</p>

<pre><code class="go">func (c *MongoConnection) getSessionAndCollection() (session *mgo.Session,
                                                    urlCollection *mgo.Collection, err error) {
    if c.originalSession != nil {
        session = c.originalSession.Copy()
        urlCollection = session.DB("LinkShortnerDB").C("UrlCollection")
    } else {
        err = errors.New("No original session found")
    }
    return
}
</code></pre>

<h4>Чтение и запись из Mongodb</h4>

<p>Для работы с Mongodb и mgo нам необходимо создать структуру, которая будет представлять собой документ. Эта структура должна соответствовать документу в коллекции. Мы будем использовать <a href="http://stackoverflow.com/questions/10858787/what-are-the-uses-for-tags-in-go">теги для структур</a> что бы точно указать какие поля структуры мапить в документ. Это выглядит вот так:</p>

<pre><code class="go">type mongoDocument struct {
    Id       bson.ObjectId `bson:"_id"`
    ShortUrl string        `bson:"shorturl"`
    LongUrl  string        `bson:"longurl"`
}
</code></pre>

<p>Теперь мы можем использовать эту структуру для запросов в базу и для чтения и записи. Для пример, что бы найти сокращенный URL "blah" в Mongodb, нужно выполнить такой запрос:</p>

<pre><code class="js">db.collection.find({shorturl:"blah"})
</code></pre>

<p>В случае с Go и mgo этот запрос будет выглядеть так:</p>

<pre><code class="go">result := mongoDocument{}
err = urlCollection.Find(bson.M{"shorturl": shortUrl}).One(&amp;result)
if err != nil {
    return
}
</code></pre>

<p><code>bson.M{}</code> используется для представления сообщения в bson формате. Это обязательно для вызова метода <code>Find()</code> из mgo.</p>

<p>Посмотрим, как функция будет выглядеть целиком:</p>

<pre><code class="go">func (c *MongoConnection) FindlongUrl(shortUrl string) (lUrl string, err error) {
    // создаем новый документ
    result := mongoDocument{}
    // получаем копию оригинальной сессии
    session, urlCollection, err := c.getSessionAndCollection()
    if err != nil {
        return
    }
    defer session.Close()
    // ищем сокращенный url в базе
    err = urlCollection.Find(bson.M{"shorturl": shortUrl}).One(&amp;result)
    if err != nil {
        return
    }
    return result.LongUrl, nil
}
</code></pre>

<p>Теперь для сохранения данных в Mongodb нам нужно использовать функцию вставки. В эту функцию мы передаем структуру, представляющую документ который вы хотите записать. После этого мы проверяем были ли ошибки записи. В зависимости от типа ошибки мы можем или повторить запись, или вернуть это сообщение клиенту. Для проверки связанна ли эта ошибка с дубликатами мы используем метод <code>mgo.IsDup(err)</code>.</p>

<pre><code class="go">func (c *MongoConnection) AddUrls(longUrl string, shortUrl string) (err error) {
    // получаем копию сессии
    session, urlCollection, err := c.getSessionAndCollection()
    if err == nil {
        defer session.Close()
        // добавляем новый аргумент
        err = urlCollection.Insert(
            &amp;mongoDocument{
                Id:       bson.NewObjectId(),
                ShortUrl: shortUrl,
                LongUrl:  longUrl,
            },
        )
        if err != nil {
            // проверяем, была ли это ошибка вызванная дублированием 
            if mgo.IsDup(err) {
                err = errors.New("Duplicate name exists for the shorturl")
            }
        }
    }
    return
}
</code></pre>

<p>На этом мы закончили обсуждение основных частей нашего веб-сервиса. Надеюсь, эта статья вам понравилась :)</p>

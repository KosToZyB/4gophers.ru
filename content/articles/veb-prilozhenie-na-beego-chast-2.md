+++
date = "2014-08-10T17:00:05+03:00"
draft = false
title = "Beego. Часть 2"

+++

<p>Перевод второй части небольшого введения в фреймворк Beego от <a href="http://www.sitepoint.com/author/msetter/">Matthew Setter</a>. <a href="http://www.sitepoint.com/go-building-web-applications-beego-part-2/">Оригинал тут</a>. <a href="http://4gophers.com/article/veb-prilozhenie-s-beego#.U-aJn9R_vQo">Перевод первой части</a>.</p>

<p>Приветствую читалелей 2-й части серии, в которой мы продолжаем изучение веб-фрейморка Beego написанного на Go. Если вы пропустили первую часть, то я рекомендую вернуться к ней, так ка там изложены фундаментальный принципы, которые мы будем использовать.</p>

<p>В первой части у нас получился неплохой старт. Мы начали понимать принципы работы с Beego, установили этот фреймворк и тулзу Bee для работы с ним, создали базовый проект, добавили экшены к контроллеру, создали шаблон представления, добавили кастомный роут и закончили на работе с параметрами запроса</p>

<p>Во второй части мы углубимся в изучение аспектов веб-разработки, научимся пользоваться базой данных, в частности SQLite3, подробно рассмотрим работу с моделями, формами и валидацию данных. Надеюсь, вы готовы начать.</p>

<h3>Вложенные представления</h3>

<p>Наверняка, вы обратили внимание на код в функциях контроллера:</p>

<pre>
<code>
manage.Layout = &quot;basic-layout.tpl&quot;
manage.LayoutSections = make(map[string]string)
manage.LayoutSections[&quot;Header&quot;] = &quot;header.tpl&quot;
manage.LayoutSections[&quot;Footer&quot;] = &quot;footer.tpl&quot;
</code></pre>

<p>Тут мы устанавливаем наше вложенное представление. Если вы не в курсе этого термина, то просто знайте, что это структура, при которой у нас есть внешний layout, в котором определены некоторые базовые и всегда отображаемые элементы, такие как сайдбар, навигация, заголовок, подвал и место куда будет выводиться внутренний контент.</p>

<p><img alt="" src="https://dl.dropboxusercontent.com/u/750049/4gophers.com/screen_2013-12.png" /></p>

<p>Это иллюстрация, того о чем я говорю. Зеленая часть - это непосредственно часть внешнего layout, красная - это изменяемый контент в зависимости от текущего экшена.</p>

<p>Используя параметры <code>Layout</code> и <code>LayoutSections</code> мы можем указать базовое представление(layout) <code>basic-layout.tpl</code> и сабтемплейты, в нашем случае это заголовок(<code>header.tpl</code>) и подвал(<code>footer.tpl</code>).</p>

<p>После того, как мы это сделаем, сгенерированный экшеном контент вставляется в шаблон представления с помощью <code>{{.LayoutContent}}</code>, а заголовок и подвал доступны в переменных <code>{{.Header}}</code> и <code>{{.Footer}}</code> соответственно.</p>

<h3>Модели</h3>

<p>Чтобы добавить возможность работы с базой данных, мы должны сделать несколько вещей. Для начала нам нужно создать несколько моделей. Модели - это просто обычные структуры с некоторой дополнительной информацией. Ниже показан файл <code>models/models.go</code> с моделями, которые можно будет использовать по всему сайту</p>

<pre>
<code>
package models

type Article struct {
    Id     int    `form:&quot;-&quot;`
    Name   string `form:&quot;name,text,name:&quot; valid:&quot;MinSize(5);MaxSize(20)&quot;`
    Client string `form:&quot;client,text,client:&quot;`
    Url    string `form:&quot;url,text,url:&quot;`
}

func (a *Article) TableName() string {
    return &quot;articles&quot;
}
</code></pre>

<p>Как вы можете видеть, тут представлена одна модель - <code>Article</code>. Это простое описание статьи, которое содержит четыре параметра: <code>Id</code>, <code>Name</code>, <code>Client</code> и <code>Url</code>. Заметьте, для каждого поля отдельно указана метаинформация, которая будет использоваться в формах и при валидаци.</p>

<p>Это действительно простой путь, позволяющий использовать модели для генерации форм и валидации этих форм. Давайте рассмотрим каждое из четырех полей и разберем что к чему.</p>

<pre>
<code>
Id int `form:&quot;-&quot;`
</code></pre>

<p>В нашей базе данных id - это автоинкрементное поле. Это прекрасно работает, когда нужно добавить новую запись в табличку, нам не нужно руками указывать id, только при удалении, обновлении или поиске. И так, указываем тег для этого поля как <code>form:&quot;-&quot;</code>, что обозначает необязательность поля Id.</p>

<pre>
<code>
Name   string `form:&quot;name,text,name:&quot; valid:&quot;MinSize(5);MaxSize(20)&quot;`
</code></pre>

<p>Это немного более комплексный пример. Давайте начнем с <code>&quot;name,text,name:&quot;</code>. Вот что это будет обозначать, когда будет строиться форма.</p>

<ul>
    <li>Значения поля с именем name будет инициализированное значением из параметра <code>Name</code></li>
    <li>Это будет текстовым полем</li>
    <li>Лейбл для этого поля будет <code>name</code></li>
</ul>

<p>Теперь посмотрим на <code>valid:&quot;MinSize(5);MaxSize(20)&quot;</code>. Это специальные правила валидации: <code>MinSize</code> и <code>MaxSize</code>. Это означает, что значение поля должно быть не короче 5 символов, но и не длиннее 20.</p>

<p>Тут можно посмотреть <a href="http://beego.me/docs/mvc/controller/validation.md">все правила валидации</a>, которыми можно пользоваться, включая <code>Range</code>, <code>Email</code>, <code>IP</code>, <code>Mobile</code>, <code>Base64</code> и <code>Phone</code>.</p>

<pre>
<code>
Client string `form:&quot;client,text,client:&quot;`
Url    string `form:&quot;url,text,url:&quot;`
</code></pre>

<p>Последние две записи означают, что в параметр <code>Client</code> попадает значение из поля формы <code>client</code>, которое является текстовым инпутом с лейблом &quot;client:&quot;. В параметр <code>Url</code> попадают значения из поля <code>url</code>, которое так же является текстовым инпутом с лейблом <code>url:</code>. Теперь посмотрим на функцию <code>TableName</code></p>

<p>Причина, по которой я добавил эту функцию, кроется в том, что название структуры, в моем случае, не совпадает с именем таблицы. Таблица называется <code>articles</code>. Если бы наша структура называлась так же, тогда <a href="http://beego.me/docs/mvc/model/orm.md">таблица бралась бы автоматически</a>.</p>

<p>Я специально внес такое различие, что бы показать что необходимо сделать в случае различных имен таблицы и структуры. И раз уж мы начали говорить про структуру таблицы, давайте посмотрим на нее внимательней.</p>

<pre>
<code>
CREATE TABLE &quot;articles&quot; (
    &quot;id&quot; integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    &quot;name&quot; varchar(200) NOT NULL,
    &quot;client&quot; varchar(100),
    &quot;url&quot; varchar(400) DEFAULT NULL,
    &quot;notes&quot; text,
    UNIQUE (name)
);
</code></pre>

<h3>Интеграция моделей внутри приложения</h3>

<p>Мы создали и настроили наши модели, добавили информацию для валидации и создания форм. Теперь нам нужно сделать модели доступными в приложении. Мы должны добавить импорт моделей и дополнительных пакетов в <code>main.go</code>.</p>

<pre>
<code>
&quot;github.com/astaxie/beego/orm&quot;
_ &quot;github.com/mattn/go-sqlite3&quot;
models &quot;sitepointgoapp/models&quot;
</code></pre>

<p>Первый импорт - это подключение ORM библиотеки, второй импорт подключает драйвер для SQLite3, который обязателен при использовании этой базы. Третий импорт - это подключение наших моделей.</p>

<pre>
<code>
func init() {
    orm.RegisterDriver(&quot;sqlite&quot;, orm.DR_Sqlite)
    orm.RegisterDataBase(&quot;default&quot;, &quot;sqlite3&quot;, &quot;database/orm_test.db&quot;)
    orm.RegisterModel(new(models.Article))
}
</code></pre>

<p>На финальном шаге мы должны зарегистрировать драйвер, базу данных и наши модели, которые будем использовать в приложении. Мы делаем это в три этапа, показанные выше. Указываем, что мы используем SQLite и устанавливаем его для дефолтного соединения с нашей базой данных, которая находится в файле <code>database/orm_test.db</code>.</p>

<p>И в конце мы регистрируем только одну модель, которую будем использовать <code>models.Article</code>.</p>

<h3>CRUD Операции</h3>

<p>После всех этих манипуляций у нас есть настроенное соединение с базой данных и ее интеграция с нашим приложением. Давайте начнем с двух простых CRUD операций: удаление(D) и обновление(U). Ни одна из этих операций не содержит форму, потому что я максимально упрощаю экшен и концентрируюсь на ORM коде без учена построения формы или валидации. Мы работаем в <code>Add</code> экшене.</p>

<p>Удаление записи</p>

<p>Создадим экшен удаления, в котором будем пытаться удалить некоторую статью из нашей базы данных по параметру <code>id</code>. В файле <code>routers/routers.go</code> добавим еще один роут в init функции.</p>

<pre>
<code>
beego.Router(&quot;/manage/delete/:id([0-9]+)&quot;, &amp;controllers.ManageController{}, &quot;*:Delete&quot;)
</code></pre>

<p>Теперь добавим новый экшен <code>Delete</code> в файл controllers/manage.go`.</p>

<pre>
<code>
func (manage *ManageController) Delete() {
    // convert the string value to an int
    articleId, _ := strconv.Atoi(manage.Ctx.Input.Param(&quot;:id&quot;))
</code></pre>

<p>Здесь мы получаем параметр <code>id</code> и конвертируем его в тип int из типа string используя для этого функцию <code>Atoi</code> из пакета <code>strconv</code>. Это очень простой пример, в котором пропущена обработка ошибок и сразу сохраняем значение параметра в переменной <code>articleId</code>.</p>

<pre>
<code>
o := orm.NewOrm()
o.Using(&quot;default&quot;)
article := models.Article{}
</code></pre>

<p>Далее мы инициализируем новый ORM инстанс и указываем, что будем использовать дефолтную базу данных. Мы можем указать какое угодно количество подключений, например одно для записи, а другое для чтения. И создаем новый пустой экземпляр модели <code>Article</code>.</p>

<pre>
<code>
    // Check if the article exists first
    if exist := o.QueryTable(article.TableName()).Filter(&quot;Id&quot;, articleId).Exist(); exist {
        if num, err := o.Delete(&amp;models.Article{Id: articleId}); err == nil {
            beego.Info(&quot;Record Deleted. &quot;, num)
        } else {
            beego.Error(&quot;Record couldn&#39;t be deleted. Reason: &quot;, err)
        }
    } else {
        beego.Info(&quot;Record Doesn&#39;t exist.&quot;)
    }
}
</code></pre>

<p>Это сердце функции. Для начала, делаем запрос, проверяем существует ли статья, с указанным <code>Id</code>. Если статья существует, мы вызываем метод <code>Delete</code>, передаем ему объект нашей статьи только с одним указанным полем <code>Id</code>.</p>

<p>Если никаких ошибок не произошло, то статья удаляется и вызывается функция <code>beego.Info</code>, которая <a href="http://beego.me/docs/mvc/controller/logs.md">записывает в лог</a>, что статья удалена. Если мы не можем удалить статью, то вызываем функцию <code>Error</code>, которая, кроме всего прочего, принимает в качестве параметра объет ошибки <code>err</code>, в котором указано что именно пошло не так и почему статья не удалена.</p>

<p>Обновление записи</p>

<p>Мы научились удалять записи, давайте теперь попробуем что ни будь обновить и заодно научимся использовать флеш-сообщения.</p>

<pre>
<code>
func (manage *ManageController) Update() {
    o := orm.NewOrm()
    o.Using(&quot;default&quot;)
    flash := beego.NewFlash()
</code></pre>

<p>Как и раньше, начинаем с инициализации ORM объекта и указания соединение с базой. После этого мы получаем Beego флеш компонент, который может сохранять сообщения между запросами.</p>

<pre>
<code>
// convert the string value to an int
if articleId, err := strconv.Atoi(manage.Ctx.Input.Param(&quot;:id&quot;)); err == nil {
    article := models.Article{Id: articleId}
</code></pre>

<p>Здесь мы пытаемся получить <code>id</code> параметр и инициализировать новую модель <code>Article</code> если это возможно.</p>

<pre>
<code>
if o.Read(&amp;article) == nil {
    article.Client = &quot;Sitepoint&quot;
    article.Url = &quot;http://www.google.com&quot;
    if num, err := o.Update(&amp;article); err == nil {
        flash.Notice(&quot;Record Was Updated.&quot;)
        flash.Store(&amp;manage.Controller)
        beego.Info(&quot;Record Was Updated. &quot;, num)
    }
</code></pre>

<p>Потом мы вызываем <code>Read</code> метод и передаем в него объект <code>Article</code> и этот метод пытается загрузить все остальные параметры статьи из базы данных из записи в котрой есть id равный указанному нами в <code>Article.Id</code>.</p>

<p>Как только это возможно, мы устанавливаем значения <code>Client</code> и <code>Url</code> и передаем этот объект в метод <code>Update</code>, который обновляет запись в базе данных.</p>

<p>Если после этого у нас нет никаких ошибок, мы вызываем метод <code>Notice</code> у объекта <code>Flash</code>, указываем просто сообщение м вызываем метод <code>Store</code> для сохранения информации.</p>

<pre>
<code>
} else {
    flash.Notice(&quot;Record Was NOT Updated.&quot;)
    flash.Store(&amp;manage.Controller)
    beego.Error(&quot;Couldn&#39;t find article matching id: &quot;, articleId)
} else {
    flash.Notice(&quot;Record Was NOT Updated.&quot;)
    flash.Store(&amp;manage.Controller)
    beego.Error(&quot;Couldn&#39;t convert id from a string to a number. &quot;, err)
}
</code></pre>

<p>Если вдруг что то пошло не так, например не обновилась модель или мы не смогли распарсить id параметр, тогда мы записываем это как флеш сообщение и логируем этот момент.</p>

<pre>
<code>
    // redirect afterwards
    manage.Redirect(&quot;/manage/view&quot;, 302)
}
</code></pre>

<p>В самом конце мы вызываем метод <code>Redirect</code>, передаем ему url на который мы должны перейти и HTTP код статуса. Редирект на <code>/manage/view</code> произойдет независимо от того обновилась модель или нет.</p>

<p>Просмотр всех записей</p>

<p><code>View</code> функция состоит из двух блоков. Во-первых отображаем все статьи из таблицы, во вторых показываем все накопившиеся флеш сообщения. По флеш сообщениям сразу будет понятно как прошла операция.</p>

<pre>
<code>
func (manage *ManageController) View() {
    flash := beego.ReadFromRequest(&amp;manage.Controller)

    if ok := flash.Data[&quot;error&quot;]; ok != &quot;&quot; {
        // Display error messages
        manage.Data[&quot;errors&quot;] = ok
    }

    if ok := flash.Data[&quot;notice&quot;]; ok != &quot;&quot; {
        // Display error messages
        manage.Data[&quot;notices&quot;] = ok
    }
</code></pre>

<p>Первым делом, инициализируем переменную<code>flash</code> и пытаемся получить два значения <code>error</code> и <code>notice</code>. Это результаты вызовов функций <code>flash.Error</code> и <code>flash.Notice</code> соответственно. Если эти значения установленны, мы сохраняем эту информацию в <code>Data</code> и в дальнейшем можем получить к ней доступ.</p>

<pre>
<code>
    o := orm.NewOrm()
    o.Using(&quot;default&quot;)

    var articles []*models.Article
    num, err := o.QueryTable(&quot;articles&quot;).All(&amp;articles)

    if err != orm.ErrNoRows &amp;&amp; num &gt; 0 {
        manage.Data[&quot;records&quot;] = articles
    }
}
</code></pre>

<p>Как и в предыдущих примерах, сначала мы указываем какое соединение нужно использовать, после этого инициализируем слайс моделей <code>Article</code> в переменной <code>articles</code>. Затем, вызываем метод <code>QueryTable</code> указывая имя таблицы. И вызываем метод <code>All</code>, которому передаем указатель на слайс для наших статей, в который будет загружен весь результат.</p>

<p>Если все прошло хорошо, то мы получим список всех статей, который сможем передать в шаблон представления.</p>

<p>Вставка записи</p>

<p>Теперь посмотрим как добавлять новые записи используя метод <code>Add</code>, который получает данные из формы и валидирует их.</p>

<pre>
<code>
func (manage *ManageController) Add() {
    o := orm.NewOrm()
    o.Using(&quot;default&quot;)
    article := models.Article{}
</code></pre>

<p>Это я пропущу, мы подробно рассматривало это ранее.</p>

<pre>
<code>
if err := manage.ParseForm(&amp;article); err != nil {
    beego.Error(&quot;Couldn&#39;t parse the form. Reason: &quot;, err)
} else {
    manage.Data[&quot;Articles&quot;] = article
</code></pre>

<p>Здесь вызывается метод <code>ParseForm</code> передавая ему <code>article</code> объект. Если никаких ошибок не произошло мы передаем переменную <code>article</code> в шаблон для более простого рендеринга формы.</p>

<pre>
<code>
if manage.Ctx.Input.Method() == &quot;POST&quot; {
valid := validation.Validation{}
isValid, _ := valid.Valid(article)
if !isValid {
    manage.Data[&quot;Errors&quot;] = valid.ErrorsMap
    beego.Error(&quot;Form didn&#39;t validate.&quot;)
} else {
</code></pre>

<p>В этом месте проверяется тип запроса. Нам нужно обрабатывать только POST. Если это так, то мы инициализируем новый объект валидации и передаем в метод <code>Valid</code> объект <code>artcile</code> для валидации данных из POST согласно правилам в моделе.</p>

<p>Если данные не проходят валидацию, то все ошибки сохраняются и будут доступны в параметре <code>valid.ErrorsMap</code>. Все ошибки будут доступны в шаблоне в переменной <code>Errors</code>. В противном случае, мы сохраняем новую статью в базе. Если при этом произошла ошибка, мы логируем ее.</p>

<pre>
<code>
id, err := o.Insert(&amp;article)
if err == nil {
    msg := fmt.Sprintf(&quot;Article inserted with id:&quot;, id)
    beego.Debug(msg)
} else {
    msg := fmt.Sprintf(&quot;Couldn&#39;t insert new article. Reason: &quot;, err)
    beego.Debug(msg)
}
</code></pre>

<h3>Итоги</h3>

<p>Мы закончили обзор фреймворка Beego. Еще очень много фич осталось нерассмотренными, но они бы не поместились в эти две статьи.</p>

<p>Более того, некоторые примеры из данного урока сегодня уже могут быть несколько устаревшими. Причиной так сделать было не желание показать лучшие практики кодирование, а именно показать основную функциональность. Поэтому, не переживайте если структура кода покажется вам немного странной. Надеюсь вам понравилось это краткое введение в Beego и выдадите этому фреймворку стать вашим любимым инструментом. Учитывая все время, которое я на него потратил, я собираюсь использовать его и в будущем.</p>

<p>Если у вас есть какие ни будь вопросы, можете посмотреть онлайн документацию и задать их в комментария к посту. Не забывайте что все исходники примеров <a href="https://github.com/settermjd/Learning-Beego">доступны на GitHub</a>, поэтому, можете экспериментировать cколько угодно.</p>

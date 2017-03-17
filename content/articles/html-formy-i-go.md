+++
date = "2015-04-11T10:38:10+03:00"
draft = false
title = "HTML формы и Go"

+++

<p>Перевод статьи "<a href="http://blog.saush.com/2015/03/18/html-forms-and-go/">HTML Forms and Go</a>".</p>

<p>Это небольшой отрывок из моей книги "<a href="http://www.manning.com/chang/">Go Web Programming</a>", в котором рассказывается о использовании языка программирования Go для обработки данных из HTML форм. Это звучит достаточно тривиально, но, как и многое в веб-программировании (и программировании в принципе), эти тривиальные вещи часто оказываются камнем преткновения.</p>

<p>Перед тем как мы разберемся с обработкой данных из форм на стороне сервера, давайте чуть более внимательно посмотрим на их HTML описание. Чаще всего, POST реквесты приходят из HTML форм которые выглядят аналогично этому примеру:</p>

<pre><code class="html">&lt;form action="/process" method="post"&gt;
&lt;input type="text" name="first_name"/&gt;
&lt;input type="text" name="last_name"/&gt;
&lt;input type="submit"/&gt;
&lt;/form&gt;
</code></pre>

<p>Внутри HTML формы мы можем разместить различные элементы, такие как текстовые поля ввода(text), текстовые области(textarea), переключатели(radiobutton), поля для множ. выбора(checkboxes), поля для загрузки файлов и многое другое. Эти элементы позволяют пользователям ввести данные, которые будут отправлены на сервер. Отправка происходит после того, как пользователь нажмет кнопку, или некоторым другим способом сработает триггер отправки формы.</p>

<p>Мы знаем, что данные отправлены на сервер в виде HTTP POST запроса и размещены в теле самого запроса. Но как они будут отформатированы? Данные из HTML форм всегда образуют пары ключ-значение, но в каком формате эти пары ключ-значение отправляются в запросе? Для нас очень важно знать это, потому что нам придется разбирать эти данные из POST запроса на сервере и извлекать пары ключ-значение. Формат отправляемых данных зависит от типа контента в HTML форме. Он задается с помощью атрибута <code>enctype</code> в таком виде:</p>

<pre><code class="html">&lt;form action="/process" method="post" enctype="application/x-www-form-urlencoded"&gt;
&lt;input type="text" name="first_name"/&gt;
&lt;input type="text" name="last_name"/&gt;
&lt;input type="submit"/&gt;
&lt;/form&gt;
</code></pre>

<p>Значение по умолчанию для <code>enctype</code> это <code>application/x-www-form-urlencoded</code>, но браузеры должны поддерживать как минимум <code>application/x-www-form-urlencoded</code> и <code>multipart/form-data</code>  (HTML5 еще требуется поддержка <code>text/plain</code>).</p>

<p>Если мы укажем значение аттрибута <code>enctype</code> как <code>application/x-www-form-urlencoded</code>, то браузер преобразует данные из формы в длинную строку запроса в которой пары ключ-значение разделены амперсандами (&amp;) и имя с значением разделены символом равно (=), что выглядит также, как строка URL с параметрами, собственно, отсюда и название. Другими словами, тело HTTP запроса выглядит как-то так:</p>

<pre><code>first_name=sau%20sheong&amp;last_name=chang
</code></pre>

<p>Если установить значение аттрибута <code>enctype</code> как <code>multipart/form-data</code>, то каждая пара ключ-значение преобразуется в часть MIME сообщения, каждая с указанием типа и места размещения для контента. Для примера, данные из нашей подопытной формы будут выглядеть так:</p>

<pre><code class="sh">------WebKitFormBoundaryMPNjKpeO9cLiocMw
Content-Disposition: form-data; name="first_name"

sau sheong
------WebKitFormBoundaryMPNjKpeO9cLiocMw
Content-Disposition: form-data; name="last_name"

chang
------WebKitFormBoundaryMPNjKpeO9cLiocMw--
</code></pre>

<p>Когда стоит использовать тот или другой способ передачи? Если мы отправляем простые текстовые данные, то лучше выбрать URL форматирование, так как разбирать такие данные проще. Если же необходимо отправить большое количество данных, особенно если отправляем файлы, то следует использовать multipart-MIME формат. В таком случае, мы можем указать кодировку base64 для бинарных файлов, чтоб отправить их как текст.</p>

<p>До сих пор мы говорили только о POST запросах, как насчет GET? HTML формы позволяют указывать значения атрибута <code>method</code> как POST так и GET, поэтому следующая форма также валидна:</p>

<pre><code class="html">&lt;form action="/process" method="get"&gt;
&lt;input type="text" name="first_name"/&gt;
&lt;input type="text" name="last_name"/&gt;
&lt;input type="submit"/&gt;
&lt;/form&gt;
</code></pre>

<p>В таком случае нет никакого тела запроса (Это специфично для GET запросов), все данные устанавливаются в URL как пары ключ-значение.</p>

<p>Теперь мы знаем в каком виде данные из HTML форм отправляются на сервер, можем вернутся к серверу и используя пакет <code>net/http</code> обработать запрос.</p>

<h3>Form</h3>

<p>Один из способов извлечения данных из HTTP запроса это выдернуть данные из тела запроса и/или из URL в сыром виде, и самим распарсить эти данные. К счастью, в этом нет необходимости, потому что пакет <code>net/http</code> предоставляет нам полный набор функций, хотя и не всегда очевидно называющихся, для работы с формами. Давайте, для затравки, рассмотрим пару часто используемых функций.</p>

<p>Функции из <code>Request</code>, который позволяют нам извлечь данные формы из тела запроса и/или из URL, вращаются вокруг полей <code>Form</code>, <code>PostForm</code> и <code>MultipartForm</code>. Как вы помните, данные это пары ключ-значение(которые мы получаем из POST запроса в том или ином виде). Основной алгоритм работы с этими данными такой:</p>

<ul>
<li>Вызов <code>ParseForm</code> или <code>ParseMultipartForm</code> для разбора запроса</li>
<li>Обращения к <code>Form</code>, <code>PostForm</code> или <code>MultipartForm</code> соответственно</li>
</ul>

<p>Давайте напишем немного кода.</p>

<pre><code class="go">package main

import (
    "fmt"
    "net/http"
)

func process(w http.ResponseWriter, r *http.Request) {
    r.ParseForm()
    fmt.Fprintln(w, r.Form)
}

func main() {
    server := http.Server{
    Addr: "127.0.0.1:8080",
}
http.HandleFunc("/process", process)
    server.ListenAndServe()
}
</code></pre>

<p>Сфокусируем внимание на двух строчках этого кода:</p>

<pre><code class="go">r.ParseForm()
fmt.Fprintln(w, r.Form)
</code></pre>

<p>Как уже было сказано выше, прежде всего нам нужно распарсить запрос с использованием <code>ParseForm</code> и после этого можем обращаться к полю <code>Form</code></p>

<p>Давайте посмотрим, как выглядит клиент для вызова нашего серверного кода. Создадим минимальную HTML форму для отправки запроса. Поместите код в файл <code>client.html</code>.</p>

<pre><code class="html">&lt;html&gt;
&lt;head&gt;
&lt;meta http-equiv="Content-Type" content="text/html; charset=utf-8" /&gt;
&lt;title&gt;Go Web Programming&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;form action="http://127.0.0.1:8080/process?hello=world&amp;thread=123" 
method="post" enctype="application/x-www-form-urlencoded"&gt;
&lt;input type="text" name="hello" value="sau sheong"/&gt;
&lt;input type="text" name="post" value="456"/&gt;
&lt;input type="submit"/&gt;
&lt;/form&gt;
&lt;/body&gt;
&lt;/html&gt;
</code></pre>

<p>В этой форме мы:</p>

<ul>
<li>Отправляем на сервер данные по URL http://localhost:8080/process?hello=world&amp;thread=123 методом POST</li>
<li>Указываем тип контента в поле <code>enctype</code> как <code>application/x-www-form-urlencoded</code></li>
<li>Отправляем на сервер две пары ключ-значение - <code>hello=sau sheong</code> и <code>post=456</code></li>
</ul>

<p>Обратите внимание, у нас есть 2 значения для ключа <code>hello</code>. Одно из них это <code>world</code> в URL, другое это <code>sau sheong</code> в HTML форме.</p>

<p>Откройте в браузере файл <code>client.html</code> напрямую (вам не нужен сервер для этого) и кликните по кнопке отправки. В браузере вы должны увидеть:</p>

<pre><code>map[thread:[123] hello:[sau sheong world] post:[456]]
</code></pre>

<p>Это строка показывает содержимое поля <code>Form</code> после POST запроса и его парсинга. <code>Form</code> это обычный мап со строками в качестве ключей и слайсами строк для значений. Стоит упомянуть, что это не сортированный map и вы всегда будете получать поля в случайном порядке. Тем не менее, мы получаем все значения из строки запроса <code>hello=world</code> и <code>thread=123</code>, а также значения из формы <code>hello=sau sheong</code> и <code>post=456</code>. Значения из URL декодированы (это видно по пробелу между <code>sau</code> and <code>sheong</code>).</p>

<h3>PostForm</h3>

<p>Конечно, вы можете обратиться только к одному ключу, например <code>post</code>. Для это надо указать этот ключ <code>Form["post"]</code> и вы получите слайс с одним элементом - <code>[456]</code>. Если и в форме и в URL есть одинаковые ключи, они оба попадут в слайс , значение из формы будет считаться более приоритетным и это значение будет расположено перед значением из URL.</p>

<p>Что если нам нужны только пары ключ-значение из формы, а параметры из URL нужно игнорировать? Для этого у нас есть метод <code>PostForm</code>, который дает доступ только к значениям из формы. Если в нашем примере заменить строку <code>r.Form</code> на <code>r.PostForm</code>, то вывод приложения будет таким:</p>

<pre><code class="go">map[post:[456] hello:[sau sheong]]
</code></pre>

<p>Мы используем <code>application/x-www-form-urlencoded</code> для указания типа контента. Что случится, если будем использовать <code>multipart/form-data</code>? Внесите необходимые изменения в наш html файл и верните назад <code>r.Form</code>. Смотрим, что получилось:</p>

<pre><code class="go">map[hello:[world] thread:[123]]
</code></pre>

<p>Что мы видим? В <code>r.Form</code> попали только значения из URL и никаких пар ключ-значения из формы. Это  происходит потому что <code>PostForm</code> поддерживает только <code>application/x-www-form-urlencoded</code>. Для получения multipart данных из тела запроса нам нужно использовать <code>MultipartForm</code>.</p>

<h3>MultipartForm</h3>

<p>Вместо вызова <code>ParseForm</code> а затем обращения к <code>Form</code> нам необходимо использовать <code>ParseMultipartForm</code> и обращаться к полю <code>MultipartForm</code>. Метод <code>ParseMultipartForm</code> при необходимости сам вызывает <code>ParseForm</code>.</p>

<pre><code class="go">r.ParseMultipartForm(1024)
fmt.Fprintln(w, r.MultipartForm)
</code></pre>

<p>Нам нужно сообщить <code>ParseMultipartForm</code>, как много данных необходимо извлечь из multipart формы(в байтах). Теперь можем посмотреть результат:</p>

<pre><code class="go">&amp;{map[hello:[sau sheong] post:[456]] map[]}
</code></pre>

<p>Тут мы видим пары ключ-значения из формы, но нет никаких значений из URL. Это означает, что <code>MultipartForm</code> содержит только данные из формы. Обратите внимание, что полученное значение это структура с двумя мапами, а не с одним. Первый мап со строковыми ключами и слайсами строк для значений, а второй мап пустой. В этом втором мапе ключи также строковые, а вот значения уже файлы.</p>

<p>Существует еще один набор функций, который позволяет нам работать с парами ключ-значение намного проще. Метод <code>FormValue</code> позволяет обратиться к значению по ключу, аналогично обращению к <code>Form</code>, но нам не нужно перед этим вызывать <code>ParseForm</code> или <code>ParseMultipartForm</code>, <code>FormValue</code> сделает это за нас.</p>

<p>Это означает, что если мы чуть-чуть подправим наш пример:</p>

<pre><code class="go">fmt.Fprintln(w, r.FormValue("hello"))
</code></pre>

<p>И укажем параметр формы <code>enctype</code> как <code>application/x-www-form-urlencoded</code>, то результат будет таким:</p>

<pre><code class="go">sau sheong
</code></pre>

<p>Мы получили только <code>sau sheong</code>, потому что <code>FormValue</code> возвращает только первый элемент из слайса, даже если реально у нас есть оба значения в поле <code>Form</code>. Что бы убедиться в этом, давайте добавим еще пару строчек кода:</p>

<pre><code class="go">fmt.Fprintln(w, r.FormValue("hello"))
fmt.Fprintln(w, r.Form)
</code></pre>

<p>И увидим вот такую картину:</p>

<pre><code class="go">sau sheong
map[post:[456] hello:[sau sheong world] thread:[123]]
</code></pre>

<p>Функция <code>PostFormValue</code> работает аналогично, только не для <code>Form</code>, а для <code>PostForm</code>. Давайте внесем некоторые изменения в наш для использования функции <code>PostFormValue</code>:</p>

<pre><code class="go">fmt.Fprintln(w, r.PostFormValue("hello"))
fmt.Fprintln(w, r.PostForm)
</code></pre>

<p>В результате получем что то такое:</p>

<pre><code class="go">sau sheong
map[hello:[sau sheong] post:[456]]
</code></pre>

<p>Как можно заметить, тут вывелись только пары ключ-значение указанные в форме.</p>

<p>Обе функции <code>FormValue</code> и <code>PostFormValue</code> самостоятельно вызывают вызывают <code>ParseMultipartForm</code> и нам не нужно этого делать в ручную, но тут кроется небольшая ловушка и вам нужно быть осторожным(по крайней мере для Go 1.4). Если мы установим значение атрибута <code>enctype</code> как <code>multipart/form-data</code> и попробуем получить значения с использованием <code>FormValue</code> или <code>PostFormValue</code>, то ничего не получится, даже если использовать <code>MultipartForm</code>!</p>

<p>Чтобы понять, о чем идет речь, давайте еще немного поэкспериментируем с нашим примером:</p>

<pre><code class="go">fmt.Fprintln(w, "(1)", r.FormValue("hello"))
fmt.Fprintln(w, "(2)", r.PostFormValue("hello"))
fmt.Fprintln(w, "(3)", r.PostForm)
fmt.Fprintln(w, "(4)", r.MultipartForm)
</code></pre>

<p>При установленном значении <code>enctype</code> как <code>multipart/form-data</code> результаты будут такими:</p>

<pre><code class="go">(1) world
(2)
(3) map[]
(4) &amp;{map[hello:[sau sheong] post:[456]] map[]}
</code></pre>

<p>Первая строчка результата - это значение параметра <code>hello</code> из URL, не из формы. Вторая и третья строка демонстрируют, что мы не сможем получить пары ключ-значения из формы. Так происходит, потому что <code>FormValue</code> и <code>PostFormValue</code> работают с <code>Form</code> и <code>PostForm</code>, никак не касаясь <code>MultipartForm</code>. Последняя строка - демонстрация того, что функция <code>ParseMultipartForm</code> была вызвана и сделала возможным доступ к параметрам через <code>MultipartForm</code></p>

<p>Мы можем резюмировать все что было описано в этой статье в виде небольшой таблицы, в которой показано, как функции для работы с данными запроса отличаются друг от друга.</p>

<p><img src="https://dl.dropboxusercontent.com/u/750049/4gophers.com/form.png" alt="" /></p>

+++
date = "2015-08-16T18:46:06+03:00"
draft = false
title = "Go и SQL базы данных"

+++

<p>Перевод статьи "<a href="http://www.alexedwards.net/blog/practical-persistence-sql">Practical Persistence in Go: SQL Databases</a>".</p>

<p>Это первый туториал из серии материалов про работу с данными в веб приложениях.</p>

<p>В этом посте мы погрузимся в работу с SQL базами данных. Я объясню работу с стандартным пакетом <a href="http://golang.org/pkg/database/sql/"><code>database/sql</code></a>, приведу примеры рабочих приложений и продемонстрирую несколько хитростей для более красивого структурирования кода.</p>

<p>Для начала, вам необходимо установить <a href="https://github.com/golang/go/wiki/SQLDrivers">драйвера для работы с <code>database/sql</code></a>.</p>

<p>В этом посте мы будем использовать Postgres и замечательный драйвер <a href="https://github.com/lib/pq">pg</a>. Тем не менее, весь код из этого туториала должен нормально работает и с другими драйверами, включая MySQL и SQLite. По ходу я буду указывать на специфические для Postgres моменты(которых будет не очень много).</p>

<pre><code class="sh">$ go get github.com/lib/pq
</code></pre>

<h2>Основы</h2>

<p>Давайте напишем простое приложение для книжного магазина, которое будет выполнять CRUD операции с нашей таблицей для книг.</p>

<p>Прежде всего, нам нужно создать эту самую таблицу для книг, как показано ниже:</p>

<pre><code class="sql">CREATE TABLE books (
  isbn    char(14) NOT NULL,
  title   varchar(255) NOT NULL,
  author  varchar(255) NOT NULL,
  price   decimal(5,2) NOT NULL
);

INSERT INTO books (isbn, title, author, price) VALUES
('978-1503261969', 'Emma', 'Jayne Austen', 9.44),
('978-1505255607', 'The Time Machine', 'H. G. Wells', 5.99),
('978-1503379640', 'The Prince', 'Niccolò Machiavelli', 6.99);

ALTER TABLE books ADD PRIMARY KEY (isbn);
</code></pre>

<p>После этого, необходимо настроить свое Go окружение, создать папку bookstore и файл main.go:</p>

<pre><code class="sh">$ cd $GOPATH/src
$ mkdir bookstore &amp;&amp; cd bookstore
$ touch main.go
</code></pre>

<p>Давайте начнем с простого кода, который будет выполнять запрос <code>SELECT * FROM books</code> и выводить результат в консоль.</p>

<pre><code class="go">package main

import (
    _ "github.com/lib/pq"
    "database/sql"
    "fmt"
    "log"
)

type Book struct {
    isbn  string
    title  string
    author string
    price  float32
}

func main() {
    db, err := sql.Open("postgres", "postgres://user:pass@localhost/bookstore")
    if err != nil {
        log.Fatal(err)
    }

    rows, err := db.Query("SELECT * FROM books")
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()

    bks := make([]*Book, 0)
    for rows.Next() {
        bk := new(Book)
        err := rows.Scan(&amp;bk.isbn, &amp;bk.title, &amp;bk.author, &amp;bk.price)
        if err != nil {
            log.Fatal(err)
        }
        bks = append(bks, bk)
    }
    if err = rows.Err(); err != nil {
        log.Fatal(err)
    }

    for _, bk := range bks {
        fmt.Printf("%s, %s, %s, £%.2f\n", bk.isbn, bk.title, bk.author, bk.price)
    }
}
</code></pre>

<p>В этом куске кода довольно много разных действий. Пройдемся по шагам и рассмотрим как все это работает.</p>

<p>Первый интересный момент, это импортирование пакета драйвера. Мы ничего не используем напрямую из этого пакета, а это означает, что компилятор Go ругнется, если вы попробуете импортировать пакет как обычно. Но нам необходим вызов функции <code>init()</code> этого пакета для того, чтобы <a href="http://golang.org/pkg/database/sql/#Register">драйвер зарегистрировал себя</a> для использования в <code>database/sql</code>. Мы можем обойти это ограничение используя пустой <a href="http://learntogoogleit.com/post/63748050636/aliasing-imports-in-golang">алиас для импортируемого пакета</a>. И это означает, что <code>pq.init()</code> будет выполняться, но благодаря алиасу мы избавимся от ошибок во время компиляции. Такая практика является стандартом для большинства SQL драйверов в Go.</p>

<p>Далее, мы определим наш тип для книги, в котором поля и типы полей будут зависеть от таблицы books. Тут стоит уточнить, что мы можем безопасно использовать <code>string</code> и <code>float32</code>, так как мы указали <code>NOT NULL</code> для колонок в нашей таблице. Если в таблице есть поля, которые могут содержать NULL, то следует использовать типы <code>sql.NullString</code> и <code>sql.NullFloat64</code> (<a href="https://gist.github.com/alexedwards/dc3145c8e2e6d2fd6cd9">тут можно глянуть рабочий пример</a>). Вообще, если у вас есть возможность, старайтесь избегать полей, в которых могут быть NULL значения.</p>

<p>В функции <code>main()</code> мы инициализируем экземпляр <code>sql.DB</code> с помощью вызова <code>sql.Open()</code>.  Мы указываем название нашего драйвера(в нашем случае это "postgres") и строку соединения, формат которой должен быть описан в документации к драйверу. Важное замечание, <code>sql.DB</code> это не <em>соединение с базой</em>, это некоторая абстракция над пулом соединений. Вы можете менять максимальное количество открытых и простаиваемых соединений в пуле с помощью методов <code>db.SetMaxOpenConns()</code> и <code>db.SetMaxIdleConns()</code> соответственно. И обратите внимание, что <code>sql.DB</code> можно безопасно использовать в конкурентных приложениях(которыми являются и веб-приложения).</p>

<p>Рассмотрим использованные стандартные паттерны:</p>

<ol>
<li>Мы получаем данные из таблицы, используя метод <code>DB.Query()</code> и присваиваем результат переменной <code>rows</code>. После этого, мы пользуемся <code>defer rows.Close()</code>, чтобы наверняка закрыть сет с результатами до выхода из функции. Очень важно не забывать закрывать сет. Все время, пока открыт сет, используемое соединение невозможно вернуть в пул. Если вдруг что-то пойдет не так и ваш сет не будет закрываться, то соединения в пуле могут быстро закончиться. Еще одна ловушка в том(и это оказалось для меня сюрпризом), что <code>defer</code> должен идти <em>после</em> проверки на ошибки <code>DB.Query()</code>. Если <code>DB.Query()</code> вернет ошибку, то вместо сета будет получен <code>nil</code> и при вызове <code>rows.Close()</code> стрельнет паника.</li>
<li>Для итерации по строкам мы используем  <code>rows.Next()</code>. Этот метод подготавливает строку для использования метода <code>rows.Scan()</code>. Не забывайте, что по завершению итерации по всем строкам сет автоматически закрывается и соединение возвращается в пул.</li>
<li>Мы используем метод <code>rows.Scan()</code>, чтобы скопировать значения всех полей из строки в созданный нами экземпляр <code>Book</code>. Далее, мы проверяем была ли ошибка при работе метода <code>rows.Scan()</code> и добавляем новый экземпляр <code>Book</code> в слайс <code>bks</code>, который мы создали ранее.</li>
<li>После итераций с помощью <code>rows.Next()</code> мы вызываем <code>rows.Err()</code>. Этот метод возвращает любую ошибку, которая произошла во время выполнения итераций. Этот момент достаточно важен, он позволяет убедиться, что мы прошлись по всему сету без ошибок.</li>
</ol>

<p>Если все хорошо и мы нормально заполнили на слайс <code>bks</code>, то теперь мы итерируемся по нему и выводим информацию в консоль.</p>

<p>Если вы запустите код, то должно получиться что-то такое:</p>

<pre><code class="go">$ go run main.go
978-1503261969, Emma, Jayne Austen, £9.44
978-1505255607, The Time Machine, H. G. Wells, £5.99
978-1503379640, The Prince, Niccolò Machiavelli, £6.99
</code></pre>

<h3>Использование в веб-приложении</h3>

<p>Давайте изменим наш код, что бы получилось RESTful веб-приложение с 3 роутами:</p>

<ul>
<li>GET /books – Список всех книг в магазине</li>
<li>GET /books/show – Информация о конкретной книге по ISBN</li>
<li>POST /books/create – Добавление новой книги в магазин</li>
</ul>

<p>Мы уже реализовали основную логику необходимую для <code>GET /books</code>. Давайте адаптируем ее для использования в HTTP хендлере <code>booksIndex()</code> нашего приложения.</p>

<pre><code class="go">package main

import (
    _ "github.com/lib/pq"
    "database/sql"
    "fmt"
    "log"
    "net/http"
)

type Book struct {
    isbn   string
    title  string
    author string
    price  float32
}

var db *sql.DB

func init() {
    var err error
    db, err = sql.Open("postgres", "postgres://user:pass@localhost/bookstore")
    if err != nil {
        log.Fatal(err)
    }

    if err = db.Ping(); err != nil {
        log.Fatal(err)
    }
}

func main() {
    http.HandleFunc("/books", booksIndex)
    http.ListenAndServe(":3000", nil)
}

func booksIndex(w http.ResponseWriter, r *http.Request) {
    if r.Method != "GET" {
        http.Error(w, http.StatusText(405), 405)
        return
    }

    rows, err := db.Query("SELECT * FROM books")
    if err != nil {
        http.Error(w, http.StatusText(500), 500)
        return
    }
    defer rows.Close()

  bks := make([]*Book, 0)
  for rows.Next() {
      bk := new(Book)
      err := rows.Scan(&amp;bk.isbn, &amp;bk.title, &amp;bk.author, &amp;bk.price)
      if err != nil {
          http.Error(w, http.StatusText(500), 500)
          return
      }
      bks = append(bks, bk)
  }
  if err = rows.Err(); err != nil {
      http.Error(w, http.StatusText(500), 500)
      return
  }

  for _, bk := range bks {
      fmt.Fprintf(w, "%s, %s, %s, £%.2f\n", bk.isbn, bk.title, bk.author, bk.price)
  }
}
</code></pre>

<p>И в чем же тут отличия?</p>

<ul>
<li>Мы используемые функцию <code>init()</code> для настройки нашего пула соединений и указываем его в качестве значения глобальной переменной <code>db</code>. Мы используем глобальную переменную, которая предоставляет доступ к пулу соединений, чтобы иметь возможность использовать ее в разных HTTP хендлерах, но это не единственный возможный способ. Так как <code>sql.Open()</code> не проверяет соединение, то мы вызываем <code>DB.Ping()</code>, чтобы убедиться, что все работает нормально.</li>
<li>В хендлере <code>booksIndex</code> мы возвращаем <code>405 Method Not Allowed</code> ответ для всех не GET запросов. Дальше мы работаем с нашими данными. Все работает как в примере выше, за исключением что ошибки теперь возвращаются как HTTP ответ и нет выхода из программы. В результате мы записываем описания книг как обычный текст в <code>http.ResponseWriter</code>.</li>
</ul>

<p>Запускаем приложение и делаем запрос к нему:</p>

<pre><code>$ curl -i localhost:3000/books
HTTP/1.1 200 OK
Content-Length: 205
Content-Type: text/plain; charset=utf-8

978-1503261969, Emma, Jayne Austen, £9.44
978-1505255607, The Time Machine, H. G. Wells, £5.99
978-1503379640, The Prince, Niccolò Machiavelli, £6.99
</code></pre>

<h3>Выборка одной строки</h3>

<p>Для <code>GET /books/show</code> нам нужно реализовать получение одной книги из базы по ее ISBN, который будет указываться как параметр в запросе:</p>

<pre><code>/books/show?isbn=978-1505255607
</code></pre>

<p>Для этого мы добавим хендлер <code>bookShow()</code>:</p>

<pre><code class="go">// ...

func main() {
    http.HandleFunc("/books", booksIndex)
    http.HandleFunc("/books/show", booksShow)
    http.ListenAndServe(":3000", nil)
}
// ...

func booksShow(w http.ResponseWriter, r *http.Request) {
    if r.Method != "GET" {
        http.Error(w, http.StatusText(405), 405)
        return
    }

    isbn := r.FormValue("isbn")
    if isbn == "" {
        http.Error(w, http.StatusText(400), 400)
        return
    }

    row := db.QueryRow("SELECT * FROM books WHERE isbn = $1", isbn)

    bk := new(Book)
    err := row.Scan(&amp;bk.isbn, &amp;bk.title, &amp;bk.author, &amp;bk.price)
    if err == sql.ErrNoRows {
        http.NotFound(w, r)
        return
    } else if err != nil {
        http.Error(w, http.StatusText(500), 500)
        return
    }

    fmt.Fprintf(w, "%s, %s, %s, £%.2f\n", bk.isbn, bk.title, bk.author, bk.price)
}
</code></pre>

<p>Первым делом, в обработчике проверяется действительно ли пришел GET запрос.</p>

<p>После этого, мы используем метод <code>Request.FormValue()</code> для получения параметров из строки запроса. В случае если нет необходимых параметров, то мы получаем пустую строку и возвращаем ответ <code>400 Bad Request</code>.</p>

<p>Тут мы подходим к самому интересному. Метод <code>DB.QueryRow()</code> работает аналогично <code>DB.Query()</code>, но получает только одну строку.</p>

<p>Так как у нас есть некоторый ненадежный данные от пользователя(переменная <code>isbn</code>), то в нашем SQL запросе нужно использовать плейсхолдеры для параметров, сами значения мы указываем как аргументы после строки запроса.</p>

<pre><code class="go">db.QueryRow("SELECT * FROM books WHERE isbn = $1", isbn)
</code></pre>

<p>Если чуть углубиться, то можно обнаружить, что <code>db.QueryRow</code> (а также <code>db.Query()</code> и <code>db.Exec()</code>) создают "подготовленные выражения"(prepared statement) в базе данных и выполняют запросы, подставляя параметры в плейсхолдеры этих выражений. Это означает, что все три метода безопасны в плане SQL-инъекций, если пользоваться ими правильно. Вот что говорит нам википедия:</p>

<blockquote>
  <p>Подготовленные выражения устойчивы к SQL инъекциям, поскольку значения параметров, которые передаются позже с использованием другого протокола, не нужно ескейпить. Если оригинальное выражение построено не на основании внешнего ввода, то инъекции не может произойти.
  В зависимости от базы данных, плейсхолдеры указываются по разному. В Postgres используется нотация <code>$N</code>, но в MySQL, SQL Server и в некоторых других используется символ <code>?</code>.</p>
</blockquote>

<p>Окей, давайте вернемся к нашему коду.</p>

<p>После получения строки с помощью <code>DB.QueryRow()</code>, мы используем <code>row.Scan()</code> для копирования значений в наш новы объект <code>Book</code>. Важно, что мы не узнаем про ошибки выполнения запроса в методе <code>DB.QueryRow()</code>, пока не вызовем метод <code>row.Scan()</code>.</p>

<p>Если ваш запрос не нашел ни одной строки, то вызов <code>row.Scan()</code> вернет ошибку <code>sql.ErrNoRows</code>. Мы выполняем проверку на эту ошибку и, если ничего не найдено, возвращаем <code>404 Not Found</code> ответ. Если возникают другие ошибку, то возвращаем <code>500 Internal Server Error</code>.</p>

<p>Если все хорошо, то мы записываем в <code>http.ResponseWriter</code> информацию по запрашиваемой книге.</p>

<p>Давайте попробуем:</p>

<pre><code>$ curl -i localhost:3000/books/show?isbn=978-1505255607
HTTP/1.1 200 OK
Content-Length: 54
Content-Type: text/plain; charset=utf-8

978-1505255607, The Time Machine, H. G. Wells, £5.99
</code></pre>

<p>Если вы попробуете указывать разные значения ISBN, то можете увидеть как меняется результат ответа. В случае неправильного запроса, вы должны получить соответствующий код ошибки.</p>

<h3>Выполнение выражений</h3>

<p>Для нашего роута <code>POST /books/create</code> мы создадим хендлер <code>booksCreate()</code>, в котором будем использовать <code>DB.Exec()</code> для выполнения выражения <code>INSERT</code>. Вы можете использовать схожий подход для <code>UPDATE</code>, <code>DELETE</code> или других операций, которые не подразумевают получение результата в виде строк таблиц.</p>

<p>Код выглядит так:</p>

<pre><code class="go">// ...

import (
    _ "github.com/lib/pq"
    "database/sql"
    "fmt"
    "log"
    "net/http"
    "strconv"
)
// ...

func main() {
    http.HandleFunc("/books", booksIndex)
    http.HandleFunc("/books/show", booksShow)
    http.HandleFunc("/books/create", booksCreate)
    http.ListenAndServe(":3000", nil)
}
// ...

func booksCreate(w http.ResponseWriter, r *http.Request) {
    if r.Method != "POST" {
        http.Error(w, http.StatusText(405), 405)
        return
    }

    isbn := r.FormValue("isbn")
    title := r.FormValue("title")
    author := r.FormValue("author")

    if isbn == "" || title == "" || author == "" {
        http.Error(w, http.StatusText(400), 400)
        return
    }

    price, err := strconv.ParseFloat(r.FormValue("price"), 32)

    if err != nil {
        http.Error(w, http.StatusText(400), 400)
        return
    }

    result, err := db.Exec("INSERT INTO books VALUES($1, $2, $3, $4)", isbn, title, author, price)

    if err != nil {
        http.Error(w, http.StatusText(500), 500)
        return
    }

    rowsAffected, err := result.RowsAffected()

    if err != nil {
        http.Error(w, http.StatusText(500), 500)
        return
    }

    fmt.Fprintf(w, "Book %s created successfully (%d row affected)\n", isbn, rowsAffected)
}
</code></pre>

<p>Думаю, вы уже находите много знакомого в этом коде.</p>

<p>В хендлере <code>booksCreate()</code> мы проверяем, действительно ли пришел POST запрос и получаем параметры из запроса с помощью <code>request.FormValue()</code>. Мы проверяем наличие всех необходимых параметров, а цену еще и конвертируем в <code>float</code> с помощью <code>strconv.ParseFloat()</code>.</p>

<p>После этого, мы используем <code>db.Exec()</code> с указанием полученных парметров, аналогично как мы делали это ранее. Важно, что <code>DB.Exec()</code>,  <code>DB.Query()</code> и <code>DB.QueryRow()</code>, - это функции которое могут принимать переменное число параметров.</p>

<p>Метод <code>db.Exec()</code> в качестве результата возвращает объект, который удовлетворяет интерфейс <a href="http://golang.org/pkg/database/sql/#Result">sql.Result</a>. При необходимости, этот результат можно использовать или не учитывать, используя пустой идентификатор.</p>

<p>Интерфейс <code>sql.Result</code> предоставляет метод <code>LastInsertId()</code>, который используется для получения последнего значения автоинкремента. Также, можно использовать метод <code>RowsAffected()</code>, который возвращает число строк, затронутых в запросе(удаленных, обновленных, новых и т.д.). В нашем случае, используется второй описанный метод, мы получаем количество строк и формируем сообщение.</p>

<p>Стоит отметить, что не все драйвера поддерживают методы <code>LastInsertId()</code> и <code>RowsAffected()</code> и при их вызове вы получите ошибку. К примеру, драйвер <code>pq</code> не поддерживает метод <code>LastInsertId()</code> и, если вам необходим подобный функционал, то прийдется использовать <a href="https://github.com/lib/pq/issues/24">подход вроде этого</a>.</p>

<p>Давайте проверим роут <code>/books/create</code> с передачей необходимых параметров в POST:</p>

<pre><code>$ curl -i -X POST -d "isbn=978-1470184841&amp;title=Metamorphosis&amp;author=Franz Kafka&amp;price=5.90" \ 
localhost:3000/books/create
HTTP/1.1 200 OK
Content-Length: 58
Content-Type: text/plain; charset=utf-8

Book 978-1470184841 created successfully (1 row affected)
</code></pre>

<h3>Использование DB.Prepare()</h3>

<p>Возможно, вам стало интересно, почему не используется <code>DB.Prepare()</code>.</p>

<p>Как я объяснял выше, методы методы <code>DB.Query()</code>, <code>DB.Exec()</code> и <code>DB.QueryRow()</code> создают подготовленные выражения в базе данных, запускают их с указанными параметрами и затем закрывают(точнее деаллоцируют) эти выражения.</p>

<p>Недостатки использования такого подхода очевидны. У нас аж три обращения к базе данных на каждый HTTP запрос. Чтобы избежать этого, мы можем воспользоваться <code>DB.Prepare()</code> (например, в функции <code>init()</code>).</p>

<p>Но на самом деле все не так просто. Подготовленные выражения работают только с конкретной сессией базы данных. Когда сессия заканчивается, подготовленные выражения необходимо создавать заново. Это может произойти, если есть большой таймаут между запросами или вам пришлось рестартовать базу.</p>

<p>Для веб-приложений, в которых задержка имеет большое значение, возможно стоит заморочиться и добавить некоторый мониторинг, который будет реинициализировать подготовленные выражения. Но в таком приложении, как наше, это слишком большой оверхед и нам достаточно использовать <code>DB.Query()</code> как есть.</p>

<p>В этом треде описанная проблема <a href="https://groups.google.com/forum/#!topic/golang-nuts/ISh22XXze-s">обсуждается несколько глубже</a>.</p>

<h3>Рефакторинг</h3>

<p>В данный момент, вся наша логика работы с базой перемешана с обработкой HTTP запросов. И это отличный подвод для рефакторинга, который позволит упростить наш код и сделать его более логичным.</p>

<p>Но этот туториал уже и так достаточно большой, поэтому оставим это для следующего поста - "Practical Persistence in Go: Organising Database Access" (в скором времени).</p>

<h3>Дополнительные инструменты</h3>

<p>Пакет <a href="https://github.com/jmoiron/sqlx">Sqlx</a> от Jason Moiron предоставляет расширенный функционал для работы с базой, включая именованные плейсхолдеры, и автоматический маршалинг результатов запроса в структуры.</p>

<p>Если вам интересны более ORM-ориентированные подходы, то взгляните в сторону <a href="https://github.com/jmoiron/modl">Modl</a> того же автора или <a href="https://github.com/go-gorp/gorp">gorp</a> от James Cooper.</p>

<p>Пакет <a href="https://github.com/guregu/null">null</a> может помочь вам при работе с null-значениями.</p>

<p>Позже, я нашел отличный туториал: "<a href="http://go-database-sql.org/">go-database-sql.org</a>". Обратите особое внимание на раздел про сюрпризы и ограничения.</p>

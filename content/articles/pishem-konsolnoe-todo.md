+++
date = "2014-11-23T20:05:08+03:00"
draft = false
title = "Пишем консольное todo"

+++

<p>Перевод статьи &quot;Writing a Command-line Task Tracker in Go&quot;. <a href="http://takemikazuchi.github.io/2014/04/06/command-line-task-tracker-in-go/">Оригинал тут</a>.</p>

<p>Из этого туториала вы узнаете, как с помощью Go написать простое консольное приложение. Предполагается, что вы уже <a href="http://tour.golang.org/#1">ознакомились с языком</a> и у вас <a href="http://golang.org/doc/install">настроено окружение для нормальной разработки на нем</a>.</p>

<p>Наше приложение будет напоминать известное <a href="http://todotxt.com">todo.txt</a>.</p>

<p>Мы сможем добавлять задачи, просматривать список задач и отмечать выполненные с помощью команды <code>todo</code> в консоли:</p>

<pre>
<code>$ todo ls
[1]     [2014-3-27]     Get groceries
[2]     [2014-3-27]     Fix Issue #4501
[3]     [2014-3-28]     Add more features to
$ todo add &quot;Update readme file&quot;
Task is added: Update readme file
$ todo ls
[1]     [2014-3-27]     Get groceries
[2]     [2014-3-27]     Fix Issue #4501
[3]     [2014-3-28]     Add more features to
[4]     [2014-3-29]     Update readme file
$ todo complete 1
Task Marked as complete: Get groceries
$ todo ls
[1]     [2014-3-27]     Fix Issue #4501
[2]     [2014-3-28]     Add more features to
[3]     [2014-3-29]     Update readme file
</code></pre>

<p>Реализуя функцинал этого приложения, вы узнаете как сохранять введенные пользователем данные, отображать эти данные в удобной форме и изменять их по необходимости.</p>

<p>Содержание</p>

<ul>
	<li>Разбираемся с консолью</li>
	<li>Добавление задач, хранение в JSON</li>
	<li>Список задач</li>
	<li>Выполнение задачи</li>
</ul>

<p>Разбираемся с консолью</p>

<p>Для начала нам нужен <a href="https://github.com/codegangsta/cli">пакет для работы с консолью от Codegangsta</a>, который немного упростит нам жизнь.</p>

<pre>
<code>go get github.com/codegangsta/cli
</code></pre>

<p>Если вы не поленитесь сходить на гитхаб и почитать README, то найдете отличное описание работы этого пакета и примеры, которые мы можем использовать для быстрого старта. Давайте начнем писать наше приложение с определения основных команд.</p>

<p>Создайте файл todo.go:</p>

<pre>
<code>package main

import (
    &quot;fmt&quot;
    &quot;github.com/codegangsta/cli&quot;
    &quot;os&quot;
)

func main() {
    app := cli.NewApp()
    app.Name = &quot;todo&quot;
    app.Usage = &quot;add, list, and complete tasks&quot;
    app.Commands = []cli.Command{
        {
            Name:      &quot;add&quot;,
            Usage:     &quot;add a task&quot;,
            Action: func(c *cli.Context) {
                fmt.Println(&quot;added task: &quot;, c.Args().First())
            },
        },
        {
            Name:      &quot;complete&quot;,
            Usage:     &quot;complete a task&quot;,
            Action: func(c *cli.Context) {
                fmt.Println(&quot;completed task: &quot;, c.Args().First())
            },
        },
    }
    app.Run(os.Args)
}
</code></pre>

<p><code>cli.NewApp()</code> возвращает указатель на структуру <code>App</code>. Эта структура выступает в роли обертки над основным функционалом и различными метаданными. Есть множество атрибутов и настроек, которые можно менять, <a href="https://github.com/codegangsta/cli/blob/f7c1cd9a11e75b5ad458628188f733a325 e14ca5/app.go#L10-L34">как вы можете видеть</a>. Но нас сейчас интересуют только <code>name</code>,<code>Usage</code>, и <code>Commands</code></p>

<p><code>app.Commands = []cli.Command {....}</code> - это добавление массива типа <code>Command</code> (<a href="https://github.com/codega ngsta/cli/blob/f7c1cd9a11e75b5ad458628188f733a325e14ca5 command.go#L9-L23">определение типа можно глянуть тут</a>). <code>Command</code> это тоже структура. <code>Name</code> - это поле, которое определяет когда запустится анонимная функция в поле <code>Action</code>. Это значит, что команда:</p>

<pre>
<code>$ godo run todo.go add &quot;Hello World!&quot;
</code></pre>

<p>выведет:</p>

<pre>
<code>added task: Hello World!
</code></pre>

<p>Очевидно, пока наше приложение не очень полезно. Давайте рассмотрим, как мы можем сохранять наши задачи.</p>

<p>Добавление задач, хранение в JSON</p>

<p>Go поставляется с отличной библиотекой для работы с <a href="http://www.json.org/">JSON</a>. Мы будем использовать ее для хранения списка задач в виде JSON файла.</p>

<p>Пакет <code>json</code> предоставляет возможность конвертировать обычные Go структуры в JSON данные. Мы можем определить структуру:</p>

<pre>
<code>type Task struct {
    Content  string
    Complete bool
}
</code></pre>

<p>И можем использовать метод <code>Marhsal</code> для конвертации структуры в JSON:</p>

<pre>
<code>m := Task{Content: &quot;Hello&quot;, Complete: true}
b, error := json.Marshal(m)
</code></pre>

<p><code>b</code> - это слайс байтов, который содержит JSON текст <code>{&quot;Content&quot;:&quot;Hello&quot;,&quot;Complete&quot;:true}</code>. Вот так все просто.</p>

<p>Добавим код структуры <code>Task</code> под импортом в нашем файле todo.go. Это будет выглядеть вот так:</p>

<pre>
<code>import (
    &quot;fmt&quot;
    &quot;github.com/codegangsta/cli&quot;
    &quot;os&quot;
)

type Task struct {
    Content  string
    Complete bool
}
</code></pre>

<p>Теперь нам нужен экземпляр структуры <code>Task</code>. Поля нужно заполнить данными от пользователя. Для этого изменим <code>Action</code> нашей add команды:</p>

<pre>
<code>app.Commands = []cli.Command{
{
    Name:      &quot;add&quot;,
    ShortName: &quot;a&quot;,
    Usage:     &quot;add a task to the list&quot;,
    Action: func(c *cli.Context) {
        task := Task{Content: c.Args().First(), Complete: false}
        fmt.Println(task)
    },
},
</code></pre>

<p>Если мы сейчас запустим наше приложение <code>go run todo.go add &quot;hello!&quot;</code>, то увидим <code>hello! false</code>. Тут нужно отметить, что по умолчанию fmt.Println не выводит название полей структуры. Для этого нужно воспользоваться функцией <code>fmt.Printf(&quot;%+v&quot;, task)</code>.</p>

<p>Можем сохранять нашу задачу как JSON файл. Не забудьте указать <code>io/ioutil</code> и <code>encoding/json</code> в импорте.</p>

<pre>
<code>task := Task{Content: c.Args().First(), Complete: false}
j, err := json.Marshal(task)
if err != nil {
    panic(err)
}
ioutil.WriteFile(&quot;tasks.json&quot;, j, 0600)
</code></pre>

<p>При добавлении нового таска, он запишется в JSON файл, который будет создан в папке с вашей программой.</p>

<p>Наверняка, вы обратили внимание, что <code>ioutil.WriteFile</code> перезаписывает файл tasks.json. Технически, мы могли бы сначала прочитать файл, сохранить его в память, дополнить новыми данными и опять записать в tasks.json. Такой подход нормально работает когда у нас не очень много данных. Но что будет, если количество задач вырастит в разы? Если их будет 10 миллионов? И сколько это займет памяти? Конечно, это не про наш случай. Но будем писать правильно сразу. Сделаем так, чтобы строки дописывались в файл.</p>

<p>Для реализации этого будем открывать файл функцией <code>os.OpenFile</code> с указанием опции <code>os.O_APPEND</code>. <code>os.OpenFile</code> возвращает ошибку, если файла не существует. Поэтому будем также указывать опцию <code>os.O_CREATE</code>. Тогда, если файл не существует, то он будет создан.</p>

<pre>
<code>Action: func(c *cli.Context) {
    task := Task{Content: c.Args().First(), Complete: false}
    j, err := json.Marshal(task)
    if err != nil {
            panic(err)
    }
    // Добавляем перенос на новую строку
    // для лучшей читабельности
    j = append(j, &quot;\n&quot;...)

    // Открываем tasks.json с опциями добавления, записи и
    // создания если не существует
    f, _ := os.OpenFile(&quot;tasks.json&quot;,
                os.O_APPEND|os.O_WRONLY|os.O_CREATE, 0600)
    // Добавляем новые данные к нашему файлу tasks.json
    if _, err = f.Write(j); err != nil {
            panic(err)
    }
},
</code></pre>

<p>При выполнении команды <code>todo add &quot;task&quot;</code>, наша программа добавит задачу в конец файла.</p>

<p>Давайте сделаем наш код более структурированным и вынесем добавление задачи в отдельную функцию.</p>

<pre>
<code>func AddTask(task Task) {
    j, err := json.Marshal(task)
    if err != nil {
        panic(err)
    }
    // Добавляем перенос на новую строку
    // для лучшей читабельности
    j = append(j, &quot;\n&quot;...)
    // Open tasks.json in append-mode.
    f, _ := os.OpenFile(&quot;tasks.json&quot;,
                os.O_APPEND|os.O_WRONLY|os.O_CREATE, 0600)
    // Append our json to tasks.json
    if _, err = f.Write(j); err != nil {
        panic(err)
    }
}
</code></pre>

<p>Теперь мы можем вызывать функцию <code>AddTask(task)</code> в нашем <code>Action</code> для добавления задач в файл.</p>

<p>Список задач</p>

<p>Мы уже умеем добавлять новые задачи в наш список, но будет намного удобней, если мы сможем просматривать задачи без необходимости открывать файл tasks.json вручную.</p>

<p>Давайте добавим новую команду которую назовем &quot;list&quot;.</p>

<pre>
<code>{
    Name:      &quot;list&quot;,
    ShortName: &quot;ls&quot;,
    Usage:     &quot;print all uncompleted tasks in list&quot;,
    Action: func(c *cli.Context) {
        ListTasks()
    },
},
</code></pre>

<p>Поле <code>ShortName</code> используется для указания сокращенного имени команды. Теперь пользователь может набирать и &quot;list&quot;, и просто &quot;ls&quot;.</p>

<p>Для отображения всех задач нам нужно выполнить итерацию по всему файлу tasks.json.</p>

<p>Теперь, наиболее простое решение это загрузить все таски из файла целиком в память как слайс. Но, как уже опоминалось, наш файл с задачами может быть очень большим. Намного предпочтительней загружать в память по одной строке, делать из ней экземпляр <code>Task</code> и сразу отображать задачу в консоли.</p>

<p>Для построчного доступа к файлу мы будем использовать пакет <a href="http://golang.org/pkg/bufio/">bufio</a>. Это позволит нам загружать только одну строку в буфер, без загрузки всего файла в память. Воспользуемся <code>buffer.Scanner</code> с помощью которого можно разбить файл на строки по указанному разделителю(по умолчанию это &ldquo;\n&rdquo;).</p>

<pre>
<code>func ListTasks() {
    // Проверяем, существует ли файл
    if _, err := os.Stat(&quot;tasks.json&quot;); os.IsNotExist(err) {
        log.Fatal(&quot;tasks file does not exist&quot;)
        return
    }
    file, err := os.Open(&quot;tasks.json&quot;)
    if err != nil {
        panic(err)
    }
    defer file.Close()
    scanner := bufio.NewScanner(file)
    // Наш индекс, который мы будем использовать как номер задачи
    i := 1
    // `scanner.Scan()` перемещает сканер к следующему разделителю  
    // и возвращает true. По умолчанию разделитель это перенос на новую
    //строку. Когда сканер доходит до конца файла, то возвращает false.
    for scanner.Scan() {
        // `scanner.Text()` возвращает текущий токен как строку
        j := scanner.Text()
        t := Task{}
        // Мы передаем в Unmarshall json строку конвертированную в байт слайс
        // и указатель на переменную типа `Task`. Поля этой переменной будут 
        // заполнены значениями из json.
        err := json.Unmarshal([]byte(j), &amp;t)
        // По умолчанию мы будем показывать только 
        // не выполненные задания
        if err != nil {
            panic(err)
        }
        if !t.Complete {
            fmt.Printf(&quot;[%d] %s\n&quot;, i, t.Content)
            i++
        }
    }
}
</code></pre>

<p>Как указанно в комментариях к коду, каждый раз когда мы вызываем <code>scanner.Scan()</code> мы перемещаем сканер на следующий токен. Цикл с одним условием будет работать пока это условие возвращает true. <code>Scan</code> возвращает true пока сканирование не закончится и false по завершению. Цикл будет работать пока мы дочитаем файл до конца.</p>

<p>Можем выполнить команду <code>go run todo.go ls</code> для просмотра всех невыполненных задач:</p>

<pre>
<code>$ todo ls
[1] Task 1
[2] Task two
[3] Task number 3
</code></pre>

<p>Выполнение задачи</p>

<p>Наконец, мы сделаем функциональность, чтобы можно было сделать задачу выполненной. У нас должна быть возможность выполнить команду:</p>

<pre>
<code>todo complete #
</code></pre>

<p># это номер задачи, которая должна быть выполненной. Стоит обратить внимание, что это число, которое отображает при запуске <code>todo ls</code>, а не реальная позиция задачи в фале tasks.json. Это потому что, когда мы выводим задачи, мы игнорируем уже выполненные задачи, наш индекс не инкрементируется.</p>

<p>Мы можем реализовать выполнение задачи несколькими способами.</p>

<p>Самый простой - это загрузить все задачи в слайс <code>[]Tasks</code>, пройтись по нему(учитывая выполненность) до задачи с нужным индексом, отметить ее как выполненную, удалить файл и записать новый с измененными задачами. Но это не очень красивый подход. К тому же, у нас опять будут проблемы с большими файлами.</p>

<p>Что если мы будем относиться к задачам в файле как к обычному тексту и просто найдем поле &quot;bool&quot; которое false и заменим его на true? Написать такое лексер или регулярку будет не так просто. А что будет если пользователь сделает вот так <code>todo add &quot;&quot;bool&quot;:true&quot;</code>? Вы никогда не будете на 100% уверенными, что это сработает. Кроме того, если строки будут разной длины, то файл будет поврежден. В общем, это весьма болезненный подход.</p>

<p>Самый безопасный способ - это читать построчно из файла с задачам, писать их в временный файл, изменить необходимую задачу, когда доберемся до нее. После этого заменить старый файл новым. Весь процесс будет выглядеть так:</p>

<ol>
	<li>Читаем каждую строку в нашем файле с задачами.</li>
	<li>Используем <code>Unmarshal</code> для создания экземпляра <code>Task</code>.</li>
	<li>Если задача не выполнена, инкрементируем индекс.
	<ol>
		<li>Проверяем, совпадает ли индекс с числом, которое указал пользователь.</li>
		<li>Если это так, устанавливаем <code>Complete</code> равным true.</li>
	</ol>
	</li>
	<li>Используем <code>Marshall</code> для преобразования переменной и записи ее в временный файл.</li>
	<li>Как только доходим до конца файла, заменяем оригинальный файл на временный.</li>
</ol>

<p>Как вы видите, большая часть функциональности уже реализована в нашем коде. Нам нужно только немного модифицировать его.</p>

<p>Запись в файл</p>

<p>Для записи задач в временный файл мы можем использовать функцию AddTask, которую мы написали ранее. Только нам нужно добавить еще один параметр, который будет определять в какой файл мы хотим записывать(tasks.json или .temp).</p>

<pre>
<code>func AddTask(task Task, filename string) {
</code></pre>

<p>Далее, в самой <code>AddTask()</code> замените строку, в которой открывается файл:</p>

<pre>
<code>f, _ := os.OpenFile(&quot;tasks.json&quot;, os.O_APPEND|os.O_WRONLY|os.O_CREATE, 0600)
</code></pre>

<p>на такую:</p>

<pre>
<code>f, _ := os.OpenFile(filename, os.O_APPEND|os.O_WRONLY|os.O_CREATE, 0600)
</code></pre>

<p>Теперь нужно модифицировать вызов функции, добавив название файла:</p>

<pre>
<code>AddTask(task, &quot;tasks.json&quot;)
</code></pre>

<p>Открытие файла</p>

<p>Так как нам нужно открывать файл tasks.json в обоих функциях ListTasks() и CompleteTasks(), то можем перенести код отвечающий за это в отдельную функцию:</p>

<pre>
<code>func OpenTaskFile() *os.File {
    // Проверяем существование файла
    if _, err := os.Stat(&quot;tasks.json&quot;); os.IsNotExist(err) {
        log.Fatal(&quot;tasks file does not exist&quot;)
        return nil
    }
    file, err := os.Open(&quot;tasks.json&quot;)
    if err != nil {
        panic(err)
    }
    return file
}
</code></pre>

<p>После модификации и добавления <code>OpenTaskFile()</code> функция <code>ListTasks()</code> будет выглядеть так:</p>

<pre>
<code>func ListTasks() {
    file := OpenTaskFile()
    defer file.Close()
    scanner := bufio.NewScanner(file)
    i := 1
    for scanner.Scan() {
        j := scanner.Text()
        t := Task{}
        err := json.Unmarshal([]byte(j), &amp;t)
        if err != nil {
                panic(err)
        }
        if !t.Complete {
                fmt.Printf(&quot;[%d] %s\n&quot;, i, t.Content)
                i++
        }
    }
}
</code></pre>

<p>Значительно красивее.</p>

<p><code>CompleteTask()</code> принимает параметр <code>idx</code>. Это индекс задачи, который указывает пользователь. В программе мы можем получить его с помощью <code>c.Args().Flag()</code>. Но эта функция возвращает строку и нам нужно конвертировать ее в int. Для этого мы будем использовать пакет <code>strconv</code>:</p>

<pre>
<code>import (
// ...
    &quot;strconv&quot;
// ...
)
</code></pre>

<p>Нам нужна функция <code>strconv.Atoi()</code> для конвертирования нашей строки в int. После конвертирования передаем это значение в <code>CompleteTask()</code>:</p>

<pre>
<code>{
    Name:  &quot;complete&quot;,
    Usage: &quot;complete a task&quot;,
    Action: func(c *cli.Context) {
        idx, err := strconv.Atoi(c.Args().First())
        if err != nil {
                panic(err)
        }
        CompleteTask(idx)
    },
},
</code></pre>

<p>Теперь можем написать код самой функции <code>CompleteTask()</code>:</p>

<pre>
<code>func CompleteTask(idx int) {
    file := OpenTaskFile()
    defer file.Close()
    scanner := bufio.NewScanner(file)
    i := 1
    for scanner.Scan() {
        j := scanner.Text()
        t := Task{}
        err := json.Unmarshal([]byte(j), &amp;t)
        if err != nil {
            panic(err)
        }
        if !t.Complete {
            if idx == i {
                t.Complete = true
            }
            i++
        }
        // Добавляем текущую задачу к временному файлу.
        // Обратите внимание, когда мы вызываем эту функцию
        // первый раз, то создается файл и записывается задача.
        AddTask(t, &quot;.tempfile&quot;)
    }
    // Когда мы записали все в .tempfile, заменяем им файл tasks.json
    os.Rename(&quot;.tempfile&quot;, &quot;tasks.json&quot;)
    // Теперь можем удалять .tempfile
    os.Remove(&quot;.tempfile&quot;)
}
</code></pre>

<p>На этом наша работа закончена. Теперь мы можем добавлять, просматривать и выполнять задачи используя наш консольный такс трекер!</p>

<p>Вы можете обратить внимание, что цикл, который читает строки в <code>CompleteTask()</code> идентичен <code>ListTasks()</code> вплоть до <code>if !t.Complete {</code>. В следующем посте рассмотрим, как перенести этот код в отдельную функцию и использовать замыкания для уменьшения дублирования кода. Кроме того, вы наверняка заметили, что в отличии от демо сверху, у нас не отображается дата добавления задачи. Это тоже будет в следующем посте.</p>

<p><a href="https://github.com/4gophers/todo">Код приложения на GitHub</a>.</p>

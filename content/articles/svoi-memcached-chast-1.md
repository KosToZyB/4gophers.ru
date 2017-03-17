+++
date = "2015-01-27T01:48:01+03:00"
draft = false
title = "Свой memcached. Часть 1"

+++

<p>Сайты становятся все больше, информация уже не влазит на терабайтные харды, а пользователи еще более требовательны к задержкам и не готовы ждать даже пары секунд ответа от сервера. К нам на помощь спешат распределенные системы обработки информации.</p>

<p>Распеределенщина и все такое - это целый неизведанный мир, со своими законами. Нахрапам не возьмешь. Но нужно с чего-то начинать. Например, <a href="http://dcg.ethz.ch/lectures/podc_allstars/">с отличного курса лекций</a>.</p>

<p>Важной частью высоко нагруженных систем является кеширование. И очень часто, это кеш делается распределенным. Используется несколько серверов, на которых сохраняются кешированные данные.</p>

<p>Memcached - одни из самых ярких представителей подобных систем кеширования. Это прекрасный продукт, который работает и достаточно неплохо справляется со своими задачами.</p>

<p>И, если хочется разобраться как все устроено, то самый верный способ - это написать свой велосипед. Конечно, вряд ли наша поделка сможет так сразу тягаться с memcached. Но несколько практически примеров будут очень к стати.</p>

<h3>Простейший сервер</h3>

<p>Начнем с написания просто tcp сервера к которому можно обратиться по telnet. Такие сервера на Go с использованием пакета <code>net</code> пишутся просто замечательно.</p>

<pre><code class="go">package main

import (
    "log"
    "net"
)

func main() {
    ln, err := net.Listen("tcp", ":11212")
    if err != nil {
        log.Println(err)
    }
    for {
        conn, err := ln.Accept()
        if err != nil {
            log.Println(err)
        }
        go connectionHandler(conn)
    }
}

func connectionHandler(conn net.Conn) {
    buf := make([]byte, 1024)
    _, err := conn.Read(buf)
    if err != nil {
        fmt.Println("Error reading:", err.Error())
    }
    conn.Write([]byte("Message received."))
    conn.Close()
}
</code></pre>

<p>Это очень простой сервер, который висит на 11212 порту, ждет когда мы ему что-то отправим. При получении сообщения отправляет нам в ответ "Message received." и закрывает соединение.</p>

<pre><code class="go">    ln, err := net.Listen("tcp", ":11212")
</code></pre>

<p>Биндимся на порт под номером 11212.</p>

<pre><code class="go">for {
    conn, err := ln.Accept()
    if err != nil {
        log.Println(err)
    }
    go connectionHandler(conn)
}
</code></pre>

<p>В бесконечном цикле ждем подключения. При новом подключении <code>ln.Accept()</code> будет создавать новое соединение <code>conn net.Conn</code> и функция <code>connectionHandler(conn)</code> будет запушена в новой go-рутине. Обратите внимание, что запуск новых рутин обеспечивает конкурентный доступ к нашему серверу.</p>

<p>После подключения клиента, начинает работать функция <code>connectionHandler</code>:</p>

<pre><code class="go">buf := make([]byte, 1024)
_, err := conn.Read(buf)
if err != nil {
    fmt.Println("Error reading:", err.Error())
}
</code></pre>

<p>В этой функции создается буфер, затем в него сохраняются данные переданные от клиента.</p>

<pre><code class="go">conn.Write([]byte("Message received."))
conn.Close()
</code></pre>

<p>После получения сообщения от клиента, нас сервер сам шлет свое сообщение "Message received." и закрывает соединение.</p>

<p>Связаться с нашим сервером можно по telnet:</p>

<pre><code class="sh">$ telnet localhost 11212
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
hi
Message received.
Connection closed by foreign host.
</code></pre>

<p>Telnet пригодится нам для дальнейшего тестирования. По крайней мере, какое-то время, пока у нас не появится свой клиент.</p>

<h3>Протокол</h3>

<p>Раз уж мы взялись писать аналог memcached, то давайте постараемся реализовать его протокол. К 100% совместимости мы стремится не будем, но постараемся максимально приближено реализовать большинство команд.</p>

<ul>
<li>get - Чтение из кэша <code>get mysimplekey</code></li>
<li>set - Запись в кэш. Пишет не проверяя есть ли в кэше значение с этим ключом: <code>set mysimplekey 0 100 50</code></li>
<li>add - Запись в кэш. Записывает только тогда, когда в кеше нет значений с таким ключом: <code>add newkey 0 20 50</code></li>
<li>replace - Замена значения записи в кеше. Работает по принципу add: <code>replace key 0 100 50</code></li>
<li>append - Записать в кеш по указанному ключу данные, перед уже находящимися там данными: <code>append key 0 100 155</code></li>
<li>prepend - Записать в кеш по указанному ключу данные, после уже находящихся там данных <code>prepend key 0 60 15</code></li>
<li>incr - Увеличивает числовое значение ключа на указанную величину: <code>incr mykey 2</code></li>
<li>decr - Уменьшает числовое значение ключа на указанную величину: <code>decr mykey 5</code></li>
<li>delete - Удаляет значение из кеша по ключу: <code>delete mykey</code></li>
<li>flush_all - Производит инвалидацю все записей в кэше: <code>flush_all</code>. Отложенная инвалидация на указанное кол-во секунд: <code>flush_all 900</code>.</li>
<li>stats - Выводит общую статистику: <code>stats</code>. Статистика использования памяти: <code>stats slabs</code>. Еще одна статистика использования памяти: <code>stats malloc</code>
Можно посмотреть список элементов в кеше: <code>stats items</code>, <code>stats detail [on|off|dump]</code>, <code>stats sizes</code>. Сброс статистики <code>stats reset</code></li>
<li>version - Показывает версию memcached: <code>version</code></li>
<li>verbosity - Установка уровня детализации логирования: <code>verbosity</code></li>
<li>quit - Закрыть телнет сессию: <code>quit</code></li>
</ul>

<p>Полное описание протокола <a href="https://github.com/memcached/memcached/blob/master/doc/protocol.txt">можно найти на github</a></p>

<p>Стоить заметить, что если нам не обязательна поддержка memcahced протокола и все клиенты будут написаны на Go, то можно было бы реализовать более специфический протокол. Более того, мы могли бы не заморачиваться над текстовым протоколом,а придумать что то более в стиле Go. Пример такого протокола рассмотрим в конце серии.</p>

<h3>Основные команды</h3>

<p>Давайте начнем с реализации двух базовых команд: <code>get</code> и <code>set</code>. Заодно, определимся с принципами работы.</p>

<p>Нужно научится разделать команды. Для этого в <code>func connectionHandler(conn net.Conn)</code> у нас будет бесконечный цикл, в котором будут читаться данные из соединения. Эти данные будут проверятся на наличие команд.</p>

<pre><code class="go">func connectionHandler(conn net.Conn) {
    for {
        command, err := bufio.NewReader(conn).ReadString('\n')
        if err != nil {
            if err == io.EOF {
                log.Println("Error io.EOF", err)
                break
            } else {
                log.Println("Error reading:", err)
            }
        }
        if strings.HasPrefix(command, "set") {
            // выполняем команду set
        }
        if strings.HasPrefix(command, "get") {
            // выполняем команду get
        }
    }
}
</code></pre>

<p><code>bufio.NewReader(conn).ReadString('\n')</code> постоянно пытаться прочитать строку из соединения до символа <code>'\n'</code>. В зависимости от того как начинается эта строка будет срабатывать один из ифов. Давайте допишем немного кода, чтоб сервер возвращал данные в соединение сигнализируя о сработавшей команде.</p>

<pre><code class="go">if strings.HasPrefix(command, "set") {
    conn.Write([]byte("answer: set command\n"))
}

if strings.HasPrefix(command, "get") {
    conn.Write([]byte("answer: get command\n"))
}
</code></pre>

<p>Теперь попробуем подключится по telnet и выполнить пару команд:</p>

<pre><code class="sh">telnet localhost 11212
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
set key
answer: set command
get key
answer: get command
</code></pre>

<h4>set</h4>

<p>Принцип работы с соединением должен быть ясен. Теперь можно описывать наши команды. Начнем с того, что команда должна получить и сохранить некоторую информацию - ключ и сами данные. Для команды установки кеша(set) можно определить такую структуру:</p>

<pre><code class="go">// Структура для команды set
type SetCommand struct {
    Key     string
    Length  int
    Text    string
    Conn    net.Conn
}
</code></pre>

<ul>
<li><code>Key</code> - Ключ по которому будет сохраняться ключ</li>
<li><code>Length</code> - Количество данных в байтах, которые нужно сохранить</li>
<li><code>Text</code> - Строка самой команды</li>
<li><code>Conn</code> - Объект соединения. В рамках выполнения команды необходимо будет отправлять ответы пользователю.</li>
</ul>

<p>Со временем мы будем добавлять новые параметры команды и, соответственно, новые поля в структуре.</p>

<p>Всю логику этой команды можно реализовать внутри метода <code>Run()</code>.</p>

<pre><code class="go">func (s *SetCommand) Run() {
    err := ParseTextCommand(s.Text, s, func() error {
        _, err := fmt.Sscanf(s.Text, "set %s %d\n", &amp;s.Key, &amp;s.Length)
        return err
    })

    if err == nil {
        data, _ := bufio.NewReader(s.Conn).ReadBytes('\n')
        storage[s.Key] = data
        s.Conn.Write([]byte("STORED\n"))
    }
}
</code></pre>

<p>Переменная <code>storage</code> - это переменная в глобальной области видимости которая имеет тип <code>map[string][]byte</code>. По сути - это хранилище для нашего кеша. Она инициализируется в функции <code>init()</code> нашего модуля.</p>

<pre><code class="go">var storage map[string][]byte

func init() {
    storage = make(map[string][]byte)
}
</code></pre>

<p>Чтобы со всеми командами можно было единообразно работать нужно определить общий интерфейс. У всех наших команд будет как минимум метод <code>Run()</code>. А значит нам подойдет интерфейс вида:</p>

<pre><code class="go">type Command interface {
    Run()
}
</code></pre>

<p>Теперь мы можем использовать  <code>*SetCommand</code> как экземпляр с интерфейсом <code>Command</code>. Обратите внимание, что я написал именно указатель, так как в нашем случае метод <code>Run()</code> определен именно для указателя. Если мы создадим экземпляр команды таким образом:</p>

<pre><code class="go">set := SetCommand{
    Text: command,
    Conn: conn,
}
</code></pre>

<p>То не сможем его использовать как интерфейс <code>Command</code>. Нужно получить именно указатель на структуру а не саму структуру:</p>

<pre><code class="go">set := &amp;SetCommand{
    Text: command,
    Conn: conn,
}
</code></pre>

<p>Обратите внимание на выражение:</p>

<pre><code class="go">err := ParseTextCommand(s.Text, s, func() error {
    _, err := fmt.Sscanf(s.Text, "set %s\n", &amp;s.Key)
    return err
})
</code></pre>

<p><code>ParseTextCommand</code> - это вспомогательная функция, которая обеспечивает разбор строки в поля структуры. Основная логика описана в замыкании, а внутри <code>ParseTextCommand</code> выполняется вспомогательная работа(логирование, отлавливание ошибок и т.д).</p>

<p>В результате работы функции <code>fmt.Sscanf(s.Text, "set %s\n", &amp;s.Key)</code> в <code>s.Key</code> записывается значение ключа. Теперь можно приступать к чтению основных данных для кеширования.</p>

<pre><code class="go">data, _ := bufio.NewReader(s.Conn).Peek(s.Length)
storage[s.Key] = data
s.Conn.Write([]byte("STORED\n"))
</code></pre>

<p>Нам нужно учесть возможные ошибки. Данных может быть отправлено больше или меньше чем указанно в поле <code>s.Length</code>. Если меньше, то мы получим ошибку при вызове <code>Peek(s.Length)</code></p>

<pre><code class="go">reader := bufio.NewReader(s.Conn)
data, err := reader.Peek(s.Length)

if (err != nil) {
    s.Conn.Write([]byte(CLIENT_ERROR + "\n"))
    return
}
</code></pre>

<p>А чтобы определить конец сообщения, договоримся что каждое сообщения должно завершаться комбинацией <code>\r\n</code> - как это реализованно в самом memcached.</p>

<pre><code class="go">control, err := reader.Peek(s.Length + 2)
if (err != nil) {
    s.Conn.Write([]byte(CLIENT_ERROR + "\n"))
    return
}

if !strings.HasSuffix(string(control), "\r\n") {
    s.Conn.Write([]byte(CLIENT_ERROR + "\n"))
    return
}
</code></pre>

<p>На этом пока закончим функционал для команды set. В будущем нужно будет добавить время хранения кеша и поле флагов для более точного соответствия протоколу memcached.</p>

<h4>get</h4>

<p>Поняв общую логику написания команд значительно проще писать остальные команды. В get будет чуть меньше параметров и сама логика работы будет сравнительно проще. Напишем структуру с такими полями:</p>

<pre><code class="go">type GetCommand struct {
    Name string
    Key  string
    Text string
    Conn net.Conn
}
</code></pre>

<p>А всю логику реализуем в методе <code>Run()</code>, которая будет заключаться в получении значения по ключу. Обратите внимание, что в начале и в конце добавляются специальные данные.</p>

<pre><code class="go">func (g *GetCommand) Run() {
    err := ParseTextCommand(g.Text, g, func() error {
        _, err := fmt.Sscanf(g.Text, "get %s\n", &amp;g.Key)
        return err
    })

    if err == nil {
        data, ok := storage[g.Key]
        if ok {
            g.Conn.Write([]byte("VALUE " + g.Key + "\r\n"))
            g.Conn.Write(data)
            g.Conn.Write([]byte("\r\n"))
            g.Conn.Write([]byte("END\r\n"))
        }
    }
}
</code></pre>

<p>Есть один нюанс. На самом деле нам нужно запоминать и возвращать не только сами данные, но всякую и служебную информацию, такую как длину данных и время жизни кеша(на будущее). Кроме того, в стандартном проколе при записи кеша используется дополнительное полк ключей для команды set</p>

<p>Это решается довольно просто. Мы будем хранить в мапе <code>storage</code> не просто набор байтов а структуры с дополнительной метаинформацией:</p>

<pre><code class="go">type Item struct {
    Key     string
    Flags   int32
    Exptime int
    Length  int
    Data    []byte
}
</code></pre>

<p>Нам нужно немного поправить команду set, и заменить <code>var storage map[string][]byte</code> на <code>var storage map[string]Item</code>:</p>

<pre><code class="go">storage[s.Key] = Item{Key:s.Key, Length:s.Length, Data:data}
</code></pre>

<p>Поля <code>Flags</code> и <code>Exptime</code> мы пока не используем. Вносим изменения в метод <code>Run()</code> для команды get:</p>

<pre><code class="go">g.Conn.Write([]byte("VALUE " + g.Key + " " + strconv.Itoa(data.Length) + "\r\n"))
g.Conn.Write(data.Data)
g.Conn.Write([]byte("\r\n"))
g.Conn.Write([]byte("END\r\n"))
</code></pre>

<p>Теперь можем собрать получившийся сервер и протестировать как он работает с помощью все того же telnet:</p>

<pre><code class="sh">telnet localhost 11212
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
set key 4
nnnn
STORED
get key
VALUE key 4
nnnn
END
</code></pre>

<p>Все получилось, как и задумывалось.</p>

<h3>Конкурентный мап</h3>

<p>У нашего кеш-сервера есть один очень существенный недостаток - способ записи данных в память. Мы используем простую запись в map. Но наше приложение работает конкурентно, каждое соединение - это новая go-рутина. И при одновременном доступе к <code>storage</code> из двух рутин могут возникнуть большие проблемы проблемы.</p>

<p>Чтобы избежать конфликтов при доступе к элементам переменной типа map, нам нужно написать свой интерфейс, с помощью которого мы будем выполнять необходимые манипуляции с элементами этого самого map.</p>

<p>Для обеспечения безопасного использования отображения мы вынесем его в отдельный пакет и сделаем не экспортируемым, доступ к которому можно будет получить только через каналы. Именно каналы будут обеспечивать очередность доступа к значениям map которое, в свою очередь, будет завернуто в не экспортируемый метод. Этот метод будет выполнять бесконечный цикл, блокирующийся до получения команд ("вставить это значение", "удалить этот элемент" и т.д.). Для начала рассмотрим интерфейс <code>SafeMap</code>, потом разберемся с методами типа <code>safeMap</code>, затем с функцией <code>New()</code> из пакета <code>safemap</code> и в конце с не экспортируемым методом <code>safeMap.run()</code>.</p>

<pre><code class="go">type SafeMap interface {
    Insert(string, interface{})
    Delete(string)
    Find(string) (interface{}, bool)
    Len() int
    Update(string, UpdateFunc)
    Close() map[string]interface
    interface{}
}
type UpdateFunc func(interface{}, bool) interface{}
</code></pre>

<p>Все эти методы реализуются типом <code>safeMap</code>.</p>

<p>Тип <code>UpdateFunc</code> определяет сигнатуру функции обновления: она будет рассматриваться после знакомства с методом <code>Update()</code> ниже.</p>

<pre><code class="go">type safeMap chan commandData
type commandData struct {
    action  commandAction
    key     string
    value   interface{}
    result  chan&lt;- interface{}
    data    chan&lt;- map[string]interface{}
    updater UpdateFunc
}

type commandAction int
const (
    remove commandAction = iota
    end
    find
    insert
    length
    update
)
</code></pre>

<p><code>safeMap</code>, по сути, это канал, в который можно посылать и из которого можно принимать значения типа <code>commandData</code>. Когда в канал приходит значение <code>commandData</code>, то оно определяет какую операцию нужно выполнять и какие данные использовать(например ключ). Подробности про поля увидим дальше.</p>

<p>Обратите внимание, что каналы <code>result</code> и <code>data</code>, объявлены как однонаправленные. Это значит что наш поточно-безопасный map может только посылать значения в них, но не может принимать из них. Дальше будет видно, что эти каналы создаются как двунаправленные, но в рамках нашего типа нет необходимости что то принимать из этих каналов, но их можно использовать для приема вне <code>safeMap</code>.</p>

<pre><code class="go">func (sm safeMap) Insert(key string, value interface{}) {
    sm &lt;- commandData{action: insert, key: key, value: value}
}
</code></pre>

<p>Это ни что иное, как безопасный эквивалент инструкции <code>m[key] = value</code>, где <code>m</code> - значение типа <code>map[string]interface{}</code>. Метод создает значение типа <code>commandData</code> с командой <code>insert</code>, указанным ключом <code>key</code> и значением <code>value</code>, и посылает его в поточно-ориентированный map, который, как было показано выше, имеет тип <code>chan commandData</code>.</p>

<p>Когда будем рассматривать метод <code>New()</code> из пакета <code>safemap</code> увидим, что <code>safeMap</code> возвращается функцией <code>New()</code> как интерфейс <code>SafeMap</code> и уже привязанным к go-рутине. Метод <code>safeMap.run()</code> выполняется в отдельной рутине в рамках замыкания. Этот метод также содержит в себе реальный map, используемый для хранения элементов, и цикл <code>for</code>, который производит итерации по элементам в канале <code>safeMap</code> и выполняет команды, принимаемые из канала.</p>

<pre><code class="go">func (sm safeMap) Delete(key string) {
    sm &lt;- commandData{action: remove, key: key}
}
</code></pre>

<p>Этот метод посылает команду на удаление элемента с указанным
ключом.</p>

<pre><code class="go">type findResult struct {
    interface{}
    value interface
    found bool
}
func (sm safeMap) Find(key string) (value interface{}, found bool) {
    reply := make(chan interface{})
    sm &lt;- commandData{action: find, key: key, result: reply}
    result := (&lt;-reply).(findResult)
    return result.value, result.found
}
</code></pre>

<p>В методе <code>safeMap.Find()</code> создается канал <code>reply</code>, с помощью которого можно получить значение из нашего безопасного map. Для этого в методе посылается команда <code>find</code> с нужным ключем и с указанием канала <code>reply</code>. Так как это не буферизированный канал, то операция блокируется пока безопасный map не обработает все запросы. После отправки запроса, метод получает ответ в виде структуры <code>findResult</code>. Из этой структуры мы можем получить необходимые поля и вернуть из как результат работы метода.</p>

<pre><code class="go">func (sm safeMap) Len() int {
    reply := make(chan interface{})
    sm &lt;- commandData{action: length, result: reply}
    return (&lt;-reply).(int)
}
</code></pre>

<p>Принцип работы этого метода очень похож на <code>Find()</code>. Аналогичным образом создается канал <code>reply</code>, из которого забирается результат.</p>

<pre><code class="go">func (sm safeMap) Update(key string, updater UpdateFunc) {
    sm &lt;- commandData{action: update, key: key, updater: updater}
}
</code></pre>

<p>В методе создается и отправляется команда <code>update</code> с указанным ключем <code>key</code> и функцией <code>updater</code>. В момент выполнения операции вызовется функция <code>updater</code> в которую будет передано значение элемента по указанному ключи и булевый параметр, указывающий найдено ли такое значение в map или нет. Самому элементу будет присвоено новое значение, которое вернет функция <code>updater</code>. Если элемента по такому ключу раньше не существовало, то будет создан новый элемент.</p>

<p>Важное замечание - если в <code>updater</code> будут вызываться методы <code>safeMap</code>, то есть вероятность взаимоблокировок. Причина этого в методе <code>safemap.safeMap.run()</code></p>

<p>Казалось бы, у нас уже есть методы <code>Insert()</code>, <code>Delete()</code> и <code>Find()</code>, зачем нам еще один метод? Его нужно использовать, когда возникает необходимость не просто записать новое значение, а изменить старое. Например, если у нас в map хранятся цены на товары и цена одного из товаров возросла на 5%. Если мы используем обычный map, то достаточно просто написать <code>m[key] *= 1.05</code> - значение элемента будет увеличено на 5%, если такого элемента не существует, то создастся новый элемент с нулевым значением. В нашем случае, подобную операцию можно реализовать с помощью метода <code>Update()</code>.</p>

<pre><code class="go">if price, found := priceMap.Find(part); found { // ОШИБКА!
    priceMap.Insert(part, price.(float64)*1.05)
}
</code></pre>

<p>Проблема в том, что используя вызов двух методов для записи значения, мы нарушаем атомарность операции. Нет никакой уверенности, что между вызовом <code>Find()</code> и <code>Insert()</code> какая ни будь другая go-рутина не изменит значение.</p>

<p>Именно атомарность метода <code>Update()</code> обеспечивает безопасное сохранение значения.</p>

<pre><code class="go">priceMap.Update(part, func(price interface{}, found bool) interface{} {
    if found {
        return price.(float64) * 1.05
    }
    return 0.0
})
</code></pre>

<p>Если элемент с указанным ключом отсутствует, будет создан новый элемент со значением 0.0. Иначе существующие значение будет увеличено на 5%.</p>

<pre><code class="go">func (sm safeMap) Close() map[string]interface{} {
    reply := make(chan map[string]interface{})
    sm &lt;- commandData{action: end, data: reply}
    return &lt;-reply
}
</code></pre>

<p>Метод <code>Close()</code> закрывает канал <code>safeMap</code> внутри метода <code>safeMap.run()</code>, после этого завершается цикл в методе <code>safeMap.run()</code>. Затем, метод <code>Close()</code> возвращает используемый <code>map[string]interface{}</code>, который может быть использован в рамках программы. Это метод может быть вызван только один раз, несмотря на то, сколько go-рутин используют наш безопасный map.</p>

<p>Теперь глянем на функцию <code>New()</code> из пакета <code>safemap</code>. Эта функция создает значение типа <code>safeMap</code> с интерфейсом <code>SafeMap</code>. Этот интерфейс можно использовать вне модуля и внутри метода <code>safeMap.run()</code>, в котором находится используемый канал и фактически <code>map[string]interface{}</code> для хранения данных.</p>

<pre><code class="go">func New() SafeMap {
    sm := make(safeMap) // тип safeMap chan commandData
    go sm.run()
    return sm
}
</code></pre>

<p><code>safeMap</code> - по сути это канал, поэтому для его создания  и получения ссылки необходимо использовать встроенную функцию <code>make()</code>. Сразу после создания <code>safeMap</code> вызывается метод <code>run()</code>, внутри которого создается обычный map. Метод <code>run()</code> запускается в go-рутине. В конце функция <code>New()</code> возвращает созданный <code>safeMap</code> как экземпляр с интерфейсом <code>SafeMap</code>.</p>

<pre><code class="go">func (sm safeMap) run() {
    store := make(map[string]interface{})
    for command := range sm {
        switch command.action {
        case insert:
            store[command.key] = command.value
        case remove:
            delete(store, command.key)
        case find:
            value, found := store[command.key]
            command.result &lt;- findResult{value, found}
        case length:
            command.result &lt;- len(store)
        case update:
            value, found := store[command.key]
            store[command.key] = command.updater(value, found)
        case end:
            close(sm)
            command.data &lt;- store
        }
    }
}
</code></pre>

<p>После создания обычного map метод <code>run()</code> запускает бесконечный цикл, который пытается получить значения из канала. Таким образом, цикл блокируется, если в канале нет ни одного значения.</p>

<p>В принципе, обработка команд очень проста - это все стандартные операции работы с map, кроме команды <code>update</code>. В случае этой команды, элементу присваивается значение, которое вернет функция <code>command.updater()</code>.</p>

<pre><code class="go">//...
case end:
    close(sm)
    command.data &lt;- store
}
//...
</code></pre>

<p>Этот код срабатывает при вызове метода <code>Close()</code>. Закрывается канал <code>safeMap</code>, и реальный map отправляется в канал результатов.</p>

<p>Самое важное при написании функции <code>command.updater()</code> - это избегать вызовов методов из <code>safeMap</code>, так как произойдет блокировка. Обработка команды <code>update</code> не может завершиться, пока <code>command.updater()</code> не вернет управление, но если функция вызовет метод типа <code>safeMap</code>, этот вызов заблокируется в ожидании завершения обработки текущей команды, и ни то, ни другое не смогут завершиться.</p>

<p>Теперь вернемся к нашему кеш серверу. Нам нужно изменить операцию добавления нового элемента в кеш и операцию получения данных из кеша. Начнем с нового определения нашей переменной <code>storage</code></p>

<pre><code class="go">var storage safemap.SafeMap

func init() {
    storage = safemap.New()
}
</code></pre>

<p>Теперь изменяем метод <code>Run()</code> для команды set:</p>

<pre><code class="go">storage.Insert(s.Key, Item{Key: s.Key, Length: s.Length, Data: data})
</code></pre>

<p>И последним изменяем метод <code>Run()</code> для команды get. Обратите внимание на выражение <code>item := data.(Item)</code> - нам приходится кастовать тип. Приводить переменную типа <code>interface{}</code> к типу <code>Item{}</code></p>

<pre><code class="go">data, ok := storage.Find(g.Key)
item := data.(Item)
if ok {
    // Необходимо для адекватного переноса, так как при считывании
    // последний перенос не учитывался
    g.Conn.Write([]byte("VALUE " + g.Key + " " + strconv.Itoa(item.Length) + "\r\n"))
    g.Conn.Write(item.Data)
    g.Conn.Write([]byte("\r\n"))
    g.Conn.Write([]byte("END\r\n"))
}
</code></pre>

<h3>Заключение</h3>

<p>В этой части мы рассмотрели как реализовать сервер, способный выполнять команды по сети. Сделали небольшой прототип своего memcached и узнали как писать потокобезопасные программы, основанные на канал и рутинах.</p>

<p>В следующей части мы продолжим реализовывать команды из протокола memcached. Также, нам нужно будет написать клиентскую библиотеку и посмотреть, как этот кеш-сервер можно использовать совместно с распределенным приложением.</p>

<p>Исходный код примера можно <a href="https://github.com/4gophers/cache">посмотреть на гитхабе</a>.</p>

<h3>Примечания</h3>

<p>Для статьи был использован код из примеров к книге "Programming in Go: Creating Applications for the 21st Century (Developer's Library)" Mark Summerfield.</p>

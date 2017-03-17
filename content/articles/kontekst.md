+++
date = "2014-08-25T16:27:04+03:00"
draft = false
title = "Контекст"

+++

<p>Перевод статьи от Sameer Ajmani <a href="http://blog.golang.org/context">Go Concurrency Patterns: Context</a> из официального блога Go.</p>

<h3>Введение</h3>

<p>В Go сервере каждый входящий запрос обрабатывается в отдельной go-рутине. Обработчики запросов часто стартуют свои рутины для доступа к базам данным, сторонним АПИ и т.д. Всем этим рутинам, как правило, нужен доступ к спецефичным для этого запроса параметрам, таким как идентификатор пользователя, токены авторизации и время завершения запроса(request&#39;s deadline). Кроме того, эти go-рутины, работающие в рамках этого запроса, должны быстро завершаться и освобождать ресурсы.</p>

<p>В Google мы разработали пакет <code>context</code>, который упрощает передачу запросозависимых значений, сигналов отмены, и время завершения для всех go-рутин запущенных в рамках этого запроса. Сам пакет доступен тут <a href="http://code.google.com/p/go.net/context">code.google.com/p/go.net/context</a>. В этой статье описано как использовать этот пакет и показаны несколько примеров работы с ним.</p>

<h3>Контекст(Context)</h3>

<p>Самая важная часть пакета <code>context</code> это тип <code>Context</code>:</p>

<pre>
<code>// Контекст предоставляет механизм дедлайнов, сигнал отмены, и доступ к запросозависимым значениям.
// Эти методы безопасны для одновременного использования в разных go-рутинах.
type Context interface {
    // Done возвращает канал, который закрывается когда Context отменяется
    // или по таймауту.
    Done() &lt;-chan struct{}

    // Err объясняет почему контекст был отменен, после того как закрылся канал Done.
    Err() error

    // Deadline возвращает время когда этот Context будет отменен.
    Deadline() (deadline time.Time, ok bool)

    // Value возвращает значение ассоциированное с ключем или nil.
    Value(key interface{}) interface{}
}
</code></pre>

<p>(Это выжимка из <a href="http://godoc.org/code.google.com/p/go.net/context">godoc</a>. Там можно найти больше)</p>

<p>Метод <code>Done</code> возвращает канал который действует как сигнал отмены для функций запущенных от имени <code>Context</code>. Когда канал закрывается, функции должны завершить работу. Метод <code>Err</code> возвращает ошибку, которая объясняет, почему <code>Context</code> был отменен. В статье &quot;<a href="http://blog.golang.org/pipelines">Pipelines and Cancelation</a>&quot; идиома <code>Done</code> каналов объясняется более подробно.</p>

<p>У контекста нет метода <code>Cancel</code> по той же причине, по которой канал <code>Done</code> работает только на прием. Функция, которая принимает сигнал отмены, как правило, совсем не та что его отправляет. В частности, когда родительская операция стартует go-рутину для подоперации эта подоперация не должна иметь возможность отменить работу родителя. Вместо этого функция <code>WithCancel</code>(которая описана ниже) предоставляет возможность отменить новое значение <code>Context</code>.</p>

<p><code>Context</code> безопасен для использования в множестве go-рутин. Мы можем передать один <code>Context</code> любому количеству go-рутин и отметить этот <code>Context</code> по сигналу любой из них.</p>

<p>Метод <code>Deadline</code> позволяет функциям определить должны ли они начать работу. Если осталось слишком мало времени, это уже может быть не целесообразным. Так же, можно использовать дедлайны для установки таймаута для операций ввода/вывода.</p>

<p><code>Value</code> позволяет в <code>Context</code> пользоваться запросозависимыми данными. Данные должны быть безопасны для одновременного использованием множеством go-рутин.</p>

<h3>Производные контексты</h3>

<p>Пакет <code>context</code> предоставляет возможность производить новые значения <code>Context</code> из существующего. Все эти значения образуют дерево и в случае отмены родительского <code>Context</code> все производные тоже будут отменены.</p>

<p><code>Background</code> это корень для всего дерева `Context и он никогда не отменяется:</p>

<pre>
<code>// Background возвращает пустой Context. Он никогда не будет отменен и не имеет deadline
// и значений. Background обычно используется в main, init, тестах и как верхний уровень
// Context для входящих запросов.
func Background() Context
</code></pre>

<p><code>WithCancel</code> и <code>WithTimeout</code> возвращают полученное значение <code>Context</code> которое может быть отменено раньше чем родительский контекст. <code>Context</code> ассоциированный с входящим запросом, как правило, отменяется когда обработчик запроса завершает работу. <code>WithCancel</code> удобен для отмены излишних запросов, когда используется несколько реплик. <code>WithTimeout</code> удобно использовать для установки дедлайна в запросе к серверу:</p>

<pre>
<code>// WithCancel возвращает копию родителя, в котором Done закрывается как только 
// parent.Done будет закрыт или контекст будет отменен.
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)

// CancelFunc отменяет контекст.
type CancelFunc func()

// WithTimeout возвразщает копию родителя, в котором Done закрывается как только 
// parent.Done будет закрыт, контекст будет отменен или закончится таймаут. Новый дедлайн 
// контекста состоит из текущее время + таймаут и родительский дедлайн если такой имеется.
// Если таймер все еще работает, функция отмены релизит свои ресурсы.
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
</code></pre>

<p><code>WithValue</code> предоставляет возможность привязать запросозависимые значения к контексту.</p>

<pre>
<code>// WithValue возвращает копию родителя, в которой метод Value возвращает значение по ключу.
func WithValue(parent Context, key interface{}, val interface{}) Context
</code></pre>

<p>Самый лучший способ разобраться как работает пакет <code>context</code> это посмотреть на него в действии.</p>

<h3>Пример: Google Web Search</h3>

<p>В качестве примера у нас HTTP сервер, который обрабатывает URLы в таком формате <code>/search?q=golang&amp;timeout=1s</code> передает запрос из параметра &quot;golang&quot; в <a href="https://developers.google.com/web-search/docs/">Google Web Search API</a> и отображает результат. Параметр &quot;timeout&quot; говорит нашему серверу через какое время прекратить запрос.</p>

<p>Весь код разделен на три пакета:</p>

<ul>
	<li><a href="http://blog.golang.org/context/server/server.go">server</a> в этом пакете определена функция main и обработчики для <code>/search</code>.</li>
	<li><a href="http://blog.golang.org/context/userip/userip.go">userip</a> предоставляет функции для получения клиентского IP из запроса и привязка его к <code>Context</code>.</li>
	<li><a href="http://blog.golang.org/context/google/google.go">google</a> тут определена функция <code>Search</code> для отправки запроса в сервис Google.</li>
</ul>

<p>Пакет server</p>

<p>Пакет <a href="http://blog.golang.org/context/server/server.go">server</a> обрабатывает запросы вида <code>/search?q=golang</code> для получения первых нескольких результатов из Google в golang. В это пакете регистрируется <code>handleSearch</code> для обработки всех запросов к <code>/search</code>. Обработчик создает начальный <code>Context</code> с названием <code>ctx</code> и подготавливает его к закрытию после завершения работы обработчика. Если запрос включает параметр <code>timeout</code>, тогда <code>Context</code> будет автоматически отменен по завершению таймаута:</p>

<pre>
<code>func handleSearch(w http.ResponseWriter, req *http.Request) {
    // ctx это Context для этого обработчика. Отмена контекста (вызов cancel)
    // закрывает канал ctx.Done что является сигналом отмены
    // для запросов запущенных в этом обработчике.
    var (
        ctx    context.Context
        cancel context.CancelFunc
    )
    timeout, err := time.ParseDuration(req.FormValue(&quot;timeout&quot;))
    if err == nil {
        // В запросе есть параметр timeout, в таком случае создаем
        // контекст который будет отменен автоматически по
        // окончанию таймаута.
        ctx, cancel = context.WithTimeout(context.Background(), timeout)
    } else {
        ctx, cancel = context.WithCancel(context.Background())
    }
    defer cancel() // Отменяем ctx как только handleSearch закончит работу.
</code></pre>

<p>Обработчик извлекает поисковый запрос и клиентский IP адрес с помощью пакета <code>userip</code>. Клиентский IP нужен для выполнения запросов к АПИ Google. В результате <code>handleSearch</code> аттачит все это к <code>ctx</code></p>

<pre>
<code>    // Получаем посковый запрос.
    query := req.FormValue(&quot;q&quot;)
    if query == &quot;&quot; {
        http.Error(w, &quot;no query&quot;, http.StatusBadRequest)
        return
    }

    // Записываем пользовательский IP в ctx для использования в других пакетах.
    userIP, err := userip.FromRequest(req)
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    ctx = userip.NewContext(ctx, userIP)
</code></pre>

<p>Обработчик вызывает <code>google.Search</code> и передает <code>ctx</code> и <code>query</code>:</p>

<pre>
<code>    // Запускаем Google поиск и выводим результаты.
    start := time.Now()
    results, err := google.Search(ctx, query)
    elapsed := time.Since(start)
</code></pre>

<p>Если поиск отработал нормально, тогда обработчик выводит результаты:</p>

<pre>
<code>    if err := resultsTemplate.Execute(w, struct {
        Results          google.Results
        Timeout, Elapsed time.Duration
    }{
        Results: results,
        Timeout: timeout,
        Elapsed: elapsed,
    }); err != nil {
        log.Print(err)
        return
    }
</code></pre>

<p>Пакет userip</p>

<p>Пакет <a href="http://blog.golang.org/context/userip/userip.go">userip</a> предоставляет функции для извлчения IP адреса из запроса и привязки его к <code>Context</code>. Сам <code>Context</code> дает возможность мапинга ключ/значение, в котором ключи и значения имеют тип <code>interface{}</code>. Все типы ключей должны поддерживать сравнение и все типы значений должны быть безопасными для использования в нескольких go-рутинах. Такие пакеты как <code>userip</code> должны скрывать подробности реализации этого мапинга и предоставлять строго типизированный доступ к значениям в <code>Context</code>.</p>

<p>Для избежания коллизий с ключами, в пакете <code>userip</code> определена константа <code>key</code> которая используется как ключ для получения значения из контекста:</p>

<pre>
<code>// Тип key не экспортируемый для предотвращения коллизий к другими ключами определенными
// в других пакетах.
type key int

// userIPkey это ключ контекста для клиентского IP. Его значение равно нулю.
// Если в этом пакете определить другие ключи, они могут иметь различные
// целочисленные значения.
const userIPKey key = 0
</code></pre>

<p><code>FromRequest</code> извлекает значение <code>userIP</code> из <code>http.Request</code>:</p>

<pre>
<code>func FromRequest(req *http.Request) (net.IP, error) {
    s := strings.SplitN(req.RemoteAddr, &quot;:&quot;, 2)
    userIP := net.ParseIP(s[0])
    if userIP == nil {
        return nil, fmt.Errorf(&quot;userip: %q is not IP:port&quot;, req.RemoteAddr)
    }
</code></pre>

<p><code>NewContext</code> возвращает новый <code>Context</code> который содержит значение <code>userIP</code>:</p>

<pre>
<code>func NewContext(ctx context.Context, userIP net.IP) context.Context {
    return context.WithValue(ctx, userIPKey, userIP)
}
</code></pre>

<p><code>FromContext</code> извлекает <code>userIP</code> из<code>Context</code>:</p>

<pre>
<code>func FromContext(ctx context.Context) (net.IP, bool) {
    // ctx.Value возвращает nil если ctx не имеет значение для этого ключа;
    // Приведение к типу net.IP возвращает ok=false для значения nil.
    userIP, ok := ctx.Value(userIPKey).(net.IP)
    return userIP, ok
}
</code></pre>

<p>Пакет google</p>

<p>Функция <a href="http://blog.golang.org/context/google/google.go">google.Search</a> отправляет HTTP запрос к <a href="https://developers.google.com/web-search/docs/">Google Web Search API</a> и парсит результат в формате JSON. Он принимает <code>Context</code> параметр <code>ctx</code> и возвращает результат немедленно если <code>ctx.Done</code> будет закрыт пока запрос выполняется.</p>

<p>Запрос Google Web Search API содержит поисковый запрос и пользовательский IP:</p>

<pre>
<code>func Search(ctx context.Context, query string) (Results, error) {
    // Подготовка запроса к Google Search API.
    req, err := http.NewRequest(&quot;GET&quot;, &quot;https://ajax.googleapis.com/ajax/services/search/web?v=1.0&quot;, nil)
    if err != nil {
        return nil, err
    }
    q := req.URL.Query()
    q.Set(&quot;q&quot;, query)
    // Если ctx содержит пользовательский IP, передаем его серверу.
    // Google API использует пользовательский IP чтобы отличить запрос с сервера
    // от пользовательского запроса.
    if userIP, ok := userip.FromContext(ctx); ok {
        q.Set(&quot;userip&quot;, userIP.String())
    }
    req.URL.RawQuery = q.Encode()
</code></pre>

<p><code>Search</code> использует вспомогательную функцию <code>httpDo</code> для создания HTTP запроса и его отмены, если <code>ctx.Done</code> закроется во время обработки запроса или ответа. <code>Search</code> передает замыкание в <code>httpDo</code>, которое в качестве параметра принимает HTTP запрос:</p>

<pre>
<code>    var results Results
    err = httpDo(ctx, req, func(resp *http.Response, err error) error {
        if err != nil {
            return err
        }
        defer resp.Body.Close()
        // Разбираем результат в формате JSON.
        // https://developers.google.com/web-search/docs/#fonje
        var data struct {
            ResponseData struct {
                Results []struct {
                    TitleNoFormatting string
                    URL               string
                }
            }
        }
        if err := json.NewDecoder(resp.Body).Decode(&amp;data); err != nil {
            return err
        }
        for _, res := range data.ResponseData.Results {
            results = append(results, Result{Title: res.TitleNoFormatting, URL: res.URL})
        }
        return nil
    })
    // httpDo waits for the closure we provided to return, so it&#39;s safe to
    // read results here.
    return results, err
</code></pre>

<p>Функция <code>httpDo</code> запускает HTTP запрос и обрабатывает ответ в новой go-рутине. Запрос будет отменен если <code>ctx.Done</code> будет отменен до выхода из go-рутины:</p>

<pre>
<code>func httpDo(ctx context.Context, req *http.Request, f func(*http.Response, error) error) error {
    // Запускаем HTTP запрос в go-рутине и передаем запрос в f.
    tr := &amp;http.Transport{}
    client := &amp;http.Client{Transport: tr}
    c := make(chan error, 1)
    go func() { c &lt;- f(client.Do(req)) }()
    select {
    case &lt;-ctx.Done():
        tr.CancelRequest(req)
        &lt;-c // Wait for f to return.
        return ctx.Err()
    case err := &lt;-c:
        return err
    }
}
</code></pre>

<h3>Адаптация кода для использования контекстов</h3>

<p>Множество разный фреймворков предоставляют пакеты для работы с запросозависимыми значениями. Мы можем определить новую реализацию для интерфейса <code>Context</code> для связи между используемым фреймворком и кодом, который ожидает получить параметр типа <code>Context</code>.</p>

<p>Для примера, Gorilla пакета <a href="http://www.gorillatoolkit.org/pkg/context">github.com/gorilla/context</a> позволяет обработчику ассоциировать данные с входящим запросом предоставляя ключ/значение мапинг для HTTP запроса. В <a href="http://blog.golang.org/context/gorilla/gorilla.go">gorilla.go</a> представлена реализация интерфейса <code>Context</code> в котором метод <code>Value</code> возвращает значения из HTTP запроса с помощью Gorilla&#39;ского пакета.</p>

<p>Некоторые другие пакеты реализуют отмену по аналогии с нашим <code>Context</code>. Например пакет <a href="http://godoc.org/gopkg.in/tomb.v2">Tomb</a> предоставляет функцию <code>Kill</code>, которая сигнализирует об отмене закрытием канала <code>Dying</code>. <code>Tomb</code> так же предоставляет функцию для ожидания всех go-рутин аналогичную <code>sync.WaitGroup</code>. В <a href="http://blog.golang.org/context/tomb/tomb.go">tomb.go</a> мы представили реализацию <code>Context</code> который может быть отменен при отмене родительского <code>Context</code> или при условии, что <code>Tomb</code> будет &quot;убит&quot;.</p>

<h3>Заключение</h3>

<p>В Google мы требуем, чтобы программисты на Go первым аргументом использовали <code>Context</code> во всех функциях которые занимаются обработкой запроса и выдачей ответа. Это позволяет разным командам разработчиков взаимодействовать более продуктивно. Так же, такой подход предоставляет контроль над таймаутами и отменой, а так же гарантирует безопасность транзита таким критическим данным.</p>

<p>Фреймворки, которые хотят работать с <code>Context</code> должны предоставить свои реализации для связи между функциями, которые принимают <code>Context</code> в параметрах. Функции в их клиентских библиотеках должны уметь работать с <code>Context</code>. Устанавливая интерфейс для работы с запросозависимыми данными и возможностью отмены, <code>Context</code> упрощает разработку более универсальных, масштабируемых и всем понятных пакетов.</p>

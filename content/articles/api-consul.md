+++
date = "2015-06-26T18:37:06+03:00"
draft = false
title = "API Consul"

+++

<p>Это перевод статьи "<a href="http://techblog.zeomega.com/devops/golang/2015/06/09/consul-kv-api-in-golang.html">An introduction to Consul key/value store API in Golang</a>".</p>

<p>К сожалению, статья совсем не такая подробная и интересная как хотелось бы. Реквестирую больше экшена в статьях про consul и другие полезные go-тулзовины.</p>

<h3>Введение</h3>

<p><a href="https://www.consul.io/">Consul</a> это инструмент для обнаружения сервисов(service discovery) и простое хранилище пар ключ-значение, которое удобно использовать для конфигурации. Большинство свой функциональности consul предоставляет как простое <a href="https://www.consul.io/docs/agent/http.html">RESTful HTTP API</a>. Это позволяет писать клиенты к этому API на самых различных языках. В этом посте мы рассмотрим как работать с <a href="http://godoc.org/github.com/hashicorp/consul/api">Go пакетом для Consul API</a>, в частности, c хранилищем пар ключ-значение(далее просто k/v).</p>

<p>Для запуска примеров из статьи вам нужен <a href="http://muthukadan.net/golang/an-introduction-to-go-programming.html">компилятор Go</a> и установленный consul. После этого нужно установить пакет для работы с consul:</p>

<pre><code>go get github.com/hashicorp/consul/api
</code></pre>

<p>Теперь мы можем его импортировать, указав(для удобства и читаемоcти) псевдоним <code>consulapi</code>.</p>

<pre><code class="go">import (
    consulapi "github.com/hashicorp/consul/api"
)
</code></pre>

<h3>Экземпляр клиента</h3>

<p>Для создания клиента нужно вызвать функцию <code>NewClient</code> и передать ей объект <a href="http://godoc.org/github.com/hashicorp/consul/api#Config">Config</a>  как аргумент. Самый простой способ создать экземпляр <code>Config</code>, это вызвать функцию <code>DefaultConfig</code> и изменить необходимые атрибуты, например <code>Address</code>, <code>Scheme</code>, <code>Datacenter</code> и т.д.</p>

<pre><code class="go">config := consulapi.DefaultConfig()
config.Address = "192.168.1.2:8500"
consul, err := consulapi.NewClient(config)
</code></pre>

<p>В примере выше, атрибут <code>Address</code> изменен на локальный IP адрес и порт. Также, мы можем изменить некоторый атрибуты, указав правильные переменные окружения. Но не забывайте, что значения указанные в коде, более приоритетны чем значения заданные через переменные окружения.</p>

<p>Давайте рассмотрим, какие атрибуты в объекте <code>Config</code> доступны.</p>

<h4>Address</h4>

<p>Это строка, в которой указан адрес consul сервера в формате <code>HOST:PORT</code>, например <code>"192.168.1.2:8500"</code>. Его можно указать через переменную окружения <code>CONSUL_HTTP_ADDR</code>. Дефолтное значение атрибута <code>127.0.0.1:8500</code> и это будет работать для большинства наших примеров, если вы не надумаете изменить адрес consul сервера.</p>

<h4>Scheme</h4>

<p>Этот атрибут должен быть строкой, указывающей какая схема используется в URI consul сервера. Значение может быть <code>http</code> or <code>https</code>. Дефолтное значение <code>http</code>. Для управления этим значением есть <a href="http://golang.org/pkg/strconv/#ParseBool">две булевые</a> переменные окружения: <code>CONSUL_HTTP_SSL</code> и <code>CONSUL_HTTP_SSL_VERIFY</code>. Если установить <code>CONSUL_HTTP_SSL</code> как <code>true</code>, то будет использоваться <code>https</code> схема, иначе <code>http</code>. Если  <code>CONSUL_HTTP_SSL_VERIFY</code> установить <code>false</code>, то не будет SSL верификации.</p>

<h4>Datacenter</h4>

<p>Определяет какой датацентр нужно использовать. Если атрибут не указан, то используется дефолтный датацентр.</p>

<h4>HttpClient</h4>

<p>Этот атрибут определяет какой клиент необходимо использовать. Если не указан, используется клиент по умолчанию.</p>

<h4>HttpAuth</h4>

<p>Этот атрибут должен быть указателем на структуру <a href="http://godoc.org/github.com/hashicorp/consul/api#HttpBasicAuth"><code>HttpBasicAuth</code></a>, в которой указаны значения <code>Username</code> и <code>Password</code> в виде строк для HTTP Basic Authentication. Например вот так:</p>

<pre><code class="go">config := consulapi.DefaultConfig()
config.HttpAuth = &amp;consulapi.HttpBasicAuth{Username: "guest", Password: "secret"}
consul, err := consulapi.NewClient(config)
</code></pre>

<p>Также, значение <code>HttpAuth</code> можно установить с помощью переменной окружения <code>CONSUL_HTTP_AUTH</code>. В эту переменную нужно записать строку с указанием пользователя, пароля и двоеточием в качестве разделителя. Если указна строка без двоеточия, то это будет обрабатываться как указание пользователя с пустым паролем. Например:</p>

<pre><code class="go">export CONSUL_HTTP_AUTH=guest:secret
export CONSUL_HTTP_AUTH=guest
</code></pre>

<h4>WaitTime</h4>

<p>Этот параметр определяет, как долго можно блокировать <code>Watch</code>. Если ничего не указанно, то используется дефолтное значение.</p>

<h4>Token</h4>

<p>Атрибут используется для задания в реквесте специального ACL токена, который будет использоваться вместо токена агента по умолчанию.</p>

<p>Используя методы <code>Client</code> вы можете получить доступ к различным возможностям API consul'а. В рамках данной статьи, нас интересует работа c k/v хранилищем.</p>

<h3>k/v хранилище</h3>

<p>Чтобы получить объект, с помощь которого мы сможем работать с <a href="http://godoc.org/github.com/hashicorp/consul/api#KV">k/v хранилищем</a>, нам нужно вызвать метод <code>KV</code> у экземпляра клиента.</p>

<pre><code class="go">kv := consul.KV()
</code></pre>

<p>В нашем распоряжении есть множество методов для CRUD операций и работой с k/v хранилищем. Структура <code>KVPair</code> используется для представления одной записи.</p>

<pre><code class="go">type KVPair struct {
    Key         string
    CreateIndex uint64
    ModifyIndex uint64
    LockIndex   uint64
    Flags       uint64
    Value       []byte
    Session     string
}
</code></pre>

<ul>
<li>Key это ключ, как правило с разделительными слешами. Например <code>sites/1/domain</code>.</li>
<li>CreateIndex это номер индекса, указывающий, когда ключ был создан.</li>
<li>ModifyIndex это номер индекса, указывающий, когда ключ был изменен.</li>
<li>LockIndex это номер индекса, указывающий, когда была взята блокировка этого ключа.</li>
<li>Flag может использоваться в рамках приложения, туда можно записывать дополнительную информацию.</li>
<li>Value это само значение в виде байт-массива с максимальным значением 512 килобайт.</li>
<li>Session можно записывать после <a href="http://godoc.org/github.com/hashicorp/consul/api#Session.Create">создания сессии</a>.</li>
</ul>

<p>Тип <code>KVPairs</code> это ничто иное как список ссылок на объекты <code>KVPair</code>:</p>

<pre><code class="go">type KVPairs []*KVPair
</code></pre>

<p><code>QueryMeta</code> содержит некоторую дополнительную информацию по выполненному запросу:</p>

<pre><code class="go">type QueryMeta struct {
    // LastIndex. Это поле может использоваться аналогично 
    // WaitIndex для предотвращения блокировки запроса
    LastIndex uint64

    // Время последнего взаимодействия с лидером
    // для данного сервера(обработавшего запрос)
    LastContact time.Duration

    // Известен ли лидер
    KnownLeader bool

    // Время выполнения запроса
    RequestTime time.Duration
}
</code></pre>

<p><code>QueryOptions</code> используется для указания дополнительных параметров при запросе:</p>

<pre><code class="go">type QueryOptions struct {
    // Обеспечивает перезаписывание датацетром DC
    // DC, указанного в Config.
    Datacenter string

    // AllowStale позволяет любому серверу(кроме лидера) consult 
    // обрабатывать запросы на чтение. Это уменьшает задержки и
    // увеличивает пропускную способность.
    AllowStale bool

    // RequireConsistent увеличивает достоверность данных, 
    // отдаваемых клиенту. Это дороже по производительности, 
    // но предотвращает отдачу старых данных клиенту.
    RequireConsistent bool

    // WaitIndex используется для блокирующих запросов.
    // Ожидаем истечение таймаута или следующего
    // доступного индекса.
    WaitIndex uint64

    // WaitTime используется для указания времени ожидания.
    // По умолчанию берется значение из конфига, 
    // но мы можем его поменять.
    WaitTime time.Duration

    // Token используется для указания ACL в запросе,
    // который перетирает дефолтный токен агента.
    Token string
}
</code></pre>

<h4>Put</h4>

<p>Сигнатура метода выглядит так:</p>

<pre><code class="go">func (k *KV) Put(p *KVPair, q *WriteOptions) (*WriteMeta, error)
</code></pre>

<p>А вот так этим методом можно пользоваться:</p>

<pre><code class="go">d := &amp;consulapi.KVPair{Key: "sites/1/domain", Value: []byte("example.com")}
kv.Acquire(d, nil)
</code></pre>

<h4>Get</h4>

<p>Сигнатура метода</p>

<pre><code class="go">func (k *KV) Get(key string, q *QueryOptions) (*KVPair, *QueryMeta, error)
</code></pre>

<p>И его использования:</p>

<pre><code class="go">kvp, qm, error := kv.Get("sites/1/domain", nil)
if err != nil {
    fmt.Println(err)
} else {
    fmt.Println(string(kvp.Value))
}
</code></pre>

<h4>Delete</h4>

<p>Сигнатура метода <code>Delete</code>:</p>

<pre><code class="go">func (k *KV) Delete(key string, w *WriteOptions) (*WriteMeta, error)
</code></pre>

<p>И пример как им пользоваться:</p>

<pre><code class="go">wm, err := kv.Delete("sites/1/domain", nil)
if err != nil {
    fmt.Println(err)
}
</code></pre>

<h4>Keys</h4>

<p>Метод для получения списка ключей:</p>

<pre><code class="go">func (k *KV) Keys(prefix, separator string, q *QueryOptions) ([]string, *QueryMeta, error)
</code></pre>

<h4>List</h4>

<p>Получение пачки значений по префиксу ключа:</p>

<pre><code class="go">func (k *KV) List(prefix string, q *QueryOptions) (KVPairs, *QueryMeta, error)
</code></pre>

<h4>DeleteTree</h4>

<p>Метод для удаления набора ключей с одинаковым префиксом:</p>

<pre><code class="go">func (k *KV) DeleteTree(prefix string, w *WriteOptions) (*WriteMeta, error)
</code></pre>

<p>И пример использования:</p>

<pre><code class="go">wm, err := kv.DeleteTree("sites", nil)
if err != nil {
    fmt.Println(err)
}
</code></pre>

<h3>Набор примитивов для более сложных операций</h3>

<p>Существует несколько примитивов для продвинутых операций, таких как комплексная синхронизация и выбор лидера. Перечислим несколько методов:</p>

<ul>
<li><em>Acquire</em> используется для операций приобретения(acquisition). Можно применять к <code>Key</code>, <code>Flags</code>, <code>Value</code> и <code>Session</code>. В случае удачи возвращает true, иначе false.</li>
<li><em>CAS</em> используется для операций <a href="https://en.wikipedia.org/wiki/Test-and-set">"проверить и установить"(Check-And-Set)</a>. Работает с <code>Key</code>, <code>ModifyIndex</code>, <code>Flags</code> и <code>Value</code>. В случае удачи возвращает true, иначе false.</li>
<li><em>DeleteCAS</em> эта операция аналогична <em>CAS</em>. Работает с <code>Key</code> и <code>ModifyIndex</code>. В случае удачи возвращает true, иначе false.</li>
<li><em>Release</em> используется для лока релизной операции. Работает с <code>Key</code>, <code>Flags</code>, <code>Value</code> и <code>Session</code>. В случае удачи возвращает true, иначе false.</li>
</ul>

<h3>Заключение</h3>

<p>В этой статья мы рассмотрели API k/v хранилище consul'a и его использование в рамках пакета. У consul'a еще много других возможностей, кроме k/v хранилища. И очень круто, что пакет для работы API написан самими разработчиками consul.</p>

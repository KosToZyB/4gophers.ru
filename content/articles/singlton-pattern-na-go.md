+++
date = "2015-07-23T11:24:11+03:00"
draft = false
title = " Синглтон паттерн на Go"

+++

<p>Это перевод статьи "<a href="http://marcio.io/2015/07/singleton-pattern-in-go/">Singleton Pattern in Go</a>"</p>

<p>В последние несколько лет популярность Go растет с невероятной скоростью, каким бы удивительным это не было. Go привлекает разработчиков из разных областей ИТ и с различным опытом разработки. Появляется много статей о том, как компании переходят с Ruby на Go, окунаются в мир параллельного/конкурентного подхода для решения задач.</p>

<p>Последние 10 лет Ruby on Rails позволял разработчикам и стартаперам писать довольно мощные системы, как правило, без необходимости вникать в то, как все устроено внутри или беспокоиться о конкурентности и потоко-безопсности кода. Для RoR приложения создание потоков и параллельная работа - вещи довольно редкие. На хостингах и внутри фреймворков используется другой подход, через запуск параллельных процессов. Параллельные серверы, такие как <a href="http://puma.io/">Puma</a>, стали набирать популярность только последние несколько лет, но они тянут за собой много различных проблем, связанных третьесторонними гемами и кодом, не предназначенным для многопоточности.</p>

<p>Теперь, когда много новых разработчиков запрыгнули в лодку языка Go, нам приходится более внимательно и пристально относится к коду, продумывать его поведение и разрабатывать с учетом потоко-безопасности.</p>

<h3>Распространенная ошибка</h3>

<p>Недавно я начал все чаще и чаще замечать эту ошибку в различных репозиториях c Go проектами. Синглтон, реализация которого абсолютна небезопасна для использования в конкурентных приложениях. Приведу пример, о чем я говорю:</p>

<pre><code class="go">package singleton

type singleton struct {
}

var instance *singleton

func GetInstance() *singleton {
    if instance == nil {
        instance = &amp;singleton{} // Это НЕ потоко-безопасно
    }
    return instance
}
</code></pre>

<p>При таком сценарии несколько go-рутин могут пройти проверку в <code>if</code> и они все могут создать свои экземпляры <code>singleton</code>, которые затрут один другого. Нет гарантий, какой из экземпляров вернет этот метод, а выполнение операций над этим экземпляром может привести к неожиданным последствиям.</p>

<p>Это очень плохо, потому что синглтон подразумевает использование одного экземпляра в разных участках кода, а в нашем случае это будут разные объекты в разных состояниях и с разным поведением. Это может стать настоящим адом во время дебага - ошибку заметить очень трудно, так как паузы рантайма не такие большие, как в продакшене и код ведет себя почти как нужно, что скрывает ошибку от разработчика.</p>

<h3>Агрессивная блокировка</h3>

<p>Я часто замечал как неправильно пытаются решить эту проблему. Конечно, подобный способ решает проблему потоко-безопасности, но тянет за собой другую не менее опасную ошибку. Он заключается в использовании агрессивной блокировки ресурса при вызове метода.</p>

<pre><code class="go">var mu Sync.Mutex

func GetInstance() *singleton {
    mu.Lock()   // Неоправданная блокировка, в случае
                // когда экземпляр уже создан

    defer mu.Unlock()

    if instance == nil {
        instance = &amp;singleton{}
    }
    return instance
}
</code></pre>

<p>Код, приведенный выше, использует <code>sync.Mutex</code> для решения проблемы вызова метода в разных потока и приобретает блокировку до создания инстанса. Проблема в том, что мы будем приобретать лишнюю блокировку когда экземпляр уже создан и нужно просто вернуть его. Если будет много конкурентного выполнения кода, то в этом месте у нас будет "бутылочное горлышко", так как в один момент времени только одна go-рутина сможет получить доступ к экземпляру объекта.</p>

<p>Таким образом, это не самое лучшее решение. Нам стоит использовать другие подходы.</p>

<h3>Блокировка с двойной проверкой</h3>

<p>В C++ (и многих других языках) один из самых лучших способов обеспечения минимальной потоко-безопасной блокировки, это использования паттерна "блокировка с двойной проверкой"(он же "double checked locking" и "check-lock-check"). Ниже приведена реализация этого паттерна на псевдокоде:</p>

<pre><code>if check() {
    lock() {
        if check() {
            // perform your lock-safe code here
        }
    }
}
</code></pre>

<p>Основная идея в том, что мы избегаем агрессивной блокировки благодаря проверки перед этим. Естественно что операция <code>if</code> дешевле блокировки. К тому же, мы можем подождать и получить индивидуальную блокировку. Таким образом, внутри блока код будет исполнятся единоразово в один момент времени. Но между первой проверкой и взятием блокировки может успеть отработать другой поток, который получил блокировку, поэтому нам нужно выполнить вторую проверку уже после блокировки, чтобы избежать перетирания экземпляров объекта.</p>

<p>На протяжении многих лет, люди работающие со мной знают, что я очень строг, по поводу использования этого паттерна, с моими командами во время ревью.</p>

<p>Если мы применим этот паттерн в нашем методе <code>GetInstance()</code>, в результате будет что то такое:</p>

<pre><code class="go">func GetInstance() *singleton {
    if instance == nil { // Все еще не идеально. 
                         // Тут нет полной атомарности
        mu.Lock()
        defer mu.Unlock()

        if instance == nil {
            instance = &amp;singleton{}
        }
    }
    return instance
}
</code></pre>

<p>Так уже лучше, но все еще не идеально. Из-за оптимизации при компиляции нет никакой уверенности, что проверка экземпляра бут выполнятся атомарно. Тем не менее, мы двигаемся в нужном направлении.</p>

<p>Мы можем улучшить этот способ, используя пакет <code>sync/atomic</code>, с помощью которого можно будет устанавливать и проверять флаги, которые будут указывать инициализирован наш экземпляр или нет.</p>

<pre><code class="go">import "sync"
import "sync/atomic"

var initialized uint32

// ...

func GetInstance() *singleton {

    if atomic.LoadUInt32(&amp;initialized) == 1 {
        return instance
    }

    mu.Lock()
    defer mu.Unlock()

    if initialized == 0 {
         instance = &amp;singleton{}
         atomic.StoreUint32(&amp;initialized, 1)
    }

    return instance
}
</code></pre>

<p>Но... Мне кажется, нам стоит присмотреться к реализации синхронизации go-рутин в стандартной библиотеке.</p>

<h3>Идиоматически правильный синглтон в Go</h3>

<p>Мы хотим реализовать синглтон паттерн идиоматическим для Go способом. Для этого у нас есть прекрасный пакет <code>sync</code>. В этом пакете есть тип <code>Once</code>. Этот тип позволяет выполнять действие только один раз. Ниже приведен код из стандартной библиотеки Go.</p>

<pre><code class="go">// Once это объект, которые позволяет выполнять некоторое 
// действие только один раз
type Once struct {
    m    Mutex
    done uint32
}

// Do вызывает функцию f только в том случае, если это первый вызов Do для 
// этого экземпляра Once. Другими словами, если у нас есть var once Once и 
// once.Do(f) будет вызываться несколько раз, f выполнится только в 
// момент первого вызова, даже если f будет иметь каждый раз другое значение.
// Для вызова нескольких функций таким способом нужно несколько
// экземпляров Once.
//
// Do предназначен для инициализации, которая должна выполняться единожды
// Так как f ничего не возвращает, может быть необходимым использовать
// замыкание для передачи параметров в функцию, выполняемую Do:
//  config.once.Do(func() { config.init(filename) })
//
// Поскольку ни один вызов к Do не завершится пока не произойдет 
// первый вызов f, то f может заблокировать последующие вызовы
// Do и получится дедлок.
//
// Если f паникует, то Do считает это обычным вызовом и, при последующих
// вызовах, Do не будет вызывать f.
//
func (o *Once) Do(f func()) {
    if atomic.LoadUint32(&amp;o.done) == 1 { // Check
        return
    }
    // Медленный путь.
    o.m.Lock()                           // Lock
    defer o.m.Unlock()
    if o.done == 0 {                     // Check
        defer atomic.StoreUint32(&amp;o.done, 1)
        f()
    }
}
</code></pre>

<p>А это означает, что мы можем, со спокойной душой, использовать пакет <code>sync</code> для вызова метода единоразово. Использовать метод <code>Do</code> можем вот так:</p>

<pre><code class="go">once.Do(func() {
    // в этом месте можно безопасно инициализировать экземпляр
})
</code></pre>

<p>Ниже вы можете видеть полный код реализации паттерна синглтон, в которой используется тип <code>sync.Once</code> для синхронизации доступа в <code>GetInstance()</code> и обеспечивается единоразовое создание нужного экземпляра.</p>

<pre><code class="go">package singleton

import (
    "sync"
)

type singleton struct {
}

var instance *singleton
var once sync.Once

func GetInstance() *singleton {
    once.Do(func() {
        instance = &amp;singleton{}
    })
    return instance
}
</code></pre>

<p>Как видите, использование <code>sync.Once</code> это самый элегантный путь для реализации потоко-безопасности. Аналогично в Objective-C и Swift (Cocoa) реализован метод <code>dispatch_once</code> для выполнения подобных инициализаций.</p>

<h3>Заключение</h3>

<p>Когда приходится работать с распределенностью и параллельностью, то необходимо совсем по другому смотреть на код. Старайтесь всегда проводить ревью кода чтобы быть в курсе всей кодовой базы.</p>

<p>Все новоприбывшие Go разработчики должны, прежде всего, разобраться как нужно писать потоко-безопасный код. Несмотря на то, что Go позволяет проектировать многопоточный код без значительный усилий, все же остается достаточно много моментов, в которых нужно подумать и найти лучшее решение.</p>

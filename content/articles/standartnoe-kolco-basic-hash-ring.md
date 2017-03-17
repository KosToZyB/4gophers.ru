+++
date = "2016-02-29T02:23:02+03:00"
draft = false
title = "Стандартное кольцо (Basic Hash Ring)"

+++

<p>Перевод статьи "<a href="https://github.com/gholt/ring/blob/master/BASIC_HASH_RING.md">Basic Hash Ring</a>"</p>

<p>В этой статье мы поговорим о структуре данных, которую называют стандартным кольцом(basic hash ring) и о наиболее частых способах ее реализации. В итоге, я проведу небольшое сравнение описанной библиотеки. Я вынес в отдельную статью про различия между <a href="https://github.com/gholt/ring/blob/master/PARTITION_RING_VS_HASH_RING.md">стандартным кольцом и партиционным кольцом(partition кing)</a>.</p>

<p>"Консистентное хеширование" - это термин, используемый для описания процесса, при котором данные(или выполнение некоторой работы) распределяются с помощью алгоритма хеширования, который определяет где именно должны быть размещены эти данные. Используя только хеш по идентификатору этих данных, вы можете определить где находятся эти данные. Этот список хешей(hash map), как правило, называют кольцом.</p>

<p>Вероятно, самый простой хеш выглядит как некоторый модуль от идентификатора. Например, если у вас всего две машины, то вы можете на первую отправлять все данные с четными идентификаторами, а на вторую с нечетными. Если у вас будет сбалансированное количество четных и нечетных идентификаторов, то все ваши данные сбалансировано распределяться между двумя нодами.</p>

<p>Но, как правило, идентификаторы это текстовые названия, например пути к файлам, URL, а не числовые значения. Это вынуждает использовать "настоящие" алгоритмы хеширования, прежде всего, для конвертирования текстовых идентефикаторов в числовые. Например, используя MD5 вы можете хешировать название файла 'mom.png' и получить '4559a12e3e8da7c2186250c2f292e3af' и хешировать название 'dad.png' получив '096edcc4107e9e18d6a03a43b3853bea'. Теперь, используя модуль, мы можем разместить 'mom.jpg' на нечетной ноде и 'dad.png' на четной ноде. И еще одно преимущество использования хеширования, такого как MD5, это равномерное распределение хешей и вам не прийдется беспокоится о значениях самих идентификаторов.</p>

<blockquote>
  <p>Если вам интересна тема выбора алгоритма хеширования, то вам стоит познакомиться с <a href="http://programmers.stackexchange.com/questions/49550/which-hashing-algorithm-is-best-for-uniqueness-and-speed">этим тредом на StackExchange</a>.</p>
</blockquote>

<p>Для простоты реализации я буду использовать встроенную в Go библиотеку хеширования FNV. Пример реализации кольца с использованием распределением по модулю:</p>

<pre><code class="go">package main

import (
    "fmt"
    "hash/fnv"
)

func Hash(item int) uint64 {
    hasher := fnv.New64a()
    hasher.Write([]byte(fmt.Sprintf("%d", item)))
    return hasher.Sum64()
}

const ITEMS = 1000000
const NODES = 100

func main() {
    countPerNode := make([]int, NODES)
    for i := 0; i &lt; ITEMS; i++ {
        countPerNode[int(Hash(i)%NODES)]++
    }
    min := ITEMS
    max := 0
    for n := 0; n &lt; NODES; n++ {
        if countPerNode[n] &lt; min {
            min = countPerNode[n]
        }
        if countPerNode[n] &gt; max {
            max = countPerNode[n]
        }
    }
    t := ITEMS / NODES
    fmt.Printf("%d to %d assigments per node, target was %d.\n", min, max, t)
    fmt.Printf("That's %.02f%% under and %.02f%% over.\n",
        float64(t-min)/float64(t)*100, float64(max-t)/float64(t)*100)
}
</code></pre>

<pre><code>9780 to 10215 assigments per node, target was 10000.
That's 2.20% under and 2.15% over.
</code></pre>

<p>Как ожидалось, мы добились довольно хорошего баланса. Но что будет, если одна нода отвалится? Давайте протестируем нашу теорию и добавим немного кода, который просто сравнивает использование ноды каждым элементом для 100 нод и 101 ноды:</p>

<pre><code class="go">moved := 0
for i := 0; i &lt; ITEMS; i++ {
    hasher := fnv.New64a()
    hasher.Write([]byte(fmt.Sprintf("%d", i)))
    x := hasher.Sum64()
    if int(x%NODES) != int(x%(NODES+1)) {
        moved++
    }
}
fmt.Printf("%d items moved, %.02f%%.\n",
    moved, float64(moved)/float64(ITEMS)*100)
</code></pre>

<pre><code>990214 items moved, 99.02%.
</code></pre>

<p>Теперь результаты далеки от идеальных. Более 99% всех элементов сменили ноду. Если бы это был настоящий кластер с реальными данными, то нам пришлось бы переместить все данные только из-за добавления одной ноды.</p>

<p>Это тот самый момент, когда на сцену выходит консистентное хеширование. Вместо использования распределения по модулям, мы привязываем определенный диапазон хешей к определенной ноде. Теперь при добавление новой ноды в кластер нужно будет переместить лишь небольшую часть диапазона с других нод, не перемещая все-все данные.</p>

<p>Например, представьте себе что наш хеш это числа от 0 до 9 и у нас есть две ноды. Нода А получает идентификатор 3, в таком случае эта нода будет работать с диапазоном хеша от 0 до 3. Нода В получает идентификатор 9 и будет работать с диапазоном хеша от 4 до 9. Таким образом, нода А работает с четырьмя элементами, а нода В с шестью. Это не очень сбалансировано, но пока не страшно.</p>

<p>Все преимущество такого подхода мы увидим, когда попытаемся добавить ноду C. Пусть ее идентификатор будет 6 и она будет работать с диапазоном хешей от 4 до 6. Таким образом, нужно будет перенести только три элемента с ноды В на ноду С. В результате, нода А так и будет работать с 4 элементами, нода B теперь будет работать с 3 элементами и нода С тоже работает с 3 элементами. Нам пришлось перенести только три элемента, а это намного лучше чем при использовании модулей.</p>

<p>В этом примере, для упрощения, я ограничился только 9 значениями хеша. Если бы значений было 8, и вдруг появилась необходимость добавить 9 элемент, то у нас будут проблемы, так как нет ноды, на которой можно разместить этот элемент. В таком случае, необходимо двигаться по кругу и размещать элементы на ноде А, затем С и так, далее. Вот почему этот механизм называется кольцом, визуально это лучше всего представить как кольцо по окружности которого размещены ноды в виде узлов.</p>

<p><img src="https://github.com/gholt/ring/raw/master/BASIC_HASH_RING.png" alt="" /></p>

<p>Теперь рассмотрим код очень простого кольца. Это кольцо будет состоять из упорядоченного списка нод и карты, которая связывает хеш с идентификатором ноды. Что бы определить какую ноду нужно нужно использовать для конкретного элемента, мы ищем по хешу элемента в сортированном списке, выбирая ближайшую ноду, хеш которой больше или равен хешу элемента, затем используем карту для получения идентификатора ноды. Результат работы примера также будет показывать сколько элементов было перемещено между нодами при добавлении новой.</p>

<pre><code class="go">package main

import (
    "fmt"
    "hash/fnv"
    "sort"
)

func Hash(x int) uint64 {
    hasher := fnv.New64a()
    hasher.Write([]byte(fmt.Sprintf("%d", x)))
    return hasher.Sum64()
}

const ITEMS = 1000000
const NODES = 100

type nodeSlice []uint64

func (ns nodeSlice) Len() int               { return len(ns) }
func (ns nodeSlice) Swap(a int, b int)      { ns[a], ns[b] = ns[b], ns[a] }
func (ns nodeSlice) Less(a int, b int) bool { return ns[a] &lt; ns[b] }

func main() {
    ring := make([]uint64, NODES)
    hashesToNode := make(map[uint64]int, NODES)
    for n := 0; n &lt; NODES; n++ {
        h := Hash(n)
        ring[n] = h
        hashesToNode[h] = n
    }
    sort.Sort(nodeSlice(ring))

    countPerNode := make([]int, NODES)
    for i := 0; i &lt; ITEMS; i++ {
        h := Hash(i)
        x := sort.Search(len(ring), func(x int) bool { return ring[x] &gt;= h })
        if x &gt;= len(ring) {
            x = 0
        }
        countPerNode[hashesToNode[ring[x]]]++
    }
    min := ITEMS
    max := 0
    for n := 0; n &lt; NODES; n++ {
        if countPerNode[n] &lt; min {
            min = countPerNode[n]
        }
        if countPerNode[n] &gt; max {
            max = countPerNode[n]
        }
    }
    t := ITEMS / NODES
    fmt.Printf("%d to %d assigments per node, target was %d.\n", min, max, t)
    fmt.Printf("That's %.02f%% under and %.02f%% over.\n",
        float64(t-min)/float64(t)*100, float64(max-t)/float64(t)*100)

    ring2 := make([]uint64, NODES+1)
    copy(ring2, ring)
    hashesToNode2 := make(map[uint64]int, NODES+1)
    for k, v := range hashesToNode {
        hashesToNode2[k] = v
    }
    h := Hash(NODES)
    ring2[NODES] = h
    hashesToNode2[h] = NODES
    sort.Sort(nodeSlice(ring2))

    moved := 0
    for i := 0; i &lt; ITEMS; i++ {
        h := Hash(i)
        x := sort.Search(len(ring), func(x int) bool { return ring[x] &gt;= h })
        if x &gt;= len(ring) {
            x = 0
        }
        x2 := sort.Search(len(ring2), func(x int) bool { return ring2[x] &gt;= h })
        if x2 &gt;= len(ring2) {
            x2 = 0
        }
        if hashesToNode[ring[x]] != hashesToNode2[ring2[x2]] {
            moved++
        }
    }
    fmt.Printf("%d items moved, %.02f%%.\n",
        moved, float64(moved)/float64(ITEMS)*100)
}
</code></pre>

<pre><code>1 to 659651 assigments per node, target was 10000.
That's 99.99% under and 6496.51% over.
240855 items moved, 24.09%.
</code></pre>

<p>В результате мы получили жуткий баланс, на одну узел перегружен, а другой совсем недогружен. Но нам пришлось переместить значительно меньше элементов при добавлении новой ноды. Мы добавили 1% новых нод и при этом нам пришлось переместить 24% процента элементов. Конечно, это не идеальный результат, но это значительно лучше чем 99%.</p>

<p>Для улучшения этого алгоритма, добавим концепцию <em>виртуальных нод</em>. Виртуальные ноды представляют собой набор хешей, который привязан к какой-то конкретной ноде. Представьте себе кольцо еще раз, мы размещаем на нем несколько точек, все они привязаны к конкретной ноде, затем повторить это для остальных нод. В итоге у вас будет больше хешей в кольце, но это позволит сгладить баланс.</p>

<p>Сейчас я изменю нашу программу, мы будем использовать 1000 виртуальных нод для каждой реальной ноды:</p>

<pre><code class="go">package main

import (
    "fmt"
    "hash/fnv"
    "sort"
)

func Hash(x int) uint64 {
    hasher := fnv.New64a()
    hasher.Write([]byte(fmt.Sprintf("%d", x)))
    return hasher.Sum64()
}

const ITEMS = 1000000
const NODES = 100
const VIRTUAL_NODES_PER_NODE = 1000

type nodeSlice []uint64

func (ns nodeSlice) Len() int               { return len(ns) }
func (ns nodeSlice) Swap(a int, b int)      { ns[a], ns[b] = ns[b], ns[a] }
func (ns nodeSlice) Less(a int, b int) bool { return ns[a] &lt; ns[b] }

func main() {
    ring := make([]uint64, NODES*VIRTUAL_NODES_PER_NODE)
    hashesToNode := make(map[uint64]int, NODES*VIRTUAL_NODES_PER_NODE)
    for p, n := 0, 0; n &lt; NODES; n++ {
        for v := 0; v &lt; VIRTUAL_NODES_PER_NODE; v++ {
            h := Hash(n*1000000 + v)
            ring[p] = h
            p++
            hashesToNode[h] = n
        }
    }
    sort.Sort(nodeSlice(ring))

    countPerNode := make([]int, NODES)
    for i := 0; i &lt; ITEMS; i++ {
        h := Hash(i)
        x := sort.Search(len(ring), func(x int) bool { return ring[x] &gt;= h })
        if x &gt;= len(ring) {
            x = 0
        }
        countPerNode[hashesToNode[ring[x]]]++
    }
    min := ITEMS
    max := 0
    for n := 0; n &lt; NODES; n++ {
        if countPerNode[n] &lt; min {
            min = countPerNode[n]
        }
        if countPerNode[n] &gt; max {
            max = countPerNode[n]
        }
    }
    t := ITEMS / NODES
    fmt.Printf("%d to %d assigments per node, target was %d.\n", min, max, t)
    fmt.Printf("That's %.02f%% under and %.02f%% over.\n",
        float64(t-min)/float64(t)*100, float64(max-t)/float64(t)*100)

    ring2 := make([]uint64, (NODES+1)*VIRTUAL_NODES_PER_NODE)
    copy(ring2, ring)
    hashesToNode2 := make(map[uint64]int, (NODES+1)*VIRTUAL_NODES_PER_NODE)
    for k, v := range hashesToNode {
        hashesToNode2[k] = v
    }
    for p, v := NODES*VIRTUAL_NODES_PER_NODE, 0; v &lt; VIRTUAL_NODES_PER_NODE; v++ {
        h := Hash(NODES*1000000 + v)
        ring2[p] = h
        p++
        hashesToNode2[h] = NODES
    }
    sort.Sort(nodeSlice(ring2))

    moved := 0
    for i := 0; i &lt; ITEMS; i++ {
        h := Hash(i)
        x := sort.Search(len(ring), func(x int) bool { return ring[x] &gt;= h })
        if x &gt;= len(ring) {
            x = 0
        }
        x2 := sort.Search(len(ring2), func(x int) bool { return ring2[x] &gt;= h })
        if x2 &gt;= len(ring2) {
            x2 = 0
        }
        if hashesToNode[ring[x]] != hashesToNode2[ring2[x2]] {
            moved++
        }
    }
    fmt.Printf("%d items moved, %.02f%%.\n",
        moved, float64(moved)/float64(ITEMS)*100)
}
</code></pre>

<pre><code>2920 to 27557 assigments per node, target was 10000.
That's 70.80% under and 175.57% over.
10279 items moved, 1.03%.
</code></pre>

<p>Баланс стал только чуть лучше, хотя пока он еще не идеален, можно попробовать добавить больше виртуальных нод. Зато число перемещенных элементов впечатляет: всего 1% элементов был перемещен между нодами.</p>

<p>Теперь понятно почему консистентное хеширование так популярно в распределенных системах. Однако, я обещал затронуть тему работы партиционного кольца. Партиционное кольцо выглядит очень похоже на консистентное, но реализация отличается. Вы можете почитать <a href="https://github.com/gholt/ring/blob/master/PARTITION_RING_VS_HASH_RING.md">Partition Ring vs. Hash Ring</a>, чтобы разобраться в различиях. В рамках этой статьи я уделяю больше внимания стандартному кольцу, как наиболее распространенному алгоритму. Но приведу пример партиционного кольца, чтобы вы смогли посмотреть о чем идет речь.</p>

<pre><code class="go">package main

import (
    "fmt"
    "hash/fnv"

    "github.com/gholt/ring"
)

func Hash(x int) uint64 {
    hasher := fnv.New64a()
    hasher.Write([]byte(fmt.Sprintf("%d", x)))
    return hasher.Sum64()
}

const ITEMS = 1000000
const NODES = 100

func main() {
    nodeIDsToNode := make(map[uint64]int)
    b := ring.NewBuilder(64)
    for n := 0; n &lt; NODES; n++ {
        bn, _ := b.AddNode(true, 1, nil, nil, "", nil)
        nodeIDsToNode[bn.ID()] = n
    }
    ring := b.Ring()

    nodeIDsToNode2 := make(map[uint64]int, NODES+1)
    for k, v := range nodeIDsToNode {
        nodeIDsToNode2[k] = v
    }
    b.PretendElapsed(b.MoveWait() + 1)
    bn, _ := b.AddNode(true, 1, nil, nil, "", nil)
    nodeIDsToNode2[bn.ID()] = NODES
    ring2 := b.Ring()

    countPerNode := make([]int, NODES)
    for i := 0; i &lt; ITEMS; i++ {
        h := Hash(i)
        x := ring.ResponsibleNodes(uint32(h &gt;&gt; (64 - ring.PartitionBitCount())))[0].ID()
        countPerNode[nodeIDsToNode[x]]++
    }
    min := ITEMS
    max := 0
    for n := 0; n &lt; NODES; n++ {
        if countPerNode[n] &lt; min {
            min = countPerNode[n]
        }
        if countPerNode[n] &gt; max {
            max = countPerNode[n]
        }
    }
    t := ITEMS / NODES
    fmt.Printf("%d to %d assigments per node, target was %d.\n", min, max, t)
    fmt.Printf("That's %.02f%% under and %.02f%% over.\n",
        float64(t-min)/float64(t)*100, float64(max-t)/float64(t)*100)

    moved := 0
    for i := 0; i &lt; ITEMS; i++ {
        h := Hash(i)
        x := ring.ResponsibleNodes(uint32(h &gt;&gt; (64 - ring.PartitionBitCount())))[0].ID()
        x2 := ring2.ResponsibleNodes(uint32(h &gt;&gt; (64 - ring2.PartitionBitCount())))[0].ID()
        if nodeIDsToNode[x] != nodeIDsToNode2[x2] {
            moved++
        }
    }
    fmt.Printf("%d items moved, %.02f%%.\n",
        moved, float64(moved)/float64(ITEMS)*100)
}
</code></pre>

<pre><code>9395 to 10933 assigments per node, target was 10000.
That's 6.05% under and 9.33% over.
11710 items moved, 1.17%.
</code></pre>

<p>Теперь результаты для under и over значительно лучше. Конечно, пришлось переносить больше сущностей, чем в случае с базовым кольцом, но этим можно пожертвовать ради улучшенного баланса. Напомню, что узнать больше о различиях между партиционным кольцом и стандартным кольцом <a href="https://github.com/gholt/ring/blob/master/PARTITION_RING_VS_HASH_RING.md">можно в этой статье</a>.</p>

<p>Каждый пример я запускал 100 раз. Для этого я переименовал функцию main и сделал другую функцию, которая уже вызывала рабочий пример. Первый пример без виртуальных нод отрабатывал за 1m0.861s, второй пример с использованием виртуальных нод отработал за 1m22.554s, пример с партиционным кольцом отработало за 1m17.088s. Неплохой результат, если учесть, что мы значительно улучшили баланс. Конечно, это не самый идеальный бенчмарк. По хорошему, нужно разделять скорость создания, скорость модификации и скорость поиска. Но в принципе, тест дает верные результаты.</p>

<p>Есть более сложные статьи, описывающие принципы работы кольца, такие как веса для нод(или емкость нод), когда вам приходится учитывать дополнительные параметры при распределении элементов, а также работу с репликами и многие другие фичи. Но это явно выходит за рамки одной статьи.</p>

<p>Что еще есть интересного по теме:</p>

<ul>
<li><a href="http://arxiv.org/abs/1406.2294">Прыгающее консистентное хеширование(Jump consistent hashing)</a> - <a href="https://github.com/dgryski/go-jump">реализация от dgryski</a> и его распределенное <a href="https://github.com/dgryski/go-shardedkv">K/V хранилище</a>.</li>
<li><a href="http://arxiv.org/pdf/1505.00062.pdf">Мульти-зондирующее консистентное хеширование</a> - <a href="https://github.com/dgryski/go-mpchash">реализация от dgryski</a>.</li>
<li><a href="http://storageconference.us/2015/Papers/16.Zhao.pdf">Схема репликации GreenCHT</a></li>
</ul>

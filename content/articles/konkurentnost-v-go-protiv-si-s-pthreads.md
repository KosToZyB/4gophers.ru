+++
date = "2015-12-30T02:34:02+03:00"
draft = false
title = "Конкурентность в Go против Си с pthreads"

+++

<p>Перевод статьи "<a href="http://denis.papathanasiou.org/posts/2015.12.26.post.html">Go Concurrency versus C and pthreads</a>".</p>

<p>Недавно я принимал участие в одном программистком соревновании:</p>

<blockquote>
  <p>Необходимо написать программу для тестирования реакции пользователя. При запуске эта программа ожидает случайный промежуток времени от 1 до 10 секунд и выводит сообщение "GO!" в консоль.</p>
  
  <p>После этого программа ожидает когда пользователь нажмет enter. В момент нажатия, замеряется промежуток времени от отображения надписи до нажатия кнопки.</p>
  
  <p>Если пользователь умудрился нажать enter до сообщения "GO!", то выводится другое сообщение - "FAIL".</p>
</blockquote>

<p>По сути, это упражнение на применение <a href="https://en.wikipedia.org/wiki/Concurrency_%28computer_science%29">конкуренности(concurrency)</a>: пока одна часть логики следит за нажатием кнопки, вторая часть генерирует случайное время ожидание перед выводом сообщения и в конце нужно выполнить сравнение времени нажатия и задержки.</p>

<p>Так как я изучаю Go, то мне показалось, что описанная задача идеально ложится па концепцию <a href="https://golang.org/doc/effective_go.html#goroutines">go-рутин</a> и <a href="https://golang.org/doc/effective_go.html#channels">каналов</a>.</p>

<p>И действительно, решить эту задачу на Go очень просто.</p>

<pre><code>package main

import (
    "bufio"
    "fmt"
    "math/rand"
    "os"
    "time"
)

func init() {
    rand.Seed(time.Now().UnixNano())
}

func main() {
    channel := make(chan time.Time)

    go func() {
        reader := bufio.NewReader(os.Stdin)
        reader.ReadString('\n')
        channel &lt;- time.Now()
    }()

    time.Sleep(time.Second * time.Duration(rand.Intn(11)))
    paused := time.Now()
    fmt.Println("GO!")

    entered := &lt;- channel
    if paused.Sub(entered) &gt; 0 {
        fmt.Println("FAIL")
    } else {
        fmt.Printf("%v\n", time.Since(entered))
    }
}
</code></pre>

<p>И тут я вспомнил, что на развитие <a href="https://en.wikipedia.org/wiki/Go_%28programming_language%29#Language_design">Go очень повлиял Си</a>. Или, если быть точнее, <a href="http://thenewstack.io/a-closer-look-at-golang-from-an-architects-perspective/">Go это современный Си</a>. Я заинтересовался, насколько сложнее сделать аналогичную программу на Си?</p>

<p>В <a href="http://flash-gordon.me.uk/ansi.c.txt">стандартном ANSI Си</a> нет никаких функций для работы с потоками и во <a href="http://www.amazon.com/gp/product/0131103628/ref=as_li_tl?ie=UTF8&amp;camp=1789&amp;creative=9325&amp;creativeASIN=0131103628&amp;linkCode=as2&amp;tag=denispapath-20&amp;linkId=YZU35D35AZNKMNHE">втором издании K &amp; R</a> тоже нет никаких упоминаний о потоках. Но есть <a href="https://computing.llnl.gov/tutorials/pthreads/">POSIX потоки</a>(они же нити, они же треды), или pthreads, которые можно использовать как отдельную библиотеку.</p>

<p>В результате у нас получилось больше текста, но концептуально это практически одинаковые программы:</p>

<pre><code>#include &lt;stdio.h&gt;
#include &lt;stdlib.h&gt;
#include &lt;unistd.h&gt;
#include &lt;sys/time.h&gt;
#include &lt;pthread.h&gt;

void *listen(void *timestamp) {
    int c;
    while((c = getc(stdin)) != EOF) {
        if( c == '\n' ) {
            gettimeofday((struct timeval *)timestamp, NULL);
            break;
        }
    }

    return NULL;
}

int main(void) {
    pthread_t listener;
    struct timeval entered, paused, diff;

    if( pthread_create(&amp;listener, NULL, listen, &amp;entered) ) {
        fprintf(stderr, "Error: could not create keyboard listening thread\n");
        return 1;
    }

    srand(time(NULL));
    sleep(rand() % 10 + 1);
    printf("GO!\n");
    gettimeofday(&amp;paused, NULL);

    if( pthread_join(listener, NULL) ) {
        fprintf(stderr, "Error: could not join keyboard listening thread\n");
        return 2;
    }

    if( timercmp(&amp;paused, &amp;entered, &gt;) )
        printf("FAIL\n");
    else {
        timersub(&amp;entered, &amp;paused, &amp;diff);
        printf("%ld\n", (1000LL * diff.tv_sec) + (diff.tv_usec / 1000));
    }

    return 0;
}
</code></pre>

<p>Мы также делаем обработчик, который ожидает нажатия клавиши enter и используем функцию <a href="http://linux.die.net/man/2/gettimeofday">gettimeofday</a> для получения текущего времени и функцию для сравнения дат, которые работают аналогично <a href="https://golang.org/pkg/time/">пакету time</a>.</p>

<p>Создавать новые потоки с помощью pthread достаточно просто, но вместо использования каналов для обмена данными между потоками, приходится использовать <a href="http://beej.us/guide/bgc/output/html/multipage/pointers.html#ptpass">передачу структуры по ссылке</a>.</p>

<p>Еще одно большое отличие - при использовании pthread необходимо выполнять <a href="https://computing.llnl.gov/tutorials/pthreads/man/pthread_join.txt">"присоединение"(joined)</a> чтобы дождаться завершения работы потока.</p>

<p><a href="http://timmurphy.org/2010/05/04/pthreads-in-c-a-minimal-working-example/">Вот это замечательный пример</a> с подробными объяснениями помогли мне вникнуть в суть того, как работает pthread.</p>

<p>И при создании обработчика, и при "присоединении" нам необходимо проверять ошибки. В нашем случае, для этой игрушечной программы я просто делаю выход в случае ошибки. Однако, для более объемных проектов, нужно будет реализовать значительно больше дополнительной логики.</p>

<p>Кроме некоторого различия в реализации логики этих двух примеров, существует еще много нюансов, скрытых под капотом.</p>

<p>Несмотря на то, что потоки, реализованные с помощью pthread, значительно легче чем процессы операционной системы, они все еще не настолько легковесные как go-рутины. Конкурентность в Go реализована на принципах взаимодействующих последовательных процессов(<a href="http://www.usingcsp.com/">Communicating Sequential Processes (CSP)</a>).</p>

<p>В результате, go-рутины объединены и работают на <a href="https://tleyden.github.io/blog/2014/10/30/goroutines-vs-threads/">небольшом количестве системных процессов</a>, вместо использования одного потока для каждой go-рутины.</p>

<p>Конечно, это не совсем честное соревнование, так как Си намного старше Go, а phthread это вообще сторонняя библиотека.</p>

<p>Go вобрал в себя весь опыт полученный с Си и еще много всего полезного из других языков программирования.</p>

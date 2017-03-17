+++
date = "2016-02-12T01:09:01+03:00"
draft = false
title = "SMID оптимизация в Go"

+++

<p>Перевод статьи "<a href="https://goroutines.com/asm">SIMD Optimization in Go</a>".</p>

<p>Из этой статьи вы узнаете много прекрасного о программировании на ассемблере.</p>

<p>Из-за сложности современных компьютеров, очень просто забыть что на самом низком уровне они предоставляют для программиста очень простой интерфейс. Машинный код это очень тонкая прослойка над физическим слоем транзисторов и кондесаторов из которых состоит компьютер. Все абстракции нужны для удобства человека. Они надежны и хорошо протестированы. Как правило, мы программируем сверху вниз, на более высоких уровнях. В свою очередь, программирование на ассемблере позволяет увидеть как машины работают на самом деле.</p>

<p>Программирование на ассемблере может быть очень медленным и сложным в сравнении с программированием на более высокоуровневых языках, таких как Go. Но иногда в этом есть смысл, к тому же, это бывает весело. Например замечательная игра на ассемблере <a href="http://www.zachtronics.com/tis-100/">TIS-100</a> с ее <a href="http://www.vidarholen.net/contents/junk/files/TIS-100%20Reference%20Manual.pdf">14 страничной инструкцией</a> доставляет удовольствие без всяких реальных компьютеров с его <a href="http://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-manual-325462.pdf">3,883 страничным мануалом</a> и целым арсеналом инструментов сборки.</p>

<p>Зачем тратить свое время на ассемблер, если есть значительно более удобные языки программирования? Ваш отец сделал этот новый язык собственноручно, но для вас он недостаточно хорош? Он потратил годы и перепробовал кучу подходов для решения проблем, встретившихся на этом пути, а после кристализировал все это в навороченных компиляторах, которые не позволят вам повторить его ошибки.</p>

<p>И тем не менее, есть еще парочка мест в которых ассемблер будет к стати. Явно можно выделить <a href="https://golang.org/src/crypto/aes/asm_amd64.s">криптографию</a>, <a href="https://go-review.googlesource.com/#/c/8968/">оптимизацию производительности</a>, <a href="https://golang.org/src/syscall/asm_linux_amd64.s">доступ к местам, недоступным</a> из обычного языка программирования. Нас больше всего интересует оптимизация производительности.</p>

<p>Если производительность некоторого куска вашей программы имеет очень большое значение для пользователя и вы уже перепробовали все более простые способы, то ассемблер может стать неплохим подспорьем в этом деле. Конечно, компилятор лучше знает, как оптимизировать код, но вы лучше разбираетесь в вашем конкретном случае.</p>

<h3>Пишем ассемблер в Go</h3>

<p>Чтобы быстро вникнуть в тему, лучше всего написать простейшую функцию. Ниже пример одной из таких функций, которая называется and. Все что она делает - это складывает два числа типа int64.</p>

<pre><code>package main

import "fmt"

func add(x, y int64) int64 {
    return x + y
}

func main() {
    fmt.Println(add(2, 3))
}
</code></pre>

<p>Для реализации аналогичного кода на ассемблере необходимо создать файл с названием <code>add_amd64.s</code> в котором, собственно, мы будем писать наш ассемблерный код. Так как я использую мак, то я буду писать реализацию для архитектуры AMD64.</p>

<pre><code>// add.go
package main

import "fmt"

func add(x, y int64) int64

func main() {
    fmt.Println(add(2, 3))
}

// add_amd64.s
TEXT ·add(SB),NOSPLIT,$0
    MOVQ x+0(FP), BX
    MOVQ y+8(FP), BP
    ADDQ BP, BX
    MOVQ BX, ret+16(FP)
    RET
</code></pre>

<p>Для запуска этого примера, скопируйте его в GOPATH и скомпилируйте его. На самом деле, не обязательно даже в GOPATH копировать:</p>

<pre><code>mkdir -p src
unzip add.zip -d src/add
GOPATH=$PWD go build add
./add
</code></pre>

<p>Синтаксис ассемблера... обескураживает. В официальном руководстве Go есть небольшой, несколько древний, гайд по ассемблеру в Plan 9, он дает некоторое представление о том как работает Go ассемблер. Но самая лучшая документация, конечно же, это существующий код на Go ассемблере и скомпилированные в ассемблер Go функции, которые вы можете получить с помощью команды:</p>

<pre><code>go tool compile -S &lt;go file&gt;
</code></pre>

<p>Самое важно, что следует запомнить в первую очередь, это объявление функций и использование стека.</p>

<p>Наше магическое заклинание начинается с <code>TEXT ·add(SB),NOSPLIT,$0</code>. Да, обратите внимание, что используется специальный юникодный символ <code>·</code>, который играет роль разделителя для названия пакета и функции. Так как мы находимся в пакете <code>main</code>, его название не нужно указывать, а функция в нашем примере называется <code>add</code>. Директива <code>NOSPLIT</code> означает что не нужно указывать размер аргументов следующим параметров. Константа <code>$0</code> стоит как раз на том месте, где нужно было указать размер аргументов, но так как мы уже используем <code>NOSPLIT</code>, то просто оставим <code>$0</code>. К сожалению, не совсем понятно, почему после названия функции стоит <code>(SB)</code>, но без этого ничего не работает.</p>

<p>Аргументы в функцию передаются из стека в соответствующем порядке, начиная с <code>0(FP)</code>, что означает нулевой байт смещения от указателя <code>FP</code>, затем идут все остальные параметры до самого возвращаемого значения. Для функции <code>func add(x, y int64) int64</code> это выглядит примерно так:</p>

<p><img src="https://box.everhelper.me/attachment/389728/26da152b-2f64-4196-b100-2340f0ec03da/126220-GK3Aw8wiSDaN8Htz/screen.png" alt="" /></p>

<p>И ассемблерный код для справки:</p>

<pre><code>TEXT ·add(SB),NOSPLIT,$0
    MOVQ x+0(FP), BX
    MOVQ y+8(FP), BP
    ADDQ BP, BX
    MOVQ BX, ret+16(FP)
    RET
</code></pre>

<p>Ассемблерная версия функции <code>add</code> загружает переменную <code>a</code> из памяти по адресу <code>+0(FP)</code> в регистр <code>BX</code>. Затем грузит <code>b</code> из памяти по адресу <code>+8(FP)</code> в регистр <code>BP</code>, добавляет <code>BP</code> к <code>BX</code> сохраняя результаты в <code>BX</code> и, в конце концов, копирует <code>BX</code> в память по адресу <code>+16(FP)</code> и выходит из функции. Вызывающая функция, которая размещает аргументы на стеке, в итоге читает возвращаемое значение по конкретному адресу в памяти.</p>

<h3>Оптимизация функций с помощью ассемблера</h3>

<p>Конечно, нам не нужно писать на ассемблере программу для складывания двух чисел. Но что мы вообще будем с ним делать?</p>

<p>Давайте представим, что у вас есть пачка векторов и вам нужно умножить их на матрицу преобразований. Возможно, векторы это точки и вы хотите <a href="http://blog.wolfire.com/2010/07/Linear-algebra-for-game-developers-part-3">переместить их в 3D пространстве</a>. Мы будем использовать 4D векторы и матрицу преобразования 4x4, векторы будут упакованы в массив. Это лучше чем использование массива с объектами которые представляют вектор, так как линейное сканирование упакованных данных в памяти позволяет лучше использовать кеш.</p>

<pre><code>type V4 [4]float32
type M4 [16]float32

func M4MultiplyV4(m M4, v V4) V4 {
    return V4{
        v[0]*m[0] + v[1]*m[4] + v[2]*m[8] + v[3]*m[12],
        v[0]*m[1] + v[1]*m[5] + v[2]*m[9] + v[3]*m[13],
        v[0]*m[2] + v[1]*m[6] + v[2]*m[10] + v[3]*m[14],
        v[0]*m[3] + v[1]*m[7] + v[2]*m[11] + v[3]*m[15],
    }
}

func multiply(data []V4, m M4) {
    for i, v := range data {
        data[i] = M4MultiplyV4(m, v)
    }
}
</code></pre>

<p>На моем 2012 Macbook Pro с использованием Go 1.5.3 этот код выполняется за 140ms при размере данных в 128MB. Насколько быстрой может быть реализация? Для копирования памяти, кажется, хорошим показателем <a href="https://goroutines.com/asm-files/vectors-copy.go">14ms</a>.</p>

<p>Ниже приведена функция написанная на ассемблере с использованием SIMD инструкций для умножения. Она позволяет умножать 4 числа типа float32 параллельно:</p>

<pre><code>// func multiply(data []V4, m M4)
//
// memory layout of the stack relative to FP
//  +0 data slice ptr
//  +8 data slice len
// +16 data slice cap
// +24 m[0]  | m[1]
// +32 m[2]  | m[3]
// +40 m[4]  | m[5]
// +48 m[6]  | m[7]
// +56 m[8]  | m[9]
// +64 m[10] | m[11]
// +72 m[12] | m[13]
// +80 m[14] | m[15]

TEXT ·multiply(SB),NOSPLIT,$0
  // data ptr
  MOVQ data+0(FP), CX
  // data len
  MOVQ data+8(FP), SI
  // index into data
  MOVQ $0, AX
  // return early if zero length
  CMPQ AX, SI
  JE END
  // load the matrix into 128-bit wide xmm registers
  // load [m[0], m[1], m[2], m[3]] into xmm0
  MOVUPS m+24(FP), X0
  // load [m[4], m[5], m[6], m[7]] into xmm1
  MOVUPS m+40(FP), X1
  // load [m[8], m[9], m[10], m[11]] into xmm2
  MOVUPS m+56(FP), X2
  // load [m[12], m[13], m[14], m[15]] into xmm3
  MOVUPS m+72(FP), X3
LOOP:
  // load each component of the vector into xmm registers
  // load data[i][0] (x) into xmm4
  MOVSS    0(CX), X4
  // load data[i][1] (y) into xmm5
  MOVSS    4(CX), X5
  // load data[i][2] (z) into xmm6
  MOVSS    8(CX), X6
  // load data[i][3] (w) into xmm7
  MOVSS    12(CX), X7
  // copy each component of the matrix across each register
  // [0, 0, 0, x] =&gt; [x, x, x, x]
  SHUFPS $0, X4, X4
  // [0, 0, 0, y] =&gt; [y, y, y, y]
  SHUFPS $0, X5, X5
  // [0, 0, 0, z] =&gt; [z, z, z, z]
  SHUFPS $0, X6, X6
  // [0, 0, 0, w] =&gt; [w, w, w, w]
  SHUFPS $0, X7, X7
  // xmm4 = [m[0], m[1], m[2], m[3]] .* [x, x, x, x]
  MULPS X0, X4
  // xmm5 = [m[4], m[5], m[6], m[7]] .* [y, y, y, y]
  MULPS X1, X5
  // xmm6 = [m[8], m[9], m[10], m[11]] .* [z, z, z, z]
  MULPS X2, X6
  // xmm7 = [m[12], m[13], m[14], m[15]] .* [w, w, w, w]
  MULPS X3, X7
  // xmm4 = xmm4 + xmm5
  ADDPS X5, X4
  // xmm4 = xmm4 + xmm6
  ADDPS X6, X4
  // xmm4 = xmm4 + xmm7
  ADDPS X7, X4
  // data[i] = xmm4
  MOVNTPS X4, 0(CX)
  // data++
  ADDQ $16, CX
  // i++
  INCQ AX
  // if i &gt;= len(data) break
  CMPQ AX, SI
  JLT LOOP
END:
  // since we use a non-temporal write (MOVNTPS)
  // make sure all writes are visible before we leave the function
  SFENCE
  RET
</code></pre>

<p>Выполнение функции занимает 18ms, а это уже очень близко к скорости копирования памяти. Еще большую оптимизацию можно получить при использовании GPU вместо CPU. Вот время выполнения разных версий реализации, включая заинлайненную Go версию и реализацию без использования SMID:</p>

<table>
<thead>
<tr>
  <th>program</th>
  <th>time</th>
  <th>speedup</th>
</tr>
</thead>
<tbody>
<tr>
  <td><a href="https://goroutines.com/asm-files/process-vectors.go">original</a></td>
  <td>140ms</td>
  <td>1x</td>
</tr>
<tr>
  <td><a href="https://goroutines.com/asm-files/process-vectors-inline.go">inline</a></td>
  <td>69ms</td>
  <td>2x</td>
</tr>
<tr>
  <td><a href="https://goroutines.com/asm-files/process-vectors-asm-plain.zip">assembly</a></td>
  <td>43ms</td>
  <td>3x</td>
</tr>
<tr>
  <td><a href="https://goroutines.com/asm-files/process-vectors-asm-simd.zip">simd</a></td>
  <td>17ms</td>
  <td>8x</td>
</tr>
<tr>
  <td><a href="https://goroutines.com/asm-files/vectors-copy.go">copy</a></td>
  <td>15ms</td>
  <td>9x</td>
</tr>
</tbody>
</table>

<p>При оптимизации нам приходится платить сложностью кода, чтобы сделать программу более легкой для понимания машиной. Ассемблер это достаточно сложный способ повышения производительности, но иногда это самый лучший способ.</p>

<h3>Замечания по реализации</h3>

<p>Я разрабатывал ассемблерную часть на C и x64 ассемблере используя XCode и затем портировал ассемблер в Go формат. В XCode есть замечательный дебагер, который позволяет просматривать регистры во время выполнения кода. Если добавить .s файл в XCode то он будет компилироваться и линковаться с вашим бинарником.</p>

<p>Я использовал <a href="http://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-instruction-set-reference-manual-325383.pdf">документацию по набору инструкций к Intel x64</a> и <a href="https://software.intel.com/sites/landingpage/IntrinsicsGuide/">Intel Intrinsics Guide</a> что бы разобраться, какие инструкции использовать. Конвертирование в Go ассемблер это не так уж и просто, но большинство инструкций есть в <code>x86/anames.go</code>, а все остальные можно <a href="https://golang.org/doc/asm#unsupported_opcodes">кодировать непосредственно</a> в бинарное представление.</p>

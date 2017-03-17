+++
date = "2014-06-16T19:59:07+03:00"
draft = false
title = "Работа с изображениями"

+++

<p>В стандартной библиотеки Go много всего интересного. Например, пакет <code>image</code> для работы с изображениями. Сейчас попробуем нарисовать что нить простенькое для понимания основных принципов.</p>

<h3>Начало</h3>

<p>Сообществом уже написана целая куча библиотек для работы с графикой. Но, чтобы продуктивно работать с этими инструментами, нужно разбираться, как это все устроено.</p>

<p>Начинаем с самого простого. Создадим простую png картинку абсолютно без ничего.</p>

<pre>
<code>package main

import (
    &quot;fmt&quot;
    &quot;image&quot;
    &quot;image/png&quot;
    &quot;os&quot;
)

func main() {
    file, err := os.Create(&quot;someimage.png&quot;)

    if err != nil {
        fmt.Errorf(&quot;%s&quot;, err)
    }
    img := image.NewRGBA(image.Rect(0, 0, 500, 500))
    png.Encode(file, img)   
}

</code></pre>

<p><code>img := image.NewRGBA(image.Rect(0, 0, 500, 500))</code> - создаем наше изображение, которое состоит из одной области размером 500x500 пикселей.</p>

<p><code>png.Encode(file, m)</code> - сохраняем изображение в файл. Стандартная библиотека поддерживает два формата изображений: png и jpeg.</p>

<p>Теперь заполним картинку цветом. Для этого нам нужны пакеты <code>image/draw</code> и <code>image/color</code></p>

<pre>
<code>package main

import (
    &quot;fmt&quot;
    &quot;image&quot;
    &quot;image/color&quot;
    &quot;image/draw&quot;
    &quot;image/png&quot;
    &quot;os&quot;
)

var teal color.Color = color.RGBA{0, 200, 200, 255}

func main() {
    file, err := os.Create(&quot;someimage.png&quot;)

    if err != nil {
        fmt.Errorf(&quot;%s&quot;, err)
    }

    img := image.NewRGBA(image.Rect(0, 0, 500, 500))

    draw.Draw(img, img.Bounds(), &amp;image.Uniform{teal}, image.ZP, draw.Src)
    // или draw.Draw(img, img.Bounds(), image.Transparent, image.ZP, draw.Src)

    png.Encode(file, img)
}
</code></pre>

<p><code>var teal color.Color = color.RGBA{0, 200, 200, 255}</code> - цвет, которым будет заполнена наша картинка. Первое значение - это прозрачность.</p>

<p><code>draw.Draw(img, img.Bounds(), &amp;image.Uniform{rectaleColor}, image.ZP, draw.Src)</code> - заполняем картинку цветом. Функция <code>draw.Draw()</code> в параметрах принимает:</p>

<ul>
	<li>Создаваемое изображение типа <code>draw.Image</code>,</li>
	<li>Прямоугольную область рисования. У нас это все изображение <code>img.Bounds()</code>)</li>
	<li>Исходное изображение <code>&amp;image.Uniform{rectaleColor}</code>. Это изображение с бесконечными размерами и залитое цветом <code>rectaleColor</code></li>
	<li>Координаты прямоугольника для рисования <code>image.ZP</code> - это точка с координатами (0, 0)</li>
	<li><code>draw.Src</code> - метод рисования, который указывает функции просто скопировать исходное изображение в целевое не применяя никаких преобразований. В нашем случае, просто окрасит пиксели изображения в нужный цвет.</li>
</ul>

<h3>Линии</h3>

<p>Пришло время нарисовать пару линий.</p>

<pre>
<code>// ...

var red  color.Color = color.RGBA{200, 30, 30, 255}

// ...

draw.Draw(img, img.Bounds(), &amp;image.Uniform{teal}, image.ZP, draw.Src)

for x := 20; x &lt; 380; x++ {
    y := x/3 + 15
    img.Set(x, y, red)
}

// ...
</code></pre>

<p>Линия начинается в точке с координатами x = 20 y = 21 и заканчивается в точке x = 379 y = 141.</p>

<p><code>y := x/3 + 15</code> - простейшая математическая функция рисования линии на координатной плоскости. Меняя тройку - меняем наклон линии.</p>

<p><code>img.Set(x, y, red)</code> - функция, которая заполняет определенную точку на изображении указанным цветом.</p>

<h3>Фигуры</h3>

<p>Линии это просто. Нам нужно больше математики! Будем рисовать круги. Создаем новый тип <code>Circle</code></p>

<pre>
<code>type Circle struct {
    p   image.Point
    r   int
}

func (c *Circle) ColorModel() color.Model {
    return color.AlphaModel
}

func (c *Circle) Bounds() image.Rectangle {
    return image.Rect(c.p.X-c.r, c.p.Y-c.r, c.p.X+c.r, c.p.Y+c.r)
}

func (c *Circle) At(x, y int) color.Color {
    xx, yy, rr := float64(x-c.p.X)+0.5, float64(y-c.p.Y)+0.5, float64(c.r)
    if xx*xx+yy*yy &lt; rr*rr {
        return color.Alpha{255}
    }
    return color.Alpha{0}
}
</code></pre>

<p>Тип <code>Circle</code> определяет интерфейс image.Image</p>

<pre>
<code>type Image interface {
    ColorModel() color.Model
    Bounds() Rectangle
    At(x, y int) color.Color
}
</code></pre>

<p><code>ColorModel() color.Model</code> - определяет какая <a href="http://golang.org/pkg/image/color/#Model">цветовая модель</a> будет использоваться для нашего изображения.</p>

<p><code>Bounds() Rectangle</code> - определяем границы, в переделах которых будет размещена наша фигура.</p>

<p><code>At(x, y int) color.Color</code> - возвращет цвет для определенных координат.</p>

<p>Теперь нам нужно изображение, которое будет исходником для нашей маски:</p>

<pre>
<code>mask := image.NewRGBA(image.Rect(0, 0, 500, 500))
draw.Draw(mask, mask.Bounds(), &amp;image.Uniform{red}, image.ZP, draw.Src)
</code></pre>

<p>Это изображение заполнено красным цветом.</p>

<p>Рисуем три круга используя функцию draw.DrawMask(), которая принимает аж 7 параметров.</p>

<pre>
<code>draw.DrawMask(img, img.Bounds(), mask, image.ZP, &amp;Circle{image.Point{20, 21}, 20}, image.ZP, draw.Over)
draw.DrawMask(img, img.Bounds(), mask, image.ZP, &amp;Circle{image.Point{379, 141}, 20}, image.ZP, draw.Over)
draw.DrawMask(img, img.Bounds(), mask, image.ZP, &amp;Circle{image.Point{239, 321}, 70}, image.ZP, draw.Over)
</code></pre>

<p>Сигнатура функции <code>draw.DrawMask</code> выглядит так:</p>

<pre>
<code>func DrawMask(dst Image, r image.Rectangle, src image.Image, sp image.Point, mask image.Image, mp image.Point, op Op)
</code></pre>

<ul>
	<li><code>dst Image</code> - это изображение, на которое мы будем накладывать маску</li>
	<li><code>r image.Rectangle</code> - границы изображения, в котором сохранятся результаты преобразования</li>
	<li><code>src image.Image</code> - это исходное изображение, которое будет использоваться для создания маски.</li>
	<li><code>sp image.Point</code> - позиция исходного изображения</li>
	<li><code>mask image.Image</code> - изображение, которое будет играть роль маски. У нас это <code>&amp;Circle{image.Point{x, y}, r}</code></li>
	<li><code>mp image.Point</code> - позиция для создаваемой маски</li>
	<li><code>op Op</code> - оператор композиции изображения. В нашем случае draw.Over.</li>
</ul>

<p>Собираем все это вместе и запускаем. В результате у нас получится вот такая картинка:</p>

<p><img alt="" src="https://dl.dropboxusercontent.com/u/750049/4gophers.com/someimage.png" /></p>

<p>Конечный вариант этой программы можно <a href="https://gist.github.com/horechek/c28c8d9e1b054675c62a">найти на гисте</a>.</p>

<p>Почитать по теме:</p>

<ul>
	<li><a href="http://blog.golang.org/go-image-package">The Go image package</a></li>
	<li><a href="http://blog.golang.org/go-imagedraw-package">The Go image/draw package</a></li>
</ul>

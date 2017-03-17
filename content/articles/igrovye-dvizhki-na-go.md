+++
date = "2014-10-08T17:28:05+03:00"
draft = false
title = "Игровые движки на Go"

+++

<p>Конечно, Go не задумывался как язык для написания игр. Но чем еще заняться холодными осенними вечерами? Давайте попробуем запустить несколько игровых движков и посмотреть, как в Go-мире обстоят дела с геймдевом.</p>

<h2><a href="https://github.com/vova616/GarageEngine">GarageEngine</a></h2>

<p>Наш первый экземпляр - это 2d игровой движок для Go, который работает на OpenGL. Он состоит из модулей/компонентов. Сейчас уже есть приличное количество таких модулей для шрифтов, спрайтов, текстур, физики, сцен и так далее. Очень сильно ощущается влияние Unity3d. Особенно в названиях компонентов: Scene, Coroutines, Components, Transform, GameObjects и т.д.</p>

<p>К сожалению это проект для обучения. Не стоит ожидать от него супер-килер-фич и обратной совместимости.</p>

<h3>Установка</h3>

<p>Нам нужно установить либы <code>glfw</code> и <code>glew</code></p>

<pre><code class="shell">sudo apt-get update
sudo apt-get install build-essential binutils-gold \
freeglut3 freeglut3-dev libalut-dev libglew-dev \
libglfw-dev libxrandr2 libxrandr-dev
</code></pre>

<p>Если вы используете windows вам нужно юзать mingw и собрать glew.</p>

<pre><code class="shell">go get github.com/vova616/GarageEngine
</code></pre>

<h3>Примеры</h3>

<p>SpaceCookies - Небольшой пример для тестирования движка. Есть несколько нюансов, чтобы нормально запустить этот пример.</p>

<p>Прежде всего заходим в папку <code>src/github.com/vova616/GarageEngine</code> и собираем все руками:</p>

<pre><code class="shell">go build main.go
</code></pre>

<p>Если появилась ошибка вида:</p>

<pre><code class="shell"># github.com/vova616/GarageEngine/engine/audio
.godeps/src/github.com/vova616/GarageEngine/engine/audio/AudioSource.go:129: 
undefined: openal.DistanceModel
</code></pre>

<p>Комментируем строки 128-130 в файле <code>src/github.com/vova616/GarageEngine/engine/audio/AudioSource.go</code></p>

<pre><code class="go">if currentDistanceModel != this.distanceModel {
  openal.SetDistanceModel(openal.DistanceModel(this.distanceModel))
}
</code></pre>

<p>Пытаемся запустить и получаем ошибку <code>listen tcp 0.0.0.0:123: bind: permission denied</code>. Меняем номера портов в файле <code>src/github.com/vova616/GarageEngine/spaceCookies/server/Server.go</code> на строке 88</p>

<pre><code class="go">addr, err := net.ResolveTCPAddr("tcp", "0.0.0.0:12345")
</code></pre>

<p>и в файле <code>src/github.com/vova616/GarageEngine/spaceCookies/game/Client.go</code> строки 19 и 20</p>

<pre><code class="go">const ServerIP = "localhost:12345"
const ServerLocalIP = "localhost:12345"
</code></pre>

<p>Опять пробуем запустить и ничего не получается. Рядом с нашим бинарником лежит файл <code>log.txt</code> c примерно таким содержимым:</p>

<pre><code class="shell">Enginge started!
GLFW Initialized!
Opengl Initialized!
open ./data/spaceCookies/background.png: no such file or directory
open ./data/spaceCookies/button.png: no such file or directory
runtime error: invalid memory address or nil pointer dereference panic.c:482, 
os_linux.c:234, Sprite.go:42, LoginScene.go:83, Engine.go:96, 
main.go:78, main.go:44, proc.c:220, proc.c:1394
</code></pre>

<p>Видим, что программа пытается найти файлы <code>./data/spaceCookies/background.png</code> и <code>./data/spaceCookies/button.png</code>. А папка, на самом деле, называется <code>./data/SpaceCookies</code>. Переименовываем папку и снова запускаем. Ура! Теперь все работает.</p>

<p><img src="https://dl.dropboxusercontent.com/u/750049/4gophers.com/garage.png" alt="" /></p>

<p>Если лень фиксить все эти баги, можете <a href="http://www.youtube.com/watch?v=iMMbf6SRb9Q">посмотреть демки</a>, как <a href="http://www.youtube.com/watch?v=BMRlY9dFVLg">это все работает</a>.</p>

<h2><a href="http://azul3d.org/">☣ Azul3D</a></h2>

<p>Azul3D это 3D движок написанный полностью с нуля на языке программирования Go. Azul3D подходит для создания 2D и 3D игр. Так же, его можно использовать для создания не игровых, а просто интерактивных приложений. В отличии от большинства современных движков(таких как Unity, JMonkey) у Azul3D нет дополнительных фич типа редакторов уровней, IDE. Это просто набор необходимых разработчику пакетов.</p>

<p>Это игровой движок от программистов для программистов. Он очень минималистичен, но легко расширяется в отличии от других движков.</p>

<p>Azul3D предоставляет самые необходимые инструменты, которые будут использоваться изо дня в день. И делает это действительно хорошо.</p>

<h3>Установка</h3>

<p>Начинаем с зависимостей для Ubuntu 14.04. Для других систем <a href="http://azul3d.org/doc/install">смотрим тут</a>.</p>

<pre><code class="shell">sudo apt-get install build-essential git mesa-common-dev \
libx11-dev libx11-xcb-dev libxcb-icccm4-dev \
libxcb-image0-dev libxcb-randr0-dev libxcb-render-util0-dev \
libxcb-xkb-dev libfreetype6-dev libbz2-dev \
libxxf86vm-dev libgl1-mesa-dev
</code></pre>

<h3>Примеры</h3>

<p>Теперь пробуем установить примеры. Они вынесены в отдельный пакет.</p>

<pre><code class="shell">go get azul3d.org/examples.v1
</code></pre>

<p>нам понадобится куча других пакетов:</p>

<pre><code class="shell">go get azul3d.org/keyboard.v1
go get azul3d.org/lmath.v1
go get azul3d.org/gfx.v1
go get azul3d.org/gfx/window.v2
go get azul3d.org/mouse.v1
go get azul3d.org/tmx.dev
</code></pre>

<p>Теперь можем посмотреть примеры. Заходим в <code>src/azul3d.org/examples.v1/azul3d_tmx</code> и запускаем пример</p>

<pre><code class="shell">go run azul3d_tmx.go
</code></pre>

<p>Вы увидите что-то такое:</p>

<p><img src="https://dl.dropboxusercontent.com/u/750049/4gophers.com/azul3d.png" alt="" /></p>

<p>Были проблемы с версией Go ниже 1.3. <a href="https://github.com/moovweb/gvm">Обновился до самой последней</a> и все отлично заработало на моей ubuntu 14.04.</p>

<h2><a href="https://bitbucket.org/krepa098/gosfml2/wiki/Home">gosfml</a></h2>

<p>Возможность использовать C-код в Go приложениях - это просто прекрасно. Мы можем использовать кучу готовых библиотек.</p>

<p>И конечно же, нельзя обойти стороной Go биндинги к популярной библиотеке <a href="http://www.sfml-dev.org/">SFML</a>. Есть несколько вариантов, но я смог запустить <a href="https://bitbucket.org/krepa098/gosfml2/wiki/Home">только этот</a>.</p>

<p>SFML - это кросплатформенная библиотека для мультимедиа. Она написана на C++, но есть куча привязок к разным языкам и к Go в том числе.</p>

<p>Кроме работы с мультимедиа в этой либе есть несколько пакетов для программирования простых игр.</p>

<p>Модули, которые нам доступны в этой библиотеке:</p>

<ul>
<li>System - управление временем и потоками.</li>
<li>Window - управление окнами и взаимодействием с пользователем.</li>
<li>Graphics - делает простым отображение графических примитивов и изображений.</li>
<li>Audio - предоставляет интерфейс для управления звуком.</li>
<li>Network - для сетевых приложений.</li>
</ul>

<h3>Устанавливаем</h3>

<p>Сначала ставим саму SFML</p>

<pre><code class="shell">sudo apt-get install libsfml-dev
</code></pre>

<p>Так как SFML написана на C++, нам нужен C биндинг к SFML.</p>

<pre><code class="shell">sudo apt-get install libcsfml-dev
</code></pre>

<p>Теперь устанавливаем Go биндинг</p>

<pre><code class="shell">go get -u bitbucket.org/krepa098/gosfml2
</code></pre>

<h3>Примеры</h3>

<p>Можем собирать и запускать примеры</p>

<pre><code class="shell">cd ./src/bitbucket.org/krepa098/gosfml2/samples
./buildSamples
cd sfmlPong/
./sfmlPong
</code></pre>

<p>И видим вот такую картинку:</p>

<p><img src="https://dl.dropboxusercontent.com/u/750049/4gophers.com/sfml.png" alt="" /></p>

<h2>Еще инструменты</h2>

<p>Список некоторых специфических движков и инструментов, которые попадались в поиске, но до которых не дошли руки.</p>

<ul>
<li><a href="http://go-ngine.com/">go:ngine</a> - Многообещающий движок, но я так и не понял как его собрать и где примеры посмотреть.</li>
<li><a href="https://github.com/sarenji/terrago">terrago</a> - Генератор для ландшафта</li>
<li><a href="https://bitbucket.org/tshannon/gonewton/src">Newton Dynamics</a> - Это либа 3d физики</li>
<li><a href="https://bitbucket.org/tshannon/gohorde/src">horde3d</a> - Интересный 3d движок. Стоит попробовать в будущем.</li>
<li><a href="https://bitbucket.org/tshannon/vmath/src">vmath</a> - Либа для векторных, матричных и 3дешных расчетов</li>
<li><a href="https://github.com/ajhager/engi">ENGi</a>  Мультиплатформенная либа для 2d игр. Пока в глубокой альфе.</li>
<li><a href="https://github.com/go-gl">OpenGL with Golang</a> - Go биндинг к OpenGL.</li>
</ul>

<h2>Выводы</h2>

<p>Go не задумывался для написания игр. Конечно, если вы хотите потратить несколько вечеров ради развлечения, то все нормально. Но для серьезных проектов стоит выбрать что то более подходящее.</p>

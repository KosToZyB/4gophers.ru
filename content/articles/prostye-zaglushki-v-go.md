+++
date = "2015-06-10T11:41:11+03:00"
draft = false
title = "Простые заглушки в Go"

+++

<p>И множестве других языков...</p>

<p>Перевод статьи "<a href="http://paytonrules.com/software-development/2015/05/31/simple-mock-objects-in-go.html">Simple Mock Objects in Go</a>"</p>

<h3>У меня нет фреймворка для заглушек.</h3>

<p>Недавно я вычитал в оном из тредов вот такую идею:</p>

<p><em>В Go нет встроенного фреймока для создания заглушек(mocks) и gomock пока еще не готов для использования в продакшене! Когда же, в конце-то концов, мы сможем нормально TDD'ешить в Go?</em></p>

<p>Чувак, да ладно. Ты можешь делать свое TDD без всяких фреймворков для заглушек. Люди занимаются этим каждый день. Ты можешь начать разрабатывать через тестирование прямо сейчас. И без использования gomock и подобным ему вещей, если они тебе не нравятся.</p>

<p>Вся проблема в отношении разработчика, задающего вопрос. "У меня нет вот таких инструментов, предоставьте их мне или я не буду работать по TDD!". Навряд ли это ленивый или глупый разработчик. Go, как правило, не привлекает таких людей, но они уверены, что все делают правильно. В этом вопросе меня раздражает то, что как правило, он задается по привычке. Девелоперы приходят после работы с другими средами разработки, в частности ентерпрайзными, аналогичными C# например, и они привыкли к тем "официальным" инструментам, которыми они пользовались. И когда таких инструментов не обнаруживается, то они ждут их появления. Как ине кажется, это просто неприемлемо.</p>

<p>Что-то я много разглагольствую. Предположим, что у нас есть только gomock и нет никакого способа улучшить его - действительно ли это такая большая проблема, что стоит отказываться от TDD? Давайте на примере Go посмотрим, как можно самому писать заглушки и хелперы для тестирования.</p>

<h3>Пишем собственные "Заглушки"</h3>

<p>Давайте рассмотрим один из моих любимых пример - игровой цикл(Game Loop). Я писал про него в <a href="http://blog.8thlight.com/eric-smith/2014/08/18/test-driving-the-game-loop-part-1.html">блоге "8th Light"</a> и это было про C#. Пост была основан на <a href="http://gameprogrammingpatterns.com/game-loop.html">вот этой замечательной статье</a>. Почти каждая игра начинается с базового цикла:</p>

<pre><code class="go">while (true) {
    processInput();
    update();
    render();
}
</code></pre>

<p>На самом деле, это достаточно стремный игровой цикл, потому что он не учитывает частоту кадров или время работы каждой итерации, а это необходимо для реализации правильной физике в игре, но для примера он подходит. Давайте посмотрим, как бы мы могли протестировать этот код.</p>

<p>Напишем первый тест:</p>

<pre><code class="go">package gameloop

import "testing"

func TestLoopUpdatesOnStart(t *testing.T) {
    gl := &amp;GameLoop{}
}
</code></pre>

<p>Это не соберется, пока не будет реализован объект GameLoop. У меня нет желания растягивать эту писанину на тысячу строк, поэтому я пропущу несколько шагов. И так, первый тест:</p>

<pre><code class="go">func TestLoopUpdatesOnStart(t *testing.T) {
    game := &amp;PhonyGame{}
    gl := &amp;GameLoop{Game: game}

    gl.Update()

    if true != game.Updated {

        t.Error("Ожидается, что игра будет обновлена, но это не произошло")
    }
}
</code></pre>

<p>В первой строке мы создаем указатель на <code>PhonyGame</code> без указания дополнительных параметров. Посмотрим, что такое <code>PhonyGame</code>:</p>

<pre><code class="go">type PhonyGame struct {
    Updated bool
}
</code></pre>

<p>Откуда взялась эта структура? Возможно, это какая-то магия фреймворка для заглушек? Я забыл показать вам вызов <code>go get</code>? Нет, все значительно проще:</p>

<pre><code class="go">func TestLoopUpdatesOnStart(t *testing.T) {
    //...
}

type PhonyGame struct {
    Updated bool
}
</code></pre>

<p>Ага, это просто объект, точнее структура. Структура с булевым параметром, который сетится в момент вызова <code>Update</code>. Как мы ее устанавливаем?</p>

<pre><code class="go">func (g *PhonyGame) Update() {
    g.Updated = true
}
</code></pre>

<p>Этот метод размещен, конечно же, прямо под определением структуры <code>PhonyGame</code>. Все что я сейчас делаю - это создаю фейковые объекты для тестирования обновления. Но что насчет реального кода? Окей:</p>

<pre><code class="go">type Updater interface {
    Update()
}

type GameLoop struct {
    Game Updater
}

func (g *GameLoop) Update() {
    g.Game.Update()
}
</code></pre>

<p><code>Updater</code> - это ужасное название, но я не придумал чего-то получше. Что он делает? Он обновляется. Объект <code>GameLoop</code> оперирует объектом <code>Game</code> типа <code>Updater</code>, который является <em>интерфейсом</em>. Этот интерфейс реализован в объекте <code>PhonyGame</code>, который я использую для своих тестов. Это одна из причин почему Go прекрасен. Принцип сегрегации интерфейса(<a href="http://www.objectmentor.com/resources/articles/isp.pdf">Interface Segregation Principle</a>) подразумевает, что клиент управляет интерфейсом, хотя мне больше нравится говорить "владеет" интерфейсом. В Go вы можете определить интерфейсы где угодно. И если объекты соответствуют этому интерфейсу, то они просто работают и не требуют никаких директив "implements". Такой подход, работа с парами интерфейс-реализация, это именно то, чего вам необходимо придерживаться при программировании на любом языке.</p>

<p>Что это означает? Это означает, что я могу протестировать свою игру, используя фейковые объекты(которые я создаю прям в тесте), если они реализуют необходимый мне интерфейс(ы). Показанный пример пока еще бесполезный, так что давайте немного расширим его. Игра должна остановить обновление по завершению, но должна делать обновления и прорисовку на каждой итерации. Давайте напишем все необходимые тесты:</p>

<pre><code class="go">func TestLoopUpdatesOnEachUpdate(t *testing.T) {
    game := NewPhonyGame()
    gl := &amp;GameLoop{Game: game, Canvas: game}
    game.SetTurnsUntilGameOver(1)

    gl.Start()

    AssertTrue(t, game.Updated())
}

func TestLoopDrawsOnEachUpdate(t *testing.T) {
    game := NewPhonyGame()
    gl := &amp;GameLoop{Game: game, Canvas: game}
    game.SetTurnsUntilGameOver(1)

    gl.Start()

    AssertTrue(t, game.Drawn())
}

func TestLoopDoesntUpdateWhenTheGameIsOver(t *testing.T) {
    game := NewPhonyGame()
    gl := &amp;GameLoop{Game: game, Canvas: game}
    game.SetTurnsUntilGameOver(0)

    gl.Start()

    AssertFalse(t, game.Drawn())
    AssertFalse(t, game.Updated())
}

func TestLoopUpdatesUntilTheGameIsOver(t *testing.T) {
    game := NewPhonyGame()
    gl := &amp;GameLoop{Game: game, Canvas: game}
    game.SetTurnsUntilGameOver(2)

    gl.Start()

    AssertEquals(t, 2, game.DrawCount)
    AssertEquals(t, 2, game.UpdateCount)
}

type PhonyGame struct {
    UpdateCount   int
    DrawCount     int
    IsOverAnswers []bool
}

func NewPhonyGame() *PhonyGame {
    g := &amp;PhonyGame{}
    g.IsOverAnswers = make([]bool, 0)
    return g
}

func (g *PhonyGame) Updated() bool {
   return g.UpdateCount &gt; 0
}

func (g *PhonyGame) Update() {
    g.UpdateCount++
}

func (g *PhonyGame) Drawn() bool {
    return g.DrawCount &gt; 0
}

func (g *PhonyGame) Draw() {
    g.DrawCount++
}

func (g *PhonyGame) SetTurnsUntilGameOver(turns int) {
    for i := 0; i &lt; turns; i++ {
        g.IsOverAnswers = append(g.IsOverAnswers, false)
    }
}

func (g *PhonyGame) IsOver() bool {
    if len(g.IsOverAnswers) &gt; 1 {
        answer := g.IsOverAnswers[0]
        g.IsOverAnswers = append(g.IsOverAnswers[:0], g.IsOverAnswers[1:]...)
        return answer
    }

    if len(g.IsOverAnswers) == 1 {
        answer := g.IsOverAnswers[0]
        g.IsOverAnswers = make([]bool, 0)
        return answer
    }
    return true
}

func AssertTrue(t *testing.T, value bool) {
    if !value {
       t.Error("Expected true, got false")
    }
}

func AssertFalse(t *testing.T, value bool) {
    if value {
        t.Error("Expected false, got true")
    }
}

func AssertEquals(t *testing.T, expected, actual int) {
    if expected != actual {
        t.Errorf("Expected %d but got %d", expected, actual)
    }
}
</code></pre>

<p>Это довольно длинный пример, поэтому я попытался кое-что сократить:</p>

<pre><code class="go">func TestLoopUpdatesUntilTheGameIsOver(t *testing.T) {
    game := NewPhonyGame()
    gl := &amp;GameLoop{Game: game, Canvas: game}
    game.SetTurnsUntilGameOver(2)

    gl.Start()

    AssertEquals(t, 2, game.DrawCount)
    AssertEquals(t, 2, game.UpdateCount)
}
</code></pre>

<p>Я заменил в тестах использование <code>&amp;PhonyGame{}</code> на вызов фабричной функции <code>NewPhonyGame()</code>. Внутри этой функции используется слайс и я должен быть уверен, что он правильно инициализируется. На следующей строчке инициализируется <code>GameLoop</code> с указанием параметра <code>Game</code>(наш <code>Updater</code>) и <code>Canvas</code>(объект для рисования). Для этих параметров нужны разные интерфейсы, но <code>PhonyGame</code> реализует сразу оба интерфейса. Конечно, <code>GameLoop</code> может использовать один объект, но я считаю, что правильно разделять и использовать два объекта для <code>Draw</code> и <code>Update</code>. Поэтому это два интерфейса. Функция <code>SetTurnsUntilGameIsOver()</code> настроит <code>PhonyGame</code> так что бы <code>IsOver</code> возвращала true после двух "повторов". В динамический фреймворках мы можем писать <code>game.stub(IsOver).andReturn([false, false, true])</code> и мне кажется так намного наглядней. Это говорит о том, что заглушка более "вещественна" чем хотелось бы, в основном, потому что у нас нет встроенного типа <code>Queue</code>. Весьма вероятно, что в будущем я реализую этот тип.</p>

<p>О, и я написал несколько хелпереров для асертов.</p>

<p>Актуальный код:</p>

<pre><code class="go">package gameloop

type Updater interface {
    Update()
    IsOver() bool
}

type Canvas interface {
    Draw()
}

type GameLoop struct {
    Game   Updater
    Canvas Canvas
}

func (g *GameLoop) update() {
    g.Game.Update()
    g.Canvas.Draw()
}

func (g *GameLoop) Start() {
    for !g.Game.IsOver() {
        g.update()
    }
}
</code></pre>

<p>Этот код не очень труден для понимания и его удобно использовать в тестах, но обратите внимание на проверку <code>!g.Game.IsOver()</code>. Это некоторое поведение и я реализовал это поведение не правильно. Собственно, сам игровой цикл может иметь ограничения на частоту кадров, получать входные данные. И убедитесь, что игровой цикл запускается в своем собственном треде(go-рутине), отдельном от прорисовки. Все эти различные варианты поведений нужно проверить и это можно сделать изолированно.</p>

<h3>Вы можете делать это в любых языках</h3>

<p>Я регулярно общаюсь с разработчиками, будь то мои коллеги, студенты, или случайные люди в интернете, которые жалуются, что причина, по которой они не могут делать "хорошее" TDD, в отсутствии инструментов. На самом деле, инструменты для TDD не обязательны. И тут уместна цитата:</p>

<blockquote>
  <p>Первая атака на проблему проверки может быть сделана еще до написания кода. Чтобы быть уверенным в точности ответа, необходимо знать ручной способ расчета результатов. Нужно проверить результаты по некоторым кейсам, которые позднее будут проверяться на машине.</p>
</blockquote>

<p>Отсылка к "машине" может дать вам подсказку. Эта цитата из бородатого <a href="https://arialdomartini.wordpress.com/2012/07/20/you-wont-believe-how-old-tdd-is/">1957</a>. Так что имейте ввиду, когда ваш архитектор скажет что вы не можете использовать фрейворк для заглушек или у IDE нет ранера, просто вспомните, что люди делали это еще на перфокартах!</p>

<h3>Agile отпусти!</h3>

<p>Помните как Agile грозился изменить мир? Да, этого не произошло, но у вас все еще есть возможность писать чистый, работающий, быстрый и надежный код.</p>

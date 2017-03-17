+++
date = "2014-04-14T20:37:08+03:00"
draft = false
title = "Редактор на Go и QML"

+++

<p>Вольный пересказ <a href="http://qt-project.org/doc/qt-5/gettingstartedqml.html">документации к QML</a>, адаптированный для языка программирования golang.</p>

<p>Все манипуляции производились на убунте. Не обещаю, что это взлетит на других дистрах. И вообще не факт, что будет работать на винде. Но, чем черт не шутит?</p>

<p>QML - хорошая вещь для написания&nbsp;десктопных&nbsp;интерфейсов. Сейчас активно пилится&nbsp;пакет <a href="https://github.com/go-qml/qml">go-qml</a>. Пока что, этот пакет еще не достаточно стабилен, но уже многое умеет.</p>

<p>Надо заметить, что в этом примере не будет использоваться мощный инструмент <code>RegisterType</code>, который позволяет использовать свои типы в QML</p>

<p>Презентация возможностей пакета в <a href="https://www.youtube.com/watch?v=FVQlMrPa7lI">коротком видео</a>.</p>

<h2>Установка Qt</h2>

<p>Для моей убунты 12.04 сперва нужно установить зависимости.</p>

<pre>
<code>$ sudo add-apt-repository ppa:ubuntu-sdk-team/ppa
$ sudo apt-get update
$ sudo apt-get install ubuntu-sdk qtbase5-private-dev qtdeclarative5-private-dev libqt5opengl5-dev
</code></pre>

<p>Правда, есть одно но. Стандартная установка затащит Qt версии 5.0.1, а в ней нет, например, QtQuick.Controls. Поэтому немного поизвращаемся с установкой свежей версии.</p>

<p>Самый удобный вариант установить последнюю версию Qt - это воспользоваться <a href="http://qt-project.org/downloads">.run файлом с оф. сайта</a>. Место для установки можно выбрать любое. После установки, чтобы приложение с go-qml могли нормально собираться, нужно указать где находятся исходники нашей версии библиотеки:</p>

<pre>
<code>$ export LD_LIBRARY_PATH=/home/artem/Qt5.2.1/5.2.1/gcc/lib:$LD_LIBRARY_PATH
</code></pre>

<p>Теперь устанавливаем сам пакет go-qml:</p>

<pre>
<code>$ go get gopkg.in/qml.v0
</code></pre>

<h2>Простая программа</h2>

<p>Для проверки правильности установки и работоспособности пакета напишем минимальную программу, которая почти ничего не делает.</p>

<pre>
<code>package main

import (
    &quot;fmt&quot;
    &quot;gopkg.in/qml.v0&quot;
    &quot;os&quot;
)

func main() {
    if err := run(); err != nil {
        fmt.Fprintf(os.Stderr, &quot;error: %v\n&quot;, err)
        os.Exit(1)
    }
}

func run() error {
    qml.Init(nil)

    engine := qml.NewEngine()
    component, err := engine.LoadFile(&quot;Example.qml&quot;)
    if err != nil {
        return err
    }

    win := component.CreateWindow(nil)

    win.Show()
    win.Wait()

    return nil
}
</code></pre>

<p>Любая Go программа, работающая с QML, должна выполнять несколько шагов:</p>

<ul>
	<li>Инициализировать пакет qml (<code>qml.Init(nil)</code>)</li>
	<li>Создать движок для загрузки и выполнения QML контента (<code>engine := qml.NewEngine()</code>)</li>
	<li>Сделать значения и типы из Go доступными для QML (<code>Context.SetVar</code> и <code>RegisterType</code>)</li>
	<li>Загружать QML контент (<code>component, err := engine.LoadFile(&quot;Main.qml&quot;)</code>)</li>
	<li>Создавать новое окно с контентом (<code>win := component.CreateWindow(nil)</code>)</li>
	<li>Показывать созданное окно и ждать его закрытия (<code>win.Show()</code>, <code>win.Wait()</code>)</li>
</ul>

<p>И очень простой QML код из файла <em>Example.qml</em>, который создает регион с текстом:</p>

<pre>
<code>import QtQuick 2.0

Rectangle {
    width: 360
    height: 360
    color: &quot;grey&quot;

    Text {
        id: windowText
        anchors.centerIn: parent
        text: &quot;Hello QML in Go!&quot;
    }
}
</code></pre>

<p>В результате должно полуится серенькое окно с текстом:</p>

<p><img alt="img" src="https://dl.dropboxusercontent.com/u/750049/4gophers.com/go-qml-1.png" /></p>

<h2>Кнопки для редактора</h2>

<p>Теперь можем приступать к созданию нашего редактора. Начнем с кнопочек открытия и сохранения файла. Как всегда, есть два способа. В первом случае, мы сделаем отдельный файл с названием <em>Botton.qml</em> и содержанием:</p>

<pre>
<code>import QtQuick 2.0

Rectangle {
    id: button
    radius: 6
    border.width: 3
    border.color: &quot;#ffffff&quot;
    width: 150; height: 75
    property string label: &quot;&quot;
    color: &quot;#eeeeee&quot;
    signal buttonClick()
    onButtonClick: {
    }

    Text{
        id: buttonLabel
        anchors.centerIn: parent
        text: label
    }

    MouseArea {
        id: buttonMouseArea

        anchors.fill: parent
        onClicked: buttonClick()
    }
}
</code></pre>

<p>Это будет новый тип в QML который мы можем использовать в нашем главном файле. Не нужно писать ни каких импортов, так как <em>Button.qml</em> находится в той же папке, что и <em>Example.qml</em></p>

<pre>
<code>import QtQuick 2.0

Rectangle{
    width: 360
    height: 360
    color: &quot;grey&quot;

    Button{
        id: loadButton
        label: &quot;Load&quot;
        onButtonClick: {
            console.log(&quot;Hello!&quot;)
        }
    }
}
</code></pre>

<p>Вполне работоспособный пример, который выглядит примерно вот так: <img alt="img" src="https://dl.dropboxusercontent.com/u/750049/4gophers.com/go-qml-2.png" /></p>

<p>При клике на кнопку, в консоли будет писаться &quot;Hello worl!&quot;.</p>

<h2>Готовые компоненты</h2>

<p>Такой подход дает полный контроль над внешним видом и поведением компонента, но требует больших временных затрат. Поэтому будем обходить гору и воспользуемся компонентом ToolButton из QtQuick.Controls:</p>

<pre>
<code>import QtQuick 2.0
import QtQuick.Controls 1.0

Rectangle {
    width: 360
    height: 360
    color: &quot;grey&quot;

    ToolButton {
        id: loadButton
        x: 8
        y: 8
        text: &quot;Load&quot;
        clicked: {
            console.log(&quot;Load&quot;)
        }
    }

    ToolButton {
        id: saveButton
        x: 70
        y: 8
        text: {
            console.log(&quot;Save&quot;)
        }
    }
}
</code></pre>

<p>Теперь у нас есть две кнопочки:</p>

<p><img alt="img" src="https://dl.dropboxusercontent.com/u/750049/4gophers.com/go-qml-3.png" /></p>

<p>Добавим немного жизни и радости к этому скучному дизайну. Создадим отдельный тип кнопок основанный на ToolButton и добавим к нему стилей QtQuick.Controls.Styles</p>

<pre>
<code>import QtQuick 2.0
import QtQuick.Controls 1.0
import QtQuick.Controls.Styles 1.1

ToolButton {
    style: ButtonStyle {
        background: Rectangle {
            implicitWidth: 100
            implicitHeight: 25
            border.width: control.activeFocus ? 2 : 1
            border.color: &quot;#888&quot;
            radius: 4
            gradient: Gradient {
                GradientStop { position: 0 ; color: control.pressed ? &quot;#ccc&quot; : &quot;#eee&quot; }
                GradientStop { position: 1 ; color: control.pressed ? &quot;#aaa&quot; : &quot;#ccc&quot; }
            }
        }
    }
}
</code></pre>

<p>Используем этот тип в основном файле. А за одно, добавим многострочное текстовое поле для редактирования:</p>

<pre>
<code>import QtQuick 2.0
import QtQuick.Controls 1.0

Rectangle {
    width: 360
    height: 360
    color: &quot;grey&quot;

    TextArea {
        id: textArea
        x: 8
        y: 74
        width: 344
        height: 278
    }

    ExampleButton {
        id: loadButton
        x: 8
        y: 8
        text: &quot;Load&quot;
        onClicked: {
            console.log(&quot;Load&quot;)
        }
    }

    ExampleButton {
        id: saveButton
        x: 140
        y: 8
        text: &quot;Save&quot;
        onClicked: {
            console.log(&quot;Save&quot;)
        }
    }
}
</code></pre>

<p>В итоге, получается готовый интерфейс нашего маленького приложения.</p>

<p><img alt="img" src="https://dl.dropboxusercontent.com/u/750049/4gophers.com/go-qml-4.png" /></p>

<h2>Реализуем логику</h2>

<p>Добавляем файловый диалог и загрузку файлов для редактирования. Для этого используем компоненты из <code>QtQuick.Dialogs</code></p>

<pre>
<code>import QtQuick 2.0
import QtQuick.Controls 1.0
import QtQuick.Dialogs 1.0

Rectangle {
    ExampleButton {
        id: loadButton
        //...
        onClicked: {
            console.log(&quot;Load&quot;)
            fileDialogLoad.open()
        }
    }

    //...
    FileDialog {
        id: fileDialogLoad
        folder: &quot;.&quot;
        title: &quot;Choose a file to open&quot;
        selectMultiple: false
        onAccepted: {
            console.log(&quot;Accepted: &quot; + fileDialogLoad.fileUrl)
        }
    }
}
</code></pre>

<p><code>onAccepted</code> - сработает тогда, когда в диалоговом окне будет выбран нужный файл.</p>

<p>Следующий шаг - самое интересное. Научимся передавать значения из QML в Go и наоборот. Для этого сделаем отдельный тип <code>Editor</code>:</p>

<pre>
<code>type Editor struct {
    Text string
}

func (e *Editor) SelectFile(fileUrl string) {
    fmt.Println(&quot;Selected file: &quot;, fileUrl)
    e.Text = fileUrl
    qml.Changed(e, &amp;e.Text) // нужно, чтобы qml узнал о обновлении переменной
}
</code></pre>

<p>Как видно, метод <code>SelectFile</code> получает строку и записывает ее в параметр <code>Text</code>. Нужно привыкнуть пользоваться конструкцией <code>qml.Changed(e, &amp;e.Text)</code> - именно этот вызов говорит нашему приложению что нужно обновить параметры в qml.</p>

<p>Пока не совсем понятно, зачем все это. Нужно передать этот тип в QML. Для этого есть методы <code>SetVar</code>, <code>SetVars</code>.</p>

<pre>
<code>func run() error {
//...

context := engine.Context()
context.SetVar(&quot;editor&quot;, &amp;Editor{})

//...
}
</code></pre>

<p>Так, все немного проясняется. Теперь нужно как-то захендлить Go переменную в qml коде. И тут нет ничего сложного:</p>

<pre>
<code>TextArea {
    //...
    text: editor.text
}

ExampleButton {
    id: saveButton
    //...
    onClicked: {
        console.log(&quot;Save&quot;)
        editor.saveFile(textArea.text)
    }
}

FileDialog {
    //...
    onAccepted: {
        console.log(&quot;Accepted: &quot; + fileDialogLoad.fileUrl)
        editor.selectFile(fileDialogLoad.fileUrl)
    }
}
</code></pre>

<p>Ага, теперь все понятно. <code>editor.text</code> - это обращение к параметру <code>Editor.Text</code>, a <code>editor.selectFile(fileDialogLoad.fileUrl)</code> - это вызов метода <code>Editor.SelectFile(fileUrl string)</code></p>

<p>Последний штрих - это, собственно, работа с файлами. Загрузка контента и сохранение изменений:</p>

<pre>
<code>type Editor struct {
    Text    string
    FileUrl string
}

func (e *Editor) SelectFile(fileUrl string) {
    fmt.Println(&quot;Selected file: &quot;, fileUrl)
    e.FileUrl = strings.Replace(fileUrl, &quot;file:/&quot;, &quot;&quot;, 1)
    dat, err := ioutil.ReadFile(e.FileUrl)
    if err != nil {
        log.Println(err)
    }
    e.Text = string(dat)
 }

func (e *Editor) SaveFile(text string) {
    dat := []byte(text)
    err := ioutil.WriteFile(e.FileUrl, dat, 0644)
    if err != nil {
        log.Println(err)
    }
}
</code></pre>

<p>Вот и все. Наш маленький редактор, написанный с использованием Go и QML готов. Теперь можно браться за написание своей вижуал студии. Все исходники можно <a href="https://github.com/4gophers/qmleditor">стянуть с гитхаба</a>.</p>

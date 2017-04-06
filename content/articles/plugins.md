+++
date = "2017-04-05T19:59:09+03:00"
draft = true
title = "Пишем модульную Go программу с плагинами"

+++

Перевод статьи "[Writing Modular Go Programs with Plugins](https://medium.com/learning-the-go-programming-language/writing-modular-go-programs-with-plugins-ec46381ee1a9)"

Среди всех фич, которые появились в Go 1.8 есть система плагинов. С ее помощью можно создавать модульные программы используя пакеты как динамически загружаемые в рантайме библиотеки.

Это открывает большие возможности. Наверняка вы замечали, что разработчикам больших систем на Go неизбежно приходится структурировать по модулям свое приложение. Мы можем использовать различные инструменты для мудуляризации нашего приложения, такие как [системные вызовы](https://kubernetes.io/docs/admin/network-plugins/), [сокеты](https://docs.docker.com/engine/extend/plugin_api/), [RPC/gRPC](https://github.com/hashicorp/go-plugin) и т.д. Несмотря на то, что перечисленные подходы работают, все это говорит о том, что не плохо было бы иметь нативную поддержку системы плагинов.

В этой статье я хочу показать небольшой пример создания модульного приложения с использованием систем Go плагинов. Я постараюсь рассказать о всех деталях, знание которых вам понадобится для создание полно функционального примера и затрону тему проектирования более серьезных вещей.

### Плагины в Go

Плагины в Go, по своей сути, это пакеты, скомпилированные с указанием флага `-buildmode=plugin` в общие динамические библиотеки (файлы .so). Экспортируемые функции и переменные в этом пакете остаются открытыми как [ELF](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) символы, которые могут быть использованы в рантайме с помощью пакета `plugin`

>В одной из моих [прошлых статей](https://medium.com/learning-the-go-programming-language/calling-go-functions-from-other-languages-4c7d8bcc69bf#.b49hgos9n) я рассказывал, что Go компилятор, при использовании флага `-buildmode=c-shared`, может делать совместимые с сишными динамические библиотеками.

#### Ограничения

В версии Go 1.8 плагины доступны доступны только для Linux. Возможно, в будущем что-то поменяется, особенно если к этой фиче проявят много интереса. 

### Простая программа с плагинами

В этом разделе посмотрим как написать маленькую программу использующую плагины, которая печает в консоли приветствие на различных языках. Каждый язык в этой программе был реализован через плагин.

>Можете сразу посмотреть на пример реализации - [https://github.com/vladimirvivien/go-plugin-example](https://github.com/vladimirvivien/go-plugin-example)

Эта программа, greeter.go, использует плагины, которые реализуются пакетами `./eng` и `./chi`, для вывода приветствия на английском и китайском, соответственно. На картинке снизу показана примерная структура программы.

![](/img/1-struct.png)

Прежде всего, рассмотрим код `eng/greeter.go` который выводит сообщение на английском языке.

Код находится в файле [./eng/greeter.go](https://github.com/vladimirvivien/go-plugin-example/blob/master/eng/greeter.go)

```go
package main

import "fmt"

type greeting string

func (g greeting) Greet() {
    fmt.Println("Hello Universe")
}

// экспортируется как символ с именем "Greeter"
var Greeter greeting
```

Код выше это все содержимое пакета. Вам нужно учитывать несколько вещей:
* Сам пакет, не зависимо от папки а которой он лежит, должен называться main
* Экспортируемые функции и переменные становятся доступными символами в динамической библиотеке. В примере выше экспортируемая переменная `Greeter` экспортируется как символ в динамической библиотеке.

#### Компилирование плагина

Плагины компилируются с помощью команд ниже:

```
go build -buildmode=plugin -o eng/eng.so eng/greeter.go
go build -buildmode=plugin -o chi/chi.so chi/greeter.go
```


#### Использование плагинов

Плагины загружаются динамически с использованием специального пакета `plugin`. Клиентская программа [./greeter.go](https://github.com/vladimirvivien/go-plugin-example/blob/master/greeter.go) использует заранее скомпилированные плагины как указано ниже:

Файл ./greeter.go

```
package main

import "plugin"; ...


type Greeter interface {
    Greet()
}

func main() {
    // определяем пакет для загрузки
    lang := "english"
    if len(os.Args) == 2 {
        lang = os.Args[1]
    }
    var mod string
    switch lang {
    case "english":
        mod = "./eng/eng.so"
    case "chinese":
        mod = "./chi/chi.so"
    default:
        fmt.Println("don't speak that language")
        os.Exit(1)
    }

    // загружаем плагин
    // 1. открываем .so файл для загрузки символов
    plug, err := plugin.Open(mod)
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }

    // 2. выполняем поиск символов(экспортированных функций или переменных)
    // в нашем случае, это переменная Greeter
    symGreeter, err := plug.Lookup("Greeter")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }

    // 3. делаем возможным работу с этими символами в нашем коде
    // нужно не забывать про тип экспортированных символов
    var greeter Greeter
    greeter, ok := symGreeter.(Greeter)
    if !ok {
        fmt.Println("unexpected type from module symbol")
        os.Exit(1)
    }

    // 4. используем загруженный плагин
    greeter.Greet()

}
```

Как видно из кода выше, нужно выполнить несколько определенных шагов для загрузки плагина и работы с его интерфейсами в вашем коде.
* Прежде всего, нужно определится с названием плагина, который нужно загрузить. В нашем случае имя плагина передается через аргументы `os.Args`.
* Необходимо добраться до нужного символа "Greeter" с помощью вызова `plguin.Lookup("Greeter")`. Название символа совпадает с названием экспортируемых переменных и функций, определенных в пакете плагина.
* Приводим найденный символ к нужному интерфейсу с помощью конструкции `symGreeter.(Greeter)`.
* Теперь можем спокойно вызывать `Greet()`, который выведет приветствие на английском.

#### Запускаем программу

Программа выводит в консоль приветствие на английском или китайском, в зависимости от того, какой параметр указан при запуске, как показано ниже.

```
> go run greeter.go english
Hello Universe
> go run greeter.go chinese
你好宇宙
```

Самое главное преимущество такого подхода к проектированию приложения, это возможность в рантайме изменять логику работы приложения(вывод приветствия) без необходимости перекомпилирования самого приложения.

### Модульный дизайн Go приложений

Создание модульных приложений на основе Go плагинов требует не менее строгого подход к разработке, чем при проектировании обычных приложений. Тем не менее, благодаря своей разделяющей сущности, плагины позволяют использовать некоторые новые концепции.

#### Clear affordances

When building pluggable software system, it is important to establish clear component affordances. The system must provide clean a simple surfaces for plugin integration. Plugin developers, on the other hand, should consider the system as black box and make no assumptions other than the provided contracts.

#### Plugin Independence

A plugin should be considered an independent component that is decoupled from other components. This allows plugins to follow their own development and deployment life cycles independent of their consumers.

#### Apply Unix modularity principles

A plugin code should be designed to focus on one and only one functional concern.

#### Clearly documented

Since plugins are independent components that are loaded at runtime, it is imperative they are well-documented. For instance, the names of exported functions and variables should be clearly documented to avoid symbol lookup errors.

#### Use interface types as boundaries

Go plugins can export both package functions and variables of any type. You can design your plugin to bundle its functionalities as a set of loose functions. The downside is you have to lookup and bind each function symbol separately.

A tidier approach, however, is to use interface types. Creating an interface to export functionalities provides a uniform and concise interaction surface with clear functional demarcations. Looking up and binding to a symbol that resolves to an interface will provide access to the entire method set for the functionality, not just one.

#### New deployment paradigm

Plugins have the potential of impacting how software is built and distributed in Go. For instance, library authors could distribute their code as pre-built components that can be linked at runtime. This would represent a departure from the the traditional `go get`, build, and link cycle.

#### Trust and security

If the Go community gets in the habit of using pre-built plugin binaries as a way of distributing libraries, trust and security will naturally become a concern. Fortunately, there are already established communities coupled with reputable distribution infrastructures that can help here.

#### Versioning

Go plugins are opaque and independent entities that should be versioned to give its users hints of its supported functionalities. One suggestion here is to use semantic versioning when naming the shared object file. For instance, the file compiled plugin above could be named `eng.so.1.0.0` where suffix `1.0.0` repents its semver.

### Gosh: a pluggable command shell

I will go ahead and plug (no pun, really) this project I started recently. Since the plugin system was announced, I wanted to create a pluggable framework for creating interactive command shell programs where commands are implemented using Go plugins. So I created [Gosh](https://github.com/vladimirvivien/gosh) (Go shell).

>Learn about Gosh in this [blog post](https://medium.com/@vladimirvivien/gosh-a-go-pluggable-console-shell-cf25102c8439)

Gosh uses a driver shell program to load the command plugins at runtime. When a user types a command at the prompt, the driver dispatches the plugin registered to handle the command. This is an early attempt, but it shows the potential power of the Go plugin systems.

### Conclusion

I am excited about the addition of shared object plugin in Go. I think it adds an important dimension to the way Go programs can be assembled and built. Plugins will make it possible to create new types of Go systems that take advantage of the late binding nature of dynamic shared object binaries such as Gosh, or distributed systems where plugin binaries are pushed to nodes as needed, or containerized system that finalize assembly at runtime, etc.

Plugins are not perfect. They have their flaws (their rather large sizes for now) and can be abused just like anything in software development. If other languages are an indication, I can predict plugin versioning hell will be a pain point. Those issues aside, having a modular solutions in Go makes for a healthier platform that can strongly support both single-binary and modularized deployment models.

As always, if you find this writeup useful, please let me know by clicking on the ♡ icon to recommend this post.

Also, don’t forget to checkout my book on Go, titled Learning Go Programming from Packt Publishing.
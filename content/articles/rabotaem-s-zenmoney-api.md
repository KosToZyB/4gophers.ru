+++
date = "2015-06-07T03:07:03+03:00"
draft = false
title = "Работаем с zenmoney API"

+++

<p>Возможно вы знаете, что я обожаю сервис для учета финансов <a href="http://zenmoney.ru">zenmoney.ru</a>. Интерфейс и юзабельность, как по мне, просто на высоте. А самое главное - это их REST API, которое позволяет писать все возможные клиенты и делать экспорт ваших транзакций.</p>

<p>К сожалению, документации по API не так уж много и иногда попадаются достаточно противные ошибки. Поэтому я постараюсь провести вас через эти дебри и покажу как написать простой консольный клиент для работы с транзакциями.</p>

<h3>Получаем доступы к API</h3>

<p>У zenmoney есть отдельная страничка для разработчиков, на которой собрана вся <a href="http://developers.zenmoney.ru/index.html">информация по работе с API</a>.</p>

<p>Прежде всего нам нужно добавить новое приложение. Для этого воспользуйтесь формой, которая появляется по клику на слове "скриптом" в разделе "Параметры OAuth-провайдера" или <a href="http://api.zenmoney.ru/consumer.html">пройдите по этой ссылке</a>.</p>

<p>Для регистрации приложения необходимо заполнить простую форму:</p>

<p><img src="https://dl.dropboxusercontent.com/u/750049/4gophers.com/reg_app.png" alt="" /></p>

<p>После отправки формы, в этом же окне(или на странице) вы увидите данные для вашего приложения, что-то вот такое:</p>

<blockquote>
  <p>Приложение зарегистрировано</p>
  
  <p>Пожалуйста, сохраните данные параметры, вам необходимо будет указать в настройках библиотеки OAuth вашего приложения:
  Consumer Key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  Consumer Secret: xxxxxxxxxx
  Request token endpoint: http://api.zenmoney.ru/oauth/request_token
  Access token endpoint: http://api.zenmoney.ru/oauth/access_token
  Authorize URL: http://api.zenmoney.ru/access/
  Authorize URL для мобильных устройств: http://api.zenmoney.ru/access/?mobile</p>
</blockquote>

<p><em>Обязательно</em> сохраните эти данные. Я так и не нашел способа как-то просматривать параметры уже зарегистрированных приложений или менять эти параметры. Поэтому надежно сохраните всю информацию.</p>

<p>После регистрации нашего приложения, можем начинать писать свой консольный клиент.</p>

<h3>Авторизация</h3>

<p>Как вы могли заметить, авторизация для API в zenmoney работает по OAuth протоколу. Cтарый добрый первый OAuth... Будем использовать пакет для безболезненной работы с этим проколом. Нас полностью устроит <a href="https://github.com/mrjones/oauth">github.com/mrjones/oauth</a></p>

<p>Небольшое отступление. Чтобы не мучатся с флагами и ифами, будем использовать пакет <a href="https://github.com/codegangsta/cli">github.com/codegangsta/cli</a>, который, как по мне, значительно упрощает написание консольных утилит.</p>

<p>И так, первым делом готовим настройки для нашей авторизации. В файле 4gophers.rum/zenmoney/main.go:</p>

<pre><code class="go">package main

import (
    "github.com/codegangsta/cli"
    "github.com/mrjones/oauth"
)

func main() {
    consumer := oauth.NewConsumer(
        "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "xxxxxxxxxx",
        oauth.ServiceProvider{
            RequestTokenUrl:   "http://api.zenmoney.ru/oauth/request_token",
            AuthorizeTokenUrl: "http://api.zenmoney.ru/access",
            AccessTokenUrl:    "http://api.zenmoney.ru/oauth/access_token",
        })

    consumer.Debug(false)

    app := cli.NewApp()
    app.Name = "zen"
    app.Usage = "auth, account"
    app.Commands = []cli.Command{
        {
            Name:  "auth",
            Usage: "get tokens",
            Action: func(c *cli.Context) {
                // ...
            },
        },
    }

    app.Run(os.Args)
</code></pre>

<p>Прежде всего, мы создаем <code>consumer</code>(новый экземпляр <code>*oauth.Consumer</code>), указывая те самые настройки, которые вы получили после регистрации приложения. Первым мы указываем Consumer Key, затем Consumer Secret. Последний параметр - экземпляр <code>oauth.ServiceProvider</code> с предустановленными настройками, необходимыми для успешной авторизации.</p>

<p>Давайте добавим отдельный пакет, назовем его <code>4gophers.ru/zenmoney/api</code>. В этом пакете будем реализовывать все наши функции для работы с API.</p>

<p>Начнем с функции <code>api.Auth()</code>. Эта функция должна принимать указатель на <code>oauth.Consumer</code>, а результатом работы этой функции должны быть токен и секретный ключ.</p>

<p>В файле 4gophers.rum/zenmoney/api/zenmoney.go добавим код:</p>

<pre><code class="go">package api

import (
    "fmt"
    "log"

    "github.com/mrjones/oauth"
)

func Auth(c *oauth.Consumer) {

    requestToken, url, err := c.GetRequestTokenAndUrl("oob")
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println("(1) Go to: " + url)
    fmt.Println("(2) Grant access, you should get back a verification code.")
    fmt.Println("(3) Enter that verification code here: ")

    var verificationCode string
    fmt.Scanln(&amp;verificationCode)

    accessToken, err := c.AuthorizeToken(requestToken, verificationCode)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("token: %s secret: %s", accessToken.Token, accessToken.Secret)
}
</code></pre>

<p>Вызов <code>c.GetRequestTokenAndUrl("oob")</code> отдает нам <code>requestToken</code> и <code>url</code>. Строка <code>"oob"</code>, которую мы передали вместо url для коллбека, означает "out-of-band" -  это значит, что нам не нужно вызов нашего коллбека(у нас его просто нет)и пользователь сам сам будет решать как поступить с полученными токенами.</p>

<p>Далее мы выводим в консоль url, по которому должен пройти пользователь и ждем пока он введет проверочный код.</p>

<p><img src="https://dl.dropboxusercontent.com/u/750049/4gophers.com/auth.png" alt="" /></p>

<p>Если вы перейдете по указанному url в браузере, то после логина вас перекинет на указанный в настройках(при регистрации приложения) коллбек url. К сожалению, насколько я вник в тему, zenmoney не может работать совсем без редиректа. Ну что ж, это не страшно. В крайнем случае, мы можем сделать страничку, которая будет красиво отображать проверочный код, который приходит нам от сервера zenmoney. А пока, можно просто скопировать значение параметра <code>oauth_verifier</code> из url и вставить его в консоль.</p>

<p>Последнее, что мы должны сделать, это получить токены для доступа:</p>

<pre><code class="go">accessToken, err := c.AuthorizeToken(requestToken, verificationCode)
</code></pre>

<p>Теперь у нас есть честно добытые <code>access.Token</code> и <code>access.Secret</code>. Их нужно сохранить. Конечно, хранить токены в открытом виде это не безопасно, но для примера приложения вполне оправдано.</p>

<p>Для простого хранения токенов, будем использовать JSON. Для этого нам нужен пакет <code>encoding/json</code>.</p>

<pre><code class="go">package api

import (
    "encoding/json"
    "os"
    // ...

    "github.com/kardianos/osext"
    // ...
)

func Auth(c *oauth.Consumer) {
    // ...

    folder, err := osext.ExecutableFolder()
    if err != nil {
        log.Fatal(err)
    }

    file, err := os.Create(folder + "/keys.json")
    if err != nil {
        log.Fatal(err)
    }

    json.NewEncoder(file).Encode(access)
}
</code></pre>

<p>Возможно вас смутит пакет <a href="https://github.com/kardianos/osext">github.com/kardianos/osext</a> и вызов <code>osext.ExecutableFolder()</code>, но эта большая хитрость необходима, чтобы файл keys.json всегда создавался рядом с бинарником программы и не зависел от папки, откуда вы его запускаете. Это сослужит вам хорошую службу, когда вы будете устанавливать приложение глобально, с помощью <code>go get</code>.</p>

<p>Не забудьте вызвать <code>Auth()</code> в main.go:</p>

<pre><code class="go">package main

import (
    // ...
    "4gophers.ru/zenmoney/api"
)

// ...
app.Commands = []cli.Command{
    {
        Name:  "auth",
        Usage: "get tokens",
        Action: func(c *cli.Context) {
            api.Auth(c)
        },
    },
}
// ...
</code></pre>

<p>На этом с авторизацией можно заканчивать. Если вас интересует больше информации по OAuth, то на странице документации zenmoney приводится несколько отличных ссылок, которые могут многое прояснить.</p>

<h3>Список транзакций</h3>

<p>Теперь у нас есть все необходимые ключи и мы можем спокойно выполнять запросы к API. Для начала, сделаем функцию, которая будет красиво выводить список последних транзакций.</p>

<p>Нам нужно добавить еще одну команду в файле main.go</p>

<pre><code class="go">package main

app.Commands = []cli.Command{
    // ...
    {
        Name:  "list, l",
        Usage: "show list of transaction",
        Action: func(c *cli.Context) {
            api.TransactionList(consumer)
        },
    },
    // ...
}
</code></pre>

<p>Теперь реализуем запрос к API в методе <code>api.TransactionList()</code>.</p>

<p>Так как, у нас уже есть ключи для авторизации, то неплохо было бы подготовить указатель на <code>oauth.AccessToken</code> для использования в запросах к API.</p>

<pre><code class="go">package api

import (
    // ...
    "io/ioutil"
)

func TransactionList(c *oauth.Consumer) {
    access := &amp;oauth.AccessToken{Token: "xxxxx", Secret: "xxxxx"}

    response, err := c.Get(
        "http://api.zenmoney.ru/v1/transaction/",
        map[string]string{},
        access)
    if err != nil {
        log.Fatal(err)
    }
    defer response.Body.Close()

    bits, err := ioutil.ReadAll(response.Body)
    fmt.Println("Транзакции: ", string(bits))
}
</code></pre>

<p>В примере выше мы напрямую задаем <code>Token</code> и <code>Secret</code>. Это совсем не правильный подход, так как у нас уже сохраненный JSON с правильными ключами. Давайте будем использовать его.</p>

<pre><code class="go">func TransactionList(c *oauth.Consumer) {
    file, err := os.Open(folder + "/keys.json")
    if err != nil {
        log.Fatal(err)
    }

    access := &amp;oauth.AccessToken{}
    if err = json.NewDecoder(file).Decode(access); err != nil {
        log.Fatal(err)
    }
    // ...
}
</code></pre>

<p>Инициализацию переменной <code>folder</code> мы вынесли в функции <code>init()</code> пакета <code>4gophers.ru/zenmoney/api</code>. Она нам еще не раз пригодится и со временем у нас добавятся еще переменные, которые будут нужны в разных функциях.</p>

<pre><code class="go">package api

var folder string

func init() {
    var err error
    folder, err = osext.ExecutableFolder()
    if err != nil {
        log.Fatal(err)
    }
}
</code></pre>

<p>Обратите внимание на URL для обращение к API:</p>

<pre><code class="go">response, err := c.Get(
    "http://api.zenmoney.ru/v1/transaction/",
    map[string]string{},
    access)
</code></pre>

<p>Тут стоит сделать небольшое лирическое отступление. К сожалению, на момент написания статьи, в документации была ошибка, и в разделе "Транзакции" нам предлагали использовать url <code>/v1/instrument/currency/</code>, что конечно же не верно(хотя описание структуры именно для транзакции). Путем непродолжительных логический размышлений было обнаружено, что транзакции можно получить по <code>/v1/transaction/</code>. Надеюсь, разработчики API в ближайшее время пофиксят эту ошибку в документации.</p>

<p>И так, теперь у нас есть большой JSON со всеми транзакциями. Давайте превратим его в список структур. Для этого нам нужно описать саму структуру транзакции. Будем надеяться, что тут нас документация не подведет.</p>

<p>Вот так выглядит одна транзакция:</p>

<pre><code>type Transaction struct {
    Id                int    `json:"id"`
    AccountIncome     int    `json:"account_income"`
    AccountOutcome    int    `json:"account_outcome"`
    Income            string `json:"income"`
    Outcome           string `json:"outcome"`
    Comment           string `json:"comment"`
    InstrumentIncome  int    `json:"instrument_income"`
    InstrumentOutcome int    `json:"instrument_outcome"`
    TypeIncome        string `json:"type_income"`
    TypeOutcome       string `json:"type_outcome"`
    Direction         int    `json:"direction"`
    Group             string `json:"tag_group"`
    Date              string `json:"date"`
    Category          int    `json:"category"`
}

type Transactions []Transaction
</code></pre>

<p>Кажется подозрительным, что <code>Income</code> и <code>Outcome</code> мы описали как <code>string</code>, хотя на самом деле это числовые типы. Тут имеет место еще одна не состыковка в API: нам эти параметры приходят в виде строк. Конечно, в таком несерьезном приложении это мелочь, но если вы захотите выполнять какие-то операции с этими полями, то имейте ввиду, все не так просто. Дальше будут описаны еще интересные нюансы, связанные с транзакциями и их представлением в виде JSON.</p>

<p>Список таких структур (<code>[]Transaction</code>) нам нужно возвращать из функции <code>TransactionList()</code>:</p>

<pre><code class="go"><br />func TransactionList(c *oauth.Consumer) Transactions {
    // ...

    bits, err := ioutil.ReadAll(response.Body)

    tarnsactions := make(Transactions, 0)
    json.Unmarshal(bits, &amp;tarnsactions)

    return tarnsactions
}
</code></pre>

<p>Конечно же, мы хотим в удобном формате просматривать все наши транзакции. Для этого воспользуемся пакетом <code>text/tabwriter</code>. Добавим немного кода после вызова <code>TransactionList()</code>:</p>

<pre><code class="go">transactions := api.TransactionList(consumer)
w := new(tabwriter.Writer)
w.Init(os.Stdout, 4, 8, 1, '\t', 0)
fmt.Fprintln(w, "Id\tDate\tIncome\tOutcome\tGroup\t")

for _, transacrtion := range transactions {
    fmt.Fprintln(w, strconv.Itoa(transacrtion.Id)+"\t"+
        transacrtion.Date+"\t"+
        transacrtion.Income+"\t"+
        transacrtion.Outcome+"\t"+
        transacrtion.Group+"\t")
}

fmt.Fprintln(w)
w.Flush()
</code></pre>

<p>Вывод транзакций, полученных от API, выглядит как симпатичная табличка:</p>

<p><img src="https://dl.dropboxusercontent.com/u/750049/4gophers.com/list.png" alt="" /></p>

<p>И тут у нас есть еще одна большая ложка дегтя. Вы наверняка заметили, что у нашей структуры есть поле <code>Transaction.Category</code>, но оно всегда будет равно 0. А вот те категории, которые вы указываете в веб приложении, будут совсем в других полях. Если, при добавлении транзакции, вы указали только одну категорию, то ее id будет в поле <code>Transaction.Group</code>(в JSON это поле <code>tag_group</code>), а если указали несколько категорий, то они будут в JSON поле <code>tag_groups</code> в виде списка. В этом, конечно, ничего страшного нет. Но вот беда, никак нельзя получить данные по этим тегам, даже названия. Да, в API есть метод <code>http://api.zenmoney.ru/v1/category/</code> (кстати, тут нашлась вторая ошибка в документации: в разделе "Категории" указан URL <code>/v1/account/</code>), но id этих категорий никак не соответствуют id <code>tag_group</code>. И нет никакой возможности связать категории с их названиями. Возможно, я не обладаю неким тайным знанием и на самом деле существует метод для получения инфы по <code>tag_group</code>, но у меня ничего не получилось.</p>

<p>А это значит, что вместо вменяемых названий категорий, у нас буду просто id. И так сойдет (с).</p>

<h3>Добавление новой транзакции</h3>

<p>В принципе, все основные идеи работы с API уже описаны. Все что нам нужно: прочитать токены из файла, сделать запрос к апи, обработать ответ. Для добавления новой транзакции нужно отправить POST запрос с JSON, сформированным из нашей структуры <code>Transaction</code>, на URL <code>/v1/transaction/</code>.</p>

<p>Но тут тоже не все так просто. Для правильного добавления транзакции, нам нужно указать номер счета(например у меня есть "наличные" и "карта"). К счастью, все счета мы можем спокойно получить с помощью самого API.</p>

<p>Реализуем функцию <code>api.AccountList()</code>, которая будет возвращать map всех существующих аккаунтов по аналогии с транзакциями.</p>

<pre><code class="go">// ...
type Account struct {
    Id         int    `json:"id"`
    Type       string `json:"type"`
    Title      string `json:"title"`
    Balance    string `json:"balance"`
    Instrument int    `json:"instrument"`
}

type Accounts map[string]Account

// ...

func AccountList(c *oauth.Consumer) Accounts {
    access := getAccess()

    response, err := c.Get(
        "http://api.zenmoney.ru/v1/account",
        map[string]string{},
        access)
    if err != nil {
        log.Fatal(err)
    }
    defer response.Body.Close()

    bits, err := ioutil.ReadAll(response.Body)

    accounts := make(Accounts, 0)
    json.Unmarshal(bits, &amp;accounts)

    fmt.Println(accounts)

    return accounts
}
</code></pre>

<p>И теперь в <code>main.go</code> добавляем новую команду и вызов функции <code>api.AccountList()</code>.</p>

<pre><code class="go">{
    Name:      "account",
    ShortName: "ac",
    Usage:     "show auth url and save tokens",
    Action: func(c *cli.Context) {
        accounts := api.AccountList(consumer)

        w := new(tabwriter.Writer)
        w.Init(os.Stdout, 4, 8, 1, '\t', 0)
        fmt.Fprintln(w, "Id\tTitle\tBalance\t")

        for _, account := range accounts {
            fmt.Fprintln(w, strconv.Itoa(account.Id)+"\t"+
                account.Title+"\t"+
                account.Balance+"\t")
        }

        fmt.Fprintln(w)
        w.Flush()
    },
},
</code></pre>

<p>Да да, это очень похоже на отображение транзакций. Аккаунты будут выводиться аналогично транзакциям в красивой табличке. Для нас самое важное - это поле <code>Id</code>. Значение из этого поля нужно будет для создания новой транзакции, мы будем указывать его как <code>Transaction.AccountIncome</code> и <code>Transaction.AccountOutcome</code></p>

<p>Можно переходить к самой реализации добавления.</p>

<pre><code class="go">func TransactionAdd(c *oauth.Consumer, 
                    dir string, 
                    amount, account int) Transaction {
    access := getAccess()

    income := 0
    outcome := 0
    direction := 0

    if dir == "in" {
        direction = 1
        income = amount
    } else {
        direction = -1
        outcome = amount
    }

    if dir == 0 {
        log.Fatal("Undefined direction")
    }

    response, err := c.Put(
        "http://api.zenmoney.ru/v1/transaction/",
        `{
            "account_income":`+strconv.Itoa(account)+`,
            "account_outcome":`+strconv.Itoa(account)+`,
            "income":`+strconv.Itoa(income)+`,
            "outcome":`+strconv.Itoa(outcome)+`,
            "comment":"",
            "instrument_income":2,
            "instrument_outcome":2,
            "type_income":"cash",
            "type_outcome":"cash",
            "direction": `+strconv.Itoa(direction)+`,
            "date":"`+time.Now().Format("2006-01-02")+`"
            }`,
        map[string]string{},
        access)

    if err != nil {
        log.Fatal(err)
    }

    defer response.Body.Close()

    transaction := Transaction{}

    bits, err := ioutil.ReadAll(response.Body)
    if err != nil {
        log.Fatal(err)
    }

    err = json.Unmarshal(bits, &amp;transaction)
    if err != nil {
        log.Fatal(err)
    }

    return transaction
}
</code></pre>

<p>Да, уважаемый читатель, это страх и ужас. Мы не можем использовать структуру <code>transaction</code>, так как, когда мы получаем список транзакций, то JSON поля <code>income</code> и <code>outcome</code> это строки, а при добавлении они становятся интами. Вот так вот. Как вариант, можно создать отдельную структуру транзакции, которую можно использовать для добавления, но это не делает код значительно лучше.</p>

<p>Если у вас есть идеи как можно упростить эту функцию, милости просим в комментарии.</p>

<p>При добавлении транзакции нам нужно указывать суму, счет и направление(расход это или доход). Было бы круто, указывать категорию, но, к сожалению, пока не понятно как это сделать. Конечно, можно получить список прям с фронтенда веб версии, но это уж чересчур хардкорно.</p>

<h3>Удаление</h3>

<p>Слава Богу, тут все просто. Все что нам нужно - это послать DELETE запрос с указанием id транзакции.</p>

<p>Добавляем новую команду для работы с транзакциями:</p>

<pre><code class="go">{
    Name:      "remove",
    ShortName: "rm",
    Usage:     "remove an existing transaction",
    Action: func(c *cli.Context) {
        if err := api.TransactionDelete(consumer, c.String("id")); err != nil {
            fmt.Println(err)
        }
        fmt.Println("Removed!")
    },
    Flags: []cli.Flag{
        cli.StringFlag{
            Name:  "id",
            Value: "",
            Usage: "transaction id",
        },
    },
},
</code></pre>

<p>И реализуем функцию удаления. Вторым параметром функция принимает id транзакции для удаления.</p>

<pre><code class="go">// TransactionDelete remove transaction by id
func TransactionDelete(c *oauth.Consumer, id string) error {
    access := getAccess()
    response, err := c.Delete(
        "http://api.zenmoney.ru/v1/transaction/"+id, 
        map[string]string{}, 
        access)
    if err != nil {
        return err
    }

    defer response.Body.Close()

    return nil
}
</code></pre>

<h3>Заключение</h3>

<p>С первого взгляда казалось, что работа с этим API достаточно тривиальна, но мелкие проблемы и опечатки в документации все портят.</p>

<p>За кадром остался обзор расширенного API для упрощенной синхронизации клиента и сервера, но тут не хватило бы размеров статьи :)</p>

<p>В целом, с API zenmoney можно сносно работать. И я надеюсь, что в будущем API будет только улучшаться, а ошибки исправят.</p>

<p>Как всегда, все примеры к статье <a href="https://github.com/4gophers/zenmoney">можно найти на github</a>.</p>

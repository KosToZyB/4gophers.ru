+++
date = "2018-01-15T02:08:02+03:00"
draft = false
title = "Разработка твиттер ботнета на основе цепей Маркова"
tags = ["bot", "twitter", "markov", "golang"]

+++

![](/img/markovchain/main.png)

Перевод "[Developing a Twitter botnet based on Markov chains in Go](http://arnaucode.com/blog/flock-botnet.html)"

Основная идея этой статьи - рассказать как написать твиттер ботнет с автономными ботами которые смогут отвечать на другие твиты текстом сгенерированным с помощью алгоритма цепей Маркова. Так как это обучающий минипроект, то мы будем делать все сами и с самого нуля.

Идея совместить алгоритм цепей Маркова и твиттер ботов появилась после общения с [x0rz](https://twitter.com/x0rz).

### Цепи Маркова

Цепь маркова это последовательность стохастических событий(основанных на вероятности) где текущее состояние переменной или системы не зависит только от предыдущего события и не зависит от всех остальных прошедших событий.

[https://en.wikipedia.org/wiki/Markov_chain](https://en.wikipedia.org/wiki/Markov_chain)

В нашем случае мы будем использовать цепочки маркова для анализа вероятности что после некоторого слова идет другое определенное слово. Нам нужно будет сгенерировать граф, вроде того что на рисунке ниже, только с тысячами слов.

![](/img/markovchain/markovchain.png)

На вход нам нужно подавать документ с тысячами слов для более качественного результата. Для нашего примера мы будем использовать книгу Иммануила Канта ["The Critique of Pure Reason"](http://www.gutenberg.org/cache/epub/4280/pg4280.txt). Просто потому что это первая книга которая мне попалась в текстовом формате.

#### Расчет цепи

Прежде всего нам нужно прочитать файл

```go
func readTxt(path string) (string, error) {
	data, err := ioutil.ReadFile(path)
	if err != nil {
		//выполняем необходимую работу
	}
	dataClean := strings.Replace(string(data), "\n", " ", -1)
	content := string(dataClean)
	return content, err
}
```

Для вычисления вероятности состояний нам нужно написать функцию, которая будет на вход принимать текст, анализировать его и сохранять состояния Маркова.

```go
func calcMarkovStates(words []string) []State {
	var states []State
	// считаем слова
	for i := 0; i < len(words)-1; i++ {
		var iState int
		states, iState = addWordToStates(states, words[i])
		if iState < len(words) {
			states[iState].NextStates, _ = addWordToStates(states[iState].NextStates, words[i+1])
		}

		printLoading(i, len(words))
	}

	// считаем вероятность
	for i := 0; i < len(states); i++ {
		states[i].Prob = (float64(states[i].Count) / float64(len(words)) * 100)
		for j := 0; j < len(states[i].NextStates); j++ {
			states[i].NextStates[j].Prob = (float64(states[i].NextStates[j].Count) / float64(len(words)) * 100)
		}
	}
	fmt.Println("\ntotal words computed: " + strconv.Itoa(len(words)))
	return states
}
```

Функция `printLoading` выводит в теримал прогресбар просто для удобства.

```go
func printLoading(n int, total int) {
	var bar []string
	tantPerFourty := int((float64(n) / float64(total)) * 40)
	tantPerCent := int((float64(n) / float64(total)) * 100)
	for i := 0; i < tantPerFourty; i++ {
		bar = append(bar, "█")
	}
	progressBar := strings.Join(bar, "")
	fmt.Printf("\r " + progressBar + " - " + strconv.Itoa(tantPerCent) + "")
}
```

И выглядит это вот так:

![](/img/markovchain/progressbarMarkov.gif)

#### Генерация текста по цепи Маркова

Для генерации текста нам нужно первое слово и длина генерируемого текста. После этого запускается цикл в котором мы выбираем слова по вероятностям, рассчитанным при составлении цепи на прошлом шаге.

```go
func (markov Markov) generateText(states []State, initWord string, count int) string {
	var generatedText []string
	word := initWord
	generatedText = append(generatedText, word)
	for i := 0; i < count; i++ {
		word = getNextMarkovState(states, word)
		if word == "word no exist on the memory" {
			return "word no exist on the memory"
		}
		generatedText = append(generatedText, word)
	}
	text := strings.Join(generatedText, " ")
	return text
}
```

Для генерации нам нужна функция, котрая принимает на вход всю цепь и некоторое слово, а возвращает другое слово на основе вероятности:

```go
func getNextMarkovState(states []State, word string) string {
	iState := -1
	for i := 0; i < len(states); i++ {
		if states[i].Word == word {
			iState = i
		}
	}
	if iState < 0 {
		return "word no exist on the memory"
	}
	var next State
	next = states[iState].NextStates[0]
	next.Prob = rand.Float64() * states[iState].Prob
	for i := 0; i < len(states[iState].NextStates); i++ {
		if (rand.Float64()*states[iState].NextStates[i].Prob) > next.Prob && states[iState-1].Word != states[iState].NextStates[i].Word {
			next = states[iState].NextStates[i]
		}
	}
	return next.Word
}
```

### Твиттер АПИ

Для работы с АПИ твиттера будем использовать пакет [go-twitter](https://github.com/dghubble/go-twitter)

Нам нужно настроить стриминг соединение - мы будем фильтровать твиты по определенным словам, которые есть в нашем исходном наборе:

```go
func startStreaming(states []State, flock Flock, flockUser *twitter.Client, botScreenName string, keywords []string) {
	// Convenience Demux demultiplexed stream messages
	demux := twitter.NewSwitchDemux()
	demux.Tweet = func(tweet *twitter.Tweet) {
		if isRT(tweet) == false && isFromBot(flock, tweet) == false {
			processTweet(states, flockUser, botScreenName, keywords, tweet)
		}
	}
	demux.DM = func(dm *twitter.DirectMessage) {
		fmt.Println(dm.SenderID)
	}
	demux.Event = func(event *twitter.Event) {
		fmt.Printf("%#v\n", event)
	}

	fmt.Println("Starting Stream...")
	// фильтруем все что нам нужно
	filterParams := &twitter.StreamFilterParams{
		Track:         keywords,
		StallWarnings: twitter.Bool(true),
	}
	stream, err := flockUser.Streams.Filter(filterParams)
	if err != nil {
		log.Fatal(err)
	}
	//  получаем сообщения пока стрим не будет остановлен
	demux.HandleChan(stream.Messages)
}
```

Теперь когда нам будет попадаться твит с искомыми словами, то будет срабатывать функция `processTweet` в которой генерируется ответ с помощью алгоритма, описанного выше:

```go
func processTweet(states []State, flockUser *twitter.Client, botScreenName string, keywords []string, tweet *twitter.Tweet) {
	c.Yellow("bot @" + botScreenName + " - New tweet detected:")
	fmt.Println(tweet.Text)

	tweetWords := strings.Split(tweet.Text, " ")
	generatedText := "word no exist on the memory"
	for i := 0; i < len(tweetWords) && generatedText == "word no exist on the memory"; i++ {
		fmt.Println(strconv.Itoa(i) + " - " + tweetWords[i])
		generatedText = generateMarkovResponse(states, tweetWords[i])
	}
	c.Yellow("bot @" + botScreenName + " posting response")
	fmt.Println(tweet.ID)
	replyTweet(flockUser, "@"+tweet.User.ScreenName+" "+generatedText, tweet.ID)
	waitTime(1)
}
```

И постим твит с помощью `replyTweet`:

```go
func replyTweet(client *twitter.Client, text string, inReplyToStatusID int64) {
	tweet, httpResp, err := client.Statuses.Update(text, &twitter.StatusUpdateParams{
		InReplyToStatusID: inReplyToStatusID,
	})
	if err != nil {
		fmt.Println(err)
	}
	if httpResp.Status != "200 OK" {
		c.Red("error: " + httpResp.Status)
		c.Purple("maybe twitter has blocked the account, CTRL+C, wait 15 minutes and try again")
	}
	fmt.Print("tweet posted: ")
	c.Green(tweet.Text)
}
```

### Стадный ботнет или как избежать ограничение твиттер АПИ

Если вы когда ни будь пользовались твиттер АПИ, то наверняка в курсе что есть целый ряд ограничений и лимитов. Это означает, что если ваш бот будет делать лишком много запросов, то его будут периодически блокировать на некоторое время.

Чтобы избежать этого мы будем использовать целую сеть ботов. Когда в стриме появится твит с нужным словом, один из ботов ответит на него и "уйдет в ждущий режим" на минуту, а обработкой следующих сообщений займутся другие боты. И так по кругу.

![](/img/markovchain/flock-botnet-scheme.png)

### Собираем все вместе

В нашем примере используются всего 3 бота. Это значит нам нужно три отдельных аккаунта. Ключи для этих аккаунтов вынесем в отдельный JSON файл который будем использовать как конфиг для нашего приложения.

```js
[
    {
        "title": "bot1",
        "consumer_key": "xxxxxxxxxxxxx",
        "consumer_secret": "xxxxxxxxxxxxx",
        "access_token_key": "xxxxxxxxxxxxx",
        "access_token_secret": "xxxxxxxxxxxxx"
    },
    {
        "title": "bot2",
        "consumer_key": "xxxxxxxxxxxxx",
        "consumer_secret": "xxxxxxxxxxxxx",
        "access_token_key": "xxxxxxxxxxxxx",
        "access_token_secret": "xxxxxxxxxxxxx"
    },
    {
        "title": "bot3",
        "consumer_key": "xxxxxxxxxxxxx",
        "consumer_secret": "xxxxxxxxxxxxx",
        "access_token_key": "xxxxxxxxxxxxx",
        "access_token_secret": "xxxxxxxxxxxxx"
    }
]
```

### Демо

Мы настроили небольшую версию нашего ботнета с тремя ботами. Как уже говорилось, в качестве входных данных для генерации цепи Маркова мы использовали книгу "The Critique of Pure Reason".

Когда ботнет запускается то все боты подключаются к стримингу и ждут когда появатся твиты с необходимыми ключевыми словами.

Каждый бот получает один из твитов, обрабатывает его и отправляет ответ с использованием цепи Маркова.

В терминале это выглядит вот так:

![](/img/markovchain/terminal00.png)

И вот так выглядит все в процессе:

![](/img/markovchain/flock-botnet-demo.gif)

Ниже примеры твиттов, сгенерированные нашей цепью Маркова

![](/img/markovchain/01.png)
![](/img/markovchain/02.jpeg)
![](/img/markovchain/03.jpeg)
![](/img/markovchain/04.jpeg)

### Заключение

У нас получилось создать небольшой ботнет на основе алгоритма цепи Маркова, который может генерировать ответы на твиты.

Мы использовали только 1 класс цепей маркова и сгенерированный текст не очень поход на настоящий человеческий. Но этого вполне достаточно для начала и в будущем можно будет использовать различные классы цепей маркова и другие техники для генерации более человеческого текста.

Твиттер АПИ может использоваться для самых различных целей. Надеюсь в будущем я смогу написать на эту тему еще несколько статей, например про [анализ нод](https://devpost.com/software/projectnsa) или [пользователей и хештегов](https://arnaucode.com/hashtagsUsersNetworkPage.html).

Весь код проекта можно найти на гихабе [https://github.com/arnaucode/flock-botnet](https://github.com/arnaucode/flock-botnet). 

Страница проекта: [http://arnaucode.com/flock-botnet/](http://arnaucode.com/flock-botnet/)
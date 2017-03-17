+++
date = "2014-05-03T13:47:01+03:00"
draft = false
title = "Go и arduino"

+++

<p>На ардуине много всего можно построить и, иногда, этим всем нужно как-то управлять. Для этого можно использовать последовательный порт, отправляя команды с помощью Go программы.</p>

<p>В ардуине есть класс <a href="http://arduino.cc/en/Reference/Serial">Serial</a>, который служит для связи устройства ардуино с компьютером или другими устройствами, поддерживающими последовательный интерфейс обмена данными. Все платы arduino имеют хотя бы один последовательный порт (UART, иногда называют USART). Для обмена данными Serial используют цифровые порты ввод/вывода 0 (RX) и 1 (TX), а также USB порт. Важно помнить, что если вы используете класс Serial, то нельзя одновременно с этим использовать порты 0 и 1 для других целей.</p>

<p>В Go работать с последовательным портом, проще всего используя, пакет <a href="file:///home/artem/Projects/articles/4gophers.com/httop://github.com/schleibinger/sio">sio</a>. У этого пакета не очень большой функционал, но его вполне хватает для оправки и получения данных с устройства.</p>

<h3>Эхо</h3>

<p>Для примера, напишем небольшую программу, которая отправляет данные на устройство и получает их обратно. Начнем с Go части. Cначала нужно установить соединение:</p>

<pre>
<code>port, err := sio.Open(&quot;/dev/ttyACM0&quot;, syscall.B9600)
if err != nil {
    log.Fatal(err)
}
</code></pre>

<p><code>/dev/ttyACM0</code> - имя порта, которое можно посмотреть в arduino ide или используя <a href="https://github.com/Robot-Will/Stino">плагин для sublime</a></p>

<p><code>syscall.B9600</code> - скорость передачи данных в бит/c (бод). Для наших экспериментов не имеет принципиального значения, на какой скорости будет происходить обмен, важно, чтоб этот параметр был одинаковым и на устройстве и в нашей Go-программе.</p>

<p>После этого можно отправлять данные в порт.</p>

<pre>
<code>_, err = port.Write([]byte(&quot;test\n&quot;))
if err != nil {
    log.Fatal(err)
}
</code></pre>

<p>В устройстве первым делом устанавливаем соединение.</p>

<pre>
<code>void setup() {
    Serial.begin(9600);
}
</code></pre>

<p>Потом получаем данные и оправляем их обратно в порт.</p>

<pre>
<code>if (Serial.available() &gt; 0) {  
    // если есть доступные данные
    // считываем байт
    incomingByte = Serial.read();

    Serial.print(incomingByte);
}
</code></pre>

<p>Теперь пришло время считывать данные на стороне Go программы.</p>

<pre>
<code>reader := bufio.NewReader(port)
reply, err := reader.ReadBytes(&#39;\n&#39;)
if err != nil {
    log.Fatal(err)
}
fmt.Printf(&quot;%q&quot;, reply)
</code></pre>

<p>Все вместе это выглядит так:</p>

<pre>
<code>// echo.go

package main

import (
    &quot;bufio&quot;
    &quot;fmt&quot;
    &quot;github.com/schleibinger/sio&quot;
    &quot;log&quot;
    &quot;syscall&quot;
)

func main() {
    // устанавливаем соединение
    port, err := sio.Open(&quot;/dev/ttyACM0&quot;, syscall.B9600)
    if err != nil {
        log.Fatal(err)
    }

    // отправляем данные
    _, err = port.Write([]byte(&quot;test\n&quot;))
    if err != nil {
        log.Fatal(err)
    }

    // получаем данные
    reader := bufio.NewReader(port)
    reply, err := reader.ReadBytes(&#39;\n&#39;)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf(&quot;%q&quot;, reply)
}
</code></pre>

<pre>
<code>// echo.ino

// переменная для хранения полученного байта
char incomingByte = 0;

void setup() {
    // устанавливаем последовательное соединение
    Serial.begin(9600);
}


void loop() {
    if (Serial.available() &gt; 0) {  
        // если есть доступные данные
        // считываем байт
        incomingByte = Serial.read();

        Serial.print(incomingByte);
    }
}
</code></pre>

<h3>Последний твит</h3>

<p>Для закрепления приобретенных знаний напишем небольшую программу, которая отображает последний твитт бегущей строкой на LCD экране.</p>

<p>Конечно, сначала нужно получить твиты. И для этого заюзаем пакет <a href="http://github.com/ChimeraCoder/anaconda">anaconda</a>.</p>

<pre>
<code>anaconda.SetConsumerKey(consumerKey)
anaconda.SetConsumerSecret(consumerSecret)
api := anaconda.NewTwitterApi(key, secretKey)
</code></pre>

<p><code>consumerKey</code>, <code>consumerSecret</code>, <code>key</code>, <code>secretKey</code> - соответственно ключи взятые из настроек <a href="https://apps.twitter.com/">твиттер приложения</a></p>

<p>Получаем твиты по ключевому слову #golang:</p>

<pre>
<code>searchResult, _ = api.GetSearch(&quot;#golang&quot;, nil)
twitt = &quot; -- &quot; + searchResult[0].Text
fmt.Println(twitt)
</code></pre>

<p>В <code>searchResult</code> будет храниться список всех найденных твитов, но нам нужен только последний по времени.</p>

<p>Оправляем твит в последовательный порт:</p>

<pre>
<code>var twitt string
var searchResult []anaconda.Tweet

for {

    searchResult, _ = api.GetSearch(&quot;#golang&quot;, nil)
    twitt = &quot; -- &quot; + searchResult[0].Text
    fmt.Println(twitt)

    // отправляем данные
    _, err = port.Write([]byte(twitt))
    if err != nil {
        log.Fatal(err)
    }

    time.Sleep(120 * time.Second)
}
</code></pre>

<p>Таким образом, в порт будет оправляться новый твит каждые три минуты.</p>

<p>Ардуино-часть нашего маленького проекта начинается с объявления и инициализации:</p>

<pre>
<code>LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

int bytesCount = 0;
char data[140] = &quot;&quot;;
char string[16] = &quot;&quot;;
</code></pre>

<p><code>LiquidCrystal</code> - это наш LCD дисплей, <code>data</code> - это полученные данные, <code>bytesCount</code> - количество полученных байтов, <code>string</code> - это строка информации, которая будет отображаться на дисплее.</p>

<p>Про установку соединения и считывание данных писалось выше. Единственное изменение, теперь будем получать не по одному байту, а все сразу:</p>

<pre>
<code>if (Serial.available() &gt; 0) {  
    memset(&amp;data[0], 0, sizeof(data));
    bytesCount = Serial.readBytes(data, 1000);
}
</code></pre>

<p><code>memset(&amp;data[0], 0, sizeof(data));</code> - это очистка массива перед получением новых данных, так как следующий твит может оказаться короче предыдущего.</p>

<p>И последний шаг - отображение данных в виде бегущей строки:</p>

<pre>
<code>for(int i = 0; i &lt; bytesCount; i++){
    lcd.setCursor(0, 1);
    for(int j = 0; j &lt; 16; j++){
        if (data[i+j] == NULL) string[j] = data[(i+j) - bytesCount];
        else string[j] = data[i+j];
    }
    lcd.print(string);
    delay(300);
}
</code></pre>

<p>Полностью исходники можно <a href="https://github.com/4gophers/goardiuno">посмотреть на гитхабе</a>. Для управления зависимостями использовался <a href="https://github.com/pote/gpm">gpm</a>.</p>

<p>Жду предложений и замечаний.</p>

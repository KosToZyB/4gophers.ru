+++
date = "2016-01-31T17:53:05+03:00"
draft = false
title = "Пример использования RSA"

+++

<p>Перевод статьи "<a href="http://blog.brainattica.com/golang-cryptography-rsa/">Golang &amp; Cryptography. RSA sample</a>"</p>

<p>Как вы вероятно знаете, большинство самых используемых криптографических библиотек написано на С (или С++). Go весь пропитан духом C, он небольшой но весьма эффективный язык с удобной инфраструктурой и такими низкоуровневыми возможностями, как указатели. Кроме того, Go предоставляет широкий набор фичей для более высокоуровневого программирования. Меньше кода, быстрее компиляция, быстрое исполнение - это философия Go.</p>

<p>Все это делает Go замечательным языком для написания криптографического приложения.</p>

<p>В этом примере мы научимся использовать функции из пакета <code>crypto/rsa</code>.</p>

<p>Предположим, что Лена хочет отправить секретное сообщение Алисе. Для этого нужно выполнить ряд действий.</p>

<ol>
<li>Для начала, у Лены и Алисы должны быть пары ключей. Одни ключ приватный, второй публичный.</li>
</ol>

<pre><code>package main

import (
    "crypto"
    "crypto/rand"
    "crypto/rsa"
    "crypto/sha256"
    "fmt"
    "os"
)

func main() {
    lenaPrivateKey, err := rsa.GenerateKey(rand.Reader, 2048)

    if err != nil {
        fmt.Println(err.Error)
        os.Exit(1)
    }

    lenaPublicKey := &amp;lenaPrivateKey.PublicKey

    alisaPrivateKey, err := rsa.GenerateKey(rand.Reader, 2048)

    if err != nil {
        fmt.Println(err.Error)
        os.Exit(1)
    }

    alisaPublicKey := &amp;alisaPrivateKey.PublicKey

    fmt.Println("Private Key : ", lenaPrivateKey)
    fmt.Println("Public key ", lenaPublicKey)
    fmt.Println("Private Key : ", alisaPrivateKey)
    fmt.Println("Public key ", alisaPublicKey)

}
</code></pre>

<ol start="2">
<li>Лене необходимо зашифровать свое сообщение публичным ключем Алисы.</li>
</ol>

<pre><code>message := []byte("the code must be like a piece of music")  
label := []byte("")  
hash := sha256.New()

ciphertext, err := rsa.EncryptOAEP(hash, rand.Reader, alisaPublicKey, message, label 

if err != nil {  
    fmt.Println(err)
    os.Exit(1)
}

fmt.Printf("OAEP encrypted [%s] to \n[%x]\n", string(message), ciphertext)  
</code></pre>

<p>Стоит обратить внимание, что OAEP этот рекомендуемый стандарт, определяющий количество байтов, добавляемых к исходной записи(padding). Стандарт PKCS1v15 стоит использовать только для поддержки старых версий протоколов.</p>

<ol start="3">
<li>Далее, Лена должна подписать сообщение Алисы с помощью своего приватного ключа. Это позволит Алисе проверить отправителя сообщения с помощью Лениного публичного ключа и убедится что его отправила именно Лена.</li>
</ol>

<pre><code>var opts rsa.PSSOptions  
opts.SaltLength = rsa.PSSSaltLengthAuto // for simple example  
PSSmessage := message  
newhash := crypto.SHA256  
pssh := newhash.New()  
pssh.Write(PSSmessage)  
hashed := pssh.Sum(nil)

signature, err := rsa.SignPSS(rand.Reader, lenaPrivateKey, newhash, hashed, &amp;opts)

if err != nil {  
    fmt.Println(err)
    os.Exit(1)
}

fmt.Printf("PSS Signature : %x\n", signature)  
</code></pre>

<p>В этом случае мы выбираем PSS алгоритм. PKCS1v15 стоит использовать только для поддержки старых версий протокола.</p>

<ol start="4">
<li>Теперь у Лены есть все части сообщения для отправки его Алисе.</li>
</ol>

<pre><code>[ciphertext, signature]
</code></pre>

<ol start="5">
<li>Когда Алиса получает сообщение, первое что она должна сделать, это расшифровать его.</li>
</ol>

<pre><code>plainText, err := rsa.DecryptOAEP(hash, rand.Reader, alisaPrivateKey, ciphertext, label)

if err != nil {  
    fmt.Println(err)
    os.Exit(1)
}

fmt.Printf("OAEP decrypted [%x] to \n[%s]\n", ciphertext, plainText)  
</code></pre>

<ol start="6">
<li>И последний шаг, это проверка отправителя. Действительно ли сообщение отправила Лена:</li>
</ol>

<pre><code>err = rsa.VerifyPSS(lenaPublicKey, newhash, hashed, signature, &amp;opts)

if err != nil {  
    fmt.Println("Who are U? Verify Signature failed")
    os.Exit(1)
} else {
    fmt.Println("Verify Signature successful")
}
</code></pre>

<p>Таким образом, Алиса получила сообщение, проверила отправителя и она просто вне себя от радости!</p>

<p>В рамках этого небольшого туториала мы создали репозиторий на GitHub с примерами кода, который вы можете использовать: <a href="https://github.com/brainattica/Golang-RSA-sample">github.com/brainattica/Golang-RSA-sample</a></p>

<p>В нашем следующем туториале мы попробуем экспортировать и импортировать пары RSA ключей в PEM формат.</p>

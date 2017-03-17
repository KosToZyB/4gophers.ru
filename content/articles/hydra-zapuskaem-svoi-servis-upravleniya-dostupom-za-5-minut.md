+++
date = "2016-01-26T00:47:12+03:00"
draft = false
title = "Hydra"

+++

<p>Перевод статьи "<a href="https://blog.gopheracademy.com/advent-2015/hydra-auth/">Hydra: Run your own Identity and Access Management service in &lt;5 Minutes</a>".</p>

<p>Это вводная статья, которая познакомит вас с проектом <a href="https://github.com/ory-am/hydra">Hydra</a>. Это опенсорсный микросервис, который предоставляет альтернативное решение для реализации авторизации. Вам понадобится всего 5 минут, чтобы поднять свой OAuth2 провайдер с расширенным набором функций, включая контроль доступа и управление идентификацией.</p>

<p>Hydra появилась на свет благодаря острой необходимости в масштабируемом 12factor приложении для OAuth сервисе корпоративного уровня и, при этом, без всяких сумасшедших фич и тонны сторонних зависимостей. Кроме того, мы добавили политики, управление аккаунтами/клиентами и еще кое-какие фичи. Мы решили не реализовывать поддержку 5 различных баз данных, что позволило сократить дерево зависимостей и теперь Hydra работает только с PostgreSQL и другими SQL'ными базами данных.</p>

<p>Краткое описание основных возможностей Hydra:</p>

<ul>
<li><em>Управление аккаунтами</em>: вход пользователей, настройки, востановление пароля</li>
<li><em>Контроль доступа/Реализация политики/Хранение политик</em> реализованные через <a href="https://github.com/ory-am/ladon">Ladon</a>.</li>
<li>Богатый набор различных фич OAuth2:

<ul>
<li>Hydra реализует протокол OAuth2 согласно спецификациям <a href="http://tools.ietf.org/html/rfc6749">rfc6749</a> и <a href="http://tools.ietf.org/html/draft-ietf-oauth-v2-10">draft-ietf-oauth-v2-10</a> с использованием using <a href="https://github.com/RangelReale/osin">osin</a> и <a href="https://github.com/ory-am/osin-storage">osin-storage</a>.</li>
<li>Hydra использует автономные токены доступа согласно <a href="http://tools.ietf.org/html/rfc6749#section-1.4">rfc6794#section-1.4</a> через эммисионные JSON веб-токены, описанные в <a href="https://tools.ietf.org/html/rfc7519">rfc7519</a> с <a href="https://tools.ietf.org/html/rfc7519#section-8">RSASSA-PKCS1-v1_5 SHA-256</a> шифрованием.</li>
<li>Hydra реализует <em>OAuth2 Introspection</em> (<a href="https://tools.ietf.org/html/rfc7662">rfc7662</a>) и <em>OAuth2 Revokation</em> (<a href="https://tools.ietf.org/html/rfc7009">rfc7009</a>)</li>
<li>Hydra предоставляет возможность для входа через сторонние провайдеры, такие как ropbox, LinkedIn, Google и т.д.</li>
</ul></li>
<li>Hydra не показывает никакого HTML. И нам кажется, что это правильный подход. Hydra предоставляет только базовый функционал.</li>
<li>В комплекте есть очень простые консольные утилиты, такие как <code>hydra-host jwt</code> для генерации jwt подписи к ключу/значению или <code>hydra-host client create</code>.</li>
<li>Hydra работает и по HTTP/2 с TLS, и по HTTP(не забывайте, что это не очень безопасно).</li>
<li>Для Hydra написаны много различных тестов. Мы используем <a href="https://github.com/ory-am/dockertest">github.com/ory-am/dockertest</a> для поднятия Postgres(и остальных зависимостей) на лету и запуска интеграционных тестов. Можете попробовать такой же подход для ускорения своих интеграционных тестов.</li>
</ul>

<p>Hydra написан мной (<a href="https://github.com/arekkas">GitHub</a>/<a href="https://de.linkedin.com/in/aeneasr">LinkedIn</a>) как часть бизнес приложения, которое пока в разработке. Сейчас Hydra поддерживает <a href="https://www.linkedin.com/in/thomasaidancurran">Thomas Aidan Curran</a>.</p>

<h3>Почему Hydra это бекенд?</h3>

<p>Hydra не предоставляет HTML страницы для логина, разлогирования или авторизации. Вместо это, если необходимо выполнить некоторое действие, то Hydra редиректит на заранее определенный URL, к примеру <code>http://sign-up-app.yourservice.com/sign-up</code> или <code>http://sign-in-app.yourservice.com/sign-in</code>. К тому же, пользователь может использовать другой OAuth2 провайдер, например Dropbox или Google.</p>

<h3>Мне нужны действия, пацанчик!</h3>

<p>Отлично, мне тоже! :) Давайте попробуем как установить Hydra и получить токен для клиентского приложения, его еще называют <a href="https://aaronparecki.com/articles/2012/07/29/1/oauth2-simplified#others">OAuth2 Client Grant</a> (секция  "Application Access"). Если вы не знаете как работать в консоли или у вас возникли вопросы, можете проконсультироваться на <a href="https://github.com/ory-am/hydra/issues">GitHub Issue Tracker</a>.</p>

<p>Обратите внимание: в будущем, Hydra будет работать в Docker контейнере. Но сейчас, чтобы ее запустить, вам необходим Vagrant, VirtualBox и Git.</p>

<pre><code>$ git clone https://github.com/ory-am/hydra.git
$ cd hydra
$ vagrant up
</code></pre>

<p>В рамках этого туториала я рассчитываю, что у вас не меняется рабочая директория. Я так же рассчитываю что вы работаете на Linux/Unix системе и у вас есть доступ к командной строке. Вы можете запускать <code>vagrant ssh</code> чтобы попасть в виртуальную машину. После этого вы можете выполнять все команды(например <code>curl</code>) без проблем. Только убедитесь что внутри виртуальной машине и избегайте всяких <code>exit</code> вызовов.</p>

<p>Для начала научимся запускать Hydra. Vagrant роутит 9000(HTTPS для Hydra) и 9001(для Postgres) на ваш локальный хост. Перейдите по URL <code>https://localhost:9000/alive</code> чтобы убедится, что Hydra запушен и все работает. Возможно, вам придется добавить исключение для HTTP сертификата, так-как он самоподписанный, и после это вы должны увидеть <code>{"status":"alive"}</code>, что означает что сервис Hydra запушен и работает.</p>

<p>Мы можем настроить тестовое клиентское приложение(id = app, secret = secret) с правами суперпользователя. Для этого в Vagrant нужно запустить команды:</p>

<pre><code>$ hydra-host client create -i app -s secret -r http://localhost:3000/
authenticate/callback --as-superuser.
</code></pre>

<p>Инструмент <code>hydra-host</code> предоставляет набор возможностей для управления вашим инстансом Hydra. Вся информация по использованию есть в <a href="https://github.com/ory-am/hydra#cli-usage">документации</a>. Внутри Vagrant вы всегда можете спокойно добраться до <code>hydra-host</code>.</p>

<pre><code>$ vagrant ssh
$ hydra-host help
</code></pre>

<p>Иногда, при загрузке у Vagrant возникают проблемы с сетью. Если вы не наблюдаете никаких других ошибок, например 404 в браузере, попробуйте выполнить <code>vagrant destroy -f &amp;&amp; vagrant up</code>. После этого, в течении нескольких минут, Hydra поднимется и все должно заработать.</p>

<h3>Получаем OAuth2 Token Client</h3>

<p>Наконец то мы запустили Hydra, давайте теперь будем делать немного токен-магии. Я рассчитываю, что у вас установлен curl. Если это не так, то посмотрите как <a href="http://curl.haxx.se/download.html">его установить</a>. Наша цель заключается в "обмене"" учетных данных клиента на токен доступа.</p>

<p>Мы запускаем <code>curl</code> с параметром <code>--insecure</code>, так как у нас самоподписанный TLS сертификат. В вашем случае токен будет отличаться, но сам ответ сервера будет примерно таким же.</p>

<pre><code>curl --insecure -X POST --user app:secret "https://localhost:9000/oauth2/
token?grant_type=client_credentials"

{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiIiLCJleHAiO
    jE0NTAzNTc4MDksImlhdCI6MTQ1MDM1NDIwOSwiaXNzIjoiIiwiamlkIjoiMmYyYjk2MGQtZjE3
    OC00MzE5LWI4OGEtODc3YzM1Y2U5NTFkIiwibmJmIjoxNDUwMzU0MjA5LCJzdWIiOiJhcHAifQ.
    cLSY3G0Ngz62hJmanADZ3LUfblB5nOZOWr7bAflE9T0pZBp-
    Qv1sTkwRCQfqv870cpHdFvN9xL_AReMmNo_o9sLmXfNZDL5WJzDhhsLximxPMD-
    rO0DjnvY5663l0fvhFMlaGREsHGWDzPN-wZLczRjlFr1JXPv80qMeCm9d343hGMu26WWZ8bfdgA
    bae8ecmSO_oP7I8U0tWn22FzVJjSRuaShKxlWyQY2K_0-VoHDQDZMTEIXxYGNPA0MmCOEK1DDAi
    UeKTbguMSLMCjXTkbxd2rMwHday1oHDH8aBkyL0CGmmfVfl20hfRYqJ0x7_0sTd__-
    inASEjozSvYkVOw",
    "expires_in": 3600,
    "token_type": "Bearer"
}
</code></pre>

<p>На текущий момент мы все еще обсуждаем, какие токены нужно использовать в Hydra. Работать только с самодостаточными JWT токенами или поддерживать различные типы? Если у вас есть идеи на этот счет, обязательно <a href="https://github.com/ory-am/hydra/issues/22">отпишитесь в дискуссии</a>.</p>

<h3>Получаем OAuth2 Token Password</h3>

<p>Это было быстро, верно? Давайте теперь добавим обычную учетную запись для пользователя. Вы можете использовать что угодно в качестве имени пользователя(имя, email, случайный идентификатор), но оно должно быть уникальным. И обязательно запомните полученный ID, он пригодится нам в будущем.</p>

<pre><code>$ vagrant ssh
$ hydra-host account create foo@bar.com --password secret

Created account as "e152f029-424f-4d4d-9d69-643225113ee5".

$ exit
</code></pre>

<p>Аутентифицируем пользователя с использованием <a href="https://aaronparecki.com/articles/2012/07/29/1/oauth2-simplified#others">паролей в OAuth2 </a>.</p>

<pre><code>curl --insecure --data "grant_type=password&amp;username=foo@bar.
com&amp;password=secret" --user app:secret "https://localhost:9000/oauth2/token"

{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiIiLCJleHAiOjE
  0NTAzNzAwODAsImlhdCI6MTQ1MDM2NjQ4MCwiaXNzIjoiIiwiamlkIjoiYTdjZDFmYWQtZTg5MS00
  ZDJmLWIwZmEtMzE2Zjg3MTI5ZGIyIiwibmJmIjoxNDUwMzY2NDgwLCJzdWIiOiJiY2U5M2QzMy05Y
  jVhLTQ5MzMtOTQ3Mi1jYWRhMDE4ZGFmNjAifQ.
  dqUHiAJ0uoUYtV4hqhgVqYqA6PSy1cmNZQruyTpmRaCBh2RHzkijFj4F-
  T8xTbrFBnysTQG3LxxeXkDNq6PZBsZ4WzvUXSy1R18MayT5FWkgAi-ROQ2lHn9Isw1IgN3XWO-
  YOaQt9rO0gG4w_hRQ-DprMMKcUkNVC1zK_pdUpaB7cEurYF3sd7krPQjIhucPVhJqDjkAIZGG54kd
  28_uLqKi3eTaDrViwGLbYzmLenfTb79Hxjfd8qFd_KBQW-f1maLy0BwQNP1pVu2I_P7CBjIwEm898
  wTPye42CFUfVzyvB6ob4sAZM60YVwzxN_zaw_SO1160HbDI4oO-HwwPig",
  "expires_in": 3600,
  "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.
  eyJpZCI6IjBlNmJmNTBlLTU1N2EtNGJiYy1iZDk1LTg3ZDJkOTNkOGQ5YSJ9.JFVgu7Tf1BZJLrMb
  gKi0wyBKXZuHB63yKbv6_UP8TUkUgH8e9S5Gi9MhlPOnU0KyiEkh8p5Z0CMN2HQeIeYj-
  0p3POFxoSkY6NPZeWKsnPXzDjlJJmXWYrqgI-N-
  BD26MmoGXLjHt_DY3hxBX_EzHHuqVk9q-2pUAfwc0BHjSidF5EZ852I5e3J0WHbiw4KnogNRKNN-l
  siIIEBSjkBxyyH85Dx4JdQZsAJVBKiXXzizWIQeQABAIutvIs5ok3T4xD8WYEiSuiHdKbPKe9bjNG
  X2OqW1X-eDts4RE0eHWatNQ-IafwMvi-7A0f5PSf26pSGPQ5TyvpA5qbnYAIXrMw",
  "token_type": "Bearer"
}
</code></pre>

<h3>Процесс авторизации в OAuth2</h3>

<p>Теперь посмотрим как работает процесс авторизации в OAuth2. Для этого на нужен ID аккаунта из примера выше. Так как Hydra это только бекенд часть, вам нужно будет использовать пример приложения для логина/разлогирования. Вам стоит посмотреть код в приложениях <a href="https://github.com/ory-am/hydra/blob/master/cli/hydra-signup/main.go">hydra-signin</a> и <a href="https://github.com/ory-am/hydra/blob/master/cli/hydra-signup/main.go">hydra-signup</a>.</p>

<pre><code>$ vagrant ssh
$ ACCOUNT_ID=&lt;account_id_from_above&gt; hydra-signin &amp; exit
</code></pre>

<p>Вам нужно заменить &lt;...> своими данными, например:</p>

<pre><code>$ vagrant ssh
$ ACCOUNT_ID=e152f029-424f-4d4d-9d69-643225113ee5 hydra-signin &amp; exit
</code></pre>

<p>Теперь в браузере перейдите на по ссылке: <code>https://localhost:9000/oauth2/auth?response_type=code&amp;client_id=app&amp;redirect_uri=http://localhost:3000/authenticate/callback&amp;state=foo</code>.</p>

<p><img src="https://blog.gopheracademy.com/postimages/advent-2015/sign-in.png" alt="Sign in page" /></p>

<p>Кликните по ссылке "Press this link to sign in" для перехода на следующую страницу.</p>

<p><img src="https://blog.gopheracademy.com/postimages/advent-2015/sign-in-cb.png" alt="Sign in callback page" /></p>

<p>Указанные пути для логниа/разлогирования настраиваются через переменные окружения <code>SIGNUP_URL</code> и <code>SIGNUP_URL</code>. Ясно, что текущие настройки это просто заглушки. Как использовать переменные окружения хорошо описано в <a href="https://github.com/ory-am/hydra/blob/master/README.md#available-environment-variables">документации</a>.</p>

<p>Для следующего шага мы будем использовать <code>curl</code> и полученный ранее код.</p>

<pre><code>curl --insecure --data "grant_type=authorization_code&amp;code=&lt;code_crom_above&gt;" -
-user app:secret "https://localhost:9000/oauth2/token"
</code></pre>

<p>Замените &lt;...> вашими значениями, например:</p>

<pre><code>curl --insecure --data 
"grant_type=authorization_code&amp;code=fEat4PS3TVeyWrwKgLxICg" --user app:secret 
"https://localhost:9000/oauth2/token"

{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiIiLCJleHAiOjE
  0NTAzNTkwNDMsImlhdCI6MTQ1MDM1NTQ0MywiaXNzIjoiIiwiamlkIjoiZmMzODg4NjktZWE2MC00
  YzE4LWI1NmMtM2I4YmYzOTJmMzU5IiwibmJmIjoxNDUwMzU1NDQzLCJzdWIiOiJhcHAifQ.foEvIJ
  X3hwuCJCQvIi6x31m3g1VQ0RAp6ouiiVFIs2mVM7GsD2O3aS8WxlKaxZ5P7VhbJpxTR2zg9GDSGRe
  -Acj26r1OVjY9QSoLIeMNg2VfA6AwpASmYhP8EOdlbyjFEK8hC14JXToWn-
  cT6UXE0IZxg0ANevzDSHlPnaLDemNBkxoQ1cQPIOxPOz7xZSSDZmw9rv-MNlPi6F-
  FNZOEig5iEyl5vzDgExr5438Qkmc5OzlLYz-
  RoOroFtiyoqPXp0aYEms4zaowzB4m_DrQd0cIuAKjrtlUnbvId0rOnx-
  PBtF6yWZfSC7_hmWwtfrmho-XFWfaawjZswRWTAgaMg",
  "expires_in": 3600,
  "token_type": "Bearer"
}
</code></pre>

<p>Отлично. Вы научились работать с авторизацией. Теперь можно переходить к политикам.</p>

<h3>Политики</h3>

<p>Политики - это очень мощная штука. Мы смотрели в сторону AWS и старались адоптировать их архитектуру работы с политиками, реализовать подобную логику в Hydra. Более подробную документацию по политикам можно найти в этом <a href="https://github.com/ory-am/ladon">репозитории на GitHub</a>.</p>

<pre><code>{
    // Это должен быть уникальный ID. По этому ID
    // в базе данных будет производится поиск.
    id: "68819e5a-738b-41ec-b03c-b58a1b19d043",

    // Описание для человеков. Это не обязательное поле.
    description: "описание для людей",

    // К кому применять эту политику?
    // Обратите внимание, что тут можно использовать 
    // регулярные выражение внутри &lt; &gt;.
    subjects: ["max", "peter", "&lt;zac|ken&gt;"],

    // Эта политика разрешает или запрещает?
    effect: "allow",

    // На какой ресурс действует эта политика?
    // И тут тоже можно использовать регулярные 
    // выражения внутри &lt; &gt;.
    resources: ["urn:something:resource_a", "urn:something:resource_b", "urn:something:foo:&lt;.+&gt;"],

    // На какие права доступа влияет эта политика.
    // И тут тоже используем регулярные выражения.
    permissions: ["&lt;create|delete&gt;", "get"],

    // При каких условия политики начинают работать.
    conditions: [
        // В этом примере только при условия срабатывания "SubjectIsOwner
        {
            "op": "SubjectIsOwner"
        }
    ]
}
</code></pre>

<p>Это пример того как выглядят политики. Как видно, есть ряд различных атрибутов:</p>

<ul>
<li><code>subject</code> - это может быт учетная запись или клиентское приложение.</li>
<li><code>resource</code> - некоторая онлайн страничка или файл в облаке.</li>
<li><code>permission</code> - также может быть любой действие, например "создать", "удалить" и все в таком же роде.</li>
<li><code>condition</code> - это некоторое логическое условие(например, является ли владельцем пользователь, запрашивающий ресурс?). На данный момент доступна только условие <code>SubjectIsOwner</code>. В скором будущем будет добавлено множество условий, таких как <code>IPAddressMatches</code> или <code>UserAgentMatches</code>.</li>
<li><code>effect</code> может быть только <em>allow</em> или <em>deny</em>.</li>
</ul>

<p>Как вы помните, тестовое клиентское приложение (app) добавлено с правами супер-пользователя, а ваш тестовый пользователь без прав супер-пользователя. Давайте посмотрим, что это значит в реальной жизни.</p>

<p>Проверим, может ли тестовое приложение создать ресурс "fileA.png". Прежде всего, нам нужно получить токен для нашего клиента.</p>

<pre><code>curl --insecure --data "grant_type=client_credentials&amp;username=foo@bar.
com&amp;password=secret" --user app:secret "https://localhost:9000/oauth2/token"

{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiIiLCJleHAiOjE
  0NTAzNTkwNDMsImlhdCI6MTQ1MDM1NTQ0MywiaXNzIjoiIiwiamlkIjoiZmMzODg4NjktZWE2MC00
  YzE4LWI1NmMtM2I4YmYzOTJmMzU5IiwibmJmIjoxNDUwMzU1NDQzLCJzdWIiOiJhcHAifQ.foEvIJ
  X3hwuCJCQvIi6x31m3g1VQ0RAp6ouiiVFIs2mVM7GsD2O3aS8WxlKaxZ5P7VhbJpxTR2zg9GDSGRe
  -Acj26r1OVjY9QSoLIeMNg2VfA6AwpASmYhP8EOdlbyjFEK8hC14JXToWn-
  cT6UXE0IZxg0ANevzDSHlPnaLDemNBkxoQ1cQPIOxPOz7xZSSDZmw9rv-MNlPi6F-
  FNZOEig5iEyl5vzDgExr5438Qkmc5OzlLYz-
  RoOroFtiyoqPXp0aYEms4zaowzB4m_DrQd0cIuAKjrtlUnbvId0rOnx-
  PBtF6yWZfSC7_hmWwtfrmho-XFWfaawjZswRWTAgaMg",
  "expires_in": 3600,
  "token_type": "Bearer"
}
</code></pre>

<p>В Hydra нужно передать определенную информацию для получения доступа:</p>

<ul>
<li>Ресурс: к какому ресурсу нужен доступ?</li>
<li>Доступы: какие доступы были запрошены?</li>
<li>Токен: с каким токеном пришел запрос?</li>
<li>Контекст: например, ID пользователя.</li>
<li>Заголовок <code>Authorization: Bearer &lt;token&gt;</code> с валидным токеном, таким образом, анонимным пользователям тут не место.</li>
</ul>

<p>Для проверки, имеет ли клиент права на создание ресурса "filA.png", нужно воспользоваться <code>curl</code> запросом с указанием токена, полученного выше, в теле POST (–data "...token=...") с использованием учетных данных клиента (–user app:secret):</p>

<pre><code>curl --insecure \
--data '{"resource": "filA.png", "permission": "create", "token": 
"&lt;client_token_from_above&gt;"}' \
--user app:secret \
"https://localhost:9000/guard/allowed"
</code></pre>

<p>Замените &lt;...> на ваши значения. Должно получится примерно так:</p>

<pre><code>curl --insecure \
--data '{"resource": "filA.png", "permission": "create", "token": 
"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiIiLCJleHAiOjE0NTAzNjkzOTQsImlhd
CI6MTQ1MDM2NTc5NCwiaXNzIjoiIiwiamlkIjoiZWZkM2M2ODMtZTQ3Ny00ODQ4LThmZTYtZWU4NGI1
YzAzZTUxIiwibmJmIjoxNDUwMzY1Nzk0LCJzdWIiOiJhcHAifQ.Q4zaiLaQvbVr9Ex3Oe9Htk-zhNsY
2mtxXQgtzvnxbIbWcvF2TE_fKoVAgOGQiUiF263CNVCpKqQkMGtWcm_c1fa_2r4HYXZvOoccxHrz7fo
aSuLDfqcfKinlhLn_UvERT5jR9sYOA5Vw7ES1cq2WdrP17LXog9V40I0aZzmhqHXFdAv5vb4y5MdUKp
aJgR_PWLBE_c12nmCRrLceSgHzVAVEyxW0BkUAK4cypIH0cz-
lsSPsFZLUogQQi0oBON3FVEuXeNBxJb-Ecp3V3C5aKjrg2bs0OKeJt-
ZItrzfsQF4Gsgh2irpLfF4tMN6fNDosulNT5-HuGLJGfzJzT2RYQ"}' \
--user app:secret \
"https://localhost:9000/guard/allowed"

{"allowed": true}
</code></pre>

<p>Теперь мы можем проверить, имеет ли тестовый пользователь foo@bar.com возможность создать ресурс "fileA.png". Спойлер: нет, не имеет, так как он не суперпользователь и мы не добавляли никаких дополнительных настроек. Для начала, получите токен для клиента, потом токен для пользователя:</p>

<pre><code>curl --insecure --data "grant_type=password&amp;username=foo@bar.
com&amp;password=secret" --user app:secret "https://localhost:9000/oauth2/token"

{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiIiLCJleHAiOjE
  0NTAzNzAwODAsImlhdCI6MTQ1MDM2NjQ4MCwiaXNzIjoiIiwiamlkIjoiYTdjZDFmYWQtZTg5MS00
  ZDJmLWIwZmEtMzE2Zjg3MTI5ZGIyIiwibmJmIjoxNDUwMzY2NDgwLCJzdWIiOiJiY2U5M2QzMy05Y
  jVhLTQ5MzMtOTQ3Mi1jYWRhMDE4ZGFmNjAifQ.
  dqUHiAJ0uoUYtV4hqhgVqYqA6PSy1cmNZQruyTpmRaCBh2RHzkijFj4F-
  T8xTbrFBnysTQG3LxxeXkDNq6PZBsZ4WzvUXSy1R18MayT5FWkgAi-ROQ2lHn9Isw1IgN3XWO-
  YOaQt9rO0gG4w_hRQ-DprMMKcUkNVC1zK_pdUpaB7cEurYF3sd7krPQjIhucPVhJqDjkAIZGG54kd
  28_uLqKi3eTaDrViwGLbYzmLenfTb79Hxjfd8qFd_KBQW-f1maLy0BwQNP1pVu2I_P7CBjIwEm898
  wTPye42CFUfVzyvB6ob4sAZM60YVwzxN_zaw_SO1160HbDI4oO-HwwPig",
  "expires_in": 3600,
  "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.
  eyJpZCI6IjBlNmJmNTBlLTU1N2EtNGJiYy1iZDk1LTg3ZDJkOTNkOGQ5YSJ9.JFVgu7Tf1BZJLrMb
  gKi0wyBKXZuHB63yKbv6_UP8TUkUgH8e9S5Gi9MhlPOnU0KyiEkh8p5Z0CMN2HQeIeYj-
  0p3POFxoSkY6NPZeWKsnPXzDjlJJmXWYrqgI-N-
  BD26MmoGXLjHt_DY3hxBX_EzHHuqVk9q-2pUAfwc0BHjSidF5EZ852I5e3J0WHbiw4KnogNRKNN-l
  siIIEBSjkBxyyH85Dx4JdQZsAJVBKiXXzizWIQeQABAIutvIs5ok3T4xD8WYEiSuiHdKbPKe9bjNG
  X2OqW1X-eDts4RE0eHWatNQ-IafwMvi-7A0f5PSf26pSGPQ5TyvpA5qbnYAIXrMw",
  "token_type": "Bearer"
}
</code></pre>

<p>Команда, которую вам нужно выполнить, очень простая, но в этот раз нам нужно указать токен пользователя внутри JSON в теле запроса.</p>

<pre><code>curl --insecure \
--data '{"resource": "filA.png", "permission": "create", "token": 
"&lt;ACCOUNT_token_from_above&gt;"}' \
--user app:secret \
"https://localhost:9000/guard/allowed"
</code></pre>

<pre><code>curl --insecure \
--data '{"resource": "filA.png", "permission": "create", "token": 
"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiIiLCJleHAiOjE0NTAzNzAwODAsImlhd
CI6MTQ1MDM2NjQ4MCwiaXNzIjoiIiwiamlkIjoiYTdjZDFmYWQtZTg5MS00ZDJmLWIwZmEtMzE2Zjg3
MTI5ZGIyIiwibmJmIjoxNDUwMzY2NDgwLCJzdWIiOiJiY2U5M2QzMy05YjVhLTQ5MzMtOTQ3Mi1jYWR
hMDE4ZGFmNjAifQ.dqUHiAJ0uoUYtV4hqhgVqYqA6PSy1cmNZQruyTpmRaCBh2RHzkijFj4F-
T8xTbrFBnysTQG3LxxeXkDNq6PZBsZ4WzvUXSy1R18MayT5FWkgAi-ROQ2lHn9Isw1IgN3XWO-
YOaQt9rO0gG4w_hRQ-DprMMKcUkNVC1zK_pdUpaB7cEurYF3sd7krPQjIhucPVhJqDjkAIZGG54kd28
_uLqKi3eTaDrViwGLbYzmLenfTb79Hxjfd8qFd_KBQW-f1maLy0BwQNP1pVu2I_P7CBjIwEm898wTPy
e42CFUfVzyvB6ob4sAZM60YVwzxN_zaw_SO1160HbDI4oO-HwwPig"}' \
--user app:secret \
"https://localhost:9000/guard/allowed"
{"allowed": false}
</code></pre>

<p>Ух! Тут было много копипасты команд в консоль, но у вас получилось. Вы попробовали основные возможности Hydra. Конечно, мы прошлись поверхностно только по базовым возможностям Hydra и есть еще много чего для более подробного исследования. Команда Ори надеется, что вам понравился этот туториал и вы будете рекомендовать Hydra друзьям. Пока что Hydra не так стабильна, как хотелось бы, но мы работаем над этим. Если вы нашли баг, обязательно сообщите <a href="https://github.com/ory-am/hydra">нам об этом</a>.</p>

<p>За лого я благодарен <a href="https://www.flickr.com/photos/pathfinderlinden/7161293044/">pathfinderlinden</a>.</p>

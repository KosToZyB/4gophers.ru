+++
date = "2016-02-15T00:27:12+03:00"
draft = false
title = "Готовим deb из наших бинарников"

+++

<p>Go прекрасный язык. Одна из его супер-сил - это возможность собирать все в один бинарник. А это очень удобно, вы можете везде таскать этот файл и использовать на любой машине. Но хочется иметь возможность устанавливать нашу программу простым способом.</p>

<p>С помощью deb пакетов довольно просто можно организовать деплой на ваши сервера. При этом у вас будет версионирование и все такое. Я чаще всего использую ubuntu, поэтому речь будет именно о deb пакетах, которые можно установить/удалить с помощью утилит <code>apt</code>.</p>

<p>Что же нужно сделать, для создания своего репозитория с пакетами? Можно воспользоваться тем же <a href="http://launchpad.net">launchpad.net</a>, например. Но, последнее время, он не очень развивается и выглядит ненадежно. К тому же, его удобно использовать для своих не коммерческих разработок, но использовать его для дистрибуции корпоративного ПО будет проблематично.</p>

<p>Подойдем к проблеме с другой стороны. Во первых, нам нужно собирать deb пакеты, а это очень просто сделать самим с помощью утилиты <code>dpkg-deb</code>. Во-вторых, нам нужно где-то эти пакеты размещать, и для этого мы воспользуемся супер простым <a href="https://github.com/esell/deb-simple">сервером apt репозиториев deb-simple</a></p>

<h3>Сборка пакетов</h3>

<p>Для всех своих проектов я использую <a href="http://getgb.io/">gb</a>. Структура проекта выглядит примерно так:</p>

<pre><code>project/
 |- bin/
 |      |- project
 |- src/
 |      |- github.com/
 |          |- 4gophers/
 |              |- project/
 |                 |- main.go
 |- vendor/
</code></pre>

<p>Когда я запускаю <code>gb build</code>, то все бинарники собираются в папке <code>bin</code>. Таким образом, все что нам нужно - это просто добавить спецификацию нашего будущего deb пакета прям в папку с проектом:</p>

<pre><code>mkdir project/DEBIAN
touch project/DEBIAN/control
</code></pre>

<p>В результате будет такая структура:</p>

<pre><code>project/
 |- DEBIAN/
 |      |- control
 |- bin/
 |      |- project 
 |- src/
 |- vendor/
</code></pre>

<p>В файле <code>control</code> нужно указать информацию о нашем пакете. Не забывайте про пустую последнюю строку:</p>

<pre><code>Package: project
Priority: optional
Section: devel
Installed-Size: 100
Maintainer: Ivan Ivanov &lt;test@test.ru&gt;
Architecture: i386
Version: 1.0
Depends: libc6 (&gt;= 2.1)
Description: Short description here
 Long description here

</code></pre>

<ul>
<li>Package — имя вашего пакета</li>
<li>Priority — приоритет пакета (optional, extra, standard, important, required) для обычных программ лучше ставить optional</li>
<li>Section — раздел к которому относится данный пакет (admin, base, comm, contrib, devel, doc, editors, electronics, embedded, games, gnome, graphics, hamradio, interpreters, kde, libs, libdevel, mail, math, misc, net, news, non-free, oldlibs, otherosfs, perl, python, science, shells, sound, tex, text, utils, web, x11)</li>
<li>Installed-Size — размер файлов пакета в килобайтах</li>
<li>Maintainer — имя и email создателя пакета</li>
<li>Architecture — архитектура процессора, для которой предназначен пакет (i386, amd64, all, source, all)</li>
<li>Version — версия пакета</li>
<li>Depends — в данном поле необходимо указать имена пакетов, от которых зависит ваш пакет (например, библиотеки)</li>
<li>Description — в первой строке пишем короткое описание пакета, в остальных более подробно</li>
</ul>

<p>Все что находится в папке <code>project</code> попадет в пакет. И папка <code>bin</code> тоже. В этой папке лежит наш бинарный файл, который нужно установить. Чтобы ваши файлы оказались в нужной директории на компьютере пользователя, нужно создать соответствующую структуру каталогов внутри вашей папки с проектом.</p>

<p>Стоит отметить, что такой подход к созданию deb пакетов не самый правильный. Конечно, в нашем случае мы осознанно идем на этот шаг, но вам нужно понимать, что в deb пакет попадет все содержимое папки <code>project</code>, в том числе папки <code>src</code>, <code>vendor</code> и так далее. Конечно, можно скопировать файлы в другую папку, и даже написать скрипт для этого, но все уже придумано до нас. Более правильный способ - это использовать утилиты <code>dh_make</code> и <code>dpkg-buildpackage</code>.</p>

<p>Теперь можно собирать пакет. Для этого на уровень выше выполним команду:</p>

<pre><code>dpkg-deb -z8 -Zgzip --build project
</code></pre>

<p>На уровень выше будет создан файл <code>project.deb</code>, который можно устанавливать с помощью команды:</p>

<pre><code>sudo dpkg -i project.deb
</code></pre>

<p>Больше информации о правильной сборке deb пакетов можно узнать в <a href="https://www.debian.org/doc/manuals/maint-guide/">официальной документации</a>.</p>

<h3>Свой репозиторий пакетов</h3>

<p>Теперь переходим к самому интересному. Как же нам распространять наши пакеты? Запустим свой сервер репозиториев, конечно же. А для этого воспользуемся <a href="https://github.com/esell/deb-simple">сервером apt репозиториев deb-simple</a>.</p>

<p>Это действительно простой сервер, который устанавливается всего одной командой:</p>

<pre><code>go get github.com/esell/deb-simple
</code></pre>

<p>Если go не установлен на той машине, где вы собираетесь запустить сервер с репозиториями, то вы можете собрать бинарник локально и просто скопировать его. Кроме этого, можно использовать docker.</p>

<p>Затем нужно запустить сервер. Это можно сделать с помощью docker, но мне больше нравится использовать <a href="http://supervisord.org/">supervisord</a>. Вот пример моей конфигурации сервиса:</p>

<pre><code>[program:deb-simple]
command=/home/user/go1.5/bin/deb-simple
directory=/home/user/deb-simple/
autorestart=true
stdout_logfile=none
</code></pre>

<p>Тут важно указать путь к бинарнику(<code>command</code>) и рабочую папку(<code>directory</code>), в которой мы разместим наш конфиг.</p>

<p>Сервер deb-simple поддерживает https, но пока нам это не нужно. Для репозиториев нужно создать папку <code>repo</code>. Наш конфиг <code>conf.json</code> будет выглядит так:</p>

<pre><code>{
    "listenPort" : "9090",
    "rootRepoPath" : "/home/user/deb-simple/repo",
    "supportedArch" : ["all","i386","amd64"],
    "enableSSL" : false,
    "SSLcert" : "server.crt",
    "SSLkey" : "server.key"
}
</code></pre>

<p>Чтобы загрузить пакет в свой репозиторий, нужно воспользоваться HTTP API самого сервиса:</p>

<pre><code>curl -XPOST 'http://localhost:9090/upload?arch=amd64' -F "file=@project.deb"
</code></pre>

<p>Точно так же для удаления:</p>

<pre><code>curl -XDELETE 'http://localhost:9090/delete' -d '{"filename":"project.deb","arch":"amd64"}'
</code></pre>

<p>Нам осталось добавить наш сервер репозиториев к списку в <code>/etc/apt/source.list.d/</code>. Можно создать отдельный файл с содержимым:</p>

<pre><code>deb http://my-hostname:9090/ stable main
</code></pre>

<p>Теперь запускайте <code>sudo apt-get update</code> и устанавливайте свои программы сколько душе угодно.</p>

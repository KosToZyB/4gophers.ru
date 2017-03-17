+++
date = "2014-12-22T18:34:06+03:00"
draft = false
title = "Пишем файловую сиcтему на Go и FUSE"

+++

<p>Перевод статьи "<a href="http://blog.gopheracademy.com/advent-2014/fuse-zipfs/">Writing file systems in Go with FUSE</a>" от Tommi Virtanen.</p>

<h3>Мотивация</h3>

<p>Некоторое время назад мне понадобилось улучшить используемое хранилище. Основная причина - это необходимость удобной синхронизации. Кроме того, было и другие причины. Мне была необходима  файловая система, которая сочетает в себе лучшее из трех стихий: локальных файлов, сетевой файловой системы и синхронизации файлов. Все это реализовалось в виде проекта <a href="http://bazil.org/">Bazil</a>, названный в честь базиллион байтов.</p>

<p>Для создания <a href="http://bazil.org/">Bazil</a> мне был необходим инструмент для простого написания файловых систем на Go. Таким образом, на свет появился пакет <a href="http://bazil.org/fuse">bazil.org/fuse</a>.</p>

<p>Сегодня, в качестве примера, мы напишем Go приложение, которое позволит использовать zip архивы как файловую систему:</p>

<pre><code class="sh">$ unzip -v archive.zip
Archive:  archive.zip
 Length   Method    Size  Cmpr    Date    Time   CRC-32   Name
--------  ------  ------- ---- ---------- ----- --------  ----
       0  Stored        0   0% 2014-12-11 04:03 00000000  buried/
       0  Stored        0   0% 2014-12-11 04:03 00000000  buried/deep/
       5  Stored        5   0% 2014-12-11 04:03 2efcceec  buried/deep/loot
      13  Stored       13   0% 2014-12-11 04:03 f4247453  greeting
--------          -------  ---                            -------
      18               18   0%                            4 files
$ zipfs archive.zip mnt &amp;
$ tree mnt
mnt
├── buried
│   └── deep
│       └── loot
└── greeting

2 directories, 2 files
$ cat mnt/greeting
hello, world
</code></pre>

<h3>FUSE</h3>

<p><a href="https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/Documentation/filesystems/fuse.txt">FUSE</a> (Filesystem In Userpace) это файловая система ядра Linux, которая работает поверх запросов к файловым дескрипторам в пространстве пользователя. Исторически, все это обрабатывается с помощью <a href="http://fuse.sourceforge.net/">сишной библиотеки с таким же именем</a>. Но FUSE - это всего лишь протокол. Сейчас его поддержка реализована в OS X, FreeBSD и OpenBSD.</p>

<p><a href="http://bazil.org/fuse">bazil.org/fuse</a> это реализация протокола на чистом Go.</p>

<h3>Структура файловой системы Unix</h3>

<p>Файловая система в никсах состоит из inodes (“index nodes”). Это файлы, каталоги, и т.д. Каталоги содержат записи (dirent от directory entries) которые указывают на вложенные ноды. Запись в каталоге идентифицируется по имени и содержит очень мало метаинформации. Через ноды можно добраться до дополнительной метаинформации(включая контроль доступа) и до содержимого самого файла.</p>

<p>Открытые файлы идентифицируются с помощью файловых дескрипторов, который являются безопасными ссылками на объекты ядра (handles или ручки).</p>

<h3>Go API</h3>

<p>Наша библиотека FUSE разделена на две части. Низкоуровневая часть <code>bazil.org/fuse</code>. И более высокоуровневая часть <code>bazil.org/fuse/fs</code> в которой есть методы для отслеживания времени жизни объектов,  их состояний.</p>

<p>Каждая файловая система имеет корневую запись. В интерфейсе <code>fs.FS</code> есть метод <code>Root</code>, который возвращает <code>fs.Node</code>.</p>

<p>Для доступа к файлу (просмотр метаданных, открытие, и т.д.) ядро будет запрашивать его по имени, отправляя <code>fuse.LookupRequest</code> к FUSE серверу. Этот запрос обрабатывается методом <code>Lookup</code> родительской <code>fs.Node</code>. Метод возвращает <code>fs.Node</code>, результат кешируется в ядре и пересчитываются ссылки. Удаление кеша инициирует <code>ForgetRequest</code> и, когда количество ссылок становится равным нулю, вызывается <code>Forget</code>.</p>

<p>Файлы переименовываются с помощью <code>Rename</code>, удаляются с помощью <code>Remove</code> и так далее.</p>

<p>Обработчики навешиваются, для примера, в момент открытия файла. Открытия любого существующего файла инициирует <code>OpenRequest</code> который обрабатывается с помощью <code>Open</code>. Все методы, которые создают новые обработчики, возвращают <code>Handle</code>. Обработчики закрываются комбинацией <code>Flush</code> и <code>Release</code>.</p>

<p>Если <code>Open</code> не определен, то как <code>Handle</code> используется <code>fs.Node</code>. Это, как правило, работает нормально, но только для неизменяемых файлов.</p>

<p>Для чтение из <code>Handle</code> используется <code>Read</code>, для записи - <code>Write</code>. И кроме всего этого есть дополнительные данные, аналогичные <code>io.ReaderAt</code> и <code>io.WriterAt</code>. Обратите внимание, что информация о размере файла изменяется только через <code>Setattr</code> и не зависит от вызовов <code>Write</code>. Нужно следить, чтобы  <code>Attr</code> возвращал корректный <code>Size</code>.</p>

<p>Листинг в каталоге реализуется благодаря чтению дескриптора каталога, который, на самом деле, тоже файл. После прочтения каталога, вместо содержимого файла, возвращается список(directory entries). Метод <code>ReadDir</code> реализует чуть более высокоуровневое API и возвращает слайс.</p>

<p>Написание файловой системы требует неплохого понимания структур данных ядра и принципов изменения их состояний, на абстрактном уровне. Но Go позволяется обойтись без лишних усложнений. Так что давайте погрузимся в код.</p>

<h3>Zipfs</h3>

<p>Для примера напишем простую файловую систему, которая позволит просматривать содержимое <a href="http://golang.org/pkg/archive/zip/">zip архива</a>.</p>

<p>Исходники готовой программы доступны тут: <a href="https://github.com/bazillion/zipfs">https://github.com/bazillion/zipfs</a></p>

<h4>Каркас</h4>

<p>Начнем с каркаса программы, которая парсит аргументы командной строки:</p>

<pre><code class="go">package main

import (
    "archive/zip"
    "flag"
    "fmt"
    "io"
    "log"
    "os"
    "path/filepath"
    "strings"

    "bazil.org/fuse"
    "bazil.org/fuse/fs"
)

// Мы должны учитывать, что zip файл также содержит записи для каталогов.

var progName = filepath.Base(os.Args[0])

func usage() {
    fmt.Fprintf(os.Stderr, "Usage of %s:\n", progName)
    fmt.Fprintf(os.Stderr, "  %s ZIP MOUNTPOINT\n", progName)
    flag.PrintDefaults()
}

func main() {
    log.SetFlags(0)
    log.SetPrefix(progName + ": ")

    flag.Usage = usage
    flag.Parse()

    if flag.NArg() != 2 {
        usage()
        os.Exit(2)
    }
    path := flag.Arg(0)
    mountpoint := flag.Arg(1)
    if err := mount(path, mountpoint); err != nil {
        log.Fatal(err)
    }
}
</code></pre>

<p>Монтирование это несколько громоздкое действо, еще и потому что монтирование в Linux отличается от монтирования в OSXFUSE. И это может быть причиной некоторых ошибок.</p>

<pre><code class="go">func mount(path, mountpoint string) error {
    archive, err := zip.OpenReader(path)
    if err != nil {
        return err
    }
    defer archive.Close()

    c, err := fuse.Mount(mountpoint)
    if err != nil {
        return err
    }
    defer c.Close()

    filesys := &amp;FS{
        archive: &amp;archive.Reader,
    }
    if err := fs.Serve(c, filesys); err != nil {
        return err
    }

    // проверяем ошибки при монтировании
    &lt;-c.Ready
    if err := c.MountError; err != nil {
        return err
    }

    return nil
}
</code></pre>

<h4>Файловая система</h4>

<p>Фактически, эта файловая система работает через указатель на zip архив:</p>

<pre><code class="go">type FS struct {
    archive *zip.Reader
}
</code></pre>

<p>И для начала нам нужно указать <code>Root</code> метод:</p>

<pre><code class="go">var _ fs.FS = (*FS)(nil)

func (f *FS) Root() (fs.Node, fuse.Error) {
    n := &amp;Dir{
        archive: f.archive,
    }
    return n, nil
}
</code></pre>

<h4>Каталоги</h4>

<p>Zip файлы содержат списки файлов. Кроме того, типичные zip-архивы содержать записи каталогов, которые заканчиваются на слеш. Мы вернемся к этому чуть позже.</p>

<p>Определим наш тип <code>Dir</code> и реализуем обязательный метод  <code>Attr</code>. Будем использовать <code>*zip.File</code> для работы с метаданными каталога.</p>

<pre><code class="go">type Dir struct {
    archive *zip.Reader
    // для корневого каталога, который не имеет ни
    // каких зипов, это будет nil
    file *zip.File
}

var _ fs.Node = (*Dir)(nil)

func zipAttr(f *zip.File) fuse.Attr {
    return fuse.Attr{
        Size:   f.UncompressedSize64,
        Mode:   f.Mode(),
        Mtime:  f.ModTime(),
        Ctime:  f.ModTime(),
        Crtime: f.ModTime(),
    }
}

func (d *Dir) Attr() fuse.Attr {
    if d.file == nil {
        // корневой каталог
        return fuse.Attr{Mode: os.ModeDir | 0755}
    }
    return zipAttr(d.file)
}
</code></pre>

<h4>Просмотр записей в каталоге</h4>

<p>Чтобы наша файловая система могла делать что то полезное, нам нужно реализовать поиск файла по имени. Мы просто будем перебирать все записи в zip архиве, сравнивая путь:</p>

<pre><code class="go">var _ = fs.NodeRequestLookuper(&amp;Dir{})

func (d *Dir) Lookup(req *fuse.LookupRequest, 
                resp *fuse.LookupResponse, 
                intr fs.Intr) (fs.Node, fuse.Error) {

    path := req.Name
    if d.file != nil {
        path = d.file.Name + path
    }
    for _, f := range d.archive.File {
        switch {
        case f.Name == path:
            child := &amp;File{
                file: f,
            }
            return child, nil
        case f.Name[:len(f.Name)-1] == path &amp;&amp; f.Name[len(f.Name)-1] == '/':
            child := &amp;Dir{
                archive: d.archive,
                file:    f,
            }
            return child, nil
        }
    }
    return nil, fuse.ENOENT
}
</code></pre>

<h4>Файлы</h4>

<p>Наш метод <code>Lookup</code> возвращает типы <code>File</code>, если запись не заканчивается на слеш. Определим тип <code>File</code>, который использует туже вспомогательную функцию <code>zipAttr</code>:</p>

<pre><code class="go">type File struct {
    file *zip.File
}

var _ fs.Node = (*File)(nil)

func (f *File) Attr() fuse.Attr {
    return zipAttr(f.file)
}
</code></pre>

<p>Но файлы не нужны, если их нельзя открыть:</p>

<pre><code class="go">var _ = fs.NodeOpener(&amp;File{})

func (f *File) Open(req *fuse.OpenRequest, 
                resp *fuse.OpenResponse,
                intr fs.Intr) (fs.Handle, fuse.Error) {

    r, err := f.file.Open()
    if err != nil {
        return nil, err
    }
    // индивидуальный записи в архиве
    resp.Flags |= fuse.OpenNonSeekable
    return &amp;FileHandle{r: r}, nil
}
</code></pre>

<h4>Обработчики</h4>

<pre><code class="go">type FileHandle struct {
    r io.ReadCloser
}

var _ fs.Handle = (*FileHandle)(nil)
</code></pre>

<p>Мы реализуем "открытый файл" с помощью обработчиков. В нашем случае это просто обертка на <code>archive/zip</code>, но в других случаях это может быть <code>*os.File</code>, сетевое соединение и многое другое. И нужно не забывать закрывать их:</p>

<pre><code class="go">var _ fs.HandleReleaser = (*FileHandle)(nil)

func (fh *FileHandle) Release(req *fuse.ReleaseRequest, intr fs.Intr) fuse.Error {
    return fh.r.Close()
}
</code></pre>

<p>Давайте напишем реальный обработчик операции <code>Read</code>:</p>

<pre><code class="go">var _ = fs.HandleReader(&amp;FileHandle{})

func (fh *FileHandle) Read(req *fuse.ReadRequest, 
                resp *fuse.ReadResponse, intr fs.Intr) fuse.Error {

    // В такой реализации мы не учитываем Offset для определения, где
    // закончилось прошлое чтение. Для этого нам пришлось бы трекать эту
    // операции. Ядро сделает все за нас основываясь
    // на флаге fuse.OpenNonSeekable.
    buf := make([]byte, req.Size)
    n, err := fh.r.Read(buf)
    resp.Data = buf[:n]
    return err
}
</code></pre>

<h4>Чтение директорий</h4>

<p>В таком виде мы имеем доступ к нашим файлам через <code>cat</code> и аналогичный способы. Но для этого нам нужно знать их имена. Давайте добавим поддержку <code>ReadDir</code>:</p>

<pre><code class="go">var _ = fs.HandleReadDirer(&amp;Dir{})

func (d *Dir) ReadDir(intr fs.Intr) ([]fuse.Dirent, fuse.Error) {
    prefix := ""
    if d.file != nil {
        prefix = d.file.Name
    }

    var res []fuse.Dirent
    for _, f := range d.archive.File {
        if !strings.HasPrefix(f.Name, prefix) {
            continue
        }
        name := f.Name[len(prefix):]
        if name == "" {
            // сама директория, не вложения
            continue
        }
        if strings.ContainsRune(name[:len(name)-1], '/') {
            // есть слеш в середине -&gt; находится во вложенном каталоге
            continue
        }
        var de fuse.Dirent
        if name[len(name)-1] == '/' {
            // каталог
            name = name[:len(name)-1]
            de.Type = fuse.DT_Dir
        }
        de.Name = name
        res = append(res, de)
    }
    return res, nil
}
</code></pre>

<h3>Тестируем zipfs</h3>

<p>Подготавливаем zip файл:</p>

<pre><code class="sh">$ mkdir -p data/buried/deep
$ echo hello, world &gt;data/greeting
$ echo gold &gt;data/buried/deep/loot
$ ( cd data &amp;&amp; zip -r -q ../archive.zip . )
</code></pre>

<p>Монтируем архив:</p>

<pre><code class="sh">$ mkdir mnt
$ zipfs archive.zip mnt &amp;
</code></pre>

<p>Просматриваем записи в каталоге:</p>

<pre><code class="sh">$ ls -ld mnt/greeting
-rw-r--r-- 1 root root 13 Dec 11  2014 mnt/greeting
$ ls -ld mnt/buried
drwxr-xr-x 1 root root 0 Dec 11  2014 mnt/buried
</code></pre>

<p>Читаем содержимое файла:</p>

<pre><code class="sh">$ cat mnt/greeting
hello, world
$ cat mnt/buried/deep/loot
gold
</code></pre>

<p>Чтение каталога ("total 0" это не правильно, но пока не значительно):</p>

<pre><code class="sh">$ ls -l mnt
total 0
drwxr-xr-x 1 root root  0 Dec 11  2014 buried
-rw-r--r-- 1 root root 13 Dec 11  2014 greeting
$ ls -l mnt/buried
total 0
drwxr-xr-x 1 root root 0 Dec 11  2014 deep
</code></pre>

<p>Размонтирование (для OS X используйте umount mnt):</p>

<pre><code class="sh">$ fusermount -u mnt
</code></pre>

<p>B на этом все. Если хотите продолжения и больше примеров, посмотрите <a href="https://github.com/bazillion/bolt-mount">https://github.com/bazillion/bolt-mount</a> (<a href="http://eagain.net/talks/bolt-mount/">скринкаст в дополнение к коду</a>) и другие <a href="http://godoc.org/bazil.org/fuse?importers">проекты использующие fuse</a>.</p>

<h3>Почитать</h3>

<ul>
<li><a href="http://bazil.org/">Bazil</a> распределенная файловая система. Позволяет шарить файлы на всех ваших компьютерах, включая облачные сервисы.</li>
<li><a href="https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/Documentation/filesystems/fuse.txt">FUSE</a> модуль для ядер UNIX-подобных операционных систем. Свободное программное обеспечение с открытым исходным кодом.</li>
<li>Сишная библиотека <a href="http://fuse.sourceforge.net/">FUSE</a>, реализующая поддержку протокола.</li>
<li><a href="http://bazil.org/fuse">bazil.org/fuse</a> Go библиотека для написания файловых систем. Смотрите godoc для <code>fuse</code> и <code>fuse/fs</code></li>
<li><a href="https://osxfuse.github.io/">OSXFUSE</a> это реализация ядра FUSE для OS X.</li>
<li><code>bolt-mount</code> это более обширный пример файловой системы, включая операции записи. Также, можете посмотреть <a href="http://eagain.net/talks/bolt-mount/">скринкаст</a>.</li>
<li><a href="http://bazil.org/talks/2013-06-10-la-gophers/">Написание файловых систем на Go</a> чуть более подробное описание FUSE.</li>
<li>Вопросы по FUSE можете задавать в <a href="https://groups.google.com/forum/#!forum/bazil-dev">bazil-dev Google Group</a> или на IRC канале <a href="irc:irc.freenode.net/go-nuts">#go-nuts на irc.freenode.net</a>.</li>
</ul>

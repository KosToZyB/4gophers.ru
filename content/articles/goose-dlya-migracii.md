+++
date = "2014-03-08T21:46:09+03:00"
draft = false
title = "Goose для миграций"

+++

<p>Программист, приученный к миграциям, это как кот, приученный к лотку - где попало гадить не будет. В проекте, который разрабатывается на больше чем на одной машине, миграции нужны как воздух.</p>

<p>Есть много разных инструментов для миграций, написанные на других языках. На Go тоже есть. Правда поменьше и попроще. Например <a href="https://bitbucket.org/liamstask/goose">goose</a>.</p>

<p>Устанавливается утилита через go get:</p>

<pre>
<code>$ go get bitbucket.org/liamstask/goose/cmd/goose
</code></pre>

<p>Почитать о том, как использовать goose можно на страничке проекта на битбакете.</p>

<p>Тулза почти идеальная. Но мне нужно было использовать ее совместно с SQLite, а доступные диалекты только mysql или potsgre, что меня никак не устраивало.</p>

<h3>Поддержка SQLite</h3>

<p>Полезем внутря. Первым делом в файле goose/lib/goose/dialect.go добавляем новый диалект:</p>

<pre>
<code>func dialectByName(d string) SqlDialect {
    switch d {
    case &quot;postgres&quot;:
        return &amp;PostgresDialect{}
    case &quot;mysql&quot;:
        return &amp;MySqlDialect{}
    case &quot;sqlite3&quot;:
        return &amp;SqliteDialect{}
    }

    return nil
}
</code></pre>

<p>И добавляем новый тип <em>SqliteDialect</em>, который соответствует интерфейсу <em>SqlDialect</em></p>

<pre>
<code>////////////////////////////
// sqlite
////////////////////////////

type SqliteDialect struct{}

func (m *SqliteDialect) createVersionTableSql() string {
    return `create table goose_db_version (
                id integer not null primary key autoincrement,
                version_id integer not null,
                is_applied integer not null,
                tstamp text null
            );`
}

func (m *SqliteDialect) insertVersionSql() string {
    return &quot;INSERT INTO goose_db_version (version_id, is_applied) VALUES (?, ?);&quot;
}

func (m *SqliteDialect) dbVersionQuery(db *sql.DB) (*sql.Rows, error) {
    rows, err := db.Query(&quot;SELECT version_id, is_applied from goose_db_version ORDER BY id DESC&quot;)

    // XXX: check for mysql specific error indicating the table doesn&#39;t exist.
    // for now, assume any error is because the table doesn&#39;t exist,
    // in which case we&#39;ll try to create it.
    if err != nil {
        return nil, ErrTableDoesNotExist
    }

    return rows, err
}
</code></pre>

<p>Теперь, в файле goose/lib/goose/dbconf.go указываем настройки для sqlite: какой драйвер и какой тип диалекта юзать:</p>

<pre>
<code>// Create a new DBDriver and populate driver specific
// fields for drivers that we know about.
// Further customization may be done in NewDBConf
func newDBDriver(name, open string) DBDriver {

    d := DBDriver{
        Name:    name,
        OpenStr: open,
    }

    switch name {
    case &quot;postgres&quot;:
        d.Import = &quot;github.com/lib/pq&quot;
        d.Dialect = &amp;PostgresDialect{}

    case &quot;mymysql&quot;:
        d.Import = &quot;github.com/ziutek/mymysql/godrv&quot;
        d.Dialect = &amp;MySqlDialect{}

    case &quot;sqlite3&quot;:
        d.Import = &quot;github.com/mattn/go-sqlite3&quot;
        d.Dialect = &amp;SqliteDialect{}
    }

    return d
}
</code></pre>

<p>В файле goose/lib/goose/migrate.go нужно предварительно указать в импорте дравер для sqlite.</p>

<pre>
<code>import (
    //...
    _ &quot;github.com/mattn/go-sqlite3&quot;
    //...
)
</code></pre>

<p>И на этом все. Теперь можно использовать goose для рботы с sqlite.</p>

<p>Только одно маленькое замечание. В отличие от драйверов postgres и mysql, драйвер sqlite не будет выполнять запросы при использовании txn.Query(). Нужно использовать txn.Exec()</p>

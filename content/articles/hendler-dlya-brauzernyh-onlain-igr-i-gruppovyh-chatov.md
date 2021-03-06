+++
date = "2014-12-13T03:32:03+03:00"
draft = false
title = "Хендлер для браузерных онлайн игр и групповых чатов"

+++

<p>NB! Данный хендлер предназначен для работы только с вебсокетами golang.org/x/net/websocket. Хотя, если ваше приложение не использует вебсокеты или использует вебсокеты из другого пакета, но есть необходимость объединять гороутины обработчиков в группы с общими данными, то, наверное, вам может быть полезен данный велосипед в качестве примера, если вы еще не написали подобный и если не знаете способа по-лучше конечно.</p>

<h3>Введение</h3>

<p>Когда функциональная нагрузка распределена по модулям программы не правильно, то переписывание или дописывание программы превращается в настоящий ад. Так вот, при написании сервера для браузерной онлайн игры или группового чата (или чего-нибудь подобного) не следует смешивать:</p>

<ul>
<li>Логику авторизации</li>
<li>Логику управления группами (создания и удаления групп, добавление и удаление участников)</li>
<li>Логику инициализации общих данных группы (например, игрового пространства)</li>
<li>Логику инициализации данных участника (инициализация танчика, змейки)</li>
</ul>

<p>Все эти операции необходимо проделать с одним и тем же соединением, и еще надо начать стримить данные, вот, и, наверное, есть некоторый соблазн, особенно, если нет сложного ветвления/структуры групп, смешать это все в кучу.</p>

<h3>Интерфейсы</h3>

<p>Написанный мной пакет не делает почти ничего. Его задача - помочь избежать беспорядка в коде. Главное - это определенные в нем интерфейсы, декларирующие функциональные части сервера:</p>

<p>1) Интерфейс для общих данных группы ("окружающая среда участника группы"). Общими данными может быть что угодно.</p>

<pre><code class="go">type Environment interface{}
</code></pre>

<p>2) Интерфейс менеджера групп. Этот объект должен хранить группы и управлять ими. Именно он должен решать, когда группа должна быть добавлена, а когда удалена, и именно он распрелеляет юзеров по группам.</p>

<pre><code class="go">type PoolManager interface {
    // AddConn должен найти подходящую группу для соединения и 
    // добавить туда информацию о нем, и вернуть общие данные
    // окружения группы. Также AddConn добавляет группы если надо
    AddConn(ws *websocket.Conn) (Environment, error)
    // DelConn удаляет информацию о соединении из группы. Может, в
    // случае если группа стала пустой удалить группу
    DelConn(ws *websocket.Conn) error
}
</code></pre>

<p>3) Интерфейс обработчика. Этот объект занимается работой с соединениями. Он получает соединение и общие данные группы и делает все что нужно, как обычный хендлер.</p>

<pre><code class="go">type ConnManager interface {
    // Handle - это обработчик. data - это общие данные группы
    Handle(ws *websocket.Conn, data Environment) error
    // HandleError для информирования пользователей об ошибках
    // Можно использовать для того, чтобы записать инфу об ошибке в
    // нужный лог, и написать что-то пользователю. Но не для закрытия
    // соединения (следуем принципу: не я открыл, не мне закрывать)
    HandleError(ws *websocket.Conn, err error)
}
</code></pre>

<p>4) Интерфейс для проверяльщика. Проверяльщик вызывается первым и может пообщавшись с клиентом по вебсокету либо авторизовать пользователя, вернув nil, либо не авторизовать, вернув ошибку. Эта ошибка будет передана в ConnManager.HandleError(), который передаст нужную информацию пользователю, запишит что-либо в лог, и, после того как HandleError завершится, соединение будет закрыто.</p>

<pre><code class="go">type RequestVerifier interface {
    // Verify проверяет соединение и возвращает ошибку в случае, если
    // что-то не так.
    Verify(ws *websocket.Conn) error
}
</code></pre>

<p>А так выглядит конструктор хендлера:</p>

<pre><code class="go">// PoolHandler создает хендлер с указанными менеджером групп,
// менеджером соединений и проверяльщиком. Проверяльщик может быть
// nil, и тагда соединения не будут проверяться
func PoolHandler(poolMgr PoolManager, connMgr ConnManager,
    verifier RequestVerifier) http.Handler {
}
</code></pre>

<h3>Использование</h3>

<p>Итак, мы имеем:</p>

<ul>
<li>RequestVerifier - займется логикой авторизации</li>
<li>PoolManager - займется логиками управления группами и инициализации общих данных группы. Здесь следует использовать фабрики (или одну фабрику) для генерации групп и генерации общих данных. Фабрики указываем при инициализации самого PoolManager. То есть тут лучше разделить: добавление и удаление групп - это задачи для PoolManager, а деталями инициализации занимаются фабрики.</li>
<li>ConnManager займется работой с соединением. В нем же инициализируем данные участника, тоже обратившись к фабрике, которая сделает все, что нужно. Фабрику указываем при инициализации ConnManager.</li>
</ul>

<h3>Ссылки</h3>

<ul>
<li>Пакет - <a href="https://github.com/ivan1993spb/pwshandler">github.com/ivan1993spb/pwshandler</a></li>
<li>Пример использования - <a href="https://bitbucket.org/pushkin_ivan/clever-snake">Змейка онлайн</a>. Смотрите ветку dev</li>
<li>Godoc <a href="http://godoc.org/github.com/ivan1993spb/pwshandler">github.com/ivan1993spb/pwshandler</a></li>
<li>Godoc <a href="http://godoc.org/golang.org/x/net/websocket">golang.org/x/net/websocket</a></li>
</ul>

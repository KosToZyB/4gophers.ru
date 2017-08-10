+++
date = "2015-06-26T18:37:06+03:00"
draft = false
title = "Event Sourcing с помощью Kafka and Go"

+++

Перевод статьи "[Writing and Testing an Event Sourcing Microservice with Kafka and Go](https://semaphoreci.com/community/tutorials/writing-and-testing-an-event-sourcing-microservice-with-kafka-and-go)"

В этой статье мы поговорим об обработке сообщений в распределенной системе с использованием Kafka и паттерна Event Sourcing. Для начала посмотрим как нам может помочь Kafka, затем используем паттерн Command-Query Responsibility Segregation ([CQRS](https://martinfowler.com/bliki/CQRS.html)), с помощью которого мы реализуем прием и обработку данных. И напоследок посмотрим как можно использовать группы консумеров, чтобы сделать нашу систему более избыточной и надежной. Само собой, все приложение будет написано не Go.

## Зачем нам микросервисы

Использование микросервисов дает нам масштабируемость. Как правило, это самый главный аргумент. Чем больше становится проект, чем сильнее разрастается кодовая база, тем сложнее вносить изменения ничего не ломая. 

Именно поэтому многие разработчики все больше внимания обращают на микросервисы. Разбивая систему на небольшие составные части(например сервис для управления пользователями, сервис для приема оплаты и т.к.), мы получаем возможность довольно просто добавлять новые фичи.

## Проблемы при использовании микросервисов

Очень часто создание микросервисов становится не простым делом. Например, некоторые части системы должны быть атомарными и уметь работать с распределенными данными. Проблема атомарности очень часто встречается в микросервисных архитектурах.

Кроме того, при такой архитектуре получение данных тоже становится проблемой. Если у вас два сервиса, которые отвечают по отдельности за заказ и заказчиков, то указанный ниже запрос просто так не выполнить:

```
select * from order o, customer c
  where o.customer_id = c.id
  and o.gross_amount > 50000
  and o.status = 'PAID'
  and c.country = 'INDONESIA';
```

В таком случае нам просто необходимо использовать паттерны Command-Query Responsibility Segregation и [Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html).

Конечно, не нужно в каждую систему впихивать Event Sourcing. Этот паттерн имеет смысл использовать в случае если необходим полный аудит системы и понимание что происходило в любой момент времени. Если необходима только базовая обработка данных от системы, то вполне достаточно использовать событийную модель

## Kafka

Если очень грубо сравнивать Kafka с обычными базами, то таблица в базе это топик в Kafka. В каждой таблице данные представлены в виде строки, в Kafka дата представляют собой [commit log](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying) строки. Каждая строка имеет свой индекс(он же offset). В Kafka порядок лога очень важен, поэтому каждая строчка помечается специальным автоинкриментящимся значением, кторый используется как offset.

В отличие от таблиц в SQL базах, топик должен иметь больше чем одну партицию. Kafka гарантирует константное время работы O(1), каждая партиция может содержать тысячи, миллионы и даже больше записей, при этом сохранять нормальную работу. Каждая партиция содержит разные логи.

Partitioning is the the process through which Kafka allows us to do parallel processing. Thanks to partitioning, each consumer in a consumer group can be assigned to a process in an entirely different partition. In other words, this is how Kafka handles load balancing.

Each message is produced somewhere outside of Kafka. The system responsible for sending a commit log to a Kafka broker is called a producer. The commit log is then received by a unique Kafka broker, acting as the leader of the partition to which the message is sent. Upon writing the data, each leader then replicates the same message to a different Kafka broker, either synchronously or asynchronously, as desired by the producer. This Producer-Broker orchestration is handled by an instance of Apache ZooKeeper, outside of Kafka.

Kafka is usually compared to a queuing system such as RabbitMQ. What makes the difference is that after consuming the log, Kafka doesn't delete it. In that way, messages stay in Kafka longer, and they can be replayed.

## Setting Up Kafka
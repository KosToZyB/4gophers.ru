+++
date = "2017-05-20T20:53:09+03:00"
draft = false
title = "Распределенный поиск Трампа"

+++

![](/img/trump/trump.jpg)

Перевод статьи "[Building a distributed Trump finder](http://gopherdata.io/post/distributed_trump_finder/)". 

В этой статье мы будем работать с инструментом для машинного обучения "[Machine Box](https://machinebox.io/)". Это очень классная штука, и вам обязательно нужно ее попробовать. По своей сути, это набор заранее подготовленных docker контейнеров, готовых к использованию и предоставляющих интерфейс для различных операций связанных с машинным обучением. К примеру, вы можете воспользоваться контейнером "facebox" для распознавания лиц. После запуска этого контейнера, у вас появится доступ к JSON api, которое позволит обучить этот "facebox" на определенных изображениях лиц людей, распознать эти лица на различных изображениях и сохранить "состояние" обученной модели для дальнейшего использования.

Эксперименты с "facebox" заставили меня задуматься, как можно было бы его интегрировать в мой воркфлоу. В частности, я хочу посмотреть, можно ли "Machine Box" контейнеры использовать в связке с распределенной обработкой данных, организованной на базе [Pachyderm](http://pachyderm.io/). Pachyderm позволяет создавать и запускать пайплайны(воркфлоу для машинного обучения) тоже на базе контейнеров. Таким образом, объединение этих инструментов не должно быть слишком сложным.

Насколько мне известно, это первый распределенный, основанный на Docker контейнерах пайплайн для поиска Трампа. Ниже мы рассмотрим создание пайплайна для распознавания лиц, который мы будем использовать для локлизации лица Дональда Трампа на фотографиях. Замечу, что этот пайплайн можно будет использовать для поиска любых лиц. Мы расширим его функциональность и научим находить лицо Хиллари Клинтон.

Я рассчитываю, что у вас уже запущен кластер Pachyderm и `pachctl` настроен для работы с этим кластером. У вас должен быть ключ API для Machine Box (регистрация бесплатна). Весь код и все детали можно найти [тут](https://github.com/dwhitena/pach-machine-box).

### Готовим входные данные для пайплайна

Для тренировки нашей модели распознавания лиц, обнаружения их на изображениях, и тегирования этих самых лиц нам нужно три "репозитория данных", которые и будут входом для нашего пайплайна:

* `trainig` - набор изображений с лицами, который будет использоваться для обучения faceboc.
* `unidentified` - набор изображений, в котором содержатся изображения, которые мы хотим распознать.
* `labels` - набор маркеров, которыми мы хотим пометить лица на изображениях.

Для начала нам нужно воспользоваться `pachctl`:

```
$ pachctl create-repo training
$ pachctl create-repo unidentified
$ pachctl create-repo labels
$ pachctl list-repo
NAME                CREATED             SIZE                
labels              3 seconds ago       0 B                 
unidentified        11 seconds ago      0 B                 
training            17 seconds ago      0 B
$ cd data/train/faces1/
$ ls
trump1.jpg  trump2.jpg  trump3.jpg  trump4.jpg  trump5.jpg
$ pachctl put-file training master -c -r -f .
$ pachctl list-repo
NAME                CREATED             SIZE                
training            5 minutes ago       486.2 KiB           
labels              5 minutes ago       0 B                 
unidentified        5 minutes ago       0 B                 
$ pachctl list-file training master
NAME                TYPE                SIZE                
trump1.jpg          file                78.98 KiB           
trump2.jpg          file                334.5 KiB           
trump3.jpg          file                11.63 KiB           
trump4.jpg          file                27.45 KiB           
trump5.jpg          file                33.6 KiB 
$ cd ../../labels/
$ ls
clinton.jpg  trump.jpg
$ pachctl put-file labels master -c -r -f .
$ cd ../unidentified/
$ ls
image1.jpg  image2.jpg
$ pachctl put-file unidentified master -c -r -f .
$ pachctl list-repo
NAME                CREATED             SIZE                
unidentified        7 minutes ago       540.4 KiB           
labels              7 minutes ago       15.44 KiB           
training            7 minutes ago       486.2 KiB
```

### Тренируем(или обучаем) facebox:

Теперь мы создадим пайплайн, который использует `training` как входной набор изображений для facebox. Результатом работы этого пайплайна будет определенное "состояние" тренируемой модели. Для этого нам нужно сформировать специальный [train.json](https://github.com/dwhitena/pach-machine-box/blob/master/pipelines/train.json) файл. Этот файл спецификации определяет Docker образ для обработки данных, команды которые нужно запустить в контейнере и какие данные должны подаваться на вход пайплайна.

В нашем примере в `train.json` указан контейнер на основе facebox образа с Machine Box, а также набор curl запросов, которые загружают данные в это контейнер. Как только тренировочные данные загружены и обработаны с помощью facebox, мы используем еще один curl запрос для экспорта "состояния" нашей модели (это "состояние" пригодится нам позже).

Мы используем небольшую магию bash для некоторых наших операций. Надеюсь, что в будущем Machine Box будет предоставлять чуть больше стандартных команд для большего количества распространенных кейсов.

```
create-MB-pipeline.sh  identify.json  tag.json  train.json
$ ./create-MB-pipeline.sh train.json 
$ pachctl list-pipeline
NAME                INPUT               OUTPUT              STATE               
model               training            model/master        running    
$ pachctl list-job
ID                                   OUTPUT COMMIT STARTED       DURATION RESTART PROGRESS STATE            
3425a7a0-543e-4e2a-a244-a3982c527248 model/-       9 seconds ago -        1       0 / 1    running 
$ pachctl list-job
ID                                   OUTPUT COMMIT                          STARTED       DURATION  RESTART PROGRESS STATE            
3425a7a0-543e-4e2a-a244-a3982c527248 model/1b9c158e33394056a18041a4a86cb54a 5 minutes ago 5 minutes 1       1 / 1    success 
$ pachctl list-repo
NAME                CREATED             SIZE                
model               5 minutes ago       4.118 KiB           
unidentified        18 minutes ago      540.4 KiB           
labels              18 minutes ago      15.44 KiB           
training            19 minutes ago      486.2 KiB           
$ pachctl list-file model master
NAME                TYPE                SIZE                
state.facebox       file                4.118 KiB
```

Видно, что выходные данные, в нашем случае, это файл `.facebox` в котором сохранено "состояние" обученной модели.

### Используем натренированную модель для распознания лиц

Теперь нам нужно запустить пайплайн на базе спецификации [identify.json](https://github.com/dwhitena/pach-machine-box/blob/master/pipelines/identify.json). Этот пайплайн будет работать с `unidentified` картинками. Это пайплайн будет получать на вход сохраненное "состояние" `model` и `unidentified` картинки. Также, он будет выполнять cURL запросы для общения с facebox. Результаты распознавания лиц будут выдаваться в JSON файлах, по одному на каждое изображение.

```
$ ./create-MB-pipeline.sh identify.json 
$ pachctl list-job
ID                                   OUTPUT COMMIT                          STARTED       DURATION  RESTART PROGRESS STATE            
281d4393-05c8-44bf-b5de-231cea0fc022 identify/-                             6 seconds ago -         0       0 / 2    running 
3425a7a0-543e-4e2a-a244-a3982c527248 model/1b9c158e33394056a18041a4a86cb54a 8 minutes ago 5 minutes 1       1 / 1    success 
$ pachctl list-job
ID                                   OUTPUT COMMIT                             STARTED            DURATION   RESTART PROGRESS STATE            
281d4393-05c8-44bf-b5de-231cea0fc022 identify/287fc78a4cdf42d89142d46fb5f689d9 About a minute ago 53 seconds 0       2 / 2    success 
3425a7a0-543e-4e2a-a244-a3982c527248 model/1b9c158e33394056a18041a4a86cb54a    9 minutes ago      5 minutes  1       1 / 1    success 
$ pachctl list-repo
NAME                CREATED              SIZE                
identify            About a minute ago   1.932 KiB           
model               10 minutes ago       4.118 KiB           
unidentified        23 minutes ago       540.4 KiB           
labels              23 minutes ago       15.44 KiB           
training            24 minutes ago       486.2 KiB           
$ pachctl list-file identify master
NAME                TYPE                SIZE                
image1.json         file                1.593 KiB           
image2.json         file                347 B
```

Ниже показан результат распознавания лица Трампа на изображении `image1.jpg`. Как видите, это JSON файл, в котором указаны координаты и размер лица на изображении.

```json
{
    "success": true,
    "facesCount": 13,
    "faces": [
        ...
        ...
        {
            "rect": {
                "top": 175,
                "left": 975,
                "width": 108,
                "height": 108
            },
            "id": "58ff31510f7707a01fb3e2f4d39f26dc",
            "name": "trump",
            "matched": true
        },
        ...
        ...
    ]
}
```

### Теггирование распознанных лиц на изображениях

Мы прошли большую часть пути. У нас получить обнаружить трампа на изображениях из набора `unidentified`. Но JSON формат не самое лучшее средство визуализации. Давайте добавим метки прямо на изображения.

Для этого мы можем использовать супер [простую Go программу](https://github.com/dwhitena/pach-machine-box/blob/master/tagimage/main.go), которая будет рисовать специальную метку в указанных координатах поверх изображения с которым мы работаем. Эта часть пайплайна определена в спецификации [tag.json](https://github.com/dwhitena/pach-machine-box/blob/master/pipelines/tag.json), ниже показано, как ее можно добавить.

```
$ pachctl create-pipeline -f tag.json 
$ pachctl list-job
ID                                   OUTPUT COMMIT                             STARTED        DURATION   RESTART PROGRESS STATE            
cd284a28-6c97-4236-9f6d-717346c60f24 tag/-                                     2 seconds ago  -          0       0 / 2    running 
281d4393-05c8-44bf-b5de-231cea0fc022 identify/287fc78a4cdf42d89142d46fb5f689d9 5 minutes ago  53 seconds 0       2 / 2    success 
3425a7a0-543e-4e2a-a244-a3982c527248 model/1b9c158e33394056a18041a4a86cb54a    13 minutes ago 5 minutes  1       1 / 1    success 
$ pachctl list-job
ID                                   OUTPUT COMMIT                             STARTED        DURATION   RESTART PROGRESS STATE            
cd284a28-6c97-4236-9f6d-717346c60f24 tag/ae747e8032704b6cae6ae7bba064c3c3      25 seconds ago 11 seconds 0       2 / 2    success 
281d4393-05c8-44bf-b5de-231cea0fc022 identify/287fc78a4cdf42d89142d46fb5f689d9 5 minutes ago  53 seconds 0       2 / 2    success 
3425a7a0-543e-4e2a-a244-a3982c527248 model/1b9c158e33394056a18041a4a86cb54a    14 minutes ago 5 minutes  1       1 / 1    success 
$ pachctl list-repo
NAME                CREATED             SIZE                
tag                 30 seconds ago      591.3 KiB           
identify            5 minutes ago       1.932 KiB           
model               14 minutes ago      4.118 KiB           
unidentified        27 minutes ago      540.4 KiB           
labels              27 minutes ago      15.44 KiB           
training            27 minutes ago      486.2 KiB           
$ pachctl list-file tag master
NAME                TYPE                SIZE                
tagged_image1.jpg   file                557 KiB             
tagged_image2.jpg   file                34.35 KiB    
```

Видно, что на выходе `tag` появилось два тегированных изображения. Если мы посмотрим эти изображения, то увидим что наш Трамп поиск работает:

![](/img/trump/tagged_images1.jpg)

### Обучение новым лицам

На самом деле, наш пайплайн совсем не ограничен поиском только лица Трампа. Мы можем обучить "facebox" поиску любого другого лица. Для этого нужно обновить `trainig`. Одно из замечательных свойств Pachyderm - версионирование данных. Как только "facebox" обучится новым лицам и данные изменятся, Pachyderm автоматически обновит все наши результаты.

```
$ cd ../data/train/faces2/
$ ls
clinton1.jpg  clinton2.jpg  clinton3.jpg  clinton4.jpg
$ pachctl put-file training master -c -r -f .
$ pachctl list-job
ID                                   OUTPUT COMMIT                             STARTED        DURATION   RESTART PROGRESS STATE            
56e24ac0-0430-4fa4-aa8b-08de5c1884db model/-                                   4 seconds ago  -          0       0 / 1    running 
cd284a28-6c97-4236-9f6d-717346c60f24 tag/ae747e8032704b6cae6ae7bba064c3c3      6 minutes ago  11 seconds 0       2 / 2    success 
281d4393-05c8-44bf-b5de-231cea0fc022 identify/287fc78a4cdf42d89142d46fb5f689d9 11 minutes ago 53 seconds 0       2 / 2    success 
3425a7a0-543e-4e2a-a244-a3982c527248 model/1b9c158e33394056a18041a4a86cb54a    20 minutes ago 5 minutes  1       1 / 1    success 
$ pachctl list-job
ID                                   OUTPUT COMMIT                             STARTED            DURATION   RESTART PROGRESS STATE            
6aa6c995-58ce-445d-999a-eb0e0690b041 tag/7cbd2584d4f0472abbca0d9e015b9829      5 seconds ago      1 seconds  0       2 / 2    success 
8a7961b7-1085-404a-b0ee-66034fae7212 identify/1bc94ec558e44e0cb45ed5ab7d9f9674 59 seconds ago     54 seconds 0       2 / 2    success 
56e24ac0-0430-4fa4-aa8b-08de5c1884db model/002f16b63a4345a4bc6bdf5510c9faac    About a minute ago 19 seconds 0       1 / 1    success 
cd284a28-6c97-4236-9f6d-717346c60f24 tag/ae747e8032704b6cae6ae7bba064c3c3      8 minutes ago      11 seconds 0       2 / 2    success 
281d4393-05c8-44bf-b5de-231cea0fc022 identify/287fc78a4cdf42d89142d46fb5f689d9 13 minutes ago     53 seconds 0       2 / 2    success 
3425a7a0-543e-4e2a-a244-a3982c527248 model/1b9c158e33394056a18041a4a86cb54a    21 minutes ago     5 minutes  1       1 / 1    success 
$ pachctl list-file tag master
NAME                TYPE                SIZE                
tagged_image1.jpg   file                557 KiB             
tagged_image2.jpg   file                36.03 KiB
```

Как видите, все результаты обновились и нам не пришлось что-то делать руками.

![](/img/trump/tagged_images2.jpg)

### Заключение:

Machine Box и Pachyderm позволяют очень быстро реализовать распределенный пайплайн для обработки данных и машинного обучения.

* Если вы хотите запустить свой собственный поисковик Трампа, то весь [код собран на гитхабе](https://github.com/dwhitena/pach-machine-box).
* У Pachyderm есть свое сообщество, которое активно [общается в слаке](http://slack.pachyderm.io/), а также заходите в канал #data-science Gophers слак.
* Следите за обновлениями Pachyderm в [twitter](https://twitter.com/pachydermIO).
* Тут можно получить бесплатный [Machine Box API ключ](https://machinebox.io/).
* И за обновлениями Machine Box тоже можно следить в [twitter](https://twitter.com/machineboxio).
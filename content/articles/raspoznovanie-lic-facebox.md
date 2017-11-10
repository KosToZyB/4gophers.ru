+++
date = "2017-09-04T23:30:00+03:00"
draft = false
title = "TT Стриминг и распознавание лиц через веб-камеру"

+++ 

Перевод статьи "[Stream and recognise people from a webcam with Go and Facebox](https://blog.machinebox.io/streaming-and-recognize-people-from-a-webcam-with-go-and-facebox-acea645b94ab)".

От видеонаблюдения для предотвращения вторжения до распознавания личности человека за дверью и автоматического открытия дверей. Все это можно сделать с помощью нескольких строк на pyton, Go и используя [Facebox](https://machinebox.io/docs/facebox).

Для начала нам нужно научиться получать видео с веб-камеры. Есть множество вариантов, как это можно сделать с Go. К сожалению, большинство из них тянут за собой CGO биндинги к OpenCV, при этом поддержка функциональности очень ограничена, а сами проекты довольно монструозны. А так как всегда нужно подбирать инструмент под задачу, то для захвата видео мы будем использовать python.

В python мы можем использовать с OpenCV для стриминга "[Motion JPEG](https://en.wikipedia.org/wiki/Motion_JPEG)" через стандартный вывод.

"Motion JPEG(M-JPEG)" звучит как что-то сложное и фантастическое. Хотя на сам деле это всего лишь конкатенация фреймов в один JPEG с определенных использованием разделителей. Больше всего это похоже на CSV для видео. Как правило, за простоту приходится платить размерами файлов, потому что не используется видео сжатие.

M-PEG используется во многих устройствах: IP камерах, цифровых камерах. Возможно у вас уже есть устройство, которое поддерживает этот протокол. В таком случае вы можете использовать аналогичный подход для стриминга.

Ниже показан кусок кода `capture.py`, который кадр за кадром захватывает видео с камеры и стримит его `stdout`

```
#!/usr/bin/env python
import cv2
import imutils
from imutils.video import VideoStream
import time, sys
vs = VideoStream(resolution=(320, 240)).start()
time.sleep(1.0)
while(True):
   #read frame by frame the webcame stream
   frame = vs.read()

   # encode as a JPEG
   res = bytearray(cv2.imencode(".jpeg", frame)[1])
   size = str(len(res))
   # stream to the stdout
   sys.stdout.write("Content-Type: image/jpeg\r\n")
   sys.stdout.write("Content-Length: " + size + "\r\n\r\n")
   sys.stdout.write( res )
   sys.stdout.write("\r\n")
   # we use 'informs' as a boundary   
   sys.stdout.write("--informs\r\n")
   
   if cv2.waitKey(1) & 0xFF == ord('q'):
      break
cv2.destroyAllWindows()
vs.stop()
```

### Небольшое отступление от перевода

На сомом деле нет ничего сложно, чтобы в этом месте использовать Go. К тому же, питоновская либа для работы c opencv тоже реализована через биндинги к сишному коду.

В качестве замены питоновского скрипта выше, можно использовать либу [github.com/blackjack/webcam](github.com/blackjack/webcam). Достаточно всего лишь чуть-чуть модернизировать пример из папки examples, чтобы все заработало:

```
package main

import (
	"fmt"
	"os"

	"github.com/blackjack/webcam"
)

func main() {
	cam, err := webcam.Open("/dev/video0")
	if err != nil {
		panic(err.Error())
	}
	defer cam.Close()

	var format webcam.PixelFormat
	for f, d := range cam.GetSupportedFormats() {
		if d == "Motion-JPEG" {
			format = f
			break
		}
	}
	_, _, _, err = cam.SetImageFormat(format, 320, 240)
	if err != nil {
		panic(err.Error())
	}

	err = cam.StartStreaming()
	if err != nil {
		panic(err.Error())
	}

	for {
		err = cam.WaitForFrame(5)

		switch err.(type) {
		case nil:
		case *webcam.Timeout:
			continue
		default:
			panic(err.Error())
		}

		frame, err := cam.ReadFrame()
		if len(frame) != 0 {
			os.Stdout.Write([]byte("Content-Type: image/jpeg\r\n"))
			os.Stdout.Write([]byte(fmt.Sprintf("Content-Length: %d\r\n\r\n", len(frame))))
			os.Stdout.Write(frame)
			os.Stdout.Write([]byte("\r\n"))
			os.Stdout.Write([]byte("--informs\r\n"))

			os.Stdout.Sync()
		} else if err != nil {
			panic(err.Error())
		}
	}
}
```

Конечно, в этом коде не хватает проверок. Например, когда мы вызываем `cam.GetSupportedFormats` и проходим по всему списку, там вполне может не быть `Motion-JPEG`. Точно также, размер 320 на 240 может не поддерживаться и неплохо было бы использовать функцию `cam.GetSupportedFrameSizes`. Такой код нельзя использовать в продакшене, тем не менее, он работает и подходит нам в качестве примера.

### Стриминг с помощью Go http сервер

Теперь у нас есть стрим, который мы можем прочитать и обработать в Go. Давайте создадим простой http сервер, который будет отправлять стрим с вебкамеры в браузер.

```
package main

import (
	"log"
	"net/http"
	"os/exec"
)

const boundary = "informs"

func main() {
	http.HandleFunc("/cam", cam)
	log.Fatal(http.ListenAndServe(":8081", nil))
}

func cam(w http.ResponseWriter, r *http.Request) {
	// указываем специальный multipart заголовок
	w.Header().Set("Content-Type", "multipart/x-mixed-replace; boundary="+boundary)
	// запускаем бинарник для работы с веб-камерой
	cmd := exec.CommandContext(r.Context(), "./bin/webcam")
	// отправляем вывод программы в response writer
	cmd.Stdout = w

	err := cmd.Run()
	if err != nil {
		log.Println("error capturing webcam", err)
	}
}
```

Если перейти по ссылке `http://localhost:8081/cam`, то запуститься новый процесс и начнется воспроизведение видео прямо в браузере. 

Обратите внимание на использование `context.Context`. С его помощью мы можем остановить процесс, когда http запрос будет прерван.

И еще одно замечание от переводчика. Так как мы теперь умеем получать стрим от камеры в нужном нам формате с помощью Go, то нам не нужно делать отдельный бинарник и вот это все. Можно добавить весь этот функционал непосредственно в программу, реализующую http хендлеры.

### Обработка стрима и распознавание лиц

Теперь когда у нас есть возможность управлять стримом в Go, мы можем приступить к самой интересной части.

Основная идея в том, чтобы читать из стандартного вывода программы `capture.py` и использовать пайп для чтения стрима фрейм за фреймом.

```
cmd := exec.CommandContext(r.Context(), "./capture.py")
stdout, err := cmd.StdoutPipe()
if err != nil {
   log.Println("error Getting the stdout pipe")
   return
}
cmd.Start()
```

Теперь мы модем использовать `multipart.Reader` для чтения одного кадра в память. Мы можем считывать и записывать кадры когда и сколько захотим.

```
mr := multipart.NewReader(stdout, boundary)
for {
      p, err := mr.NextPart()
      if err == io.EOF {
            break
      }
      if err != nil {
            log.Println("error reading next part", err)
            return
      }

      jp, err := ioutil.ReadAll(p)
      if err != nil {
            log.Println("error reading from bytes ", err)
            continue
      }
}
```

Дальше мы можем использовать [Facebox SDK](https://github.com/machinebox/sdk-go) для работы с запущенным инстансом Facebox и, собственно, распознавать лица.

```
jpr := bytes.NewReader(jp)
// проверяем, что в кадре человек, которого мы можем распознать
faces, err := fbox.Check(jpr)
if err != nil {
      log.Println("error calling facebox", err)
      continue
}
// для всех распознанных мы можем выполнить определенные действия, 
// например открыть двор  
for _, face := range faces {
      if face.Matched {
            fmt.Println("I know you ", face.Name)
      } else {
            fmt.Println("I do not know you ")
      }
}
```

Как только мы провели все манипуляции с кадром, его необходимо отдать пользователю. Нужно принять во внимание, что все выполняется в рамках одной рутины, а это неплохо так будет тормозить воспроизведение видео. К тому же, распознавание лиц очень затратная операция по ЦПУ. Если вы не хотите чтобы видео тормозило, то распознавание нужно выполнять в отдельной рутине. 

Полный код http обработчика `faceboxhandler`:

```
func facebox(w http.ResponseWriter, r *http.Request)  {
	w.Header().Set("Content-Type", "multipart/x-mixed-replace; boundary="+boundary)

	cmd := exec.CommandContext(r.Context(), "./capture.py")
	stdout, err := cmd.StdoutPipe()
	if err != nil {
		log.Println("error getting the stdout pipe")
		return
	}

	cmd.Start()

	mr := multipart.NewReader(stdout, boundary)
	for {
		p, err := mr.NextPart()
		if err == io.EOF {
			break
		}
		if err != nil {
			log.Println("error reading next part", err)
			return
		}

		jp, err := ioutil.ReadAll(p)
		if err != nil {
			log.Println("error reading from bytes ", err)
			continue
		}

		jpr := bytes.NewReader(jp)
		faces, err := fbox.Check(jpr)
		if err != nil {
			log.Println("error calling facebox", err)
			continue
		}

		for _, face := range faces {
			if face.Matched {
				fmt.Println("I know you ", face.Name)
			} else {
				fmt.Println("I do not know you ")
			}
		}

		w.Write([]byte("Content-Type: image/jpeg\r\n"))
		w.Write([]byte("Content-Length: " + string(len(jp)) + "\r\n\r\n"))
		w.Write(jp)
		w.Write([]byte("\r\n"))
		w.Write([]byte("--informs\r\n"))
	}

	cmd.Wait()
}
```

Чтобы запустить все что мы написали, нужно открыть в браузере `http://localhost:8081/facebox`. На этой странице будет крутиться видео(с большим лагом, правда), а в консоли будут выводиться сообщения. Конечно, все это будет работать только если вы обучили [свой Facebox](https://machinebox.io/docs/facebox/teaching-facebox).

### Заключение

Теперь вы знаете как с помощью Go и Python можно быстро сделать видео сервис. А при небольшом усилии еще и добавить туда распознавание лиц.

* Для использования Facebox нужно создать аккаунт на [machinebox.io](https://machinebox.io/)
* Полный код примера можно найти на github: [https://github.com/machinebox/webcamFacebox](https://github.com/machinebox/webcamFacebox) 

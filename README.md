# Radio
**Инструкция по установке IceCast и GStreamer для конвертации потоков**

Приветствую всех! В данном руководстве описывается процесс установки и настройки системы для конвертации потоков радиостанций в формат MP3 с использованием IceCast и GStreamer в Docker-контейнере. Эта конфигурация позволяет легко интегрировать потоковую передачу в Minecraft и другие приложения.

Перед тем как начать, хочу отметить, что я не несу ответственности за ваши действия. Все действия описаны с целью демонстрации, и использование их на практике происходит на ваш страх и риск.

- У вас должен быть уже установлен Docker (желательно с Portainer для упрощения управления контейнерами). В противном случае, пожалуйста, ознакомьтесь с документацией для установки Docker и Portainer.

---

## Установка IceCast

1. Заходим по SSH на сервер, где установлен Docker, и выполняем команду для установки репозитория IceCast:

```bash
docker run -p 8000:8000 -e ICECAST_SOURCE_PASSWORD=aaaa -e ICECAST_ADMIN_PASSWORD=bbbb -e ICECAST_PASSWORD=cccc -e ICECAST_RELAY_PASSWORD=dddd -e ICECAST_HOSTNAME=noise.example.com moul/icecast
```
Параметры:

Замените пароли (с параметрами ICECAST_SOURCE_PASSWORD, ICECAST_ADMIN_PASSWORD и т.д.) на свои собственные.
Параметр ICECAST_HOSTNAME должен быть заменен на IP-адрес вашего сервера.
После запуска контейнера, перейдите в веб-панель IceCast по адресу:
```
http://ВАШ_IP_АДРЕС:8000
```
В панели "Administration" выполните авторизацию:
Логин: admin
Пароль: используйте пароль, указанный для ICECAST_ADMIN_PASSWORD.
Теперь вы должны увидеть панель администратора IceCast.

Установка и настройка GStreamer
Для установки GStreamer используем следующую команду:
```
docker run restreamio/gstreamer:x86_64-latest-dev-with-source
```
После этого откройте Portainer, выберите контейнер GStreamer, затем нажмите "Console" и подключитесь к контейнеру.

В терминале контейнера выполните команду для обновления пакетов:

```
apt update
```
Установите текстовый редактор nano:
```
apt install nano
```
Создайте конфигурационный файл stream.sh с помощью команды:
```
nano stream.sh
```
Содержимое файла stream.sh:

```
#!/bin/bash
# Настройка для приема потока и отправки его в IceCast
gst-launch-1.0 -v \
    souphttpsrc location=http://127.0.0.1:8000/your-stream-url ! \
    decodebin ! \
    audioconvert ! \
    lamemp3enc bitrate=128 ! \
    shout2send ip=127.0.0.1 port=8000 password="your-password" mount="/your-mountpoint"
```
Пояснения:

location=http://127.0.0.1:8000/your-stream-url — это исходный поток радиостанции.
ip=127.0.0.1 — укажите IP-адрес вашего сервера с IceCast.
port=8000 — используйте стандартный порт IceCast, если не меняли.
password="your-password" — укажите пароль, настроенный в IceCast для источника потока.
mount="/your-mountpoint" — укажите точку монтирования, которая будет доступна по ссылке.
Сохраните файл, используя CTRL + W, затем подтвердите сохранение, нажав Y и ENTER.

Сделайте файл исполнимым:

```
chmod +x stream.sh
```
Запустите скрипт:
```
./stream.sh
```
Теперь, если вы зайдете в веб-панель IceCast, вы должны увидеть ваш поток.

Автоматический запуск при перезагрузке
Для того чтобы контейнеры автоматически запускались после перезагрузки, установите флаг Restart policies = always для контейнеров в Docker или Portainer. Это позволит автоматически запускать контейнеры при старте системы.

Однако, чтобы GStreamer также запускался автоматически, нужно использовать команду:

```
./stream.sh
```

Добавление второго потока
Если вы хотите добавить второй поток, просто откройте файл stream.sh и добавьте следующий код в конце:

```
#!/bin/bash
# Настройка для первого потока
gst-launch-1.0 -v \
    souphttpsrc location=http://127.0.0.1:8000/first-stream-url ! \
    decodebin ! \
    audioconvert ! \
    lamemp3enc bitrate=128 ! \
    shout2send ip=127.0.0.1 port=8000 password="your-password" mount="/first-mountpoint" &

# Настройка для второго потока
gst-launch-1.0 -v \
    souphttpsrc location=http://127.0.0.1:8000/second-stream-url ! \
    decodebin ! \
    audioconvert ! \
    lamemp3enc bitrate=128 ! \
    shout2send ip=127.0.0.1 port=8000 password="your-password" mount="/second-mountpoint" &
```
Важное замечание: Каждый поток должен иметь уникальную точку монтирования (mount), чтобы избежать конфликтов.

Заключение
Теперь, когда вы настроили IceCast и GStreamer для конвертации потоков, и добавили возможность запуска нескольких потоков, можно смело запускать вашу систему и использовать радиостанции в Minecraft или других приложениях. Удачи и приятного использования!

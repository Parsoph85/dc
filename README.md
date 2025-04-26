---
title: Начало работы с Docker Compose
linkTitle: Quickstart
weight: 30
---

Цель этого руководства — познакомить вас с основными концепциями Docker Compose, проведя вас через процесс разработки простого веб-приложения на Python.

Используя фреймворк Flask, приложение оснащено счетчиком посещений в Redis, что является практическим примером того, как Docker Compose может применяться в сценариях веб-разработки.

Представленные здесь концепции должны быть понятны даже тем, кто не знаком с Python.

Это ненормативный пример, который просто подчеркивает основные возможности Compose.

## Предварительные требования

Убедитесь, что у вас установлена последняя версия Docker Compose и есть базовое понимание концепций Docker и его работы.

## Шаг 1: Настройка

1. Создайте каталог для проекта:

   ```console
   $ mkdir composetest
   $ cd composetest
   ```

2. Создайте файл с именем app.py в каталоге вашего проекта и вставьте в него следующий код:

   ```python
   import time

   import redis
   from flask import Flask

   app = Flask(__name__)
   cache = redis.Redis(host='redis', port=6379)

   def get_hit_count():
       retries = 5
       while True:
           try:
               return cache.incr('hits')
           except redis.exceptions.ConnectionError as exc:
               if retries == 0:
                   raise exc
               retries -= 1
               time.sleep(0.5)

   @app.route('/')
   def hello():
       count = get_hit_count()
       return f'Hello World! I have been seen {count} times.\n'
    ```

  В этом примере redis— имя хоста контейнера Redis в сети приложения и порт по умолчанию 6379.

  
3. Создайте еще один файл с именем requirements.tx tв каталоге вашего проекта и вставьте в него следующий код:

   ```text
   flask
   redis
   ```

4. Создайте Dockerfile и вставьте следующий код:

   ```dockerfile
   # syntax=docker/dockerfile:1
   FROM python:3.10-alpine
   WORKDIR /code
   ENV FLASK_APP=app.py
   ENV FLASK_RUN_HOST=0.0.0.0
   RUN apk add --no-cache gcc musl-dev linux-headers
   COPY requirements.txt requirements.txt
   RUN pip install -r requirements.txt
   EXPOSE 5000
   COPY . .
   CMD ["flask", "run", "--debug"]
   ```

   

   > [!Важно]
   >
   >Проверьте, что у файла Dockerfile нет расширения, например .txt. Некоторые редакторы могут добавлять это расширение файла автоматически, что приводит к ошибке при запуске приложения.


## Шаг 2: Определите службы в файле Compose

Compose упрощает управление всем стеком приложений, позволяя легко управлять службами, сетями и томами в едином понятном файле конфигурации YAML.

Создайте файл с именем compose.yaml в каталоге вашего проекта и вставьте следующее:

```yaml
services:
  web:
    build: .
    ports:
      - "8000:5000"
  redis:
    image: "redis:alpine"
```

В этом файле Compose определяются две службы: web и redis.

Служба web использует образ, созданный из Dockerfile в текущем каталоге. Затем она привязывает контейнер и хост-машину к открытому порту 8000. Этот пример службы использует порт по умолчанию для веб-сервера Flask - 5000.

Сервис redis использует общедоступный образ Redis , извлеченный из реестра Docker Hub.


## Шаг 3: Создайте и запустите свое приложение с помощью Compose

С помощью одной команды вы создаете и запускаете все службы из вашего файла конфигурации.

1. Из каталога проекта запустите приложение, выполнив docker compose up.

   ```console
   $ docker compose up

   Creating network "composetest_default" with the default driver
   Creating composetest_web_1 ...
   Creating composetest_redis_1 ...
   Creating composetest_web_1
   Creating composetest_redis_1 ... done
   Attaching to composetest_web_1, composetest_redis_1
   web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
   redis_1  | 1:C 17 Aug 22:11:10.480 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
   redis_1  | 1:C 17 Aug 22:11:10.480 # Redis version=4.0.1, bits=64, commit=00000000, modified=0, pid=1, just started
   redis_1  | 1:C 17 Aug 22:11:10.480 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
   web_1    |  * Restarting with stat
   redis_1  | 1:M 17 Aug 22:11:10.483 * Running mode=standalone, port=6379.
   redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
   web_1    |  * Debugger is active!
   redis_1  | 1:M 17 Aug 22:11:10.483 # Server initialized
   redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
   web_1    |  * Debugger PIN: 330-787-903
   redis_1  | 1:M 17 Aug 22:11:10.483 * Ready to accept connections
   ```

   Compose извлекает образ Redis, создает образ для вашего кода и запускает службы, которые вы определили. В этом случае код статически копируется в образ во время сборки.

2. Перейдите на  http://localhost:8000/ в браузере, чтобы увидеть запущенное приложение.

Если это не помогло, вы также можете попробовать http://127.0.0.1:8000.

В вашем браузере должно появиться сообщение:

   ```text
   Hello World! I have been seen 1 times.
   ```

   ![hello world in browser](https://docs.docker.com/compose/images/quick-hello-world-1.png)

3. Обновите страницу.

Число должно увеличиваться.

   ```text
   Hello World! I have been seen 2 times.
   ```

   ![hello world in browser](https://docs.docker.com/compose/images/quick-hello-world-2.png)

4. Переключитесь в другое окно терминала и введите, docker image ls чтобы вывести список локальных образов.

Список изображений на этом этапе должен вернуть redis и web.

   ```console
   $ docker image ls

   REPOSITORY        TAG           IMAGE ID      CREATED        SIZE
   composetest_web   latest        e2c21aa48cc1  4 minutes ago  93.8MB
   python            3.4-alpine    84e6077c7ab6  7 days ago     82.5MB
   redis             alpine        9d8fa9aa0e5b  3 weeks ago    27.5MB
   ```

  Вы можете просматривать изображения с помощью docker inspect <tag or id>.

5. Остановите приложение, командой docker compose down из каталога проекта во втором терминале или нажав CTRL+C в исходном терминале, где вы запустили приложение.

## Шаг 4: Отредактируйте файл Compose для использования Compose Watch

Отредактируйте compose.yaml файл в каталоге вашего проекта, watch чтобы вы могли предварительно просмотреть работающие службы Compose, которые автоматически обновляются по мере редактирования и сохранения кода:

```yaml
services:
  web:
    build: .
    ports:
      - "8000:5000"
    develop:
      watch:
        - action: sync
          path: .
          target: /code
  redis:
    image: "redis:alpine"
```

Всякий раз, когда файл изменяется, Compose синхронизирует файл с соответствующим местоположением внутри /code контейнера. После копирования упаковщик обновляет работающее приложение без перезапуска.

## Шаг 5: Пересоберите и запустите приложение с помощью Compose

В каталоге проекта введите docker compose watch или , docker compose up --watch чтобы собрать и запустить приложение, а также включить режим просмотра файлов.

```console
$ docker compose watch
[+] Running 2/2
 ✔ Container docs-redis-1 Created                                                                                                                                                                                                        0.0s
 ✔ Container docs-web-1    Recreated                                                                                                                                                                                                      0.1s
Attaching to redis-1, web-1
         ⦿ watch enabled
...
```

Проверьте сообщение в веб-браузере еще раз и обновите страницу, чтобы увидеть увеличение счетчика.

## Шаг 6: Обновите приложение

1. Чтобы увидеть Compose Watch в действии:

Измените приветствие в app.py и сохраните его. Например, измените Hello World! сообщение на Hello from Docker!:

   ```python
   return f'Hello from Docker! I have been seen {count} times.\n'
   ```

2. Обновите приложение в браузере. Приветствие должно обновиться, а счетчик должен продолжать увеличиваться.

   ![hello world in browser](https://docs.docker.com/compose/images/quick-hello-world-3.png)

3. Как только вы закончите, запустите docker compose down.

## Шаг 7: Разделите сервисы

Использование нескольких файлов Compose позволяет настраивать приложение Compose для различных сред или рабочих процессов. Это полезно для больших приложений, которые могут использовать десятки контейнеров, с распределенным владением между несколькими командами.

1. В папке проекта создайте новый файл Compose с именем infra.yaml.

2. Вырежьте службу Redis из вашего compose.yaml файла и вставьте ее в новый infra.yaml файл. Убедитесь, что вы добавили services атрибут верхнего уровня в верхней части вашего файла. infra.yaml
   Теперь ваш файл должен выглядеть так:

   ```yaml
   services:
     redis:
       image: "redis:alpine"
   ```
3. В вашем compose.yaml файле добавьте атрибут include верхнего уровня вместе с путем к infra.yaml файлу.

   ```yaml
   include:
      - infra.yaml
   services:
     web:
       build: .
       ports:
         - "8000:5000"
       develop:
         watch:
           - action: sync
             path: .
             target: /code
   ```

4. Запустите docker compose up, чтобы построить приложение с обновленными файлами Compose. Вы должны увидеть сообщение Hello world в своем браузере.

Это упрощенный пример, но он демонстрирует базовый принцип include и то, как он может облегчить модуляризацию сложных приложений в подфайлы Compose.

## Шаг 8: Поэкспериментируйте с другими командами.

- Если вы хотите запустить свои службы в фоновом режиме, вы можете передать -d флаг (для «отсоединенного» режима) docker compose upи использовать docker compose ps для просмотра того, что в данный момент запущено:

   ```console
   $ docker compose up -d

   Starting composetest_redis_1...
   Starting composetest_web_1...

   $ docker compose ps

          Name                      Command               State           Ports         
   -------------------------------------------------------------------------------------
   composetest_redis_1   docker-entrypoint.sh redis ...   Up      6379/tcp              
   composetest_web_1     flask run                        Up      0.0.0.0:8000->5000/tcp
   ```

- Запустите docker compose --help, чтобы увидеть другие доступные команды.

- Если вы запустили Compose с помощью docker compose up -d, остановите свои службы, после завершения работы с ними:

   ```console
   $ docker compose stop
   ```

- Вы можете полностью уничтожить все, удалив контейнеры, с помощью команды docker compose down.

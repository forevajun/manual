# Docker

---

### Обозначения

``TAG`` - тэг  
``IMAGE_ID`` - ID образа  
``IMAGE_NAME`` - имя образа  
``CONTAINER_ID`` - ID контейнера  
``CONTAINER_NAME`` - имя контейнера (можно использовать везде вместо ID)  
``REPO`` - репозиторий  
``CMD`` - команда в bash

---

## Docker command

#### Версия Docker

- ``docker -v``

#### Вывести сведения о версиях клиента и сервера Docker

- ``docker version``

#### Залогиниться и разлогиниться в DockerHub

- ``docker login``
- ``docker logout``

#### Получить инфу о docker объекте

- ``docker inspect [DOCKER_OBJECT]``

#### Очистка ресурсов Docker

- ``docker system prune``

```bash
$ docker system prune -a --volumes # удалить неиспользуемые тома
```

---

## Образ (Image)

#### Получить образ из DockerHub

- ``docker pull [IMAGE_NAME]``
- ``docker image pull [IMAGE_NAME]``

```bash
$ docker pull ubuntu
$ docker pull alpine
```

#### Загрузить образ на DockerHub

- ``docker push [REPO]/[IMAGE_NAME]:[TAG]``
- ``docker image push [REPO]/[IMAGE_NAME]:[TAG]``

#### Посмотреть список образов (в т.ч. их размер)

- ``docker images``
- ``docker image ls``

> ``-q``, ``--quiet`` - вывести только ID

```bash
$ docker images -q
```

#### Удалить образ

- ``docker rmi [IMAGE_NAME]:[TAG]``
- ``docker image rm [IMAGE_NAME]:[TAG]``

> ``-f``, ``--force`` - принудительно удалить образ

#### Удалить все образы (и все их внутренние слои)

- ``docker rmi $(docker images -q) -f``

#### Создать новый образ из контейнера

- ``docker commit [CONTAINER_NAME] [REPO]/[IMAGE_NAME]:[TAG]``
- ``docker container commit [CONTAINER_NAME] [REPO]/[IMAGE_NAME]:[TAG]``

#### Сборка нового образа из Dockerfile

- ``docker build -t [IMAGE_NAME]:[TAG] [PATH]``
- ``docker image build -t [IMAGE_NAME]:[TAG] [PATH]``

> ``-t``, ``--tag`` - указать как будет называться образ  
> ``-f``, ``--file`` - имя Dockerfile (default ``[PATH]/Dockerfile``)

```bash
$ docker build -t forevajun/myImage:v1 .

$ docker run -d forevajun/myImage:v1
```

#### Промежуточные образы, из которых собран образ (в т.ч. их размер)

- ``docker history [IMAGE_NAME]``
- ``docker image history [IMAGE_NAME]``

#### Подробная инфа об образе (в том числе размер каждого слоя)

- ``docker inspect [IMAGE_NAME]:[TAG]``
- ``docker image inspect [IMAGE_NAME]:[TAG]``

---

## Контейнер (Container)

### Основные

#### Посмотреть список контейнеров

- ``docker ps``
- ``docker container ls``

> ``-a`` - все контейнеры (запущенные и остановленные)  
> ``-q``, ``--quiet`` - показать только ID  
> ``-s``, ``--size`` - размеры контейнеров  
> ``-f``, ``--filter`` - фильтр на основе условий

#### Удалить контейнер

- ``docker rm [CONTAINER_ID]``
- ``docker container rm [CONTAINER_ID]``

> ``-v``, ``--volumes`` - удалить анонимные тома, связанные с контейнером  
> ``-f``, ``--force`` - принудительное удаление запущенного контейнера

#### Удалить все контейнеры

- ``docker rm $(docker ps -aq)``

#### Удалить все остановленные контейнеры

- ``docker rm -v $(docker ps -aq -f status=exited)``

### Запуск контейнеров

#### Создать и запустить новый контейнер на основе образа (если образа нет локально, то он скачается)

- ``docker run -it [IMAGE_NAME] [CMD]``  
  ``docker run -d [IMAGE_NAME]``
- ``docker container run -it [IMAGE_NAME] [CMD]``

> ``-i``, ``--interactive`` - интерактивный режим. Благодаря этому флагу поток ``STDIN`` поддерживается в открытом
> состоянии даже если контейнер к ``STDIN`` не подключён  
> ``-t``, ``--tty`` - allocate a pseudo-TTY. Благодаря этому флагу выделяется псевдотерминал, который соединяет
> используемый терминал с потоками ``STDIN`` и ``STDOUT`` контейнера  
> ``-d``, ``--detach`` - запуск в фоне  
> ``-h``, ``--hostname`` - имя хоста контейнера  
> ``-e``, ``--env`` - установить переменные окружения (environment variables)  
> ``--name`` - задать имя контейнеру  
> ``-p``, ``--publish`` - проброс портов из контейнера на хост ``HOST_PORT:CONTAINER_PORT``  
> ``-P``, ``--publish-all`` - проброс всех expose портов  
> ``-rm`` - автоматически удаляет контейнер после того, как его выполнение завершится  
> ``--link`` - добавить link к другому контейнеру. Legacy feature of Docker!!!  
> ``-w``, ``--workdir`` - рабочий каталог внутри контейнера  
> ``-v``, ``--volume`` - смонтировать том ``HOST_PATH:CONTAINER_PATH``  
> ``--mount`` - монтирование файловой системы к контейнеру
>> ``type`` - тип монтирования  ``bind``, ``volume`` или ``tmpfs``.
> > ``tmpfs`` - временное хранилище информации в ОЗУ хоста. Это позволит ускорить чтение и запись данных  
> > ``source`` - источник монтирования. Для именованных томов это - имя тома.
> > Для неименованных томов этот ключ не указывают. Он может быть сокращён до ``src``  
> > ``destination`` - путь, к которому файл или папка монтируется в контейнере.
> > Этот ключ может быть сокращён до ``dst`` или ``target``  
> > ``readonly`` - монтирует том, который предназначен только для чтения.
> > Использовать этот ключ необязательно, значение ему не назначают

```bash
$ docker run -it ubuntu [bash / ls / pwd / echo ...]

$ docker run -it ubuntu bash # запуск контейнера и инициализация оболочки bash  
$ exit # остановить контейнер и выйти. Контейнер работает пока существует его основной процесс. Его ID=1 (в данном случае bash)  

$ docker run -d --name my_psql_container postgres
$ docker run -d -p 5432:5432 postgres # [HOST_PORT]:[CONTAINER_PORT]
$ docker run -d -v /home/userName/dev:/home/userNameContainer alpine # [HOST_PATH]:[CONTAINER_PATH]

# Docker создаст на хосте папку /doesnt/exist перед запуском контейнера
$ docker run -v /doesnt/exist:/foo -w /foo -it ubuntu bash  # [HOST_PATH]:[CONTAINER_PATH]
$ docker run --read-only -v /icanwrite busybox touch /icanwrite/here

$ docker run --mount source=my_volume, target=/container/path/for/volume my_image
$ docker run --mount type=volume,source=volume_name,destination=/path/in/container,readonly my_image

# Deprecated !!!
# Установка соединения между контейнерами
# СУБД mysql (mariadb) и php библиотека для управления СУБД (adminer)
$ docker run --name mysqlserver -e MYSQL_ROOT_PASSWORD=123456 -d mariadb
$ docker run --link mysqlserver:db -p 8080:8080 adminer
```

``docker run`` = ``docker create`` + ``docker start``

#### Создать новый контейнер

- ``docker create [IMAGE_NAME]``
- ``docker container create [IMAGE_NAME]``

#### Запустить контейнер

- ``docker start [CONTAINER_NAME]``
- ``docker container start [CONTAINER_NAME]``

#### Остановить контейнер

- ``docker stop [CONTAINER_NAME]``
- ``docker container stop [CONTAINER_NAME]``

#### Убить контейнер (Не пытается аккуратно завершить процесс, подобна системной команде kill)

- ``docker kill [CONTAINER_NAME]``
- ``docker container kill [CONTAINER_NAME]``

#### Подключиться к запущенному контейнеру к процессу с ID=1

- ``docker attach [CONTAINER_NAME]``
- ``docker container attach [CONTAINER_NAME]``

#### Выполнить команду внутри запущенного контейнера

- ``docker exec -it [CONTAINER_NAME] [CMD]``
- ``docker container exec [CONTAINER_NAME] [CMD]``

> ``-i``, ``--interactive`` - интерактивный режим  
> ``-t``, ``--tty`` - allocate a pseudo-TTY  
> ``-d``, ``--detach`` - запуск в фоне

```bash
$ docker exec -it my_psql_container bash
```

### Прочие команды

#### Подробная инфа о контейнере

- ``docker inspect [CONTAINER_NAME]``
- ``docker container inspect [CONTAINER_NAME]``

```bash
$ docker inspect ubuntu_container | grep IPAdress
```

#### Список файлов, измененных в контейнере

- ``docker diff [CONTAINER_NAME]``
- ``docker container diff [CONTAINER_NAME]``

#### Список всех событий, произошедших внутри контейнера

- ``docker logs [CONTAINER_NAME]``
- ``docker container logs [CONTAINER_NAME]``

---

## Том (Volume)

### Способы создать том

#### 1. В терминале

a) Создать том и смонтировать его в контейнер

```bash
$ docker volume create my-storage
my-storage
$ docker run -it -v my-storage:/data ubuntu /bin/bash
```

b) Смонтировать каталог хоста на каталог в контейнере

```bash
$ docker run -it -v /host/folder/:/data ubuntu /bin/bash
```

c) Анонимный том

```bash
$ docker run -it --name my-container -v /data ubuntu /bin/bash # каталог /data внутри контейнера
$ docker inspect -f {{.Mounts}} my-container # месторасположение данного тома в системе хоста
```

#### 2. В Dockerfile

```
VOLUME /data
```

#### 3. В docker-compose.yml

```yaml
volumes:
  - ./databases:/var/lib/mysql
```

### Команды

#### Создать том

- ``docker volume create [OPTIONS] [VOLUME]``

```bash
$ docker volume create my-storage
my-storage
$ docker run -it -v my-storage:/data ubuntu /bin/bash

$ docker run -it -v /host/folder/:/data ubuntu /bin/bash
$ docker run -it -v /anonymous ubuntu /bin/bash
```

#### Список томов

- ``docker volume ls [OPTIONS]``

#### Удалить том

- ``docker volume rm [OPTIONS] VOLUME [VOLUME...]``

> ``-f``, ``--force`` - force the removal

```bash
$ docker volume rm hello
hello
```

#### Удалить все тома, которые не используются контейнерами

- ``docker volume prune [OPTIONS]``

> ``-f``, ``--force`` - не запрашивать подтверждение  
> ``--filter`` - фильтр 'label=<label>'

```bash
# если не получается удалить том
$ docker system prune
$ docker volume prune
hello
```

#### Посмотреть подробную инфу о томе

- ``docker volume inspect [OPTIONS] VOLUME [VOLUME...]``

> ``-f``, ``--format`` - форматированный вывод

```bash
$ docker volume inspect my-storage
[
  {
    "CreatedAt": "2020-04-19T11:00:21Z",
    "Driver": "local",
    "Labels": {},
    "Mountpoint": "/var/lib/docker/volumes/my-storage/_data",
    "Name": "my-storage",
    "Options": {},
    "Scope": "local"
  }
]

$ docker volume inspect --format '{{ .Mountpoint }}' my-storage
/var/lib/docker/volumes/my-storage/_data
```

#### Дополнительная инфа о volume

Linux тома находятся по умолчанию в ``/var/lib/docker/volumes/``  
Tmpfs — временное файловое хранилище. Это некая специально отведённая область в оперативной памяти компьютера.

Тома (как образы и контейнеры) ограничены значением настройки dm.basesize.  
Она устанавливается на уровне настроек демона Docker. Как правило, что-то около 10Gb.    
Это значение можно изменить вручную, но потребуется перезапуск демона Docker.

```bash
$ sudo dockerd --storage-opt dm.basesize=40G
```

Однажды увеличив значение, его уже нельзя просто так уменьшить. При запуске Docker выдаст ошибку.

Если вам нужно вручную очистить содержимое всех томов, придётся удалять каталог, предварительно остановив демон

```bash
$ sudo service docker stop
$ sudo rm -rf /var/lib/docker
```

### Volume практика

```bash
$ docker volume create my-storage
my-storage

$ docker volume ls
DRIVER  VOLUME NAME
local   my-storage

$ docker inspect my-storage
[
  {
    "CreatedAt": "2020-12-14T15:00:37Z",
    "Driver": "local",
    "Labels": {},
    "Mountpoint": "/var/lib/docker/volumes/my-storage/_data",
    "Name": "my-storage",
    "Options": {},
    "Scope": "local"
  }
]

$ docker run --rm -v my-storage:/data -it ubuntu:20.10 /bin/bash
$ echo $RANDOM > /data/file.txt
$ cat /data/file.txt
13279
$ exit

# После самоуничтожения контейнера запустим другой и подключим к нему тот же том. Проверяем, что в нашем файле file.txt
$ docker run --rm -v my-storage:/data -it centos:8 /bin/bash -c "cat /data/file.txt"
13279

# Теперь примонтируем каталог с хоста
$ docker run -v /srv:/host/srv --name slurm --rm -it ubuntu:20.10 /bin/bash

# Теперь попробуем совместить оба типа томов сразу
$ docker run -v /srv:/host/srv -v my-storage:/data --name slurm --rm -it ubuntu:20.10 /bin/bash

# Передадим ровно те же тома другому контейнеру
$ docker run --volumes-from slurm --name backup --rm -it centos:8 /bin/bash

# Создавать том заранее необязательно, всё сработает в момент запуска docker run
$ docker run -v new-storage:/newdata -v /srv:/host/srv -v my-storage:/data --name slurm --rm -it ubuntu:20.10 /bin/bash

$ docker volume ls
DRIVER  VOLUME NAME
local   my-storage
local   new-storage

# Создадим анонимный том
$ docker run -v /anonymous -v new-storage:/newdata -v /srv:/host/srv -v my-storage:/data --name slurm --rm -it ubuntu:20.10 /bin/bash
# Такой том самоуничтожится после выхода из контейнера, так как мы указали ключ --rm

# Если ключ --rm не указать
$ docker run -v /anonymous -v new-storage:/newdata -v /srv:/host/srv -v my-storage:/data --name slurm -it ubuntu:20.10 /bin/bash
$ docker volume ls
DRIVER  VOLUME NAME
local   04c490b16184bf71015f7714b423a517ce9599e9360af07421ceb54ab96bd333
local   new-storage
local   my-storage
```


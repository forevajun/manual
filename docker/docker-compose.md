# Docker-compose

> В файле не должно быть табов!

```yaml
version: '3.1'
services:
  myapp:
    build: ./app
    volumes:
      - .:/usr/src/app
    ports:
      - "8080:8080"
    depends_on:
      - db
  db:
    image: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123456
  adminer:
    image: adminer
    restart: always
    ports:
      - 8081:8080
```

## Терминал

- ``docker compose -v``  
- ``docker machine ip default`` - IP адрес виртуальной машины, на которой работает Docker

В папке с файлом ``docker-compose.yaml`` / ``docker-compose.yml`` запустить

- ``docker compose build`` - собрать сервисы из ``docker-compose.yml`` (аналог ``docker build``)  
- ``docker compose up`` - создать и запустить контейнеры ``docker-compose.yml``  
> ``-d``, ``--detach`` - запуск в фоне  
> ``--build`` - собрать образы перед запуском контейнеров  
> ``--scale`` - запустить несколько инстансов сервиса
```bash
$ docker compose up --scale myapp=5 # запустить 5 контейнеров myapp
```

- ``docker compose run`` - запуск с параметрами  
- ``docker compose start`` - запустить остановленных контейнеров (запустятся в фоне)  
- ``docker compose stop`` - остановить контейнеры  
- ``docker compose images`` - список образов, которые используются в текущем ``docker-compose.yml``  
- ``docker compose ps`` - запущенные контейнеры  
- ``docker compose top`` - список запущеных процессов  
- ``docker compose logs`` - логи в контейнерах  
- ``docker compose down`` - останавливает и удаляет все контейнеры, относящиеся к текущему ``docker-compose.yml``   
- ``docker compose rm [SERVICE_NAME]`` - удаление контейнера

```bash
$ docker compose rm db
$ docker compose rm adminer
```

---

## docker-compose.yml

``version`` - версия docker-compose  
``services`` - образы, которые будут подключаться  
``myapp``, ``db``, ``[SERVICE_NAME]`` - названия сервисов  
``image`` - образ, из которого будет создан сервис  
``build`` - сообщает docker-compose, что образ для данного контейнера должен быть создан из Dockerfile  
``restart`` - ``no``, ``on-failure``, ``restart``, default ``no``  
``environment`` - переменные окружения  
``ports`` - проброс портов из контейнера наружу, ``[HOST_PORT]:[CONTAINER_PORT]``  
``volumes`` - том, каталоги в файловой системе хоста  
``working_dir`` - аналог ``WORKDIR`` в Dockerfile  
``command`` - аналог ``CMD`` в Dockerfile, команда, которая выполняется каждый раз при запуске сервиса  
``container_name`` - имя контейнера  
``depends_on [SERVICE_NAME]`` - указывает сервисы, для которых надо дождаться их запуска, прежде чем запсукать текущий  
``links [SERVICE_NAME]`` - Deprecated !!! Коннект сервисов между собой. Также определяют зависимость между сервисами так
же, как и ``depend_on``. Поэтому они определяют порядок запуска сервисов



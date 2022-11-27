# Dockerfile

---

#### Сборка нового образа из Dockerfile.

- ``docker build -t [IMAGE_NAME]:[TAG] [PATH]``
- ``docker image build -t [IMAGE_NAME]:[TAG] [PATH]``

> ``-t``, ``--tag`` - указать как будет называться образ  
> ``-f``, ``--file`` - имя Dockerfile (default ``[PATH]/Dockerfile``)

```bash
$ docker build -t forevajun/myImage:v1 .

$ docker run -d forevajun/myImage:v1
```

---

## .dockerignore

- ``# comment`` - комментарий
- ``*/temp*`` - исключить файлы и папки, имя которых начинается с ``temp`` в любом непосредственном подкаталоге корня. Например файл ``/somedir/temporary.txt`` или папка ``/somedir/temp``  
- ``*/*/temp*`` - исключить файлы и папки, начинающиеся с ``temp`` из любого подкаталога, который находится на два уровня ниже корня. Например ``/somedir/subdir/temporary.txt``  
- ``temp?`` - исключить файлы и папки в корневом каталоге, имена которых являются односимвольным расширением ``temp``. Например ``/tempa`` и ``/tempb``.  
- ``**`` - исключить всё, что соответствует любому количеству каталогов (включая ноль). Например ``**/*.go`` исключит все файлы, оканчивающиеся на ``.go``, которые находятся во всех каталогах, включая корень контекста сборки.  
- ``!`` - строки, начинающиеся с ``!``, могут использоваться для создания исключений из исключений

---

## Инструкции

### FROM

Образ, из которого будет создан новый образ

- ``FROM [IMAGE_NAME]:[TAG]``

```
FROM alpine
```

### RUN

Выполнить команды в новом слое текущего образа

- ``RUN [CMD]``
- ``RUN ["CMD", "param1", "param2"]``

Каждый вызов RUN создает новый слой, поэтому лучше писать через ``&&``

```
RUN apt-get update \
	&& apt-get install -y ApplicationName

RUN ["/bin/bash", "-c", "echo hello"]
```

``-y`` - подавление запросов подтверждения действия

### CMD

Команда, запускающаяся в контейрене по умолчанию  
Либо предоставляет параметры для команды, указанной в ``ENTRYPOINT``  
Задает параметры по умолчанию, которые могут быть переопределены при запуске контейнера

- ``CMD ["executable","param1","param2"]`` - (exec form, this is the preferred form)
- ``CMD ["param1","param2"]`` - (as default parameters to ENTRYPOINT)
- ``CMD command param1 param2`` - (shell form)

```
CMD http-server

CMD ["java", "-jar", "/services/app.jar"]
```

### ENTRYPOINT

Определить исполняемый файл при запуске ``docker run``  
Задает параметры по умолчанию, которые нельзя переопределить

- ``ENTRYPOINT ["executable", "param1", "param2"]``
- ``ENTRYPOINT command param1 param2``

```
ENTRYPOINT ["java"]

FROM ubuntu
ENTRYPOINT ["top", "-b"]
CMD ["-c"]
```

### LABEL

Метаданные об образе  
Например - сведения о том, кто создал и поддерживает образ

- ``LABEL <key>=<value> <key>=<value> <key>=<value> ...``

```
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```

### MAINTAINER (deprecated)

Информация об авторе данного образа

- ``MAINTAINER <name>``

```
MAINTAINER Ivan Ivanov <ivanov@email.com>
```

### EXPOSE

Указывает какие порты необходимо пробросить  
Данная инструкция просто информирует Docker, о том что данный контейнер слушает определенный порт, ничего не публикует

- ``EXPOSE <port> [<port>/<protocol>...]``

Default protocol = tcp

```
EXPOSE 8080

EXPOSE 80/udp
EXPOSE 80/tcp
```

### ENV

Устанавливает постоянные переменные среды (environment variable)

- ``ENV <key>=<value> ...``

```
ENV MY_NAME="John Doe"
ENV MY_DOG=Rex\ The\ Dog
ENV MY_CAT=fluffy
```

### ADD

Копирует файлы, папки и удаленные файлы (по URL) из ``<src>`` в файловую систему образа в ``<dest>``

- ``ADD [--chown=<user>:<group>] [--checksum=<checksum>] <src>... <dest>``
- ``ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]``

``<dest>`` - абсолютный путь или относительно ``WORKDIR``

```
ADD hom* /mydir/

ADD hom?.txt /mydir/

# <WORKDIR>/relativeDir/
ADD test.txt relativeDir/

# /absoluteDir/
ADD test.txt /absoluteDir/
```

### COPY

Копирует файлы и папки с хоста в контейнер

- ``COPY [--chown=<user>:<group>] <src>... <dest>``
- ``COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]``

```
# Копирование из директории с Dockerfile в /home/server/
# / в конце обязательно
COPY ./ /home/server/

COPY hom* /mydir/

COPY hom?.txt /mydir/

# <WORKDIR>/relativeDir/
COPY test.txt relativeDir/

# /absoluteDir/
COPY test.txt /absoluteDir/
```

### VOLUME

Создаёт точку монтирования для работы с постоянным хранилищем (томом)

- ``VOLUME ["/data"]``

```
VOLUME /home/server

VOLUME ["/var/log/", "/var/db"]
VOLUME /var/log /var/db

FROM ubuntu
RUN mkdir /myvol
RUN echo "hello world" > /myvol/greeting
VOLUME /myvol
```

### USER

Создает пользователя

- ``USER <user>[:<group>]``

```
FROM microsoft/windowsservercore
# Create Windows user in the container
RUN net user /add patrick
# Set it for subsequent commands
USER patrick
```

### WORKDIR

Задает рабочую директорию для следующей инструкции ``RUN``, ``CMD``, ``ENTRYPOINT``, ``COPY`` и ``ADD``

- ``WORKDIR /path/to/workdir``

По умолчанию ``/``

```
WORKDIR /home/server

# pwd -> /a/b/c
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd

# pwd -> /path/$DIRNAME
ENV DIRPATH=/path
WORKDIR $DIRPATH/$DIRNAME
RUN pwd
```

### ARG

Задает переменные для передачи во время сборки образа ``docker build``, используя флаг ``--build-arg <varname>=<value>``

- ``ARG <name>[=<default value>]``

```
FROM ubuntu
ARG CONT_IMG_VER
ENV CONT_IMG_VER=v1.0.0
RUN echo $CONT_IMG_VER

$ docker build --build-arg CONT_IMG_VER=v2.0.1 .
```

---

## Examples

```
FROM alpine
RUN apk add npm
RUN npm i -g http-server

FROM alpine
RUN apk add npm \
  && npm i -g http-server  
```

```
FROM ubuntu
RUN apt-get update \
  && apt-get install -y ApplicationName
```

```
FROM openjdk:8-jre-alpine
COPY target/Application-1.0-SNAPSHOT.jar /services/app.jar
EXPOSE 8080
CMD ["java", "-jar", "/services/app.jar"]
```

```
FROM alpine
RUN apk add npm \ 
  && npm i -g http-server
VOLUME /home/server
WORKDIR /home/server
COPY ./ /home/server/
EXPOSE 8080
CMD http-server
```

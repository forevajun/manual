# Network

#### Создать новую сеть

- ``docker network create [OPTIONS] NETWORK``

> ``-d``, ``--driver`` - указать драйвер сети

Встроенные драйверы ``bridge`` (если сервисы на одном хосте), ``overlay`` (для
нескольких хостов в Swarm) или создать пользовательский. Если не указать, то по умолчанию будет ``bridge``  
При запуске контейнера ``docker run`` он автоматически коннектится к сети ``bridge``. Нельзя удалить эту сеть по
умолчанию, но можно создать новую сеть.

#### Подключить контейнер к сети

- ``docker network connect [OPTIONS] NETWORK CONTAINER``

#### Отключить контейнер от сети

- ``docker network disconnect [OPTIONS] NETWORK CONTAINER``

> ``-f``, ``--force`` - принудительное отключение контейнера от сети

#### Список сетей

- ``docker network ls [OPTIONS]``

#### Удалить одну или несколько сетей

- ``docker network rm NETWORK [NETWORK...]``

#### Удалить все неиспользуемые сети

- ``docker network prune [OPTIONS]``

> ``-filter`` - фильтр (``until=[timestamp]``)  
> ``-f``, ``--force`` - не запрашивать подтверждение

#### Отобразить подробную информацию о сети

- ``docker network inspect [OPTIONS] NETWORK [NETWORK...]``

```bash
$ docker network create -d bridge my-network # Create network
$ docker run -itd --network=my-network my-app1 # Connect container

$ docker run -itd my-app2
$ docker network connect my-network my-app2 # Connect running container

$ docker network disconnect my-app1 # Disconnect container
$ docker network disconnect my-app2 # Disconnect container
```

# Links

> The ``--link`` flag is a legacy feature of Docker.
> It may eventually be removed. Unless you absolutely need to continue using it
> We recommend that you use ``networks`` to connection between two containers instead of using ``--link``

- ``--link <name or id>:alias``
- ``--link <name or id>``

Где ``name`` это имя контейнера, к которому мы коннектимся, а ``alias`` это псевдоним для ссылки

```bash
$ docker run -d --name db training/postgres
$ docker run -d -P --name web --link db:db training/webapp python app.py
# or
$ docker run -d -P --name web --link db training/webapp python app.py
```

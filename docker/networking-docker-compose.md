# Networks

> Networking features are not supported for Compose file version 1 (deprecated). Required version 2 and higher
>
> Your app’s network is given a name based on the “project name”, which is based on the name of the directory it lives
> in.
> You can override the project name with either the ``--project-name`` flag or the ``COMPOSE_PROJECT_NAME`` environment
> variable

```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres
    ports:
      - "8001:5432"
```

При запуске ``docker compose up``

1. Создается сеть ``myapp_default``
2. Создается контейнер ``web`` и присоединяется к сети ``myapp_default``
3. Создается контейнер ``db`` и присоединяется к сети ``myapp_default``

Контейнеры имеет hostname ``web`` и ``db``.
Например ``web`` сервис может коннектиться к URL ``postgres://db:5432``  
Различия между ``HOST_PORT`` и ``CONTAINER_PORT``:  
``db`` ``HOST_PORT = 8001``  
``db`` ``CONTAINER_PORT = 5432``  
Networked service-to-service communication uses the ``CONTAINER_PORT``  
Когда ``HOST_PORT`` указан, сервис доступен снаружи.  
Сервис ``web`` подключается к ``db`` по ``postgres://db:5432``  
А с хоста подключаться по ``postgres://{DOCKER_IP}:8001``

#### Specify custom networks

Можно указать свои собственные сети с помощью ключа ``networks`` на верхнем уровне.  
Это позволяет создавать более сложные топологии и задавать пользовательские сетевые драйверы и параметры.

В примере, определяются две сети ``networks ``  
Сервис ``proxy`` изолирована от сервиса ``db``, т.к. у них нет общей сети  
Только ``app`` может общаться с обоими

```yaml
version: "3.9"

services:
  proxy:
    build: ./proxy
    networks:
      - frontend
  app:
    build: ./app
    networks:
      - frontend
      - backend
  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
    # Use a custom driver
    driver: custom-driver-1
  backend:
    # Use a custom driver which takes special options
    driver: custom-driver-2
    driver_opts:
      foo: "1"
      bar: "2"
```

#### Configure the default network

```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres

networks:
  default:
    # Use a custom driver
    driver: custom-driver-1
```

#### network_mode

Зависимость между сервисами

- ``docker-compose up`` - запускает сервисы в порядке зависимости. ``db`` и ``redis`` запускаются до ``web``
- ``docker-compose up SERVICE`` - ``docker-compose up web`` также запускает ``db`` и ``redis`` до ``web``
- ``docker-compose stop`` - останавливает сервисы в порядке зависимости. ``web`` останавливается до ``db`` и ``redis``

```yaml
version: "3.9"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

``depend_on`` не ждет, пока ``db`` и ``redis`` будут «готовы» перед запуском ``web``, только когда они запустятся

# Links

> Links are a legacy option. We recommend using networks instead

Укажите имя сервиса и псевдоним ссылки ``SERVICE:ALIAS``, или только имя сервиса.

```yaml
version: "3.9"
services:

  web:
    build: .
    links:
      - "db:database"
  db:
    image: postgres
```

```yaml
web:
  links:
    - "db"
    - "db:database"
    - "redis"
```

Ссылки позволяют определить дополнительные псевдонимы, по которым сервис доступен из другого сервиса.
Они не требуются для того, чтобы сервисы могли обмениваться данными - по умолчанию любой сервис может связаться с любым
другим сервисом по имени этого сервиса.
В примере ``db`` доступен из ``web`` через hostname ``db`` и ``database``

Ссылки ``links`` также определяют зависимость между сервисами так же, как и ``depend_on``. Поэтому они определяют
порядок запуска сервисов.


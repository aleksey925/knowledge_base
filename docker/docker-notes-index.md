Заметки о работе с Docker
=============================

# Оглавление

1. <h3>[Введение](#Введение)</h3>

    - [Общие сведения](#Общие-сведения)

2. <h3>[Запуск сервисов необходимых для разработки](#Запуск-сервисов-необходимых-для-разработки)</h3>
    
    - [Запуск PostgreSQL](#Запуск-PostgreSQL)
    - [Запуск RabbitMQ](#Запуск-RabbitMQ)

  
<a name='Введение'></a>
## Введение


<a name='Общие-сведения'></a>
### Общие сведения

- [Понимая Docker | habr](https://habr.com/ru/post/253877/) - общая теория
- [Основы Docker за Х часов и Y дней | habr](https://habr.com/ru/post/337306/) - 
хорошо описана теория с простыми примерами
- [Quickstart: Compose and Django](https://docs.docker.com/compose/django/) -
простой и наглядный пример развертывания django и postgresql при помощи 
docker compose


<a name='Запуск-сервисов-необходимых-для-разработки'></a>
## Запуск сервисов необходимых для разработки


<a name='Запуск-PostgreSQL'></a>
### Запуск PostgreSQL

**Создание дирректории, которая будет хранить файлы PostgreSQL**

Если мы хотим иметь возможность продолжить работу с созданными в контейнере 
базами данных после перезапуска контейнера, то небходимо создать папку, которая 
будет хранить все данные созданные postgres.

```bash
mkdir -p $HOME/docker/volumes/postgres
```

**Запуск Postgres контейнера**

Запустить контейнер postgres, так же просто как выполнить команду `docker run`

```bash
docker run --rm --name pg-docker -e POSTGRES_USER=alex -e POSTGRES_PASSWORD=123 -e POSTGRES_DB=test -d -p 5432:5432 -v $HOME/docker/volumes/postgres:/var/lib/postgresql/data postgres
```

В команде выше использовано достаточно много аргументов, давайте разберем их:

- rm: Зставляет докер выполнять удаление контейнера и связанной с ним файловой
системы при выходе. В общем, если есть необходимость запускать много 
контейнеров, которые не должны жить долго, то считается хорошей практикой 
передавать флаг rm команде `docker run` для автоматической очистки дискового 
пространства.
Стоить заметить, что мы всегда можем использовать ключ -v для сохрнения данных
за пределами контейнера.

- name: Уникальное имя контейнера. Мы можем выбирать любые имена, главное они 
не должны повторяться, потому что в системе не может быть 2 контейнера с 
одинаковым именем. Для повтороного использования имени необходимо либо 
передавать флаг rm команде docker run, либо явно удалить контейнер 
`docker rm <container name>`.

- e: Объявляет переменную окружения в docker контейнере. POSTGRES_PASSWORD - 
задает пароль суперпользователя для PostgreSQL. POSTGRES_USER - задает имя
суперпользователя для PostgreSQL. POSTGRES_DB - задает имя базы данных, которая
будет создана.

- d: Указывает, что контейнер должен быть запущен в detached mode (в фоновом 
режиме).

-p: Привязывает порт 5432 с локальной машины, на порт 5432 в docker контейнере.
Этот параметр позволяет приложениям, работающим вне контейнера, подключаться к 
серверу postgres, работающему внутри контейнера.

-v: Монтирует папку $HOME/docker/volumes/postgres расположенную на локальной 
машине, к папке /var/lib/postgresql/ располагающейся внутри контейнера. Это 
гарантирует сохранение данных postgres даже после удаления контейнера.


**Подключение к Postgres**

Теперь, после того как контейнер запущен, подключение к postgres ни чем не 
отличается от подключения к экземпляру postgres расположеному вне docker 
контейнера. Для примера подключение через psql будет выглядеть следующим 
образом:

```bash
psql -h localhost -U alex -d test
``` 

Ссылки:

- [Don’t install Postgres. Docker pull Postgres](https://hackernoon.com/dont-install-postgres-docker-pull-postgres-bee20e200198) - на
основе данной статьи сделана данная заметка
- [How to create User/Database in script for Docker Postgres](https://stackoverflow.com/questions/26598738/how-to-create-user-database-in-script-for-docker-postgres) -
ответ на то как создать пользователя и базу данных в контейнере c postgres
- [Docker контейнер с данными на Postgres для интеграционного тестирования и лёгким расширением | habr](https://habr.com/ru/post/328226/)


<a name='Запуск-RabbitMQ'></a>
### Запуск RabbitMQ

По сути, запуск RabbitMQ аналогичен запуску PostgreSQL, по этому приведу только
основные команды необходимые для запуска RabbitMQ.

**Создание контейнера с RabbitMQ**

```bash
docker run -d \
           --name rabbit \
           -p "4369:4369" \
           -p "5672:5672" \
           -p "15672:15672" \
           -p "25672:25672" \
           -p "35197:35197" \
           -e "RABBITMQ_USE_LONGNAME=true" \
           -e "RABBITMQ_LOGS=/var/log/rabbitmq/rabbit.log" \
           -v $HOME/docker/volumes/rabbit:/var/lib/rabbitmq \
           -v $HOME/docker/volumes/rabbit/logs:/var/log/rabbitmq \
           rabbitmq:3.7.11-management
```

После выполнения данной команды будет создан и запущен контейнер содержащий 
RabbitMQ. 
 
Интерфейс управления доступен по ссылке [ссылке](http://127.0.0.1:15672).

**Остановка контейнера**

```bash
docker stop rabbit
```

**Запуск контейнера после перезагрузки или остановки контейнера**

```bash
docker container start rabbit
```


Полезные ссылки:

- [Настройка кластера RabbitMQ на Docker](https://thewebland.net/development/devops/rabbitmq/nastrojka-klastera-rabbitmq-na-docker/)
- [Microservices & RabbitMQ On Docker](https://dev.to/usamaashraf/microservices--rabbitmq-on-docker-e2f)
- [Docker Hub - rabbitmq](https://hub.docker.com/_/rabbitmq)
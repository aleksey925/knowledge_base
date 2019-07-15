Заметки о работе с Docker
=============================

# Оглавление

1. <h3>[Введение](#Введение)</h3>

    - [Общие сведения](#Общие-сведения)
    - [Полезные команды](#Полезные-команды)
    - [Пользователь в Docker](#Пользователь-в-Docker)
    - [Как устроен docker](#Как-устроен-docker)
    - [Устройство докер образов](#Устройство-докер-образов)

2. <h3>[Запуск сервисов необходимых для разработки](#Запуск-сервисов-необходимых-для-разработки)</h3>
    
    - [Запуск PostgreSQL](#Запуск-PostgreSQL)
    - [Запуск RabbitMQ](#Запуск-RabbitMQ)
    
3. <h3>[Ведение разработки вместе с docker](#Ведение-разработки-вместе-с-docker)</h3>

    - [Запуск простого web-приложения в образе созданном из Dockerfile](#Запуск-простого-web-приложения-в-образе-созданном-из-Dockerfile)
  

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


<a name='Полезные-команды'></a>
### Полезные команды

**Удаление контейнеров/образов**

- `docker rm -vf $(docker ps -a -q)` - удаляет все контейнеры
- `docker rmi -f $(docker images -a -q)` - удаляет все образы
- `docker system prune` - удаляет все не используемые containers, networks, 
  images и опционально volumes
- `docker system prune -a` - удаляет все containers, networks, images и 
  опционально volumes


<a name='Пользователь-в-Docker'></a>
### Пользователь в Docker

Все процессы в контейнере будут работать из-под пользователя root, если 
специальным образом его не указать. Это кажется очень удобно, ведь у этого 
пользователя нет никаких ограничений. Именно поэтому работать под рутом 
неправильно с точки зрения безопасности. Если на локальном компьютере никто 
в здравом уме не работает с рутовыми правами, то многие запускают процессы 
под рутом в контейнерах.

Всегда есть баги, которые позволят зловреду выбраться из контейнера и попасть 
на хостовый компьютер. Предполагая худшее, мы должны обеспечить запуск 
процессов внутри контейнера от пользователя, который не имеет никаких прав 
на хостовой машине.

Для того, чтобы создать пользователя в дистрибутиве alpine необходимо выполнить:

```
RUN addgroup app_user && adduser -G app_user -s /bin/sh -D app_user
```

В debian подобных дистрибутивах это можно сделать так:

```
RUN groupadd app_user && \
    useradd --gid app_user --shell /bin/bash --create-home app_user
```

После этого, чтобы сменить текущего пользователя необходимо прописать в 
Dockerfile 
```
USER app_user
```

После этого, все запускаемые процессы будут выпоняться от имени `app_user`.

Использованные материалы:

- [Пользователь в Docker](https://habr.com/ru/post/448480/)


<a name='Как-устроен-docker'></a>
### Как устроен docker

[“Проникновение в Docker с примерами”, Дмитрий Столяров, Flant](https://youtu.be/hdVNKmru3LM)


<a name='Устройство-докер-образов'></a>
### Устройство докер образов

[Digging into Docker layers](https://medium.com/@jessgreb01/digging-into-docker-layers-c22f948ed612)



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
docker run -d \
           --rm \
           --name pg \
           -e "POSTGRES_USER=alex" \
           -e "POSTGRES_PASSWORD=123" \
           -e "POSTGRES_DB=test" \
           -p "5432:5432" \
           -v $HOME/docker/volumes/postgres:/var/lib/postgresql/data \
           postgres
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

- p: Привязывает порт 5432 с локальной машины, на порт 5432 в docker контейнере.
Этот параметр позволяет приложениям, работающим вне контейнера, подключаться к 
серверу postgres, работающему внутри контейнера.

- v: Монтирует папку $HOME/docker/volumes/postgres расположенную на локальной 
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



<a name='Ведение-разработки-вместе-с-docker'></a>
## Ведение разработки вместе с docker


<a name='Запуск-простого-web-приложения-в-образе-созданном-из-Dockerfile'></a>
### Запуск простого web-приложения в образе созданном из Dockerfile

Для того, чтобы показать базовые принципы создания нового docker образа, 
необходимо создать проект с элементарным web приложением:

app.py

```python
from flask import Flask

app = Flask(__name__)


@app.route('/')
def hello_world():
    return 'Hello, World!'


app.run(host='0.0.0.0', port=5000, debug=True)
```

requirements.txt

```
flask==1.0.2
```

Теперь, когда проект создан можно заняться его докеризацией. Первым делом 
необходимо создать файл `Dockerfile`, который будет содержать инструкции
необходимые для сборки docker образа.

Dockerfile

```dockerfile
FROM python:3.7

# Открывает у контейнера порт 5000 и делает его доступным извне
EXPOSE 5000

# Установка необходимых пакетов
RUN apt-get update && \
	apt-get install -y supervisor && \
	apt-get install -y net-tools && \
	apt-get install -y curl && \
	apt-get install -y apt-utils && \
	apt-get install -y apt-transport-https && \
	apt-get install -y debconf-utils && \
	apt-get install -y gcc && \
	apt-get install -y build-essential && \
	apt-get install -y g++

# Установка необходимых locales
RUN apt-get update && apt-get install -y locales \
    && echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
    && locale-gen

COPY app.py requirements.txt /opt/web_app/
WORKDIR /opt/web_app/
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

Собираем образ

```bash
docker build -t simple-web-app .
```

Теперь убеждаемся, что образ успешно создался 

```bash
docker images
```

```
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
simple-web-app      latest              39e2e3bc092c        About a minute ago   982MB
python              3.7                 ac069ebfe1e1        42 hours ago         927MB
```

Как видим из вывода команды, образ был создан и имеет ID `39e2e3bc092c`.
Теперь, мы можем запустить его. Сделать это можно при помощи команды:

```bash
docker run -p 5000:5000 simple-web-app
```

Данная команда запускает образ simple-web-app и связывает 5000 порт локальной 
машины с 5000 портом контейнера, который будет создан в результате запуска 
образа. Если необходимо запустить контейнер в фоне, нужно добавить ключ `-d`. 

Если контейнер был запущен в фоне, то для того, чтобы убедиться в его успешном
запуске можно выполнить команду

```bash
docker ps
```

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
0adf5f5a5077        simple-web-app      "python app.py"     5 seconds ago       Up 2 seconds        0.0.0.0:5000->5000/tcp   objective_montalcini
```

которая покажет все запущенные контейнеры. Если контейнер запущен, можно 
посмотреть его логи при помощи:

```bash
docker logs -f 0adf5f5a5077
```

Вместо 0adf5f5a5077 нужно подставить id контейнера из вывода команды 
`docker ps`.

После того как мы убедились, что контейнер с нашим приложением успешно запущен,
можно открыть наше приложение в [браузере](http://0.0.0.0:5000).

После окончания работы контейнер с приложением можно будет остановить командой

```bash
docker stop 0adf5f5a5077
```

А если контейнер и образ более не нужны, то их можно удалить следующими 
командами

```bash
# Удаление контейнера
docker rm 0adf5f5a5077
# Удаление образа
docker rmi 39e2e3bc092c
```

Полезные ссылки:

- [Docker: build и пример Dockerfile](https://rtfm.co.ua/docker-build/) - заметка
  основана на данной статье
- [Погружаемся в Docker: Dockerfile и коммуникация между контейнерами](https://habr.com/ru/company/infobox/blog/240623/)
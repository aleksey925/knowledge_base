Tutorial alembic
================

alembic - это легковесный инструмент для создания и управления миграциями БД

## Добавление в проект миграций БД

Данный туториал описывает процесс добавления в уже существующий проект миграций для БД.

### Установка

Первым делом необходимо установить библиотеку alembic

```bash
pip3 install alembic==1.8.1
```

### Настройка проекта

Инициализируем alembic в проекте:

```bash
alembic init --template generic migrations
```

В создавшемся файле `alembic.ini` раскомментируем строку `file_template`, чтобы alembic 
генерировал понятные имена для создаваемых миграций.

Теперь, после базовой настройки, необходимо немного подправить сгенерированный 
скрипт env.py. Это нужно, чтобы научить alembic брать URI для подключения к БД 
не из `alembic.ini`, а из глобального для всего проекта конфига (который обычно 
является обычным python модулем), а так же включить поддержку автогенерации миграций 
и т д. Для этого открываем скрипт `migrations/env.py` и:

- импортируем из конфига URI БД и базовый класс `Base` из которого мы сможем 
получить объект MetaData

    ```python
    from config import SQLALCHEMY_DATABASE_URI
    from models import Base
    ```

- находим строку `fileConfig(config.config_file_name)` и заменяем ее на
  `fileConfig(config.config_file_name, disable_existing_loggers=False)`. Если
   это не сделать alembic отключит все существующие логгеры. В случае если 
   alembic вызывается исключительно как консольная команда, то это не страшно,
   но если работа с миграциями будет осуществляться из кода приложения, 
   то дефотное поведение alembic может доставить много хлопот.

- находим строку `target_metadata = None` и заменяем на:

    ```python
    target_metadata = Base.metadata
    ```

- где-то после объявления `target_metadata` вставляем следующую строку

    ```python
    config.set_main_option('sqlalchemy.url', SQLALCHEMY_DATABASE_URI)
    ```
  
    Это нужно, чтобы alembic знал, параметры подключения к БД с которой нужно 
    будет работать.
  
- находим в функции `run_migrations_online` вызов `context.configure` и 
    приводим его следующему виду:

    ```python
    context.configure(
        connection=connection,
        target_metadata=target_metadata,
        render_as_batch=connectable.url.drivername == 'sqlite',
    )
    ```

Пример настроенного проекта вы сможете найти [тут](https://gitlab.com/alex925/alembic-example).


### Автоматическая генерация миграций

После настройки можно сгенерировать нашу первую миграцию, которая зафиксирует
начальное состояние базы. Скрипт с миграцией будет создан в папке 
migrations/versions.

```bash
alembic revision --autogenerate -m "init"
```

Когда создание миграции закончено, можно применить ее к БД 

```bash
alembic upgrade head
```

### Ограничения автоматической генерации миграций

Автогенерация просто замечательна, но у нее есть некоторые ограничения, о 
которых нужно знать. Прежде всего, если вы переименуете таблицу или столбец,
то alembic будет считать это удалением и созданием нового столбца/таблицы. Это 
приведет к потере данных. По этому вы должны вручную произвести редактирование 
автоматически сгенерированной миграции в которой был переименован 
столбец/таблица. 

### Ручное создание миграций

Иногда бывает нужно создать миграцию вручную. Для этого необходимо выполнить
в консоли команду

```bash
alembic revision -m "add_field_create_date"
```

После ее выполнения, в папке `migrations/versions`, будет создан скрипт 
`2019_06_09_2107-7c86752c4a38_add_field_create_date.py` с пустыми функциями 
upgrade и downgrade, в которых нужно будет описать то, что необходимо сделать с БД.

В результате должен получиться примерно такой код:

```python
"""add_field_create_date

Revision ID: 7c86752c4a38
Revises: 2e8e603859e2
Create Date: 2019-06-09 21:07:17.573918

"""
from alembic import op
import sqlalchemy as sa


# revision identifiers, used by Alembic.
revision = '7c86752c4a38'
down_revision = '2e8e603859e2'
branch_labels = None
depends_on = None


def upgrade():
    op.add_column('student', sa.Column('create_date', sa.Date(), nullable=True))


def downgrade():
    with op.batch_alter_table('student') as batch_op:
        batch_op.drop_column('create_date')

``` 

Теперь, можно применить данную миграцию при помощи команды:

```bash
alembic upgrade head
```

Если необходимо будет откатить сделанные изменения, то это можно будет сделать
1 из ниже перечисленных команд:

```bash
# 2e8e603859e2 - это номер ревизии к которой мы должны откатиться
alembic downgrade 2e8e603859e2
```

```bash
alembic downgrade -1
```

### Добавление миграций в существующие приложение

При добавлении миграций в существующие приложение, где в БД уже созданы все 
необходимые таблицы в ручную или просто sql скриптами необходимо выполнить 
команду:

```
alembic stamp head
```

чтобы alembic проинциализировался и записал в специальной сервисной таблице
необходимую мета информацию.

Ссылки на дополнительные материалы:

- [alembic – Python migrations quick start](https://michaelheap.com/alembic-python-migrations-quick-start/)
- [Schema migrations with Alembic, Python and PostgreSQL](https://www.compose.com/articles/schema-migrations-with-alembic-python-and-postgresql/)


### Применение миграций из кода приложения

Применить миграции из кода приложения можно при помощи следующего кода

```python
import logging
import os

from alembic.config import main as alembic_commands


def root_logger_cleaner():
    """
    Сбрасывает root логгер к настройкам, которые были у root логгера при
    инициализации коррутины
    """
    root = logging.getLogger()
    default_settings = {
        'level': root.level,
        'disabled': root.disabled,
        'propagate': root.propagate,
        'filters': root.filters[:],
        'handlers': root.handlers[:],
    }
    yield

    while True:
        for attr, attr_value in default_settings.items():
            setattr(root, attr, attr_value)
        yield


def apply_migrations(root_dir):
    """
    Применяет к текущей БД все миграции
    :param root_dir: корневая дирректория проекта (дирректория в которой
    располагается папка alembic, содержащая миграции)
    """
    cwd = os.getcwd()
    os.chdir(root_dir)
    logger_cleaner = root_logger_cleaner()
    next(logger_cleaner)

    try:
        alembic_commands(argv=('--raiseerr', 'upgrade', 'head',))
    except Exception as err:
        next(logger_cleaner)
        logging.getLogger().error(
            f'Возникла ошибка при попытке применить миграции: {err}'
        )
        raise
    finally:
        os.chdir(cwd)

    next(logger_cleaner)
```


## Исправление известных проблем

### Команда для автоматической генерации миграции создает пустой скрипт

Если вы столкнулись с тем, что автоматическая генерация миграций не срабатывает,
значит вам необходимо проверить, что при импорте базового класса (`Base`) 
происходит импорт всех моделей приложения. 

>Это очень важно, так как базовый класс моделей внутри себя пользуется 
>возможностями метапрограммирования по этому, если во время исполнения 
>скрипта env.py модели не будут импортированы, то sqlalchemy ничего не будет о 
>них знать и следовательно не сможет сгенерировать миграцию.

Если вы поняли, что столкнулись именно с такой ситуацией, то один из путей 
решения проблемы, это:

- вынести все модули с моделями в отдельный пакет
- реализовать в `__init__.py` импортирование всех модулей пакета
- в env.py импортировать класс Base из этого пакета

### No support for ALTER of constraints in SQLite dialec

При использовании sqlite может возникать ошибка 
`No support for ALTER of constraints in SQLite dialect` (в моем случае данная 
ошибка возникала, когда один из столбцов был помечен как уникальный 
`unique=True`).

Для того, чтобы исправить данную ошибку, нужно внести исправления в функции
`run_migrations_offline` и `run_migrations_online`, которые находятся в скрипте 
env.

В этих функциях необходимо найти вызов context.configure и добавить в него 
параметр `render_as_batch=True`. После этого будут генерироваться корректные 
миграции.

Полезные ссылки:

- [Sqlite lack of ALTER support, Alembic migration failing because of this. Solutions?](https://stackoverflow.com/questions/30378233/sqlite-lack-of-alter-support-alembic-migration-failing-because-of-this-solutio)
- [batch_alter_table - alembic](https://kite.com/python/docs/alembic.op.batch_alter_table)


### Изменение foreign key в sqlite

Пример миграции, которая изменяет значение ondelete у внешнего ключа

```python
from alembic import op
import sqlalchemy as sa
from sqlalchemy import orm


NEW_SERIES_TABLE_ARGS = (
    'series',
    sa.Column('id', sa.Integer(), nullable=False),
    sa.Column('id_serial', sa.Integer(), nullable=True),
    sa.Column('series_number', sa.Integer(), nullable=True),
    sa.Column('season_number', sa.Integer(), nullable=True),
    sa.Column('looked', sa.Boolean(), nullable=True),
    sa.ForeignKeyConstraint(['id_serial'], ['serial.id'], ondelete='CASCADE'),
    sa.PrimaryKeyConstraint('id')
)

OLD_SERIES_TABLE_ARGS = (
    'series',
    sa.Column('id', sa.Integer(), nullable=False),
    sa.Column('id_serial', sa.Integer(), nullable=True),
    sa.Column('series_number', sa.Integer(), nullable=True),
    sa.Column('season_number', sa.Integer(), nullable=True),
    sa.Column('looked', sa.Boolean(), nullable=True),
    sa.ForeignKeyConstraint(['id_serial'], ['serial.id'], ),
    sa.PrimaryKeyConstraint('id')
)


def move_data(table_args):
    temp_serial_table_name = 'series_temp'
    column_series_tabel = [
        'id', 'id_serial', 'series_number', 'season_number', 'looked'
    ]
    sql_get_all_series = (
        f'select {", ".join(column_series_tabel)} '
        f'from {temp_serial_table_name}'
    )

    # В sqlite нельзя изменить foreign key, по этому приходится создавать новую
    # таблицу и переносить в нее данные из старой
    op.rename_table('series', temp_serial_table_name)

    new_series_table = op.create_table(*table_args)

    bind = op.get_bind()
    session = orm.Session(bind=bind)

    all_series = list(map(
        lambda row: dict(zip(column_series_tabel, row)),
        session.execute(sql_get_all_series)
    ))
    op.bulk_insert(new_series_table, all_series)
    session.commit()

    op.drop_table(temp_serial_table_name)


def upgrade():
    move_data(NEW_SERIES_TABLE_ARGS)


def downgrade():
    move_data(OLD_SERIES_TABLE_ARGS)
```

Полезные ссылки:

- [SQLite: Foreign Keys with Cascade Delete](https://www.techonthenet.com/sqlite/foreign_keys/foreign_delete.php)

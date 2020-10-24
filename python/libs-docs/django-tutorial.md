Django
======

# Оглавление

1. <h3>[ORM](#ORM)</h3>

    - [Оптимизация запросов](#Оптимизация-запросов)
    - [Профалинг запросов](#Профалинг-запросов)


<a name='ORM'></a>
## ORM

<a name='Оптимизация-запросов'></a>
### Оптимизация запросов

При работе ORM нужно быть очень осторожным, потому что не правильное его 
использование может очень сильно увеличить нагрузку на БД. Один из примеров,
когда это может случиться это работа со связаннами таблицами.

Например мы имеем вот такую модель:

```python
class Book(models.Model):
    name = models.CharField('Название', max_length=100)
    category = models.ForeignKey(Category)
    authors = models.ManyToManyField(Author)
```

**select_related**

Предположим мы хотим получить информацию о книге и ее категории. Если решать 
задачу влоб то мы напишем примерно вот такой код:

```python
book = Book.objects.get(pk=book_id)
print(book.name, book.category.name)
```

Проблема данного кода в том, что он заставит ORM сделать 2 запроса к БД. Первый
извлечет информацию о книге, а второй о катерии к которой относится книга.
Для того, чтобы этого избежать необходимо использовать метод `select_related`.
Он говорит ORM, что необходимо сделать запрос с JOIN.

```python
Book.objects.select_related('category').get(pk=book_id)
```

Теперь ORM за 1 запрос извлечет сразу всю нужную нам инфомрацию и при получении
имени категории `book.category.name` не будет генерироваться допольнительный 
запрос.

**prefetch_related**

Кроме получения информации о книге и ее категории у нас может возникнуть 
желание получить информацию о книге и о всех ее авторах. Если решать эту задачу
влоб, то мы столкнемся с проблемой N+1 select.

Для того, чтобы решить ее необходимо использовать метод `prefetch_related`.

```python
book = Book.objects.prefetch_related('authors').get(pk=1)
print([author.full_name for author in book.authors.all()])
```

Этот метод говорит ORM, что необходимо сделать 2 запроса, первый извлеч 
инормацию о книге, а второй извлечет всех авторов связанных с данной книгой.
Это очень сильно разгрузит нашу БД.


<a name='Профалинг-запросов'></a>
### Профалинг запросов

Для того, чтобы получить SQL запрос, который будет сгенерирован при 
использовании ORM удобно использовать `shell_plus` из расширения 
`django-extensions`.

Пример использования:

```
./manage.py shell_plus --print-sql
``` 

```python
User.objects.get(pk=1)
SELECT "auth_user"."id",
       "auth_user"."password",
       "auth_user"."last_login",
       "auth_user"."is_superuser",
       "auth_user"."username",
       "auth_user"."first_name",
       "auth_user"."last_name",
       "auth_user"."email",
       "auth_user"."is_staff",
       "auth_user"."is_active",
       "auth_user"."date_joined"
FROM "auth_user"
WHERE "auth_user"."id" = 1

Execution time: 0.002466s [Database: default]

<User: username>
```

Полезные ссылки:

- [Showing SQL Queries with Pytest and Django](https://blog.ploetzli.ch/2019/showing-sql-queries-with-pytest-and-django/)

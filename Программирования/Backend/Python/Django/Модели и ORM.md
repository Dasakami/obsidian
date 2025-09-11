# Django ORM и модели

Django ORM (Object-Relational Mapping) — это система, которая позволяет работать с базой данных **через Python-классы и объекты**, а не напрямую через SQL.



## 1️⃣ Основы моделей

```python
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)
    birth_date = models.DateField()
    email = models.EmailField(unique=True)

    def __str__(self):
        return self.name
````

- `models.Model` — базовый класс всех моделей.
    
- Поля (fields) описывают колонки в таблице.
    
- Метод `__str__` — возвращает человеко-читаемое имя объекта.
    

---

## 2️⃣ Типы полей (Field types)

|Поле|Описание|
|---|---|
|`CharField(max_length=)`|Строка фиксированной длины|
|`TextField()`|Текст (длинный)|
|`IntegerField()`|Целое число|
|`FloatField()`|Дробное число|
|`BooleanField()`|True/False|
|`DateField()`|Дата|
|`DateTimeField()`|Дата и время|
|`EmailField()`|email|
|`ForeignKey()`|Связь «многие к одному»|
|`ManyToManyField()`|Связь «многие ко многим»|
|`OneToOneField()`|Связь «один к одному»|

---

## 3️⃣ Взаимоотношения

### 🔹 ForeignKey (многие к одному)

```python
class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
```

- `on_delete=models.CASCADE` — если автор удалён, книги тоже удаляются.
    
- Можно использовать `SET_NULL`, `PROTECT`, `DO_NOTHING` и другие варианты.
    

### 🔹 ManyToManyField (многие ко многим)

```python
class Category(models.Model):
    name = models.CharField(max_length=50)

class Book(models.Model):
    title = models.CharField(max_length=200)
    categories = models.ManyToManyField(Category)
```

### 🔹 OneToOneField (один к одному)

```python
class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField()
```

---

## 4️⃣ Метаданные модели (`Meta`)

```python
class Book(models.Model):
    title = models.CharField(max_length=200)

    class Meta:
        ordering = ['title']  # сортировка по умолчанию
        verbose_name = 'Книга'
        verbose_name_plural = 'Книги'
```

- `ordering` — порядок объектов по умолчанию
    
- `verbose_name` — название модели в админке
    
- `db_table` — можно указать своё имя таблицы в БД
    

---

## 5️⃣ Основные методы ORM - Побольше инфы [[ORM|тут]]

### Создание объекта

```python
author = Author.objects.create(name='Пушкин', birth_date='1799-06-06')
```

### Получение объектов

```python
all_authors = Author.objects.all()
one_author = Author.objects.get(id=1)
```

### Фильтрация

```python
books = Book.objects.filter(author__name='Пушкин')
books = Book.objects.exclude(title='Евгений Онегин')
```

### Сортировка

```python
books = Book.objects.order_by('title')       # по возрастанию
books = Book.objects.order_by('-title')      # по убыванию
```

### Обновление

```python
author.name = 'Александр Пушкин'
author.save()
```

### Удаление

```python
author.delete()
```

---

## 6️⃣ Агрегации и аннотации

```python
from django.db.models import Count, Avg

# количество книг у каждого автора
Author.objects.annotate(book_count=Count('book'))

# средний возраст авторов
Author.objects.aggregate(avg_age=Avg('birth_date'))
```

---

## 7️⃣ Сигналы моделей (опционально)

Позволяют реагировать на события:

```python
from django.db.models.signals import post_save
from django.dispatch import receiver

@receiver(post_save, sender=Author)
def author_created(sender, instance, created, **kwargs):
    if created:
        print(f"Создан новый автор: {instance.name}")
```

---

💡 **Итог:**

- Модель = таблица БД
    
- Поля = колонки
    
- ORM позволяет писать **Python-код вместо SQL**
    
- Поддерживает **создание, фильтрацию, обновление, удаление, связи и агрегации**
    


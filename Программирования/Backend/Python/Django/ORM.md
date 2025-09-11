# Django ORM — продвинутая шпаргалка

Django ORM (Object-Relational Mapping) позволяет работать с базой данных **через Python объекты**, писать запросы, фильтровать, агрегировать, сортировать и оптимизировать выборки.



## 1️⃣ Получение объектов

```python
all_authors = Author.objects.all()          # все объекты
author = Author.objects.get(id=1)           # один объект, исключение если не найден
first_author = Author.objects.first()       # первый объект
last_author = Author.objects.last()         # последний объект
````

---

## 2️⃣ Фильтрация и поиск

```python
# фильтрация
books = Book.objects.filter(title__icontains='онегин')  # без учёта регистра
books = Book.objects.exclude(title='Евгений Онегин')    # исключение

# условия
books = Book.objects.filter(pages__gte=100)             # >= 100 страниц
books = Book.objects.filter(pages__lt=500)              # < 500 страниц

# логические операции
books = Book.objects.filter(title__icontains='онегин', pages__gte=200)  # AND
books = Book.objects.filter(Q(title__icontains='онегин') | Q(pages__gte=200))  # OR
```

---

## 3️⃣ Сортировка и ограничение

```python
books = Book.objects.order_by('title')   # по возрастанию
books = Book.objects.order_by('-title')  # по убыванию
books = Book.objects.all()[:10]          # первые 10 объектов
books = Book.objects.all()[10:20]        # срез
```

---

## 4️⃣ Создание, обновление и удаление

```python
# создание
author = Author.objects.create(name='Толстой')
book = Book(title='Война и мир', author=author)
book.save()

# обновление
author.name = 'Лев Толстой'
author.save()
Author.objects.filter(id=1).update(name='Лев Николаевич Толстой')

# удаление
book.delete()
Author.objects.filter(name='Толстой').delete()
```

---

## 5️⃣ Агрегации и аннотации

```python
from django.db.models import Count, Avg, Sum, Max, Min

# количество книг у авторов
Author.objects.annotate(book_count=Count('book'))

# средняя цена книг
Book.objects.aggregate(avg_price=Avg('price'))

# максимальная и минимальная цена
Book.objects.aggregate(max_price=Max('price'), min_price=Min('price'))

# суммарное количество страниц
Book.objects.aggregate(total_pages=Sum('pages'))
```

- `annotate()` добавляет вычисляемое поле к каждому объекту
    
- `aggregate()` возвращает словарь с результатами
    

---

## 6️⃣ Работа с связями (без описания моделей)

```python
# жадная загрузка для ForeignKey
books = Book.objects.select_related('author').all()   # один JOIN, оптимизация

# жадная загрузка для ManyToMany
books = Book.objects.prefetch_related('categories').all()  # отдельные запросы

# доступ к связанным объектам
book = Book.objects.first()
author_books = book.author.book_set.all()  # через ForeignKey
categories = book.categories.all()        # через ManyToMany
```

---

## 7️⃣ Функции для выборки

```python
# values() и values_list()
Book.objects.values('title', 'pages')          # словарь
Book.objects.values_list('title', flat=True)   # список кортежей или плоский список

# distinct() - уникальные значения
Book.objects.values('author').distinct()

# exists() - проверка наличия
Book.objects.filter(title__icontains='онегин').exists()
```

---

## 8️⃣ Транзакции

```python
from django.db import transaction

with transaction.atomic():
    author = Author.objects.create(name='Достоевский')
    Book.objects.create(title='Преступление и наказание', author=author)
```

- Всё внутри блока выполняется атомарно
    
- При ошибке — откат всех изменений
    

---

## 9️⃣ Оптимизация ORM

- `select_related()` — для ForeignKey, делает JOIN, экономит запросы
    
- `prefetch_related()` — для ManyToMany, делает отдельный запрос для связанных объектов
    
- `only()` / `defer()` — загружает только нужные поля
    
- `iterator()` — выгружает объекты по частям для больших выборок
    

```python
books = Book.objects.select_related('author').only('title', 'author__name')
for book in Book.objects.iterator():
    print(book.title)
```

---

## 🔟 Q-объекты для сложных условий

```python
from django.db.models import Q

books = Book.objects.filter(
    Q(title__icontains='онегин') | Q(pages__gte=300)
)
```

- Позволяют строить **OR**, **AND**, **NOT** условия
    
- Можно комбинировать произвольно
    

---

💡 **Итог:**

- ORM = Pythonic доступ к базе
    
- Методы: `filter()`, `exclude()`, `annotate()`, `aggregate()`, `order_by()`, `select_related()`, `prefetch_related()`
    
- Оптимизация важна для больших проектов
    
- Транзакции гарантируют целостность данных
    
- Q-объекты дают гибкость при сложных запросах
    
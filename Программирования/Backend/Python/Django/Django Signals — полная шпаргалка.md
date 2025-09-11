Signals — это **события в Django**, которые позволяют выполнять действия при определённых событиях (создание, сохранение, удаление объектов и т.д.) без изменения основной логики моделей.

---

## 1️⃣ Основные сигналы

| Сигнал                  | Когда вызывается                                         |
|-------------------------|---------------------------------------------------------|
| `pre_save`              | Перед сохранением объекта                                |
| `post_save`             | После сохранения объекта                                 |
| `pre_delete`            | Перед удалением объекта                                  |
| `post_delete`           | После удаления объекта                                   |
| `m2m_changed`           | При изменении many-to-many поля                           |
| `pre_migrate` / `post_migrate` | До/После миграций                                   |

---

## 2️⃣ Как подключить сигнал

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from .models import Book

@receiver(post_save, sender=Book)
def book_created(sender, instance, created, **kwargs):
    if created:
        print(f'Книга "{instance.title}" создана')
```

- `sender` — модель, к которой привязан сигнал
- `created` — True, если объект был создан (только для `post_save`)
- `instance` — сам объект модели

---

## 3️⃣ Signals с ManyToMany

```python
from django.db.models.signals import m2m_changed
from .models import Book, Author

@receiver(m2m_changed, sender=Book.authors.through)
def authors_changed(sender, instance, action, **kwargs):
    if action == 'post_add':
        print(f'Книге "{instance.title}" добавлены авторы')
```

- `sender` — через таблицу `Book.authors.through`
- `action` может быть: `pre_add`, `post_add`, `pre_remove`, `post_remove`, `pre_clear`, `post_clear`

---

## 4️⃣ Signals и пользовательская логика

```python
@receiver(post_save, sender=User)
def create_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)
```

- Создание профиля автоматически при регистрации пользователя

---

## 5️⃣ Подключение сигналов

- Создаём файл `signals.py` в приложении
- Импортируем сигналы в `apps.py`:

```python
class BooksConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'books'

    def ready(self):
        import books.signals
```

- Это нужно, чтобы Django зарегистрировал сигналы при старте проекта

---

## 🔟 Best practices

- Использовать сигналы для кросс-приложенной логики
- Не использовать сигналы для критичных операций, лучше делать через методы модели
- Разделять `signals.py` для читаемости
- Всегда проверять `created` в `post_save`, чтобы не дублировать действия
```

---

# 📄 18_Pagination.md

```md
# Django Pagination — полная шпаргалка

Pagination — это **разбиение списка объектов на страницы**, чтобы удобно показывать большое количество данных.

---

## 1️⃣ CBV Pagination (ListView)

```python
from django.views.generic import ListView
from .models import Book

class BookListView(ListView):
    model = Book
    template_name = 'books/list.html'
    context_object_name = 'books'
    paginate_by = 10  # количество объектов на странице
```

- `paginate_by` — количество объектов на одной странице
- `context_object_name` — имя переменной в шаблоне

---

## 2️⃣ Шаблон для пагинации

```html
{% if is_paginated %}
  <div class="pagination">
    {% if page_obj.has_previous %}
      <a href="?page={{ page_obj.previous_page_number }}">Previous</a>
    {% endif %}

    {% for num in page_obj.paginator.page_range %}
      {% if num == page_obj.number %}
        <strong>{{ num }}</strong>
      {% else %}
        <a href="?page={{ num }}">{{ num }}</a>
      {% endif %}
    {% endfor %}

    {% if page_obj.has_next %}
      <a href="?page={{ page_obj.next_page_number }}">Next</a>
    {% endif %}
  </div>
{% endif %}
```

- `page_obj.number` — текущая страница
- `page_obj.paginator.num_pages` — общее количество страниц
- `page_obj.has_previous/has_next` — есть ли соседние страницы

---

## 3️⃣ FBV Pagination

```python
from django.core.paginator import Paginator
from django.shortcuts import render
from .models import Book

def book_list(request):
    book_list = Book.objects.all()
    paginator = Paginator(book_list, 10)  # 10 объектов на страницу
    page_number = request.GET.get('page')
    page_obj = paginator.get_page(page_number)
    return render(request, 'books/list.html', {'page_obj': page_obj})
```

- `paginator.get_page()` автоматически обрабатывает некорректные номера страниц

---

## 4️⃣ Дополнительно

- Параметры запроса сохраняются:
```html
<a href="?page={{ page_obj.next_page_number }}&q={{ request.GET.q }}">Next</a>
```
- Можно использовать **Bootstrap стили** для красивой пагинации
- Для API (DRF) — использовать `PageNumberPagination` или `LimitOffsetPagination`

---

## 🔟 Best practices

- Всегда указывайте `paginate_by`, чтобы не выводить слишком много данных
- В шаблоне показывайте только соседние страницы для удобства
- Для сложных фильтров комбинируйте с `get_queryset()`
- В API используйте стандартные пагинаторы DRF
```

---

Если хочешь, следующим шагом могу сделать **19_Трюки_и_Лайфхаки.md**, где будут **все полезные хаки Django для ускорения разработки**, включая ORM, CBV, Signals, Forms и Templates.  

Хочешь, чтобы я это сделал?
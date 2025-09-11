URLs (маршрутизация) — связывают **HTTP-запросы с Views**.  

---

## 1️⃣ Основы `urls.py`

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),          # FBV
    path('books/', views.BookListView.as_view(), name='book-list'),  # CBV
    path('books/<int:pk>/', views.BookDetailView.as_view(), name='book-detail'),
]
````

- `path(route, view, kwargs=None, name=None)`
    
    - `route` — URL-паттерн
        
    - `view` — функция или CBV (`.as_view()`)
        
    - `name` — имя маршрута для `reverse` и шаблонов
        
- `<int:pk>` — захватывает часть URL и передает как аргумент
    
- Поддерживаются типы: `str`, `int`, `slug`, `uuid`, `path`
    

---

## 2️⃣ Использование `re_path` (регулярные выражения)

```python
from django.urls import re_path
from . import views

urlpatterns = [
    re_path(r'^articles/(?P<year>[0-9]{4})/$', views.article_year, name='article-year'),
]
```

- `(?P<name>pattern)` — именованная группа, передается в view
    
- Используется, если нужен сложный паттерн
    

---

## 3️⃣ Вложенные маршруты через `include`

```python
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('books/', include('books.urls')),  # все маршруты books
]
```

- Позволяет **разделять маршруты по приложениям**
    
- Внутри `books/urls.py`:
    

```python
urlpatterns = [
    path('', views.BookListView.as_view(), name='book-list'),
    path('<int:pk>/', views.BookDetailView.as_view(), name='book-detail'),
]
```

---

## 4️⃣ Namespacing (пространства имен)

```python
# books/urls.py
app_name = 'books'
urlpatterns = [
    path('', views.BookListView.as_view(), name='list'),
]

# шаблон или reverse
{% url 'books:list' %}
reverse('books:list')
```

- `app_name` — обязательный при `include` для уникальности имён
    
- Позволяет использовать одинаковые имена маршрутов в разных приложениях
    

---

## 5️⃣ Аргументы в URL и view

```python
# urls.py
path('book/<int:id>/', views.book_detail, name='book-detail'),

# views.py
def book_detail(request, id):
    book = get_object_or_404(Book, id=id)
    return render(request, 'books/detail.html', {'book': book})
```

- `<int:id>` — тип int, автоматически конвертируется
    
- `<str:name>` — строка
    
- `<slug:slug>` — безопасный для URL текст
    
- `<uuid:uuid>` — UUID объект
    
- `<path:path>` — включает слэши
    

---

## 6️⃣ Reverse URL и использование в шаблонах

```python
from django.urls import reverse
reverse('books:detail', kwargs={'id': 5})  # -> /books/5/

# шаблон
<a href="{% url 'books:detail' id=book.id %}">{{ book.title }}</a>
```

- Позволяет менять маршруты без изменения кода ссылок
    
- Используется и в FBV, и в CBV
    

---

## 7️⃣ CBV и маршруты

```python
from django.urls import path
from .views import BookDetailView

urlpatterns = [
    path('books/<int:pk>/', BookDetailView.as_view(), name='book-detail'),
]
```

- Для CBV нужно всегда использовать `.as_view()`
    
- Аргументы `<int:pk>` передаются автоматически
    

---

## 8️⃣ Маршруты с query-параметрами

- Query-параметры (`?page=2`) **не указываются в urls.py**
    
- Их можно получить в view:
    

```python
page = request.GET.get('page', 1)
```

---

## 9️⃣ Default маршруты и редиректы

```python
from django.views.generic import RedirectView

urlpatterns = [
    path('', RedirectView.as_view(url='/home/', permanent=False)),
]
```

- Используется для редиректа по умолчанию
    
- `permanent=True` — 301 редирект, иначе 302
    

---

## 🔟 Best practices

- Используй `path` вместо `re_path`, если нет сложных regex
    
- Для каждого приложения делай отдельный `urls.py` и подключай через `include`
    
- Используй `app_name` для namespaces
    
- Именуй маршруты (`name='...'`) для использования `reverse` и `{% url %}`
    
- Для CBV всегда `.as_view()`
    
- query-параметры обрабатывай в view через `request.GET`
    

---

💡 **Итог:**

- URL связывает запросы с view
    
- `path`, `re_path`, `include` и namespacing — основные инструменты
    
- Имена маршрутов позволяют безопасно менять URL
    
- CBV требуют `.as_view()`
    
- Query-параметры не пишем в urls.py
    

Сборник часто используемых кусков кода, чтобы быстро повторять и применять.

---

## 1️⃣ Работа с ORM

```python
# Получить все объекты
books = Book.objects.all()

# Фильтрация
books = Book.objects.filter(pages__gte=100)

# Сортировка
books = Book.objects.order_by('-publish_date')

# Получение одного объекта
book = Book.objects.get(pk=1)

# Создание объекта
book = Book.objects.create(title='New', author=user)

# Обновление объекта
book.title = 'Updated'
book.save()

# Удаление объекта
book.delete()

# Агрегации
from django.db.models import Count, Sum, Avg
Book.objects.aggregate(avg_pages=Avg('pages'))

# annotate
Book.objects.annotate(num_authors=Count('author'))
````

---

## 2️⃣ Forms

```python
# FBV
if request.method == 'POST':
    form = BookForm(request.POST)
    if form.is_valid():
        form.save()
else:
    form = BookForm()
return render(request, 'books/form.html', {'form': form})

# Update
form = BookForm(request.POST or None, instance=book)
if form.is_valid():
    form.save()
```

---

## 3️⃣ Templates

```html
<!-- Цикл -->
{% for book in books %}
    {{ book.title }}
{% endfor %}

<!-- Условие -->
{% if books %}
    Есть книги
{% else %}
    Нет книг
{% endif %}

<!-- URL по имени -->
<a href="{% url 'books:detail' id=book.id %}">Подробнее</a>

<!-- Подключение шаблона -->
{% include 'partials/header.html' %}

<!-- Наследование -->
{% extends "base.html" %}
{% block content %}...{% endblock %}
```

---

## 4️⃣ FBV

```python
def book_list(request):
    books = Book.objects.all()
    return render(request, 'books/list.html', {'books': books})

def book_detail(request, pk):
    book = get_object_or_404(Book, pk=pk)
    return render(request, 'books/detail.html', {'book': book})
```

---

## 5️⃣ CBV

```python
from django.views.generic import ListView, DetailView

class BookListView(ListView):
    model = Book
    template_name = 'books/list.html'
    context_object_name = 'books'

class BookDetailView(DetailView):
    model = Book
    template_name = 'books/detail.html'
    context_object_name = 'book'
```

- Generic CBV: `ListView`, `DetailView`, `CreateView`, `UpdateView`, `DeleteView`, `TemplateView`
    

---

## 6️⃣ Mixins

```python
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin

class BookCreateView(LoginRequiredMixin, PermissionRequiredMixin, CreateView):
    model = Book
    permission_required = 'books.add_book'
    success_url = reverse_lazy('books:list')
```

---

## 7️⃣ Authentication & Authorization

```python
# FBV
from django.contrib.auth import authenticate, login, logout

user = authenticate(username='dan', password='1234')
if user:
    login(request, user)
logout(request)

# Декораторы
@login_required
@permission_required('books.add_book')
def add_book(request):
    pass

# CBV миксины
LoginRequiredMixin, PermissionRequiredMixin, UserPassesTestMixin
```

---

## 8️⃣ Redirect и Reverse

```python
from django.shortcuts import redirect
from django.urls import reverse, reverse_lazy

# FBV
return redirect('books:list')
return redirect(reverse('books:list'))

# CBV
success_url = reverse_lazy('books:list')
```

---

## 9️⃣ Query параметры

```python
# Получение GET-параметра
page = request.GET.get('page', 1)

# Получение POST-параметра
username = request.POST.get('username')
```

---

## 🔟 Static и Media

```python
# settings.py
STATIC_URL = '/static/'
MEDIA_URL = '/media/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# шаблон
<img src="{{ MEDIA_URL }}{{ user.profile.avatar }}" />
<link rel="stylesheet" href="{% static 'css/style.css' %}">
```

---

## 11️⃣ Pagination

```python
# ListView
class BookListView(ListView):
    model = Book
    paginate_by = 10

# В шаблоне
{% if is_paginated %}
    {% if page_obj.has_previous %}
        <a href="?page={{ page_obj.previous_page_number }}">Prev</a>
    {% endif %}
    {% for num in page_obj.paginator.page_range %}
        <a href="?page={{ num }}">{{ num }}</a>
    {% endfor %}
    {% if page_obj.has_next %}
        <a href="?page={{ page_obj.next_page_number }}">Next</a>
    {% endif %}
{% endif %}
```

---

## 12️⃣ JSON / API

```python
from django.http import JsonResponse

def book_list_api(request):
    books = Book.objects.all().values('id', 'title', 'author')
    return JsonResponse(list(books), safe=False)
```

---

## 13️⃣ Signals

```python
from django.db.models.signals import post_save
from django.dispatch import receiver

@receiver(post_save, sender=Book)
def book_created(sender, instance, created, **kwargs):
    if created:
        print(f'Книга {instance.title} создана')
```

---

## 14️⃣ Misc

```python
# get_object_or_404
book = get_object_or_404(Book, pk=pk)

# settings
LANGUAGE_CODE = 'ru-ru'
TIME_ZONE = 'Asia/Bishkek'
USE_I18N = True
USE_TZ = True
```

---

💡 **Итог:**

- Быстрый доступ к часто используемым конструкциям Django
    
- FBV и CBV, ORM, Forms, Templates, Authentication, Permissions, Signals
    
- Можно добавлять свои snippets по мере изучения
    


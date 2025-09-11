CBV — это **классы для обработки HTTP-запросов**, которые позволяют переиспользовать код и создавать CRUD без дублирования логики.  

---

## 1️⃣ Базовый синтаксис CBV

```python
from django.views import View
from django.shortcuts import render
from django.http import HttpResponse

class MyView(View):
    def get(self, request, *args, **kwargs):
        return HttpResponse('GET-запрос')

    def post(self, request, *args, **kwargs):
        return HttpResponse('POST-запрос')
```

- `get` — обрабатывает GET-запрос
- `post` — POST-запрос
- `put`, `delete`, `patch` — другие методы HTTP
- `.as_view()` — обязательно при подключении в `urls.py`

---

## 2️⃣ Основные Generic CBV

| CBV               | Назначение                                         |
|------------------|---------------------------------------------------|
| `TemplateView`    | Рендерит шаблон без queryset                     |
| `ListView`        | Список объектов модели                            |
| `DetailView`      | Детальная страница объекта                        |
| `CreateView`      | Создание объекта через форму                      |
| `UpdateView`      | Редактирование объекта через форму               |
| `DeleteView`      | Удаление объекта с подтверждением               |
| `RedirectView`    | Редирект на другой URL                            |

---

## 3️⃣ Пример ListView

```python
from django.views.generic import ListView
from .models import Book

class BookListView(ListView):
    model = Book
    template_name = 'books/list.html'
    context_object_name = 'books'
    paginate_by = 10  # пагинация
```

- `model` — модель
- `template_name` — шаблон
- `context_object_name` — имя переменной в шаблоне
- `paginate_by` — количество объектов на странице

---

## 4️⃣ DetailView

```python
from django.views.generic import DetailView
from .models import Book

class BookDetailView(DetailView):
    model = Book
    template_name = 'books/detail.html'
    context_object_name = 'book'
```

- `pk` или `slug` передается из URL
- `get_object()` можно переопределить для кастомного queryset

---

## 5️⃣ CreateView / UpdateView / DeleteView

```python
from django.views.generic import CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import Book
from .forms import BookForm

class BookCreateView(CreateView):
    model = Book
    form_class = BookForm
    template_name = 'books/form.html'
    success_url = reverse_lazy('books:list')

class BookUpdateView(UpdateView):
    model = Book
    form_class = BookForm
    template_name = 'books/form.html'
    success_url = reverse_lazy('books:list')

class BookDeleteView(DeleteView):
    model = Book
    template_name = 'books/confirm_delete.html'
    success_url = reverse_lazy('books:list')
```

- `success_url` — куда перенаправлять после действия
- CreateView автоматически вызывает `form_valid` и сохраняет объект

---

## 6️⃣ Миксины (Mixins)

- Используются для добавления функциональности:

```python
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin

class BookCreateView(LoginRequiredMixin, CreateView):
    model = Book
    form_class = BookForm
    success_url = reverse_lazy('books:list')

class AdminRequiredMixin(PermissionRequiredMixin):
    permission_required = 'books.add_book'
```

- **Важно:** миксины ставятся **перед** CBV

---

## 7️⃣ Методы CBV для кастомизации

- `get_context_data(self, **kwargs)` — добавляет переменные в шаблон
- `get_queryset(self)` — возвращает кастомный queryset
- `get_object(self, queryset=None)` — возвращает объект для Detail/Update/Delete
- `form_valid(self, form)` — вызывается при валидной форме
- `form_invalid(self, form)` — вызывается при ошибках формы
- `dispatch(self, request, *args, **kwargs)` — точка входа для всех методов HTTP, можно добавить кастомную логику

---

## 8️⃣ Pagination

```python
class BookListView(ListView):
    model = Book
    paginate_by = 5
```

- В шаблоне:
```html
{% if is_paginated %}
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
{% endif %}
```

---

## 9️⃣ Кастомные методы и декораторы

```python
from django.utils.decorators import method_decorator
from django.contrib.auth.decorators import login_required

@method_decorator(login_required, name='dispatch')
class BookUpdateView(UpdateView):
    model = Book
    form_class = BookForm
    success_url = reverse_lazy('books:list')
```

- Можно использовать декораторы для CBV через `method_decorator`
- `name='dispatch'` — применяется ко всем HTTP-методам

---

## 🔟 Best practices CBV

- Используй CBV для CRUD и повторяющихся страниц  
- FBV оставляй для простых страниц и сложной логики  
- Миксины всегда **перед** основным CBV  
- Переопределяй методы `get_queryset`, `get_context_data`, `form_valid`  
- Для авторизации используйте `LoginRequiredMixin`, `PermissionRequiredMixin`  
- Всегда указывай `template_name` и `context_object_name` для ясности  
- Для редиректа после создания/удаления используйте `reverse_lazy`  

---

💡 **Итог:**
- CBV — мощный инструмент для организации кода
- Гибко комбинируются с миксинами
- Позволяют переиспользовать методы, формы и логику
- CRUD полностью автоматизируется с помощью Generic CBV

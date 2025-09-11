Mixins — это **классы, которые добавляют дополнительный функционал к CBV** без дублирования кода.  

---

## 1️⃣ Что такое Mixins

- Mixins **не являются полноценными CBV**
- Их цель — добавление повторяемой логики
- Используются вместе с CBV:
```python
class MyView(LoginRequiredMixin, ListView):
    model = Book
````

- **Важно:** миксины всегда ставятся **перед CBV**, иначе могут не работать
    

---

## 2️⃣ Встроенные Mixins Django

|Mixin|Назначение|
|---|---|
|`LoginRequiredMixin`|только авторизованные пользователи|
|`PermissionRequiredMixin`|проверка прав (`permission_required`)|
|`UserPassesTestMixin`|проверка по функции `test_func`|
|`MultipleObjectMixin`|работа с множеством объектов (ListView)|
|`ContextMixin`|добавление данных в шаблон (через `get_context_data`)|

---

## 3️⃣ LoginRequiredMixin

```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import ListView
from .models import Book

class BookListView(LoginRequiredMixin, ListView):
    model = Book
    template_name = 'books/list.html'
```

- Если пользователь не авторизован — редирект на `LOGIN_URL` из settings
    
- Можно настроить `redirect_field_name` для возврата на исходную страницу после логина
    

---

## 4️⃣ PermissionRequiredMixin

```python
from django.contrib.auth.mixins import PermissionRequiredMixin

class BookCreateView(PermissionRequiredMixin, CreateView):
    model = Book
    form_class = BookForm
    permission_required = 'books.add_book'
```

- Проверяет права пользователя
    
- Можно указать несколько прав:
    

```python
permission_required = ('books.add_book', 'books.change_book')
```

---

## 5️⃣ UserPassesTestMixin

```python
from django.contrib.auth.mixins import UserPassesTestMixin

class BookUpdateView(UserPassesTestMixin, UpdateView):
    model = Book
    form_class = BookForm
    success_url = reverse_lazy('books:list')

    def test_func(self):
        # проверка, владелец ли книги пользователь
        book = self.get_object()
        return book.created_by == self.request.user
```

- `test_func` возвращает True или False
    
- Если False — редирект на `login_url` или 403
    

---

## 6️⃣ ContextMixin

- Добавляет переменные в шаблон
    

```python
from django.views.generic.base import ContextMixin

class TitleMixin(ContextMixin):
    title = None

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['title'] = self.title
        return context

class BookListView(TitleMixin, ListView):
    model = Book
    template_name = 'books/list.html'
    title = "Список книг"
```

- Используется для **DRY-шаблонов** и повторяемого контекста
    

---

## 7️⃣ MultipleObjectMixin

- Используется в CBV для работы с **множеством объектов**
    
- Например: ListView наследует этот миксин
    

```python
class BookListView(MultipleObjectMixin, ListView):
    model = Book
```

- Позволяет:
    
    - фильтровать queryset (`get_queryset`)
        
    - добавлять пагинацию (`paginate_by`)
        

---

## 8️⃣ Кастомные Mixins

```python
class StaffRequiredMixin:
    def dispatch(self, request, *args, **kwargs):
        if not request.user.is_staff:
            from django.http import HttpResponseForbidden
            return HttpResponseForbidden()
        return super().dispatch(request, *args, **kwargs)
```

- Переопределяем `dispatch` для логики до view
    
- Можно комбинировать с другими миксинами
    

---

## 9️⃣ Порядок наследования

```python
class MyView(Mixin1, Mixin2, ListView):
    pass
```

- Python использует **MRO (Method Resolution Order)**
    
- Mixins ставим **перед CBV**
    
- Вызов `super()` внутри миксина позволяет правильно цепочку методов
    

---

## 🔟 Best practices

- Для проверки авторизации/прав — используйте встроенные миксины (`LoginRequiredMixin`, `PermissionRequiredMixin`)
    
- Для повторяемого контекста — создавайте кастомные `ContextMixin`
    
- Для логики до view — используйте `dispatch` в кастомном миксине
    
- Mixins должны быть **узко направленные**, не перегружать бизнес-логику
    
- Комбинируйте несколько миксинов перед CBV для DRY-кода
    

---

💡 **Итог:**

- Mixins — способ повторного использования логики в CBV
    
- Встроенные: авторизация, права, контекст, multiple objects
    
- Кастомные: проверка условий, общий контекст, staff-only доступ
    
- Ставьте Mixins **перед CBV**, используйте `super()`
    


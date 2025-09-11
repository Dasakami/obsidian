
FBV — это **функции, которые обрабатывают HTTP-запросы** и возвращают HTTP-ответ.  

---

## 1️⃣ Основной синтаксис FBV

```python
from django.shortcuts import render, redirect, get_object_or_404
from django.http import HttpResponse
from .models import Book
from .forms import BookForm

def home(request):
    return HttpResponse("Главная страница")

def book_list(request):
    books = Book.objects.all()
    return render(request, 'books/list.html', {'books': books})

def book_detail(request, pk):
    book = get_object_or_404(Book, pk=pk)
    return render(request, 'books/detail.html', {'book': book})
````

- `request` — объект запроса
    
- `render` — рендерит шаблон с контекстом
    
- `redirect` — редирект на другой URL
    
- `get_object_or_404` — возвращает объект или 404
    

---

## 2️⃣ Работа с формами

```python
def book_create(request):
    if request.method == 'POST':
        form = BookForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('books:list')
    else:
        form = BookForm()
    return render(request, 'books/form.html', {'form': form})
```

- Проверка метода `request.method`
    
- Валидация через `form.is_valid()`
    
- Сохранение данных через `form.save()`
    
- GET-запрос — просто отображение формы
    

---

## 3️⃣ Update и Delete

```python
def book_update(request, pk):
    book = get_object_or_404(Book, pk=pk)
    form = BookForm(request.POST or None, instance=book)
    if form.is_valid():
        form.save()
        return redirect('books:list')
    return render(request, 'books/form.html', {'form': form})

def book_delete(request, pk):
    book = get_object_or_404(Book, pk=pk)
    if request.method == 'POST':
        book.delete()
        return redirect('books:list')
    return render(request, 'books/confirm_delete.html', {'book': book})
```

- `instance=book` для редактирования
    
- DELETE через POST-запрос с подтверждением
    

---

## 4️⃣ Работа с ORM

```python
# получение всех объектов
books = Book.objects.all()

# фильтры
books = Book.objects.filter(pages__gte=100).order_by('-publish_date')

# создание объекта
book = Book.objects.create(title='New', author=user)

# обновление объекта
book.title = 'Updated'
book.save()

# удаление объекта
book.delete()
```

---

## 5️⃣ Работа с query-параметрами

```python
def book_search(request):
    query = request.GET.get('q', '')
    books = Book.objects.filter(title__icontains=query)
    return render(request, 'books/list.html', {'books': books, 'query': query})
```

- `request.GET.get('param', default)` — получение GET-параметров
    
- Не указываем query-параметры в `urls.py`
    

---

## 6️⃣ Redirect и reverse

```python
from django.urls import reverse

return redirect('books:list')
return redirect(reverse('books:list'))
```

- `reverse` — генерация URL по имени маршрута
    
- Удобно при переименовании маршрутов
    

---

## 7️⃣ Декораторы

```python
from django.contrib.auth.decorators import login_required, permission_required

@login_required
def book_create(request):
    # только для авторизованных
    pass

@permission_required('books.add_book')
def book_update(request, pk):
    pass
```

- `login_required` — проверка авторизации
    
- `permission_required` — проверка прав пользователя
    
- Можно комбинировать с другими декораторами
    

---

## 8️⃣ Работа с шаблонами

```python
return render(request, 'books/list.html', {'books': books})
```

- Контекст — словарь Python
    
- Шаблон использует переменные: `{% for book in books %}{{ book.title }}{% endfor %}`
    

---

## 9️⃣ JSON и API

```python
from django.http import JsonResponse

def book_list_api(request):
    books = Book.objects.all().values('id', 'title', 'author')
    return JsonResponse(list(books), safe=False)
```

- `JsonResponse` для отдачи JSON
    
- Используется для простых API без DRF
    

---

## 🔟 Best practices FBV

- Использовать для простых страниц и логики
    
- Для CRUD можно комбинировать с формами и ORM
    
- Всегда проверяйте `request.method`
    
- Использовать декораторы для авторизации и прав
    
- Для JSON API — `JsonResponse`
    
- Для редиректов — `redirect` или `reverse`
    

---

💡 **Итог:**

- FBV — это функции, обрабатывающие запросы
    
- Подходит для простой и прямой логики
    
- Полный контроль над формами, ORM и шаблонами
    
- Использовать декораторы для авторизации и прав
    

```

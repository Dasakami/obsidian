Views — это **функции или классы**, которые обрабатывают HTTP-запрос и возвращают HTTP-ответ.  

---

## 1️⃣ Function-Based Views (FBV)

```python
from django.shortcuts import render, get_object_or_404, redirect
from .models import Book
from .forms import BookForm

def book_list(request):
    books = Book.objects.all()
    return render(request, 'books/list.html', {'books': books})

def book_detail(request, pk):
    book = get_object_or_404(Book, pk=pk)
    return render(request, 'books/detail.html', {'book': book})

def book_create(request):
    if request.method == 'POST':
        form = BookForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('books:list')
    else:
        form = BookForm()
    return render(request, 'books/form.html', {'form': form})
````

- Простые и прямые
    
- Полный контроль над логикой
    
- Легко использовать с ORM и формами
    

---

## 2️⃣ Class-Based Views (CBV)

Django предоставляет **готовые классы для CRUD** и повторяющихся задач:

### 🔹 Основные CBV

|View|Использование|
|---|---|
|`View`|базовый класс|
|`TemplateView`|рендерит шаблон без контекста|
|`ListView`|список объектов модели|
|`DetailView`|детальная страница объекта|
|`CreateView`|создание объекта через форму|
|`UpdateView`|редактирование объекта через форму|
|`DeleteView`|удаление объекта с подтверждением|

### 🔹 Примеры

```python
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
from .models import Book
from django.urls import reverse_lazy

class BookListView(ListView):
    model = Book
    template_name = 'books/list.html'
    context_object_name = 'books'

class BookDetailView(DetailView):
    model = Book
    template_name = 'books/detail.html'
    context_object_name = 'book'

class BookCreateView(CreateView):
    model = Book
    template_name = 'books/form.html'
    form_class = BookForm
    success_url = reverse_lazy('books:list')
```

- `model` — модель
    
- `template_name` — шаблон
    
- `context_object_name` — название переменной в шаблоне
    
- `form_class` — форма
    
- `success_url` — редирект после успешной операции
    

---

## 3️⃣ Миксины (Mixins)

- Позволяют добавлять функциональность CBV:
    

```python
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin

class BookCreateView(LoginRequiredMixin, CreateView):
    model = Book
    form_class = BookForm
    success_url = reverse_lazy('books:list')
```

- `LoginRequiredMixin` — только для авторизованных пользователей
    
- `PermissionRequiredMixin` — проверка прав
    
- Mixins всегда ставить **перед** основным CBV
    

---

## 4️⃣ Методы CBV

- CBV можно переопределять через методы:
    

```python
class BookCreateView(CreateView):
    form_class = BookForm
    success_url = reverse_lazy('books:list')

    def form_valid(self, form):
        form.instance.created_by = self.request.user
        return super().form_valid(form)

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['title'] = 'Добавить книгу'
        return context
```

- `form_valid` — вызывается при валидной форме
    
- `form_invalid` — при ошибках формы
    
- `get_context_data` — добавление переменных в шаблон
    
- `get_queryset` — для ListView/DetailView кастомизация queryset
    
- `dispatch` — точка входа для request (можно добавить middleware-подобную логику)
    

---

## 5️⃣ Работа с [[ORM]] в Views

```python
# FBV
books = Book.objects.filter(pages__gte=100).order_by('-publish_date')

# CBV
class BookListView(ListView):
    model = Book

    def get_queryset(self):
        return Book.objects.filter(pages__gte=100).order_by('-publish_date')
```

- `get_queryset` позволяет кастомизировать данные
    
- Для сложной логики — использовать Q-объекты и аннотации
    

---

## 6️⃣ Работа с [[Django Forms — полная шпаргалка|Forms]] в Views

### 🔹 FBV

```python
def book_create(request):
    form = BookForm(request.POST or None)
    if form.is_valid():
        form.save()
        return redirect('books:list')
    return render(request, 'books/form.html', {'form': form})
```

### 🔹 CBV

```python
class BookCreateView(CreateView):
    model = Book
    form_class = BookForm
    template_name = 'books/form.html'
    success_url = reverse_lazy('books:list')
```

- CBV автоматизирует проверку формы и сохранение объекта
    

---

## 7️⃣ Работа с [[Django Templates — полная шпаргалка|Templates]]

- FBV:
    

```python
return render(request, 'books/list.html', {'books': books})
```

- CBV:
    

```python
template_name = 'books/list.html'
context_object_name = 'books'
```

- Контекст автоматически передается в шаблон
    

---

## 8️⃣ Redirect и reverse

```python
from django.shortcuts import redirect
from django.urls import reverse, reverse_lazy

return redirect('books:list')
return redirect(reverse('books:list'))
```

- `reverse_lazy` — для CBV, отложенная генерация URL
    
- `redirect` — удобная функция редиректа
    

---

## 9️⃣ API Views (DRF)

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from .models import Book
from .serializers import BookSerializer

class BookAPIView(APIView):
    def get(self, request):
        books = Book.objects.all()
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors)
```

- APIView похож на CBV
    
- Использует DRF сериализаторы и Response
    

---

## 🔟 Best practices

- Для простых страниц — FBV
    
- Для CRUD и повторяющихся операций — CBV
    
- Для проверки авторизации — mixins (`LoginRequiredMixin`, `PermissionRequiredMixin`)
    
- Всегда переопределяй `get_queryset` для фильтров и сортировки
    
- Для форм — используйте `form_class` и `form_valid`
    
- Для API — используйте DRF CBV/APIView/GenericViewSet
    

---

💡 **Итог:**

- Views — логика обработки HTTP-запросов
    
- FBV — функции, полный контроль
    
- CBV — классы, готовые CRUD, миксины, методы `form_valid`, `get_context_data`
    
- ORM и формы интегрируются с любым типом Views
    
- Templates подключаются через render или template_name/context
    

## 1️⃣ Основы тестирования

- Django использует **Python unittest**  
- Тесты создаются в файле `tests.py` внутри приложения или в папке `tests/`  

```bash
python manage.py test
````

- Тесты наследуются от `django.test.TestCase`
    

---

## 2️⃣ Тестирование моделей

```python
from django.test import TestCase
from .models import Book
from django.contrib.auth.models import User

class BookModelTest(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(username='dan', password='1234')
        self.book = Book.objects.create(title='Test Book', author=self.user)

    def test_book_creation(self):
        self.assertEqual(self.book.title, 'Test Book')
        self.assertEqual(self.book.author.username, 'dan')

    def test_book_str(self):
        self.assertEqual(str(self.book), 'Test Book')
```

- `setUp()` — выполняется перед каждым тестом
    
- Проверяем **создание объекта, методы модели, строковое представление**
    

---

## 3️⃣ Тестирование форм

```python
from django.test import TestCase
from .forms import BookForm

class BookFormTest(TestCase):
    def test_valid_form(self):
        data = {'title': 'Book', 'author': 1}
        form = BookForm(data=data)
        self.assertTrue(form.is_valid())

    def test_invalid_form(self):
        data = {'title': ''}
        form = BookForm(data=data)
        self.assertFalse(form.is_valid())
```

- Проверяем **валидность данных формы**
    
- Можно тестировать ошибки и required поля
    

---

## 4️⃣ Тестирование FBV

```python
from django.test import TestCase, Client
from django.urls import reverse
from .models import Book
from django.contrib.auth.models import User

class BookViewTest(TestCase):
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user(username='dan', password='1234')
        self.book = Book.objects.create(title='Book', author=self.user)

    def test_book_list_view(self):
        response = self.client.get(reverse('books:list'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'Book')

    def test_book_detail_view(self):
        response = self.client.get(reverse('books:detail', args=[self.book.id]))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'Book')
```

- `Client()` — имитация браузера
    
- `assertContains` — проверка текста на странице
    
- `reverse` — получение URL по имени маршрута
    

---

## 5️⃣ Тестирование CBV

- CBV тестируются так же, как FBV, через `Client` и `reverse`
    
- Для форм можно тестировать POST-запросы:
    

```python
def test_book_create_view(self):
    self.client.login(username='dan', password='1234')
    response = self.client.post(reverse('books:create'), {'title': 'New Book', 'author': self.user.id})
    self.assertEqual(response.status_code, 302)  # редирект после успешного сохранения
    self.assertTrue(Book.objects.filter(title='New Book').exists())
```

---

## 6️⃣ Тестирование авторизации и прав

```python
from django.contrib.auth.models import Permission

def test_permission_required(self):
    user = User.objects.create_user(username='guest', password='1234')
    self.client.login(username='guest', password='1234')
    response = self.client.get(reverse('books:create'))
    self.assertEqual(response.status_code, 403)  # доступ запрещён

    # даём права
    permission = Permission.objects.get(codename='add_book')
    user.user_permissions.add(permission)
    response = self.client.get(reverse('books:create'))
    self.assertEqual(response.status_code, 200)
```

- Проверяем **доступность страниц для разных пользователей**
    

---

## 7️⃣ Тестирование JSON / API

```python
from django.test import TestCase
from django.urls import reverse
import json

class BookApiTest(TestCase):
    def test_api_list(self):
        response = self.client.get(reverse('books:api-list'))
        self.assertEqual(response.status_code, 200)
        data = json.loads(response.content)
        self.assertIsInstance(data, list)
```

- Используем `json.loads()` для проверки ответа JSON
    

---

## 8️⃣ Тестирование Signals

```python
from django.test import TestCase
from .models import Book
from django.contrib.auth.models import User

class SignalTest(TestCase):
    def test_post_save_signal(self):
        user = User.objects.create_user(username='dan', password='1234')
        book = Book.objects.create(title='Signal Book', author=user)
        # здесь проверяем, что сигнал сработал
        # например, создавался лог или уведомление
```

- Можно тестировать side-effects сигналов
    

---

## 🔟 Best practices

- Разделять тесты по категориям: модели, формы, views, API
    
- Использовать `setUp` для подготовки данных
    
- Использовать `Client` для имитации запросов
    
- Проверять **статусы ответа, контент и side-effects**
    
- Для API — использовать `APIClient` из DRF
    
- Каждое приложение имеет свои `tests.py` или `tests/` папку
    
- Тесты должны быть **автономные и повторяемые**
    

---

💡 **Итог:**

- Django TestCase — основной класс для юнит-тестов
    
- Проверяем модели, формы, views, API, авторизацию, сигналы
    
- Используем Client для FBV/CBV, assert-методы для проверки результатов
    
- Тесты помогают не ломать проект при изменениях
    
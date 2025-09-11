Аутентификация (Authentication) — проверка **кто пользователь**  
Авторизация (Authorization) — проверка **что пользователь может делать**  

---

## 1️⃣ Пользователи (User model)

- Django предоставляет встроенную модель `User`:
```python
from django.contrib.auth.models import User

user = User.objects.create_user(username='dan', password='1234')
user = User.objects.create_superuser(username='admin', password='admin123')
````

- Поля:
    
    - `username`, `email`, `password`, `is_active`, `is_staff`, `is_superuser`, `last_login`
        
- Можно использовать **кастомную модель пользователя** через `AUTH_USER_MODEL`
    

---

## 2️⃣ Группы и права (Groups & Permissions)

```python
from django.contrib.auth.models import Group, Permission

group = Group.objects.create(name='Editors')
permission = Permission.objects.get(codename='add_book')
group.permissions.add(permission)
user.groups.add(group)
```

- `Group` — объединение пользователей
    
- `Permission` — права на модели (`add`, `change`, `delete`, `view`)
    
- Проверка:
    

```python
user.has_perm('books.add_book')        # True/False
user.has_perms(['books.add_book', 'books.change_book'])
user.has_module_perms('books')        # права на приложение
```

---

## 3️⃣ Login и Logout

### 🔹 FBV

```python
from django.contrib.auth import authenticate, login, logout
from django.shortcuts import redirect, render

def user_login(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
        user = authenticate(request, username=username, password=password)
        if user is not None:
            login(request, user)
            return redirect('home')
    return render(request, 'users/login.html')

def user_logout(request):
    logout(request)
    return redirect('home')
```

### 🔹 CBV

```python
from django.contrib.auth.views import LoginView, LogoutView

class MyLoginView(LoginView):
    template_name = 'users/login.html'

class MyLogoutView(LogoutView):
    next_page = 'home'
```

- `authenticate` — проверка username/password
    
- `login` — устанавливает сессию
    
- `logout` — удаляет сессию
    

---

## 4️⃣ Декораторы для FBV

```python
from django.contrib.auth.decorators import login_required, permission_required

@login_required
def dashboard(request):
    pass

@permission_required('books.add_book')
def add_book(request):
    pass
```

- `login_required` — только авторизованные
    
- `permission_required` — проверка прав
    
- Можно передавать `raise_exception=True` для 403 вместо редиректа
    

---

## 5️⃣ Миксины для CBV

```python
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin

class BookCreateView(LoginRequiredMixin, PermissionRequiredMixin, CreateView):
    model = Book
    form_class = BookForm
    permission_required = 'books.add_book'
    success_url = reverse_lazy('books:list')
```

- Mixins всегда **перед CBV**
    
- `LoginRequiredMixin` — авторизация
    
- `PermissionRequiredMixin` — права
    

---

## 6️⃣ Кастомные проверки пользователя

```python
from django.contrib.auth.mixins import UserPassesTestMixin

class StaffOnlyMixin(UserPassesTestMixin):
    def test_func(self):
        return self.request.user.is_staff
```

- `test_func` возвращает True/False
    
- Если False — 403 или редирект
    

---

## 7️⃣ Password Management

- `set_password()` — хэширование пароля
    

```python
user.set_password('newpass')
user.save()
```

- `check_password()` — проверка пароля
    

```python
user.check_password('1234')  # True/False
```

- Встроенные views: `PasswordChangeView`, `PasswordResetView`
    

---

## 8️⃣ JWT Authentication (DRF)

```python
# Установка: djangorestframework-simplejwt
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView

# urls.py
path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
```

- `access` токен — для запросов к API
    
- `refresh` токен — для обновления access
    
- Используется с `REST_FRAMEWORK`:
    

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
}
```

---

## 9️⃣ Single Sign-On (SSO)

- Для SSO используют внешние провайдеры (Google, GitHub, Facebook)
    
- Обычно через **django-allauth**:
    

```python
# settings.py
INSTALLED_APPS = [
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'allauth.socialaccount.providers.google',
]

# urls.py
path('accounts/', include('allauth.urls'))
```

- Поддерживает:
    
    - Регистрацию через соцсети
        
    - Логин/Logout
        
    - Email подтверждение
        

---

## 🔟 Best practices

- Использовать встроенную модель `User` или кастомную через `AUTH_USER_MODEL`
    
- Для проверки авторизации — декораторы или CBV миксины
    
- Всегда проверяйте права пользователя для sensitive действий
    
- Для API — использовать JWT
    
- Для внешних провайдеров — SSO через django-allauth
    
- Никогда не храните пароль в открытом виде
    

---

💡 **Итог:**

- Аутентификация: кто пользователь (login/logout)
    
- Авторизация: что пользователь может делать (permissions, groups)
    
- FBV: декораторы `login_required`, `permission_required`
    
- CBV: миксины `LoginRequiredMixin`, `PermissionRequiredMixin`, `UserPassesTestMixin`
    
- API: JWT через DRF
    
- SSO: соцсети через django-allauth
    

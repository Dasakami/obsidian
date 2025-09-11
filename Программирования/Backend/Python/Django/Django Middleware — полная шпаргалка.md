Middleware — это **промежуточный слой между запросом и ответом**, который позволяет перехватывать, изменять или обрабатывать HTTP-запросы и ответы.

---

## 1️⃣ Что такое Middleware

- Middleware — это Python класс с методами, которые вызываются **до и после view**
- Используется для:
  - Логирования
  - Аутентификации и авторизации
  - Обработки ошибок
  - Кеширования
  - Добавления заголовков, cookie и т.п.

---

## 2️⃣ Как Middleware работает

1. HTTP-запрос приходит в Django
2. Middleware **process_request** выполняется по порядку из списка `MIDDLEWARE`  
3. View обрабатывает запрос  
4. Middleware **process_response** выполняется в обратном порядке
5. Ответ отправляется клиенту

---

## 3️⃣ Стандартные Middleware в Django

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

- `SecurityMiddleware` — безопасность (HTTPS, HSTS)
- `SessionMiddleware` — работа с сессиями
- `CommonMiddleware` — редиректы, слэши
- `CsrfViewMiddleware` — защита от CSRF
- `AuthenticationMiddleware` — подключает `request.user`
- `MessageMiddleware` — поддержка flash-сообщений
- `XFrameOptionsMiddleware` — защита от Clickjacking

---

## 4️⃣ Методы Middleware

Django >= 1.10 использует **новый стиль Middleware** (классы с `__call__`):

```python
class SimpleMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        # выполняется один раз при старте сервера

    def __call__(self, request):
        # код до view
        print("До view")
        response = self.get_response(request)
        # код после view
        print("После view")
        return response
```

- `get_response` — функция для вызова следующего middleware или view
- Код до `get_response` → до view  
- Код после `get_response` → после view

---

## 5️⃣ Старый стиль Middleware (deprecated)

```python
class OldStyleMiddleware:
    def process_request(self, request):
        # до view
        pass

    def process_response(self, request, response):
        # после view
        return response

    def process_exception(self, request, exception):
        # при исключении
        pass
```

- Сейчас рекомендуется использовать новый стиль

---

## 6️⃣ Кастомное Middleware примеры

### 🔹 Логирование запросов

```python
class LoggingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        print(f"Request path: {request.path}")
        response = self.get_response(request)
        print(f"Response status: {response.status_code}")
        return response
```

### 🔹 Проверка API токена

```python
class TokenAuthMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        token = request.headers.get('Authorization')
        if not token or token != 'secret-token':
            from django.http import HttpResponseForbidden
            return HttpResponseForbidden('Invalid token')
        return self.get_response(request)
```

---

## 7️⃣ Порядок обработки Middleware

```text
request -> M1 -> M2 -> M3 -> view
response <- M3 <- M2 <- M1
```

- **Порядок важен**: middleware выше в списке выполняется раньше для запроса, но позже для ответа

---

## 8️⃣ Middleware и exceptions

- Если view выбрасывает исключение, middleware может:
  - обработать его в `try/except`
  - вернуть свой Response
- Можно использовать метод `process_exception` в старом стиле

---

## 9️⃣ Best practices

- Минимизируй тяжелую работу в middleware (логирование, кеширование, auth)
- Middleware должны быть **легкими и быстрыми**
- Порядок подключения в `MIDDLEWARE` влияет на работу auth, sessions, csrf
- Для асинхронных view используй **async middleware**
- Используй middleware для **кросс-приложенной логики**, а не для бизнес-логики

---

## 🔟 Async Middleware

```python
class AsyncMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    async def __call__(self, request):
        print("Before async view")
        response = await self.get_response(request)
        print("After async view")
        return response
```

- Работает с `async def view`
- Можно комбинировать sync и async middleware
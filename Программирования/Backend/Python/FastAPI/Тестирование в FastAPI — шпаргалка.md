Тестирование позволяет убедиться, что ваше приложение работает корректно, что эндпоинты возвращают правильные данные, а бизнес-логика не ломается при изменениях.

FastAPI отлично интегрируется с **pytest** и **TestClient** из Starlette.

---

## 1️⃣ Установка необходимых библиотек

```bash
pip install pytest httpx
````

- `pytest` — основной тестовый фреймворк
    
- `httpx` — асинхронный клиент для тестов API (альтернатива TestClient)
    

---

## 2️⃣ Использование TestClient для синхронных тестов

```python
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert "Welcome" in response.text
```

- `client.get/post/put/delete` — методы HTTP
    
- Можно проверять статус, тело ответа, заголовки
    

---

## 3️⃣ Тестирование эндпоинтов с Pydantic схемами

```python
from app.schemas.user import UserOut

def test_get_user():
    response = client.get("/users/1")
    data = response.json()
    user = UserOut(**data)
    assert user.id == 1
```

- Используем Pydantic для **валидации структуры ответа**
    
- Помогает ловить ошибки на уровне схем
    

---

## 4️⃣ Асинхронные тесты с httpx.AsyncClient

```python
import pytest
from httpx import AsyncClient
from app.main import app

@pytest.mark.asyncio
async def test_async_get():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        response = await ac.get("/items/")
        assert response.status_code == 200
```

- Подходит для тестов **async эндпоинтов**
    
- Не блокирует основной поток
    

---

## 5️⃣ Тестирование с зависимостями (Depends)

Если эндпоинт зависит от БД или авторизации:

```python
from fastapi import Depends
from fastapi.testclient import TestClient
from app.main import app
from app.api.v1.dependencies import get_db

# Мокаем зависимость
def override_get_db():
    # Возвращаем тестовую сессию или фиктивный объект
    yield TestSession()

app.dependency_overrides[get_db] = override_get_db

client = TestClient(app)

def test_user_list():
    response = client.get("/users/")
    assert response.status_code == 200
```

- `dependency_overrides` позволяет **подменять реальные зависимости** на тестовые
    

---

## 6️⃣ Тестирование Background Tasks

```python
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_background_task():
    response = client.post("/send-notification/", json={"msg": "hello"})
    assert response.status_code == 200
    # Проверяем, что задача добавилась в очередь или файл был создан
```

- Можно проверять **результаты фоновых задач** через файлы, БД или мок-объекты
    

---

## 7️⃣ Организация тестов

Рекомендуемая структура:

```
tests/
├── __init__.py
├── conftest.py      # фикстуры и конфигурация pytest
├── test_users.py
├── test_items.py
└── test_auth.py
```

- `conftest.py` — место для фикстур (тестовые данные, DB сессии)
    
- Каждый файл тестов покрывает отдельный модуль приложения
    

---

## 8️⃣ Примеры фикстур

```python
import pytest
from app.main import app
from fastapi.testclient import TestClient

@pytest.fixture
def client():
    return TestClient(app)

@pytest.fixture
def test_user(client):
    response = client.post("/users/", json={"username": "test"})
    return response.json()
```

- Фикстуры позволяют **повторно использовать объекты** в тестах
    
- Удобно для пользователей, авторизации, сессий БД
    

---

## 9️⃣ Тестирование ошибок и исключений

```python
def test_not_found(client):
    response = client.get("/users/9999")
    assert response.status_code == 404
    assert response.json() == {"detail": "User not found"}
```

- Всегда проверяйте обработку ошибок
    
- Проверяйте как `status_code`, так и `detail`
    

---

## 🔟 Best practices

1. Разделяйте тесты на **unit и integration**
    
2. Используйте `TestClient` для синхронных эндпоинтов, `httpx.AsyncClient` для асинхронных
    
3. Мокируйте зависимости через `dependency_overrides`
    
4. Покрывайте ошибки и крайние случаи
    
5. Фикстуры помогают избежать дублирования кода
    
6. Тестируйте **BackgroundTasks** и асинхронные операции
    
7. Проверяйте как **структуру данных**, так и **статусы HTTP**
    

---

💡 **Итог:**  
FastAPI отлично подходит для тестирования через `pytest` и TestClient.  
Корректная организация тестов и использование фикстур обеспечивает **надежность и стабильность приложения**.

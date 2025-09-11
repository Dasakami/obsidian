FastAPI использует **dependency injection** через `Depends` для повторного использования кода, авторизации и подключения к базе данных.

---

## 1️⃣ Основы Depends

```python
from fastapi import FastAPI, Depends

app = FastAPI()

def common_parameters(q: str = None, skip: int = 0, limit: int = 10):
    return {"q": q, "skip": skip, "limit": limit}

@app.get("/items/")
def read_items(commons: dict = Depends(common_parameters)):
    return commons
````

- `Depends` позволяет использовать функции как зависимости
    
- `common_parameters` автоматически вызывается FastAPI
    
- Возвращаемое значение функции передаётся в эндпоинт
    

---

## 2️⃣ Зависимости с классами

```python
class CommonQuery:
    def __init__(self, q: str = None, skip: int = 0, limit: int = 10):
        self.q = q
        self.skip = skip
        self.limit = limit

@app.get("/items/")
def read_items(commons: CommonQuery = Depends()):
    return {"q": commons.q, "skip": commons.skip, "limit": commons.limit}
```

- Можно использовать классы для зависимостей с состоянием
    

---

## 3️⃣ Подключение базы данных

```python
from fastapi import Depends
from sqlalchemy.orm import Session
from app.db.session import SessionLocal

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/users/")
def read_users(db: Session = Depends(get_db)):
    return db.query(User).all()
```

- `yield` позволяет автоматически закрывать сессию после запроса
    
- Каждый роут, который использует `Depends(get_db)`, получает рабочую сессию
    

---

## 4️⃣ Авторизация через Depends

```python
from fastapi import Depends, HTTPException, status

def get_current_user(token: str = Depends(oauth2_scheme)):
    user = verify_token(token)
    if not user:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Invalid token")
    return user

@app.get("/me")
def read_me(current_user: User = Depends(get_current_user)):
    return current_user
```

- `Depends` удобно использовать для авторизации
    
- Все проверки выносим в отдельные функции
    

---

## 5️⃣ Вложенные зависимости

```python
def common_params(q: str = None):
    return q

def query_or_default(q: str = Depends(common_params)):
    return q or "default"

@app.get("/items/")
def read_items(q: str = Depends(query_or_default)):
    return {"q": q}
```

- Зависимости могут вызывать другие зависимости
    
- Позволяет строить цепочки DI
    

---

## 6️⃣ Скоупы зависимостей

- **per-request** — по умолчанию: новая функция на каждый запрос
    
- **singleton / global** — можно использовать объекты классов или cached_property
    

```python
from functools import lru_cache
from app.config import Settings

@lru_cache()
def get_settings():
    return Settings()
```

- Настройки приложения удобно выносить в singleton через lru_cache
    

---

## 🔹 Best practices

1. Используйте **Depends** для DB, авторизации, общих параметров
    
2. Выносите логику в отдельные функции или сервисы
    
3. Для сложных зависимостей используйте **классы**
    
4. Используйте вложенные зависимости для повторного кода
    
5. Закрывайте ресурсы через `yield` (DB, файлы)
    
6. Для конфигураций используйте singleton с `lru_cache()`
    

---

💡 **Итог:**  
`Depends` делает FastAPI **чистым и модульным**, позволяет централизовать работу с DB, авторизацией, параметрами запросов и другими ресурсами.

Эта шпаргалка содержит полезные советы, трюки и рекомендации для более эффективной работы с FastAPI.

---

## 1️⃣ Использование Pydantic моделей

- **ORM mode** для сериализации объектов SQLAlchemy:
```python
from pydantic import BaseModel

class UserOut(BaseModel):
    id: int
    username: str

    class Config:
        orm_mode = True
````

- Позволяет напрямую возвращать объекты SQLAlchemy без преобразования в dict.
    
- **Валидация данных** через Pydantic:
    

```python
class UserCreate(BaseModel):
    username: str
    email: str

    @validator("username")
    def name_not_empty(cls, v):
        if not v:
            raise ValueError("Username cannot be empty")
        return v
```

---

## 2️⃣ Быстрое тестирование API

- Используй `TestClient` для unit-тестов:
    

```python
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
```

- Позволяет тестировать роуты без запуска сервера.
    

---

## 3️⃣ Background tasks и асинхронность

- Для неблокирующих задач используем `BackgroundTasks`:
    

```python
from fastapi import BackgroundTasks

def write_log(msg: str):
    with open("log.txt", "a") as f:
        f.write(msg + "\n")

@app.post("/send/")
def send(background_tasks: BackgroundTasks, msg: str):
    background_tasks.add_task(write_log, msg)
    return {"status": "ok"}
```

- Для тяжелых или критичных задач — лучше использовать **Celery**.
    

---

## 4️⃣ Dependency Injection и Depends

- Используй `Depends` для:
    
    - Подключения к БД
        
    - Авторизации
        
    - Общих параметров запросов
        

```python
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

- Позволяет **чисто разделять логику** между роутами и сервисами.
    

---

## 5️⃣ Работа с middleware

- Middleware — удобно для логирования, CORS и обработки ошибок:
    

```python
@app.middleware("http")
async def log_requests(request, call_next):
    response = await call_next(request)
    print(request.method, request.url)
    return response
```

- Не используйте middleware для авторизации или работы с DB — лучше Depends.
    

---

## 6️⃣ Pagination

- Используй `fastapi-pagination` для готовой пагинации:
    

```python
from fastapi_pagination import add_pagination, Page, paginate

@app.get("/items/", response_model=Page[Item])
def get_items():
    return paginate(items)

add_pagination(app)
```

- Автоматически добавляет `page` и `size` параметры.
    

---

## 7️⃣ Swagger и OpenAPI

- **Теги и описание для документации**:
    

```python
@app.get("/users/", tags=["Users"], summary="Список всех пользователей")
def read_users():
    ...
```

- Можно кастомизировать документацию:
    

```python
app = FastAPI(
    title="My API",
    description="Документация API для проекта",
    version="1.0.0",
    docs_url="/swagger",
    redoc_url="/redoc"
)
```

---

## 8️⃣ Асинхронность и SQLAlchemy

- Для async запросов используем `AsyncSession` и `asyncpg`:
    

```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine

engine = create_async_engine(DATABASE_URL, echo=True)
AsyncSessionLocal = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)
```

- Не блокируйте основной поток тяжелыми sync-операциями.
    

---

## 9️⃣ Обработка ошибок

- Используем `HTTPException`:
    

```python
from fastapi import HTTPException, status

@app.get("/items/{item_id}")
def read_item(item_id: int):
    if item_id != 1:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Item not found")
    return {"item_id": item_id}
```

- Можно создавать **глобальные обработчики** через middleware.
    

---

## 🔟 Кэширование и оптимизация

- Для уменьшения нагрузки можно использовать:
    
    - **Redis** для кэша
        
    - `lru_cache` для функций
        

```python
from functools import lru_cache

@lru_cache()
def get_settings():
    return Settings()
```

- Особенно полезно для конфигураций и внешних запросов.
    

---

## 1️⃣1️⃣ Лайфхаки

1. Разделяй **роуты через APIRouter** и версии API (`v1`, `v2`)
    
2. Используй `response_model` для всех публичных эндпоинтов
    
3. Настраивай **logging** через стандартный модуль logging
    
4. Для сложной логики создавай **слой services** между роутами и БД
    
5. Всегда используйте **async** где это возможно для масштабируемости
    
6. Проверяй зависимости через `Depends` и валидируй входные данные
    
7. Для больших файлов и тяжелых операций используйте **StreamingResponse**
    

---

💡 **Итог:**  
Эти советы помогают писать **чистый, быстрый и поддерживаемый код** на FastAPI.  
Правильное использование Pydantic, Depends, BackgroundTasks и pagination делает проект готовым к продакшну.

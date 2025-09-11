FastAPI использует **роуты** для обработки HTTP-запросов.  
Эндпоинт — это URL + HTTP-метод, который выполняет определённую функцию.

---

## 1️⃣ Основы роутов

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello, FastAPI!"}

@app.post("/items/")
def create_item(name: str, price: float):
    return {"name": name, "price": price}
````

- `@app.get("/path")` — GET запрос
    
- `@app.post("/path")` — POST запрос
    
- `@app.put("/path")` — PUT запрос
    
- `@app.delete("/path")` — DELETE запрос
    

---

## 2️⃣ Path parameters (параметры пути)

```python
@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}
```

- `item_id: int` — FastAPI автоматически проверяет тип
    
- Path parameters используются для идентификации конкретного объекта
    

---

## 3️⃣ Query parameters (параметры строки запроса)

```python
@app.get("/items/")
def read_items(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}
```

- Параметры после `?` в URL
    
- Можно задавать значения по умолчанию
    

---

## 4️⃣ Body parameters (тело запроса)

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float

@app.post("/items/")
def create_item(item: Item):
    return {"item_name": item.name, "item_price": item.price}
```

- FastAPI автоматически парсит JSON тело запроса в Pydantic модель
    

---

## 5️⃣ Response Model

```python
from fastapi import FastAPI
from pydantic import BaseModel

class ItemOut(BaseModel):
    name: str
    price: float

@app.get("/items/{item_id}", response_model=ItemOut)
def read_item(item_id: int):
    return {"name": "Book", "price": 12.5, "extra_field": "ignored"}
```

- `response_model` сериализует ответ
    
- Поля, которых нет в Pydantic, **игнорируются**
    

---

## 6️⃣ APIRouter (разделение роутов)

```python
from fastapi import APIRouter

router = APIRouter()

@router.get("/users/")
def read_users():
    return [{"username": "Dan"}]

@router.post("/users/")
def create_user():
    return {"message": "User created"}
```

- В `main.py` подключаем роутер:
    

```python
from fastapi import FastAPI
from app.api.v1.routers import users

app = FastAPI()
app.include_router(users.router, prefix="/api/v1/users", tags=["users"])
```

- `prefix` — общий путь
    
- `tags` — для автодокументации Swagger
    

---

## 7️⃣ Path + Query + Body вместе

```python
@app.put("/items/{item_id}")
def update_item(item_id: int, item: Item, q: str = None):
    return {"item_id": item_id, "item_name": item.name, "query": q}
```

- Можно комбинировать path, query и body параметры
    

---

## 8️⃣ Status codes и HTTPException

```python
from fastapi import HTTPException, status

@app.get("/items/{item_id}")
def read_item(item_id: int):
    if item_id != 1:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Item not found")
    return {"item_id": item_id}
```

- `HTTPException` позволяет возвращать ошибки с нужным кодом и сообщением
    

---

## 🔹 Best practices

1. Разделяйте роуты с помощью **APIRouter**
    
2. Используйте **response_model** для сериализации ответа
    
3. Чётко разделяйте **path, query и body параметры**
    
4. Обрабатывайте ошибки через **HTTPException**
    
5. Группируйте роуты по логике и версиям API (`v1`, `v2`)
    
6. Используйте **tags** для автодокументации Swagger
    
7. Для больших проектов создавайте отдельный слой **services** для логики, чтобы роуты оставались чистыми
    

---

💡 **Итог:**  
FastAPI роуты позволяют **чисто и быстро обрабатывать HTTP-запросы**, комбинируя path, query и body параметры, используя Pydantic для валидации и APIRouter для структуры проекта.
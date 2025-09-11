Пагинация (Pagination) используется для **разделения больших наборов данных на страницы**, чтобы:

- уменьшить нагрузку на сервер и клиент
- ускорить ответы API
- улучшить UX при отображении списков

---

## 1️⃣ Ручная пагинация (limit/offset)

```python
from fastapi import FastAPI, Query
from typing import List

app = FastAPI()

# Пример данных
items = [{"id": i, "name": f"Item {i}"} for i in range(1, 101)]

@app.get("/items/")
def read_items(skip: int = 0, limit: int = 10):
    """
    skip  - сколько элементов пропустить
    limit - сколько элементов вернуть
    """
    return items[skip : skip + limit]
````

- `skip` — offset, пропустить N элементов
    
- `limit` — сколько элементов вернуть на страницу
    
- Подходит для небольших проектов
    

---

## 2️⃣ Параметры запроса для страниц (page/page_size)

```python
@app.get("/items/")
def read_items(page: int = 1, page_size: int = 10):
    start = (page - 1) * page_size
    end = start + page_size
    return items[start:end]
```

- `page` — номер страницы
    
- `page_size` — количество элементов на странице
    
- Логика: `start = (page - 1) * page_size`
    

---

## 3️⃣ Pydantic схема для пагинации

```python
from pydantic import BaseModel
from typing import List

class Item(BaseModel):
    id: int
    name: str

class PaginatedResponse(BaseModel):
    total: int         # Общее количество элементов
    page: int          # Текущая страница
    page_size: int     # Размер страницы
    items: List[Item]  # Список объектов на странице

@app.get("/items/", response_model=PaginatedResponse)
def get_items(page: int = 1, page_size: int = 10):
    start = (page - 1) * page_size
    end = start + page_size
    return {
        "total": len(items),
        "page": page,
        "page_size": page_size,
        "items": items[start:end]
    }
```

- Удобно для фронтенда — легко строить навигацию по страницам
    
- `total` позволяет показывать общее количество страниц
    

---

## 4️⃣ Использование библиотеки `fastapi-pagination`

Установка:

```bash
pip install fastapi-pagination
```

Пример использования:

```python
from fastapi import FastAPI
from fastapi_pagination import Page, add_pagination, paginate
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    id: int
    name: str

items = [Item(id=i, name=f"Item {i}") for i in range(1, 101)]

@app.get("/items/", response_model=Page[Item])
def get_items():
    return paginate(items)

add_pagination(app)
```

- Автоматически добавляет параметры `page` и `size`
    
- Возвращает структуру с `items`, `total`, `page`, `size`
    
- Поддерживает списки и SQLAlchemy query
    

---

## 5️⃣ Pagination с SQLAlchemy

```python
from sqlalchemy.orm import Session
from fastapi import Depends
from fastapi_pagination.ext.sqlalchemy import paginate, paginate_query
from app.db.models import Item
from app.db.session import get_db
from fastapi_pagination import Page

@app.get("/items/", response_model=Page[Item])
def get_items(db: Session = Depends(get_db)):
    query = db.query(Item)
    return paginate(query)
```

- `paginate(query)` автоматически делает `LIMIT/OFFSET` для БД
    
- Работает с SQLAlchemy ORM, упрощая реализацию постраничной выдачи
    

---

## 6️⃣ Дополнительные возможности `fastapi-pagination`

- Сортировка:
    

```python
from fastapi_pagination.ext.sqlalchemy import paginate

@app.get("/items/", response_model=Page[Item])
def get_items(db: Session = Depends(get_db)):
    query = db.query(Item).order_by(Item.id.desc())
    return paginate(query)
```

- Фильтрация:
    

```python
@app.get("/items/", response_model=Page[Item])
def get_items(db: Session = Depends(get_db), active: bool = True):
    query = db.query(Item).filter(Item.is_active == active)
    return paginate(query)
```

- Настройка размера страницы по умолчанию:
    

```python
from fastapi_pagination import LimitOffsetPage, add_pagination

@app.get("/items/", response_model=LimitOffsetPage[Item])
def get_items(db: Session = Depends(get_db)):
    query = db.query(Item)
    return paginate(query)
```

---

## 🔹 Best practices

1. Использовать **fastapi-pagination** для удобства и консистентности
    
2. Возвращать `total` для фронтенда
    
3. Ограничивать `page_size` (например, max 100), чтобы не перегружать сервер
    
4. Для больших наборов данных — использовать пагинацию на уровне БД (`LIMIT/OFFSET`)
    
5. Комбинировать с фильтрацией и сортировкой
    
6. Разделять схемы для **request/response**, особенно для API с пагинацией
    

---

💡 **Итог:**  
Пагинация в FastAPI позволяет безопасно и удобно отдавать большие наборы данных.  
Можно делать вручную через `skip/limit` или использовать готовую библиотеку `fastapi-pagination` для автоматизации.

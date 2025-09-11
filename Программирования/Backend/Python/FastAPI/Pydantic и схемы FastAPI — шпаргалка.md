FastAPI использует **Pydantic** для валидации данных и сериализации.  
Все входящие и исходящие данные проходят через Pydantic модели (схемы).

---

## 1️⃣ Основы Pydantic

```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    username: str
    is_active: bool = True  # значение по умолчанию
````

- Наследуемся от `BaseModel`
    
- Атрибуты автоматически валидируются по типу
    
- Поддерживаются значения по умолчанию
    

---

## 2️⃣ Валидация данных

```python
from pydantic import BaseModel, EmailStr, constr

class UserCreate(BaseModel):
    username: constr(min_length=3, max_length=50)
    email: EmailStr
    password: constr(min_length=6)
```

- `EmailStr` — проверяет корректность email
    
- `constr` — ограничение строк (длина, regex)
    
- Типизация автоматически проверяет int, float, bool, list, dict
    

---

## 3️⃣ Дополнительные валидаторы

```python
from pydantic import BaseModel, validator

class UserCreate(BaseModel):
    username: str
    password: str

    @validator("password")
    def password_strength(cls, v):
        if len(v) < 6:
            raise ValueError("Пароль должен быть минимум 6 символов")
        return v
```

- `@validator` — кастомные проверки полей
    
- Можно проверять несколько полей через `@root_validator`
    

---

## 4️⃣ Pydantic и FastAPI роуты

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

@app.post("/items/")
def create_item(item: Item):
    return {"item_name": item.name, "item_price": item.price}
```

- `item: Item` — FastAPI автоматически валидирует запрос
    
- Ответ можно тоже сериализовать через `response_model`
    

```python
@app.get("/items/{item_id}", response_model=Item)
def read_item(item_id: int):
    return {"name": "Book", "price": 12.5}
```

---

## 5️⃣ Вложенные схемы

```python
class Address(BaseModel):
    city: str
    street: str

class User(BaseModel):
    username: str
    address: Address

user = User(username="Dan", address={"city": "Bishkek", "street": "Main St"})
```

- Можно использовать словари для вложенных объектов
    
- Pydantic автоматически конвертирует dict → объект
    

---

## 6️⃣ Дополнительные возможности

- **alias** — другое имя поля для внешнего API
    

```python
class User(BaseModel):
    user_name: str = Field(..., alias="username")
```

- **ORM mode** — для работы с SQLAlchemy моделями
    

```python
class UserOut(BaseModel):
    id: int
    username: str

    class Config:
        orm_mode = True
```

- **Конфиг Pydantic**
    

```python
class Config:
    anystr_strip_whitespace = True   # убирает пробелы
    min_anystr_length = 1            # минимальная длина строки
```

---

## 7️⃣ Работа с списками и словарями

```python
from typing import List, Dict

class ItemList(BaseModel):
    items: List[str]
    prices: Dict[str, float]
```

- `List[str]` — список строк
    
- `Dict[str, float]` — словарь с ключами str и значениями float
    

---

## 8️⃣ Best practices

1. Всегда используйте **схемы для запросов и ответов**
    
2. Для ответов используйте `response_model` в роуте
    
3. Используйте `orm_mode = True` при работе с SQLAlchemy
    
4. Пишите кастомные валидаторы для сложных правил
    
5. Разделяйте схемы для **создания, обновления и вывода**
    
    - `UserCreate`, `UserUpdate`, `UserOut`
        

---

💡 **Итог:**  
Pydantic делает FastAPI мощным: **валидация, сериализация, автодокументация**.  
Правильное использование схем повышает безопасность и удобство разработки.

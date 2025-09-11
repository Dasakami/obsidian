FastAPI использует **SQLAlchemy** как основной ORM для работы с базой данных.  
ORM позволяет работать с таблицами как с объектами Python.

---

## 1️⃣ Установка

```bash
pip install sqlalchemy
pip install databases   # если нужен async доступ
pip install psycopg2   # PostgreSQL драйвер
pip install alembic    # миграции
````

---

## 2️⃣ Базовая модель

```python
from sqlalchemy import Column, Integer, String, Boolean
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    username = Column(String, unique=True, index=True, nullable=False)
    email = Column(String, unique=True, index=True, nullable=False)
    is_active = Column(Boolean, default=True)
```

- `__tablename__` — имя таблицы в БД
    
- `Column` — поле таблицы
    
- `primary_key=True` — первичный ключ
    
- `index=True` — создаёт индекс
    

---

## 3️⃣ Подключение к базе и сессии

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"

engine = create_engine(
    SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False}
)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
```

- `engine` — соединение с БД
    
- `SessionLocal` — фабрика сессий для CRUD
    

---

## 4️⃣ Dependency для FastAPI

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
```

- Используется через `Depends(get_db)` в роутерах
    

---

## 5️⃣ CRUD операции

```python
from sqlalchemy.orm import Session
from app.db.models.user import User

# Create
def create_user(db: Session, username: str, email: str):
    user = User(username=username, email=email)
    db.add(user)
    db.commit()
    db.refresh(user)
    return user

# Read
def get_user(db: Session, user_id: int):
    return db.query(User).filter(User.id == user_id).first()

# Update
def update_user(db: Session, user: User, username: str):
    user.username = username
    db.commit()
    db.refresh(user)
    return user

# Delete
def delete_user(db: Session, user: User):
    db.delete(user)
    db.commit()
```

- `db.add()` — добавляет объект в сессию
    
- `db.commit()` — сохраняет изменения
    
- `db.refresh()` — обновляет объект после коммита
    
- `db.delete()` — удаляет объект
    

---

## 6️⃣ Работа с фильтрами

```python
users = db.query(User).filter(User.is_active == True).all()
user = db.query(User).filter(User.username == "Dan").first()
```

- `filter()` — условия WHERE
    
- `all()` — список всех объектов
    
- `first()` — первый объект или None
    

---

## 7️⃣ Async SQLAlchemy (опционально)

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "postgresql+asyncpg://user:pass@localhost/db"

engine = create_async_engine(DATABASE_URL, echo=True)
AsyncSessionLocal = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)
```

- Используется для **асинхронных операций**
    
- Подключается через `async def get_db():` и `async with db as session:`
    

---

## 8️⃣ Alembic — миграции

- Инициализация:
    

```bash
alembic init migrations
```

- Настройка `alembic.ini` и `env.py` с `Base.metadata`
    
- Создание миграций:
    

```bash
alembic revision --autogenerate -m "Initial"
alembic upgrade head
```

---

## 🔹 Best practices

1. Разделяй **models, schemas, services**
    
2. Всегда используйте **dependency get_db()**
    
3. Для сложных проектов — создавай отдельный слой **services** для CRUD
    
4. Используй `db.refresh()` после commit, чтобы получить обновлённый объект
    
5. Используй **ORM mode в Pydantic** для сериализации SQLAlchemy объектов
    
6. Для асинхронного FastAPI используй async SQLAlchemy и asyncpg
    

---

💡 **Итог:**  
SQLAlchemy в FastAPI — это мощный инструмент для работы с БД, с поддержкой sync и async, легко интегрируется с Pydantic и роутами.

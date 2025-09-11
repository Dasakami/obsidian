Middleware — это функция, которая выполняется **между запросом и ответом**.  
Позволяет обрабатывать запросы глобально: логирование, авторизация, CORS, обработка ошибок и т.д.

---

## 1️⃣ Базовый middleware

```python
from fastapi import FastAPI, Request

app = FastAPI()

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    print(f"Request: {request.method} {request.url}")
    response = await call_next(request)
    response.headers["X-Custom-Header"] = "FastAPI"
    return response
````

- `@app.middleware("http")` — декоратор для middleware
    
- `call_next(request)` — передаёт запрос дальше
    
- Можно модифицировать запрос и ответ
    

---

## 2️⃣ Использование BaseHTTPMiddleware

```python
from starlette.middleware.base import BaseHTTPMiddleware
from fastapi import FastAPI

class CustomMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        print("Before request")
        response = await call_next(request)
        print("After request")
        return response

app = FastAPI()
app.add_middleware(CustomMiddleware)
```

- Позволяет создавать **классовые middleware**
    
- Можно использовать состояние объекта
    

---

## 3️⃣ CORS Middleware

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Можно указать список доменов
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

- Разрешает кросс-доменные запросы (CORS)
    
- Важно для фронтенда, когда API и клиент на разных доменах
    

---

## 4️⃣ Обработка ошибок через middleware

```python
from fastapi.responses import JSONResponse
from fastapi import Request

@app.middleware("http")
async def catch_exceptions_middleware(request: Request, call_next):
    try:
        return await call_next(request)
    except Exception as e:
        return JSONResponse(status_code=500, content={"detail": str(e)})
```

- Middleware может перехватывать ошибки глобально
    
- Можно возвращать кастомные ответы
    

---

## 5️⃣ Логирование запросов и ответов

```python
import time
from fastapi import Request

@app.middleware("http")
async def log_requests(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    print(f"{request.method} {request.url} completed in {process_time:.2f}s")
    return response
```

- Полезно для мониторинга производительности
    
- Можно сохранять логи в базу или файл
    

---

## 🔹 Best practices

1. Middleware выполняется **для всех запросов** → используйте экономно
    
2. Для авторизации и DB лучше использовать **Depends**, а не middleware
    
3. Middleware удобно использовать для **логирования, CORS, обработки ошибок**
    
4. Для сложных кейсов используйте **BaseHTTPMiddleware классы**
    
5. Порядок добавления middleware важен: они выполняются сверху вниз
    

---

💡 **Итог:**  
Middleware в FastAPI — мощный инструмент для глобальной обработки запросов, логирования, CORS и обработки ошибок.
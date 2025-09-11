В веб-приложениях есть два типа файлов:  

- **Static** — неизменяемые файлы: CSS, JS, изображения для фронтенда  
- **Media** — загружаемые пользователями файлы: аватары, документы, изображения  

FastAPI позволяет работать с обоими типами через `StaticFiles` и ручные эндпоинты.

---

## 1️⃣ Подключение Static файлов

FastAPI использует `StaticFiles` из Starlette:

```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles

app = FastAPI()

# Подключение статических файлов
app.mount("/static", StaticFiles(directory="app/static"), name="static")
````

- `directory` — путь к папке со статикой
    
- `name` — имя монтированного маршрута
    
- `/static/...` — URL, по которому будут доступны файлы
    

Пример структуры статических файлов:

```
app/static/
├── css/
│   └── style.css
├── js/
│   └── script.js
└── images/
    └── logo.png
```

Теперь можно обращаться к `/static/css/style.css`.

---

## 2️⃣ Раздача медиа-файлов (Media)

- Для файлов, которые загружают пользователи, используем отдельную папку:
    

```
app/media/
├── avatars/
├── uploads/
```

- Эндпоинт для отдачи файлов:
    

```python
from fastapi.staticfiles import StaticFiles

app.mount("/media", StaticFiles(directory="app/media"), name="media")
```

Теперь можно получить файл через `/media/avatars/user1.png`.

---

## 3️⃣ Загрузка файлов через эндпоинт

```python
from fastapi import UploadFile, File

@app.post("/upload/")
async def upload_file(file: UploadFile = File(...)):
    file_location = f"app/media/uploads/{file.filename}"
    with open(file_location, "wb") as f:
        f.write(await file.read())
    return {"filename": file.filename, "url": f"/media/uploads/{file.filename}"}
```

- `UploadFile` — асинхронная работа с файлами
    
- `file.read()` — чтение содержимого файла
    
- Можно хранить **имя файла, путь и URL** в базе данных
    

---

## 4️⃣ Ограничение типов файлов

```python
@app.post("/upload/")
async def upload_file(file: UploadFile = File(...)):
    if file.content_type not in ["image/jpeg", "image/png"]:
        return {"error": "Invalid file type"}
    file_location = f"app/media/uploads/{file.filename}"
    with open(file_location, "wb") as f:
        f.write(await file.read())
    return {"filename": file.filename}
```

- Проверка MIME типа файла
    
- Полезно для безопасности
    

---

## 5️⃣ Использование Pydantic для meta-информации

```python
from pydantic import BaseModel

class FileResponse(BaseModel):
    filename: str
    url: str
    size: int

@app.post("/upload/", response_model=FileResponse)
async def upload_file(file: UploadFile = File(...)):
    content = await file.read()
    file_location = f"app/media/uploads/{file.filename}"
    with open(file_location, "wb") as f:
        f.write(content)
    return {
        "filename": file.filename,
        "url": f"/media/uploads/{file.filename}",
        "size": len(content)
    }
```

- Можно возвращать **размер файла, URL и имя**
    
- Удобно для фронтенда
    

---

## 6️⃣ StreamingResponse для больших файлов

```python
from fastapi.responses import StreamingResponse

@app.get("/download/{file_name}")
def download_file(file_name: str):
    file_path = f"app/media/uploads/{file_name}"
    file_like = open(file_path, mode="rb")
    return StreamingResponse(file_like, media_type="application/octet-stream")
```

- Не загружает весь файл в память
    
- Используется для больших медиа и видео
    

---

## 7️⃣ Best practices

1. Разделяй **static** и **media** файлы
    
2. Храни загруженные файлы в отдельной папке (`media`) и не смешивай со статикой
    
3. Используй `UploadFile` для асинхронной загрузки
    
4. Проверяй MIME типы и размер файлов для безопасности
    
5. Для больших файлов отдавай через `StreamingResponse`
    
6. Можно добавлять уникальные имена файлов (`uuid4`) для предотвращения коллизий
    
7. Для продакшена — отдавать статические и медиа файлы через **Nginx или CDN**
    

---

💡 **Итог:**  
FastAPI позволяет легко работать со **статикой и медиа**.  
StaticFiles и UploadFile + StreamingResponse покрывают большинство задач по работе с файлами.

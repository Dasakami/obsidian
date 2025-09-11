Middleware — это **промежуточный слой**, который обрабатывает **входящие обновления (update)** до передачи их хендлерам.  
Используется для:

- Проверки прав доступа пользователей  
- Логирования действий  
- Модификации сообщений или callback данных  
- Добавления дополнительной информации в context

---

## 1️⃣ Что такое Middleware в Aiogram 3.x

- Подключается к Dispatcher через `dp.update.middleware(...)`  
- Может быть асинхронным (`async`)  
- Каждый Middleware получает:
  - `update` — объект обновления (Message, CallbackQuery и др.)  
  - `data` — словарь, который передается хендлеру  
- Middleware может:
  - Прервать дальнейшую обработку (`raise CancelHandler()`)  
  - Изменить `update` или добавить данные в `data`

---

## 2️⃣ Пример AuthMiddleware

```python
from aiogram import BaseMiddleware
from aiogram.types import Message
from aiogram.dispatcher.event.handler import CancelHandler
from bot.config import settings

class AuthMiddleware(BaseMiddleware):
    """
    Middleware для проверки доступа пользователя
    """
    async def __call__(self, handler, event: Message, data: dict):
        user_id = event.from_user.id

        # Проверка, есть ли пользователь в базе (можно добавить)
        # user = await get_user(user_id)

        # Проверка на доступ (например, только админ)
        if user_id not in settings.admin_ids:
            await event.answer("❌ У вас нет доступа к этой команде.")
            raise CancelHandler()  # Прерываем дальнейшую обработку

        # Можно передать данные в хендлер через data
        data["user_id"] = user_id

        # Вызываем хендлер
        return await handler(event, data)
````

---

## 3️⃣ Подключение Middleware в main.py

```python
from bot.middlewares.auth import AuthMiddleware

dp.update.middleware(AuthMiddleware())
```

- Все входящие сообщения проходят через middleware перед хендлером
    
- Можно использовать несколько middleware, они выполняются в порядке подключения
    

---

## 4️⃣ Зачем использовать Middleware для Auth

1. Не дублируем проверку в каждом хендлере
    
2. Централизованная проверка доступа
    
3. Можно динамически менять правила доступа (например, добавлять новых админов)
    
4. Легко интегрировать с FSM и базой данных
    

---

## 5️⃣ Best practices

- Используйте **асинхронные методы** для работы с БД или API
    
- Middleware должен быть **легким и быстрым**
    
- Для многоуровневого доступа (админ, модератор, пользователь) создавайте **несколько middleware** или параметризуйте их
    
- Добавляйте нужные данные в `data`, чтобы хендлеры могли их использовать без повторных запросов
    
- Если пользователь не имеет прав — прерывайте хендлер через `CancelHandler`
    
- Логику хранения пользователей/прав лучше выносить в сервисы (`services/user_service.py`)
    

---

## 6️⃣ Пример расширенной логики

```python
class AdminMiddleware(BaseMiddleware):
    async def __call__(self, handler, event: Message, data: dict):
        user_id = event.from_user.id

        is_admin = user_id in settings.admin_ids
        data["is_admin"] = is_admin

        # Если не админ, можно разрешить только определённые команды
        if not is_admin and event.text.startswith("/admin"):
            await event.answer("❌ Только для админов!")
            raise CancelHandler()

        return await handler(event, data)
```

- Middleware может быть универсальным и передавать флаги (`is_admin`) для любого хендлера
    
- Упрощает проверку доступа во всех местах проекта
    

---

## 7️⃣ Итог

- Middleware — промежуточный слой между Telegram и хендлером
    
- Используется для **авторизации, логирования и модификации данных**
    
- Прерывает обработку при необходимости через `CancelHandler`
    
- Позволяет централизованно управлять доступом и добавлять контекст в хендлеры
    
- Чистый код и масштабируемость проекта достигаются через правильное использование middleware
    

```

---

Если хочешь, следующим логично сделать **09_Services/01_DB_Service.md**, чтобы подробно разобрать **работу с базой данных через сервисы для бота**.  

Хочешь, чтобы я сделал его следующим?
```
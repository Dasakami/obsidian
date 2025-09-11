Логирование — важная часть любого бота.  
Middleware позволяет **централизованно фиксировать все входящие сообщения, команды, ошибки и callback**, прежде чем они попадут в хендлеры.

---

## 1️⃣ Зачем нужен Logging Middleware

- Отслеживание всех действий пользователей  
- Отладка ошибок и unexpected behavior  
- Сбор статистики по командам и сообщениям  
- Лёгкая интеграция с внешними сервисами (Sentry, Prometheus, файл, база данных)

---

## 2️⃣ Пример LoggingMiddleware

```python
import logging
from aiogram import BaseMiddleware
from aiogram.types import Message, CallbackQuery

logger = logging.getLogger(__name__)

class LoggingMiddleware(BaseMiddleware):
    """
    Middleware для логирования сообщений и callback
    """
    async def __call__(self, handler, event, data: dict):
        # Проверяем тип обновления
        if isinstance(event, Message):
            user_id = event.from_user.id
            text = event.text
            logger.info(f"Message from {user_id}: {text}")
        elif isinstance(event, CallbackQuery):
            user_id = event.from_user.id
            data_text = event.data
            logger.info(f"CallbackQuery from {user_id}: {data_text}")

        # Можно добавить в data для хендлеров
        data["log_user_id"] = event.from_user.id

        # Передаем управление хендлеру
        return await handler(event, data)
````

---

## 3️⃣ Подключение Middleware

В `main.py`:

```python
from bot.middlewares.logging import LoggingMiddleware

dp.update.middleware(LoggingMiddleware())
```

- Все сообщения и callback проходят через middleware
    
- Логи можно записывать в файл, базу данных или внешние сервисы
    

---

## 4️⃣ Настройка Python logging

Пример настройки логирования:

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
    handlers=[
        logging.FileHandler("bot.log", encoding="utf-8"),
        logging.StreamHandler()
    ]
)
```

- `FileHandler` — записывает логи в файл
    
- `StreamHandler` — выводит в консоль
    
- `level=logging.INFO` — уровень логирования (DEBUG, INFO, WARNING, ERROR)
    

---

## 5️⃣ Расширенный пример с уровнем DEBUG и обработкой ошибок

```python
class AdvancedLoggingMiddleware(BaseMiddleware):
    async def __call__(self, handler, event, data: dict):
        try:
            if isinstance(event, Message):
                logger.debug(f"[DEBUG] Message from {event.from_user.id}: {event.text}")
            elif isinstance(event, CallbackQuery):
                logger.debug(f"[DEBUG] CallbackQuery from {event.from_user.id}: {event.data}")
            
            return await handler(event, data)

        except Exception as e:
            logger.error(f"Error in handler: {e}", exc_info=True)
            raise
```

- `exc_info=True` добавляет трассировку ошибки в лог
    
- Полезно для дебага и продакшена
    

---

## 6️⃣ Best practices

1. Используйте **централизованное логирование через Middleware**
    
2. Логи должны содержать: ID пользователя, тип события, текст/данные, время
    
3. Ошибки и исключения всегда логируйте с `exc_info=True`
    
4. Для продакшена используйте ротацию файлов логов (`RotatingFileHandler`)
    
5. Не храните в логах токены и конфиденциальные данные
    
6. Можно интегрировать с Sentry или другими сервисами мониторинга
    

---

## 7️⃣ Итог

- Logging Middleware фиксирует все сообщения и callback до хендлеров
    
- Позволяет централизованно управлять логами
    
- Улучшает отладку, мониторинг и безопасность бота
    
- Best practice — лёгкий, быстрый, не блокирующий хендлеры
    

```

---

Если хочешь, следующим логично сделать **09_Services/01_DB_Service.md**, чтобы подробно разобрать **сервис для работы с базой данных в боте**.  

Хочешь, чтобы я сделал его следующим?
```
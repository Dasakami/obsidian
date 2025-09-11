`main.py` — это **точка входа** бота. Здесь мы запускаем приложение, регистрируем хендлеры, middlewares и клавиатуры.  

Правильная организация main.py делает проект **масштабируемым, читаемым и удобным для команды**.

---

## 1️⃣ Основная структура main.py

```python
from aiogram import Bot, Dispatcher
from aiogram.types import BotCommand
from aiogram.fsm.storage.memory import MemoryStorage
from bot.config import settings
from bot.handlers import start, help, admin
from bot.middlewares import auth

# Создание бота
bot = Bot(token=settings.bot_token, parse_mode="HTML")

# Хранилище для FSM (Finite State Machine)
storage = MemoryStorage()

# Диспетчер
dp = Dispatcher(storage=storage)

# Регистрация middlewares
dp.update.middleware(auth.AuthMiddleware())

# Регистрация хендлеров
start.register_handlers(dp)
help.register_handlers(dp)
admin.register_handlers(dp)

# Настройка команд бота (меню команд в Telegram)
async def set_commands(bot: Bot):
    commands = [
        BotCommand(command="/start", description="Запуск бота"),
        BotCommand(command="/help", description="Помощь"),
        BotCommand(command="/admin", description="Админ команды"),
    ]
    await bot.set_my_commands(commands)

# Точка входа
if __name__ == "__main__":
    import asyncio
    from aiogram import F

    async def main():
        # Установка команд
        await set_commands(bot)
        # Запуск бота
        try:
            print("Bot is starting...")
            await dp.start_polling(bot)
        finally:
            await bot.session.close()

    asyncio.run(main())
````

---

## 2️⃣ Разбор компонентов

### 2.1 `Bot`

- Создаёт объект бота с токеном
    
- Параметр `parse_mode="HTML"` позволяет использовать HTML-разметку в сообщениях
    

### 2.2 `Dispatcher`

- Диспетчер обрабатывает обновления Telegram
    
- Хранит состояние FSM через `storage`
    

### 2.3 `MemoryStorage`

- Временное хранилище для FSM (подходит для dev)
    
- Для продакшена лучше использовать Redis или PostgreSQL
    

### 2.4 Middlewares

- Подключаются через `dp.update.middleware(...)`
    
- Можно использовать для:
    
    - Проверки пользователя (`auth`)
        
    - Логирования
        
    - Подмены данных
        

### 2.5 Хендлеры

- Регистрируются через функции `register_handlers(dp)`
    
- Разделение на модули (`start.py`, `help.py`, `admin.py`) упрощает структуру
    

### 2.6 Команды бота

- Настраиваются через `BotCommand`
    
- Отображаются в меню Telegram (`/start`, `/help`, `/admin`)
    

---

## 3️⃣ Запуск бота

```bash
python -m bot.main
```

- Для live-обновлений можно использовать `--reload` через `uvicorn`, но обычно для бота достаточно просто Python
    
- Проверить корректность токена и соединения
    

---

## 4️⃣ Best practices

1. Разделяйте хендлеры по модулям
    
2. Middleware используйте для общих проверок (логирование, авторизация)
    
3. FSM храните через Redis для продакшена
    
4. Команды бота прописывайте централизованно в main.py
    
5. Вся конфигурация (токен, админы) берется из `settings`
    
6. Минимизируйте код в main.py — оставляйте только инициализацию и запуск
    
7. Для больших проектов используйте пакетную структуру и `__init__.py` для импортов
    

---

## 5️⃣ Итог

`main.py` — **сердце проекта**:

- Создает объект бота и диспетчер
    
- Подключает хранилище состояний (FSM)
    
- Регистрирует middlewares и хендлеры
    
- Настраивает команды Telegram
    
- Запускает бота с безопасным завершением сессии
    

Правильная структура main.py обеспечивает **масштабируемость, читаемость и поддержку командной работы**.


## Что такое RabbitMQ?
   это брокер сообщений (message broker), то есть система, которая помогает **микросервисам обмениваться данными** между собой, **не общаясь напрямую**.
## 🧩 1. Подключение и базовая отправка сообщений

```python
import aio_pika
import json
import asyncio

RABBIT_URL = "amqp://guest:guest@localhost/"
QUEUE_NAME = "articles"

async def publish_message(message: dict):
    connection = await aio_pika.connect_robust(RABBIT_URL)
    async with connection:
        channel = await connection.channel()
        await channel.default_exchange.publish(
            aio_pika.Message(
                body=json.dumps(message).encode(),
                content_type="application/json"
            ),
            routing_key=QUEUE_NAME
        )
        print("📤 Отправлено:", message)
```

🧠 **Примечание:**  
`default_exchange` — это встроенный **direct exchange**, который просто отправляет сообщение в очередь с тем же именем, что и `routing_key`.

---

## 📨 2. Получение сообщений (Consumer)

```python
import aio_pika
import asyncio
import json

async def consume_articles():
    connection = await aio_pika.connect_robust("amqp://guest:guest@localhost/")
    async with connection:
        channel = await connection.channel()
        queue = await channel.declare_queue("articles", durable=True)

        async with queue.iterator() as queue_iter:
            async for message in queue_iter:
                async with message.process():
                    data = json.loads(message.body)
                    print("📩 Получено:", data)
                    # Здесь твоя логика: сохранить в БД, обновить и т.д.
```

🧠 **Важно:**  
`message.process()` — автоматически подтверждает, что сообщение обработано.  
Если будет ошибка — сообщение останется в очереди.

---

## ⚙️ 3. Типы Exchange

RabbitMQ не просто “отправляет сообщение”, а **распределяет его по логике через Exchange**.

### 💠 3.1. Direct Exchange (по названию)

```python
exchange = await channel.declare_exchange("direct_logs", aio_pika.ExchangeType.DIRECT)
await exchange.publish(
    aio_pika.Message(body=b"Hello Direct!"),
    routing_key="info"
)
```

🧠 Используется, когда сообщение должно пойти **в конкретную очередь**.

---

### 🌐 3.2. Fanout Exchange (всем сразу)

```python
exchange = await channel.declare_exchange("logs", aio_pika.ExchangeType.FANOUT)
await exchange.publish(aio_pika.Message(body=b"New event!"), routing_key="")
```

→ Все очереди, привязанные к `logs`, получат сообщение.  
🔥 Отлично для рассылок, уведомлений, аналитики.

---

### 🧩 3.3. Topic Exchange (по шаблонам)

```python
exchange = await channel.declare_exchange("topic_logs", aio_pika.ExchangeType.TOPIC)

# Отправка
await exchange.publish(
    aio_pika.Message(body=b"Article created"),
    routing_key="article.created"
)
```

Очередь можно подписать на `"article.*"` или `"*.deleted"`.  
→ Универсальная маршрутизация для микросервисов.

---

## 🗄 4. Durable и auto_delete

При создании очереди:

```python
queue = await channel.declare_queue("articles", durable=True)
```

- `durable=True` — очередь сохраняется при перезапуске RabbitMQ.
    
- `auto_delete=True` — очередь удалится, когда потребителей не будет.
    

---

## 🔁 5. Подтверждения и переотправка

### Автоматическое подтверждение

```python
async with message.process():
    ...
```

### Ручное подтверждение

```python
try:
    # обработка
    await message.ack()  # подтверждение
except Exception as e:
    await message.nack(requeue=True)  # вернуть обратно в очередь
```

---

## 🧨 6. Dead Letter Queue (DLQ)

Когда сообщение **не удалось обработать**, оно может уходить в “мусорную” очередь (для логирования ошибок).

```python
args = {
    "x-dead-letter-exchange": "dlx_exchange"
}
queue = await channel.declare_queue("main_queue", arguments=args)
dlx = await channel.declare_exchange("dlx_exchange", aio_pika.ExchangeType.FANOUT)
await channel.declare_queue("dlq")
await queue.bind(dlx)
```

🧠 Это нужно, если не хочешь терять сообщения при сбое.

---

## 🧠 7. Пример системы CRUD через RabbitMQ

Один exchange `article_events` (тип — topic),  
и три действия:

```python
await exchange.publish(Message(body=b"{id:1}", content_type="json"), routing_key="article.created")
await exchange.publish(Message(body=b"{id:1}", content_type="json"), routing_key="article.updated")
await exchange.publish(Message(body=b"{id:1}", content_type="json"), routing_key="article.deleted")
```

А другой микросервис слушает:

```python
queue.bind(exchange, routing_key="article.*")
```

→ Всё: теперь он автоматически реагирует на создание, обновление и удаление.

---

## ⚡ 8. Просмотр и управление из терминала

|Команда|Описание|
|---|---|
|`rabbitmqctl list_queues`|список очередей|
|`rabbitmqctl list_exchanges`|список обменников|
|`rabbitmqctl purge_queue queue_name`|очистить очередь|
|`rabbitmqctl delete_queue queue_name`|удалить очередь|
|`rabbitmqctl list_bindings`|показать связи exchange ↔ queue|

---

## 🚀 9. Советы для микросервисов

1. Используй **topic exchange** — это гибко и удобно.
    
2. Каждому сервису можно давать **свою очередь**, но **один общий exchange**.
    
3. Делай **retry и DLQ** — это спасёт от потерь данных.
    
4. Логируй **каждое сообщение** (особенно ошибки).
    
5. Если данных очень много — добавь **prefetch_count**:
    
    ```python
    await channel.set_qos(prefetch_count=10)
    ```
    
    → Consumer будет брать максимум 10 сообщений за раз.
    


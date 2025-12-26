## 🔌 Подключение

### Подключиться к RabbitMQ
```python
import aio_pika

connection = await aio_pika.connect_robust("amqp://guest:guest@localhost/")
````

📘 `connect_robust` — устойчивое подключение, автоматически переподключается при сбоях.  
(используется почти всегда в продакшене)

---

## 📡 Каналы

### Создать канал

```python
channel = await connection.channel()
```

📘 Канал — логическое соединение внутри одного TCP-подключения.  
Через него создаются очереди, обменники и публикуются сообщения.

---

## 🔁 Обменники (Exchanges)

### Объявить обменник

```python
exchange = await channel.declare_exchange(
    "articles_exchange",
    aio_pika.ExchangeType.DIRECT,
    durable=True
)
```

📘

- `DIRECT` — по маршруту (`routing_key`)
    
- `FANOUT` — широковещательно всем
    
- `TOPIC` — по шаблонам
    
- `HEADERS` — по заголовкам
    

---

## 📦 Очереди (Queues)

### Объявить очередь

```python
queue = await channel.declare_queue("articles", durable=True)
```

### Привязать очередь к обменнику

```python
await queue.bind(exchange, routing_key="articles_key")
```

---

## 📨 Отправка сообщений

### Отправить сообщение

```python
message = aio_pika.Message(
    body=b"Hello from producer!",
    delivery_mode=aio_pika.DeliveryMode.PERSISTENT
)
await exchange.publish(message, routing_key="articles_key")
```

📘

- `body` — данные в `bytes`
    
- `delivery_mode=PERSISTENT` — сохраняет сообщение при перезапуске брокера
    

---

## 📥 Получение сообщений

### Подписаться на очередь (consumer)

```python
async with queue.iterator() as queue_iter:
    async for message in queue_iter:
        async with message.process():
            print("Получено:", message.body)
```

📘

- `queue.iterator()` — асинхронный итератор по очереди
    
- `message.process()` — автоматически подтверждает получение
    

---

## ✅ Подтверждение сообщений

Если хочешь управлять вручную:

```python
await message.ack()     # подтвердить (удалить из очереди)
await message.reject()  # отклонить (удалить без обработки)
await message.nack()    # вернуть обратно в очередь
```

---

## ⚙️ Закрытие соединений

```python
await channel.close()
await connection.close()
```

📘 Закрывать соединение нужно корректно, особенно при завершении приложения.

---

## 🧩 Пример полного цикла

```python
import aio_pika
import asyncio

async def main():
    connection = await aio_pika.connect_robust("amqp://guest:guest@localhost/")
    async with connection:
        channel = await connection.channel()
        queue = await channel.declare_queue("tasks", durable=True)
        exchange = await channel.declare_exchange("main", aio_pika.ExchangeType.DIRECT)
        await queue.bind(exchange, routing_key="tasks")

        # Отправляем сообщение
        message = aio_pika.Message(body=b"Process this task")
        await exchange.publish(message, routing_key="tasks")

        # Получаем сообщение
        async with queue.iterator() as iterator:
            async for message in iterator:
                async with message.process():
                    print("✅ Получено:", message.body)
                    break

asyncio.run(main())
```

---

## 💡 Дополнительно полезное

|Метод / Класс|Описание|
|---|---|
|`aio_pika.Message`|Формирует сообщение (тело, заголовки, TTL, приоритет)|
|`exchange.publish()`|Отправка сообщения|
|`channel.declare_queue()`|Создание очереди|
|`queue.bind()`|Привязка к обменнику|
|`queue.iterator()`|Получение сообщений|
|`message.ack()`|Подтверждение обработки|
|`message.nack()`|Вернуть в очередь|
|`connect_robust()`|Безопасное подключение|

---

🧠 **Итог:**  
`aio-pika` — это асинхронная библиотека для общения с RabbitMQ.  
Она полностью повторяет концепции обычного RabbitMQ:  
**Exchange → Queue → Message**, но в асинхронной форме.


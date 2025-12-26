
Предположим, у нас есть **ArticleService**, который добавляет, обновляет, удаляет статьи, и **InfoService**, который синхронизирует эти изменения.

---

## 1. **Create (Добавление статьи)**

### Exchange: `direct`, ключ `article.create`

**Publisher (ArticleService)**

```python
import aio_pika, asyncio, json

async def publish_create(article: dict):
    connection = await aio_pika.connect_robust("amqp://guest:guest@localhost:5672/")
    async with connection:
        channel = await connection.channel()
        exchange = await channel.declare_exchange("articles_direct", aio_pika.ExchangeType.DIRECT, durable=True)

        message = aio_pika.Message(
            body=json.dumps(article).encode(),
            delivery_mode=aio_pika.DeliveryMode.PERSISTENT
        )
        await exchange.publish(message, routing_key="article.create")
        print("Article create published:", article)

# Example
asyncio.run(publish_create({"id": 1, "title": "New Article", "content": "Content"}))
```

**Consumer (InfoService)**

```python
async def consume_create():
    connection = await aio_pika.connect_robust("amqp://guest:guest@localhost:5672/")
    async with connection:
        channel = await connection.channel()
        exchange = await channel.declare_exchange("articles_direct", aio_pika.ExchangeType.DIRECT, durable=True)
        queue = await channel.declare_queue("article_create_queue", durable=True)
        await queue.bind(exchange, routing_key="article.create")

        async with queue.iterator() as queue_iter:
            async for message in queue_iter:
                async with message.process():
                    data = json.loads(message.body)
                    print("Article received in InfoService:", data)
                    # здесь добавляем в локальную БД
```

- **Принцип:** точное соответствие ключа `routing_key`.
    
- **Durable:** очередь и обменник живут после рестарта.
    

---

## 2. **Update (Обновление статьи)**

### Exchange: `fanout` (все сервисы должны узнать)

**Publisher**

```python
async def publish_update(article: dict):
    connection = await aio_pika.connect_robust("amqp://guest:guest@localhost:5672/")
    async with connection:
        channel = await connection.channel()
        exchange = await channel.declare_exchange("articles_fanout", aio_pika.ExchangeType.FANOUT, durable=True)
        message = aio_pika.Message(
            body=json.dumps(article).encode(),
            delivery_mode=aio_pika.DeliveryMode.PERSISTENT
        )
        await exchange.publish(message, routing_key="")
        print("Article update published:", article)
```

**Consumer** — может быть несколько:

```python
queue1 = await channel.declare_queue("info_service1", durable=True)
queue2 = await channel.declare_queue("info_service2", durable=True)
await queue1.bind(exchange)
await queue2.bind(exchange)
```

- **Принцип:** Fanout шлёт сообщение всем очередям, не смотря на ключ.
    

---

## 3. **Delete (Удаление статьи)**

### Exchange: `topic` (удаление по шаблону)

**Publisher**

```python
async def publish_delete(article_id: int):
    connection = await aio_pika.connect_robust("amqp://guest:guest@localhost:5672/")
    async with connection:
        channel = await connection.channel()
        exchange = await channel.declare_exchange("articles_topic", aio_pika.ExchangeType.TOPIC, durable=True)
        message = aio_pika.Message(
            body=json.dumps({"id": article_id}).encode(),
            delivery_mode=aio_pika.DeliveryMode.PERSISTENT
        )
        await exchange.publish(message, routing_key="article.delete")
        print("Article delete published:", article_id)
```

**Consumer**

```python
queue = await channel.declare_queue("info_service_delete", durable=True)
await queue.bind(exchange, routing_key="article.delete")
```

- **Принцип:** Topic позволяет фильтровать по шаблону ключа (`article.*`)
    

---

## 4. **Read (Получение данных)**

- В RabbitMQ прямого “read” нет. Обычно читают **свою очередь** (Consumer) или **делают RPC**, если нужен ответ.
    
- Пример RPC (упрощённо):
    

**RPC Publisher**

```python
correlation_id = str(uuid.uuid4())
callback_queue = await channel.declare_queue(exclusive=True)

await exchange.publish(
    aio_pika.Message(
        body=b"Request article 1",
        correlation_id=correlation_id,
        reply_to=callback_queue.name
    ),
    routing_key="article.rpc"
)
```

**RPC Consumer** отправляет ответ в `reply_to`.

---

## 5. **Подтверждение и nack**

- `async with message.process():` — автоматически ack
    
- Можно вручную:
    

```python
await message.ack()    # подтверждение
await message.nack(requeue=True)  # вернуть в очередь
await message.reject(requeue=False) # удалить без повторной отправки
```

- Важно для **непотерянных данных**.
    

---

## 6. **Общие советы**

- **Durable + PERSISTENT** — сохраняй важные сообщения
    
- **Fanout** — широковещательная рассылка
    
- **Direct** — точная доставка по ключу
    
- **Topic** — гибкая фильтрация по шаблону
    
- **Exclusive** — для временных очередей
    
- **Auto-delete** — для тестов или временных очередей
    

---

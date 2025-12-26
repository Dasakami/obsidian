
## 1. **Durable**

```python
queue = await channel.declare_queue("articles", durable=True)
exchange = await channel.declare_exchange("articles_exchange", aio_pika.ExchangeType.DIRECT, durable=True)
```

- **Что делает:** очередь или обменник выживает при перезапуске RabbitMQ.
    
- **Если `False`:** после перезапуска брокера очередь/обменник исчезнет.
    
- **Практика:** всегда ставить `durable=True` для важной информации (статьи, платежи, задания).
    

---

## 2. **Типы обменников**

### a) DIRECT (Прямое соответствие)

```python
exchange = await channel.declare_exchange("direct_exchange", aio_pika.ExchangeType.DIRECT)
await queue.bind(exchange, routing_key="articles")
```

- Сообщение идёт только в очередь с **точным соответствием `routing_key`**.
    
- Пример: у нас очередь `articles`, ключ `articles`. Только она получит сообщение.
    

---

### b) FANOUT (Широковещательный)

```python
exchange = await channel.declare_exchange("fanout_exchange", aio_pika.ExchangeType.FANOUT)
await queue.bind(exchange)
```

- Игнорирует ключ маршрутизации (`routing_key`).
    
- Сообщение уходит **во все очереди**, привязанные к обменнику.
    
- Применение: рассылка уведомлений, всех событий системы.
    

---

### c) TOPIC (По шаблону)

```python
exchange = await channel.declare_exchange("topic_exchange", aio_pika.ExchangeType.TOPIC)
await queue.bind(exchange, routing_key="articles.*")
```

- Сообщения доставляются в очередь, если **ключ сообщения совпадает с шаблоном**.
    
- `*` — один сегмент ключа, `#` — несколько сегментов.
    
- Пример: ключи сообщений: `articles.news`, `articles.blog`. Очередь с `articles.*` получит все, что начинается с `articles.`
    

---

### d) HEADERS (По заголовкам)

- Сообщение отправляется на очередь, если **заголовки совпадают с фильтром очереди**.
    
- Редко используют, больше для сложной фильтрации, когда ключи не подходят.
    

---

## 3. **Message параметры**

```python
message = aio_pika.Message(
    body=b"Hello",
    delivery_mode=aio_pika.DeliveryMode.PERSISTENT,
    headers={"type": "article"},
    expiration=60000
)
```

- **body:** данные в `bytes`
    
- **delivery_mode:**
    
    - `PERSISTENT` — сохраняется в базе RabbitMQ, survives перезапуск
        
    - `TRANSIENT` — в памяти, теряется при перезапуске
        
- **headers:** любые дополнительные данные для фильтрации (важно для `HEADERS` обменников)
    
- **expiration:** время жизни сообщения в мс. Через указанное время сообщение удалится.
    

---

## 4. **Очереди**

```python
queue = await channel.declare_queue("articles", durable=True, exclusive=False, auto_delete=False)
```

- **durable=True** — очередь выживет после перезапуска
    
- **exclusive=True** — только одно соединение может использовать очередь
    
- **auto_delete=True** — очередь удаляется, если никто к ней не подключен
    

---

## 5. **Подтверждение сообщений**

```python
async with queue.iterator() as queue_iter:
    async for message in queue_iter:
        async with message.process():
            print("Получено:", message.body)
```

- `message.process()` — автоматически вызывает `ack` (подтверждение)
    
- Можно вручную:
    
    ```python
    await message.ack()   # подтвердить, удалить из очереди
    await message.nack()  # вернуть обратно в очередь
    await message.reject() # отклонить без повторной отправки
    ```
    

---

## 6. **Пример Fanout + Durable**

```python
# создаём обменник Fanout, чтобы все очереди получили сообщение
exchange = await channel.declare_exchange(
    "news_broadcast",
    aio_pika.ExchangeType.FANOUT,
    durable=True
)

# очередь подписана на этот обменник
queue1 = await channel.declare_queue("subscriber1", durable=True)
queue2 = await channel.declare_queue("subscriber2", durable=True)

await queue1.bind(exchange)
await queue2.bind(exchange)

# публикуем сообщение
message = aio_pika.Message(body=b"New article published!", delivery_mode=aio_pika.DeliveryMode.PERSISTENT)
await exchange.publish(message, routing_key="")
```

- Все подписанные очереди получают сообщение, даже если брокер перезапустился.
    


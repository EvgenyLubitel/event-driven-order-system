#  Event-Driven Order Processing System

Event-Driven система из 5 микросервисов для асинхронной обработки заказов с полной отказоустойчивостью. Система гарантирует, что **ни один заказ не потеряется**, даже при падении отдельных сервисов.

**Особенность:** В качестве брокера сообщений используется **сам n8n** (встроенная очередь), без необходимости разворачивать отдельные сервисы типа RabbitMQ или Kafka.

**поток данных**
1. Клиент → POST /order/create
         ↓
2. API Gateway → возвращает orderId (мгновенно)
         ↓
3. Заказ отправляется в очередь n8n (Order Processor)
         ↓
4. Order Processor обрабатывает (retry 3 раза при ошибке)
         ↓
5. Успех → Event Store (сохраняем событие)
         ↓
6. Send Notification → уведомляем клиента
         ↓
7. Ошибка после 3 попыток → Dead Letter Queue → администратор
---

##  БЫСТРЫЙ СТАРТ

### Создание заказа

```bash
curl -X POST https://n-wformatagent-evgenylubitel.amvera.io/webhook-test/order/create \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": "customer_123",
    "items": [
      {"productId": "prod_001", "quantity": 2},
      {"productId": "prod_002", "quantity": 1}
    ],
    "total": 2999.99
  }'

事件總線（Event Bus）傳遞的事件內容設計（JSON 格式）
---

- eventType︰事件類型，用於事件訂閱者判斷與處理。
- timestamp：事件發生的時間，採用 ISO 8601 格式。
- orderId：訂單唯一識別碼，關聯訂單
- 

1. OrderCreated（訂單建立事件）
```json
{
  "eventType": "OrderCreated",
  "timestamp": "2025-05-05T17:05:00Z",
  "orderId": "ORD123456",
  "userId": "USER789",
  "restaurantId": "REST001",
  "orderItems": [
    {
      "itemId": "ITEM001",
      "name": "牛肉漢堡",
      "quantity": 2,
      "price": 120
    },
    {
      "itemId": "ITEM005",
      "name": "薯條",
      "quantity": 1,
      "price": 50
    }
  ],
  "totalAmount": 290,
  "deliveryAddress": {
    "street": "中正路100號",
    "city": "台北市",
    "zipcode": "100"
  },
  "paymentStatus": "Paid",
  "orderStatus": "Created"
}
```

2. OrderUpdated（訂單狀態更新事件）
```json
{
  "eventType": "OrderUpdated",
  "timestamp": "2025-05-05T17:10:00Z",
  "orderId": "ORD123456",
  "updatedFields": {
    "orderStatus": "Confirmed",
    "confirmedAt": "2025-05-05T17:09:58Z"
  }
}
```

3. OrderAssigned（訂單派送餐廳指派事件）
```json
{
  "eventType": "OrderAssigned",
  "timestamp": "2025-05-05T17:12:00Z",
  "orderId": "ORD123456",
  "restaurantId": "REST001",
  "estimatedPreparationTime": 15,
  "assignedAt": "2025-05-05T17:11:55Z"
}

```

4. OrderPrepared（訂單備餐完成事件）
```json
{
  "eventType": "OrderPrepared",
  "timestamp": "2025-05-05T17:27:00Z",
  "orderId": "ORD123456",
  "restaurantId": "REST001",
  "preparedAt": "2025-05-05T17:26:58Z"
}
```

5. DeliveryStarted（配送開始事件）
```json
{
  "eventType": "DeliveryStarted",
  "timestamp": "2025-05-05T17:30:00Z",
  "orderId": "ORD123456",
  "deliveryId": "DELIV987",
  "deliveryPersonId": "DP123",
  "startedAt": "2025-05-05T17:29:59Z",
  "currentLocation": {
    "latitude": 25.0330,
    "longitude": 121.5654
  }
}
```

6. DeliveryStatusUpdated（配送狀態更新事件）
```json
{
  "eventType": "DeliveryStatusUpdated",
  "timestamp": "2025-05-05T17:40:00Z",
  "orderId": "ORD123456",
  "deliveryId": "DELIV987",
  "deliveryPersonId": "DP123",
  "status": "OnTheWay",
  "location": {
    "latitude": 25.0340,
    "longitude": 121.5660
  },
  "updatedAt": "2025-05-05T17:39:58Z"
}
```

7. DeliveryCompleted（配送完成事件）
```json
{
  "eventType": "DeliveryCompleted",
  "timestamp": "2025-05-05T17:50:00Z",
  "orderId": "ORD123456",
  "deliveryId": "DELIV987",
  "deliveryPersonId": "DP123",
  "deliveredAt": "2025-05-05T17:49:59Z",
  "recipientConfirmation": true
}
```
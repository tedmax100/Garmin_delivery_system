# 資料表設計（Schema）

## 1. 使用者資料表（Users）

| 欄位名稱       | 資料型態       | 說明              | 備註            |
|----------------|----------------|-------------------|-----------------|
| user_id        | UUID (PK)    | 使用者唯一識別碼  | UUID        |
| username       | VARCHAR(50)    | 使用者帳號        | unique key            |
| password_hash  | VARCHAR(60)   | 使用 bcrypt 加密的密碼hash          | 安全存儲        |
| email          | VARCHAR(100)   | 電子郵件          | unique key            |
| phone          | VARCHAR(20)    | 電話號碼          |                 |
| user_type      | ENUM           | 使用者類型        | 'customer', 'delivery', 'restaurant_staff' |
| created_at     | DATETIME       | 建立時間          |                 |
| updated_at     | DATETIME       | 更新時間          |                 |

---

## 2. 訂單資料表（Orders）

| 欄位名稱       | 資料型態       | 說明              | 備註            |
|----------------|----------------|-------------------|-----------------|
| order_id       | UUID (PK)    | 訂單唯一識別碼    | UUID        |
| user_id        | BIGINT (FK)    | 下單使用者ID      | 參考 Users.user_id |
| restaurant_id  | BIGINT (FK)    | 餐廳ID            | 參考 Restaurants.restaurant_id |
| delivery_id    | BIGINT (FK)    | 外送員ID          | 參考 Users.user_id（外送員） |
| status         | ENUM           | 訂單狀態          | 'pending', 'preparing', 'delivering', 'completed', 'cancelled' |
| total_amount   | DECIMAL(10,2)  | 訂單總金額        |                 |
| order_time     | DATETIME       | 下單時間          |                 |
| delivery_time  | DATETIME       | 預計送達時間      |                 |
| created_at     | DATETIME       | 建立時間          |                 |
| updated_at     | DATETIME       | 更新時間          |                 |

---

## 3. 訂單明細資料表（Order_Items）

| 欄位名稱       | 資料型態       | 說明              | 備註            |
|----------------|----------------|-------------------|-----------------|
| order_item_id  | BIGINT (PK)    | 訂單明細唯一識別碼| 自動遞增        |
| order_id       | BIGINT (FK)    | 所屬訂單ID        | 參考 Orders.order_id |
| product_id     | BIGINT (FK)    | 商品ID            | 參考 Products.product_id |
| quantity       | INT            | 數量              |                 |
| price          | DECIMAL(10,2)  | 單價              | 下單時價格      |

---

## 4. 商品資料表（Products）

| 欄位名稱       | 資料型態       | 說明              | 備註            |
|----------------|----------------|-------------------|-----------------|
| product_id     | UUID (PK)    | 商品唯一識別碼    | UUID        |
| restaurant_id  | BIGINT (FK)    | 所屬餐廳ID        | 參考 Restaurants.restaurant_id |
| name           | VARCHAR(100)   | 商品名稱          |                 |
| description    | TEXT           | 商品描述          |                 |
| price          | DECIMAL(10,2)  | 商品價格          |                 |
| stock_quantity | INTEGER        | 商品庫存          |                 |
| available      | BOOLEAN        | 是否可販售        |                 |
| created_at     | DATETIME       | 建立時間          |                 |
| updated_at     | DATETIME       | 更新時間          |                 |

---

## 5. 餐廳資料表（Restaurants）

| 欄位名稱       | 資料型態       | 說明              | 備註            |
|----------------|----------------|-------------------|-----------------|
| restaurant_id  | UUID (PK)    | 餐廳唯一識別碼    | UUID        |
| name           | VARCHAR(100)   | 餐廳名稱          |                 |
| address        | VARCHAR(255)   | 餐廳地址          |                 |
| phone          | VARCHAR(20)    | 餐廳電話          |                 |
| created_at     | DATETIME       | 建立時間          |                 |
| updated_at     | DATETIME       | 更新時間          |                 |

---

## 6. 外送員狀態資料表（Delivery_Status）

| 欄位名稱       | 資料型態       | 說明              | 備註            |
|----------------|----------------|-------------------|-----------------|
| delivery_id    | UUID (PK, FK)| 外送員ID          | 參考 Users.user_id |
| current_status | ENUM           | 外送員狀態        | 'available', 'busy', 'offline' |
| location_lat   | DECIMAL(9,6)   | 緯度              |                 |
| location_lng   | DECIMAL(9,6)   | 經度              |                 |
| updated_at     | DATETIME       | 狀態更新時間      |                 |

---

## 7. 訂單事件資料表（Order_Events）

| 欄位名稱       | 資料型態       | 說明              | 備註            |
|----------------|----------------|-------------------|-----------------|
| event_id       | UUID (PK)    | 事件唯一識別碼    | UUID        |
| order_id       | BIGINT (FK)    | 相關訂單ID        |                 |
| event_type     | VARCHAR(50)    | 事件類型          | 如 'order_created', 'order_dispatched' 等 |
| event_data     | JSON           | 事件內容詳細資料  | 可存放事件相關資訊 |
| created_at     | DATETIME       | 事件發生時間      |                 |

---

## 8. 通知資料表（Notifications）

| 欄位名稱       | 資料型態       | 說明              | 備註            |
|----------------|----------------|-------------------|-----------------|
| notification_id| UUID (PK)    | 通知唯一識別碼    | UUID        |
| user_id        | UUID (FK)    | 接收通知使用者ID  |                 |
| type           | VARCHAR(50)    | 通知類型          | 'push', 'sms', 'email' |
| content        | TEXT           | 通知內容          |                 |
| status         | ENUM           | 通知狀態          | 'pending', 'sent', 'failed' |
| created_at     | DATETIME       | 建立時間          |                 |
| sent_at        | DATETIME       | 發送時間          |                 |

---

# 備註  
- `ENUM` 欄位可依需求擴充狀態種類。  
- JSON 欄位用於存放彈性結構的結構化事件，方便擴充。
- PK 不選用 `Auto increment` 原因有
  - 容易被猜到範圍
  - `UUID v7` 已經是 **time sortability**，是嚴格遞增的

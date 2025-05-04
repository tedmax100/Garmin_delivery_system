Delivery System Design
---

# 題目
```
請設計一個前後端架構的連鎖餐廳點餐外送系統，
● 使用者可在任何地方透過點餐系統點餐送單，上傳至點餐核心系統。
● 核心系統收單後自動派送至距離使用者最近的餐廳據點備餐。
● 備餐完成後透過待送餐點系統提供內部外送員外送資訊、取單確定外
送功能。
● 開始外送時於外送系統顯示外送路徑與餐點資料
● 使用者端同步顯示外送員外送進度
● 資料於外送系統餐點送達後結束。
設計內容包含此系統整體架構、後端程式功能模組拆分，你認為會使用到
的所有類型Service、資料交換方式，並說明你的設計理由，可用任何型式
表示。
```

| 名稱  | 數量 |
| ----- | -------- |
| 蘋果 | 3        |
| 雞蛋 | 12       |

# 根據 C4 Model 梳理
## System Context Diagram
1. 角色（Person）
主要角色

| 角色	| 說明 |
| ----- | -------- |
| 使用者	| 一般顧客，透過前端系統（App/網頁）下單、查詢訂單、追蹤外送進度 |
| 餐廳員工 |	負責備餐，接收訂單、確認備餐、通知外送員取單 |
| 外送員 |	餐廳內部外送人員，負責取餐、配送、更新外送狀態 |
| 系統管理員 | 後台管理，維護餐廳據點、菜單、帳號管理（可選，非主要流程） |

2. 系統（Software System）
主要系統

| 系統名稱	| 說明 |
| ----- | -------- |
| 使用者點餐系統	| 前端App/網頁，供顧客點餐、查詢、追蹤外送進度 |
| 點餐核心系統	| 負責訂單管理、智能派單、狀態協調、支付結算及通知管理 |
| 餐廳據點系統	| 各餐廳據點接收訂單、通知備餐完成、通知外送員 |
| 外送管理系統	| 管理外送員、指派外送、追蹤外送進度、回報送達 |
| 外送員App/系統	| 外送員端App/介面，查看外送任務、導航、更新狀態 |

其他輔助系統（可選）
- 通知服務（推播/簡訊/Email等）
- 地理定位服務（計算距離、路徑導航）
- 支付系統（金流串接）

```
[顧客] --下單/查詢--> [使用者點餐系統] --API--> [點餐核心系統]
[點餐核心系統] --派單--> [餐廳據點系統]
[餐廳員工] --接收訂單/備餐完成通知--> [餐廳據點系統]
[餐廳據點系統] --備餐完成通知--> [外送管理系統]
[外送管理系統] --任務指派/狀態更新--> [外送員App/系統]
[外送員] --接收任務/進度回報--> [外送員App/系統]
[外送員App/系統] --進度回報--> [外送管理系統] --同步進度--> [使用者點餐系統]
```

![](./image/README/c4_context.png)

```mermaid
C4Context
title 點餐系統 - 系統上下文圖

Person(customer, "顧客", "使用點餐App下單、查詢訂單狀態")
Person(restaurant_staff, "餐廳員工", "接收訂單、備餐、更新備餐狀態")
Person(delivery_person, "外送員", "接收配送任務、更新配送狀態")

System(user_order_system, "使用者點餐系統", "提供顧客點餐及訂單查詢功能")
System(order_core_system, "點餐核心系統", "處理訂單管理、派單與狀態流轉")
System(restaurant_system, "餐廳據點系統", "管理餐廳訂單與備餐流程")
System(delivery_management_system, "外送管理系統", "任務指派與配送狀態管理")
System(delivery_app_system, "外送員App/系統", "外送員接單與進度回報")

Rel(customer, user_order_system, "下單 / 查詢訂單")
Rel(user_order_system, order_core_system, "API 呼叫")
Rel(order_core_system, restaurant_system, "派單")
Rel(restaurant_staff, restaurant_system, "接收訂單 / 備餐完成通知")
Rel(restaurant_system, delivery_management_system, "備餐完成通知")
Rel(delivery_management_system, delivery_app_system, "任務指派 / 狀態更新")
Rel(delivery_person, delivery_app_system, "接收任務 / 進度回報")
Rel(delivery_app_system, delivery_management_system, "進度回報")
Rel(delivery_management_system, user_order_system, "同步配送進度")
```

## Container Diagram
### 使用者點餐系統 User Ordering System

| 容器名稱 | 	技術或類型 | 	職責說明 |
| ----- | -------- | -------- |
| Web App |	React/Vue |	提供網頁介面給顧客點餐與查詢訂單 |
| Mobile App |	iOS/Android |	手機App介面，提供點餐與訂單追蹤 |
| API Gateway |	REST API |	接收前端請求，轉發給點餐核心系統 |
| Notification Client |	Push Notification |	接收並顯示訂單狀態推播 |

![](./image/README/c4_user_ordering_container.bmp)

### 點餐核心系統 Order Core System

| 容器名稱 | 	技術或類型 | 	職責說明 |
| ----- | -------- | -------- |
| 訂單API	REST API |	提供訂單接收、查詢與修改功能 |
| 訂單服務 | C# |	處理訂單業務邏輯，管理訂單生命周期 |
| 派單服務 | 微服務 |	智能派單，根據餐廳狀態分配訂單 |
| 支付服務 | 第三方支付整合 |	處理支付及結算 |
| 通知服務 | 事件驅動 |	推送訂單狀態更新通知 |
| 訂單資料庫 |	RDBMS |	存儲訂單及狀態資料 |

![](./image/README/c4_order_core_container.bmp)

### 餐廳據點系統 Restaurant System

| 容器名稱	| 技術或類型	| 職責說明 |
| ----- | -------- | -------- |
| 訂單接收模組 |	桌面App/Web |	接收並顯示訂單給餐廳員工 |
| 備餐顯示系統 |	專用顯示裝置 |	顯示備餐進度及狀態 |
| 狀態更新服務 |	REST API |	將備餐狀態回報點餐核心系統及外送管理系統 |
| 餐廳訂單資料庫 |	關聯式資料庫 |	儲存訂單及備餐狀態 |

![](./image/README/c4_restaurant_container.bmp)

### 外送管理系統 Delivery Management System

| 容器名稱 |	技術或類型 |	職責說明 |
| ----- | -------- | -------- |
| 配送指派服務 |	微服務（事件消費者） |	根據訂單與外送員狀態指派配送任務 |
| 配送追蹤服務 |	事件驅動 |	追蹤外送進度並更新訂單狀態 |
| 配送資料庫 |	RDBMS |	儲存配送任務與狀態 |
| API Gateway |	REST API |	提供外送員App與其他系統介接 |

![](./image/README/c4_delivery_management_container.bmp)

[Component diagram](./Delivery_management_system_component.md)

### 外送員App/系統 Delivery App/System

| 容器名稱 |	技術或類型 |	職責說明 |
| ----- | -------- | -------- |
| 外送員手機App |	iOS/Android |	查看任務、導航、更新配送狀態 |
| API 客戶端 |	REST Client |	與外送管理系統API通訊 |
| 通知客戶端 |	Push Notification |	接收配送任務與狀態推播 |

![](./image/README/c4_delivery_app_container.bmp)

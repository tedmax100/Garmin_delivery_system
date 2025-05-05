# Delivery Management System Component Diagram
----

## 使用場景
- 管理配送員狀態與訂單派送，計算最佳配送方案。
- 將配送任務分配給配送員，並追蹤配送進度。

![](./image/README/c4_delivery_management_system_component.png)

##  配送指派服務 (Delivery Dispatcher Service) 
負責接收訂單事件，管理外送員的狀態與資訊，根據訂單需求與外送員狀態計算最佳配送指派方案，並將配送任務發送給配送追蹤服務或外送員App，確保配送任務能有效指派。

| 元件名稱 |	功能說明 |
| ----- | -------- | 
| 訂單事件消費元件 Order Event Consumer |	從事件總線接收新訂單或訂單狀態更新事件 |
| 外送員狀態管理元件 Courier Status Manager |	管理外送員在線狀態、位置、工作負載等資訊 |
| 派單決策引擎 Dispatch Decision Engine |	根據訂單需求及外送員狀態，計算最佳配送指派方案 |
| 指派任務發送元件 Dispatch Task Sender |	將配送任務發送到配送追蹤服務或外送員App |




## 配送追蹤服務 (Delivery Tracking Service) 
負責接收外送員回傳的配送進度更新，將配送狀態同步回訂單系統，並發布配送狀態變更事件到事件總線，協調系統間的配送狀態同步與追蹤。

| 元件名稱 |	功能說明 |
| ----- | -------- | 
| 配送狀態接收元件 Delivery Status Receiver |	接收外送員App或設備回傳的配送進度更新 |
| 訂單狀態同步元件 Order Status Synchronizer |	將配送狀態更新同步回訂單系統，更新訂單狀態 |
| 事件發佈元件 Event Publisher |	發佈配送狀態變更事件到事件總線，通知其他系統 |

##  配送資料庫 (Delivery Database)
負責儲存配送相關的資料，包括配送任務基本資料、配送狀態變更記錄及外送員資訊與狀態，為配送指派與追蹤服務提供資料支援。

| 元件名稱 |	功能說明 |
| ----- | -------- | 
| 配送任務資料表 Delivery Task Table |	儲存配送任務基本資料（訂單ID、外送員ID、指派時間等） |
| 配送狀態資料表 Delivery Status Table |	儲存配送過程中狀態變更記錄 |
| 外送員資料表 Courier Info Table |	儲存外送員資訊及狀態 |


## API Gateway
負責驗證外送員App的身份與權限，根據API請求路徑路由請求到相應的配送指派或配送追蹤服務，並進行請求格式或協議轉換，同時統一處理API錯誤與異常，保障系統安全與穩定。

| 元件名稱 |	功能說明 |
| ----- | -------- | 
| 認證授權元件 Authentication & Authorization |	驗證外送員App的身份與權限 |
| 請求路由元件 Request Router |	根據API路徑將請求路由到配送指派或配送追蹤服務 |
| 請求轉換元件 Request Transformer |	格式轉換與協議轉換（如JSON格式、RESTful） |
| 錯誤處理元件 Error Handler |	統一處理API錯誤與異常 |


## 目標
- 精準且快速派送訂單，降低配送延誤。
- 即時更新配送狀態與外送員位置。
- 保持系統高可用性，避免配送任務積壓。

### 建議 SLO
| 指標 |	目標值 |	備註 |
| ----- | -------- | -------- |
| 派單成功率 |	≥ 99.9% |	訂單能成功派送給配送員 |
| 配送狀態更新延遲 |	95% 狀態更新 < 3秒 |	保持配送進度即時性 |
| 系統可用率 |	≥ 99.9% |	保持系統穩定運行 |
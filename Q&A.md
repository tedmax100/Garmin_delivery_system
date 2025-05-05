**Q1︰怎防止超賣問題?**

Ans :

1. 資料庫層面交易與鎖定，在 Order Core Service 中將 `訂單建立`與 `庫存扣減`包成一個 atomic 操作，避免中途被打斷或其他交易干擾。雖然 **Pessimistic Locking** 與 **Optimistic Locking** 都能設計來使用，但熱門商品被競爭可能性很高，使用 Pessimistic Locking 盡可能縮短 transaction 範圍趕緊結束，會比 Optimistic Locking 機制在比對 version 不同而 retry 會來的快速。
2. 雖然有 EventBus  的設計，但若把訂單建立與庫存扣減拆開成不是一個 atomic，而是兩個有關連的 event的設計，是沒辦法處理關鍵資源鎖定的問題，只能處理狀態同步與結果通知。


**Q2︰這樣的多模組與事件驅動的系統設計怎麼進行監控與事件追蹤**

Ans︰

1. 事件追蹤，可以藉由 `OpenTelemetry` 的 tracing 來提供 context。並搭配 `auto instrumentation` 與 `propagate` 機制來解析上游服務傳送來的請求取得 trace context，然後置入到 log、metrics、tracing 等telemetry中。能參考我去年於[鐵人講堂的簡單說明](https://itplus.ithome.com.tw/webinar-page/234)。
2. 監控的部份，每個服務設計時，都需要知道目標以及風險。就能透過 `OpenTelemetry` 在程式內部產生 `延遲時間`、`錯誤率`等 SLI 來計算出 SLO 與 error budget，當 error budget 在短時間內燒完預算時就能發出警報。也能在每個主機上去監控主機的資源使用程度以及服務使用到多少資源。

Q3︰怎計算出路徑與時間

Ans︰

在派單決策計畫，以及配送進度追蹤等，甚至顧客點餐要知道餐廳在哪，這些都需要地圖路線與圖釘做配合。能透過 [**Google Maps Distance Matrix API**](https://developers.google.com/maps/documentation/distance-matrix/overview) 計算顧客與每個餐廳據點之間的實際距離與預估行車/步行時間。呼叫 [**Google Maps Directions API**](https://developers.google.com/maps/documentation/directions/?hl=zh_TW)取得詳細路線規劃，供外送員導航或預估外送時間。

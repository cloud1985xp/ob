
請更新 shippings 下的 batches 功能
參考 ai_docs/guidelines/form_ui.md 
來套用 InfiniteScrollList 的功能

並在 query 的篩選加入以下的條件

- 套用分類 filter，layout 用 left，分類選項是用 ShipBatch 的 user，請先從 ShipBatch 取得 distinct 的 user 清單當作分類選項

- 使用 Date Range Filter 加上日期條件，可以依 ShipBatch 的 date 來當條件進行篩選
- 加上額外的 text field 來對相關的訂單標題或編號進行模糊比對查詢：
	- 相關的訂單是透過 ShipBatch join 的 Orders 的 through ShipBatchOrder 找到關聯的訂單，比對 Order 的 label 或 number 做 LIKE %{term}% 的查詢 

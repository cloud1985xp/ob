
請更新 Pricings 下的 tables 價目表管理功能，實現以下目標

一，更新 CRUD 功能
- 最用新的 CRUD patterns，包括
	- 使用 infinite scroll
	- 獨立 show/new/edit 頁面 (共用 form component)
- index 頁套用 Category 為做 filter，filter label 為 code

二，根據以下規格對價目表管理做全面的檢查與修改


包括以下內容：
### 以年份為瀏覽
價目表是依年份為基礎，預設用當下年份，(以系統設定的月份作為臨界值)
因此 #index 列表必須要套用年份的條件，
例如當前年份為 2026，那在 all 的分類下也只有 2026 年的價目表

在 header 的標題處要顯示正在瀏覽的年份
可將顯示的年份直接用下拉選單呈現，可直接點擊來切換年份
超始年份請從系統設定，預設為 2014 年

可參考舊版本的設計
- PriceTable::THRESHOLD_MONTH
- ApplicationHelper#price_table_year_range

### 實作 PriceTable 的價目更新

PriceTable 的 form 應該要可以上傳檔案(csv)，來更新定價內容(price_table_cells)
更新的方式可能會依 price table 的用途不同

請參考舊版本
- Service::PriceTableImporter

### 更新 #show 頁面
請在 show 頁面將 price_table_cells 的內容
應要是依寬&高維度展開的 table，每個尺寸的價格


## 確認並詳細了解規格

請用 rails-migration skill 詳細了解舊版本對 price_table 的新增/編輯相關功能
整理成完整的規格，紀錄在 ai_docs/specs/price_table.md


請在 Price Table 的 show 畫面，編輯按鈕旁加上一個「下載」的按鈕
並增加下載的功能
讓使用者可以下載該 price table 的價目表 csv 檔案

csv 檔案的內容必須是從當前 db table 裡的 price_table_cell 資料組成的


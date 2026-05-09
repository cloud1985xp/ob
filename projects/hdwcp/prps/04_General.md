
我要對 app layout 下的功能也開始更新套用 InfiniteScrollList 的功能
參考 ai_docs/ui/crud-pattern.md，但只先套用 index 的部分

我打算一步一步來，不要一次套到全部的功能，以便過程中再檢查這個 macro 是否有改善之步
請先套用到以下的功能

- categories
- models, 要加上 filter，用 category
- systems, 要加上 filter，用 category
- operations, 要加上 filter，用 category
- colors, 要加上 filter，用 category
- fabrics要加上 filter，用 category
- accessories
- category_options, 要加上 filter，用 category
- serials, 要加上 filter，用 category

請確保 infinite scroll, term search，category filter (如果有的話)，以及排序的功能都要正常運作。

接下來將 InfiniteScrollList 套用到 contacts 的 dealers 功能，包括 infinite scroll, term search, category filter 和排序的功能

目前 dealers 已有用 scope 做篩選的功能
我希望保有 scope 的篩選，同時加上 InfiniteScrollList 用 DealerType 做為 filter

# Migrate 會計功能
請參照舊版 rails 專案(位於 ./hdwcp-main)
將會計 (accountings/) 下的所有功能，migrate 至現在的 phoenix 版專案下

功能包括
- 經銷商帳目 (accountings/reports)
- 銷售出貨明細 (accountings/reports/delivery)
- 年度結算 (accountings/reports/annual)
- 付款紀錄管理 (accountings/payments)
- 拆讓紀錄管理 (accountings/exchanges)

有幾項我希望能滿足的：
- 建立 Accountings Context module (Hdwcp.Accountings) 來實作這個 domain 的功能
	- 建立相關所需的 schema module
- 有用到「經銷售篩選」的部分，請使用現有的 component 
- 在 accountings/reports 的功能，有「依月份」或「依年份」為範圍的資料查詢，請將「切換年份」、「切換月份」使用既有的 ui component，或做成可以重複使用的 component
- 付款紀錄管理、折讓紀錄管理功能
	- 有依「起始月份」「結束月份」的範圍選擇，請製作成可重複使用的 component
	- 實作對應資料的 crud
- 「自訂經銷商名稱」這個功能先不實作，列在未來 todo
- 「銷售出貨明細」有輸出成 csv 下載的功能，也先不實作，列在未來 todo，之後在一起規劃各處與 csv 下載有關的功能


可用 brainstorming 詳細研究舊版本後，規劃清楚後再進行
程式架構請實作成依 elixir phoenix 的風格與符合最佳實務
ui 也請依現有專案的方式做調整，使用 daisyui 和既有元件做配置
若有任何問題請提出來討論


# Migrate Model Management
請參照舊版 rails 專案(位於 ../hdwcp-main)
將產品的葉片與布料(/products/models) 管理的功能，migrate 至現在的 phoenix 版專案下

管理功能是指，products/models/new 以及 products/models/:id/edit
的新增與編輯功能
請將它們實作成獨立的頁面，並共用 model_form 這個 component
model_form 裡要如同舊版本一樣，可以設定該 model 的基本資料，以及價目表、產品組合與極限規格等設定

請詳細研究舊版本後，規劃清楚後再進行
程式架構請實作成依 elixir phoenix 的風格與符合最佳實務
ui 也請依現有專案的方式做調整，使用 daisyui 和既有元件做配置
若有任何問題請提出來討論

Migrate Serial Management

## 優化產品管理
請將產品管理下的 categories、operations、systems 都調整成 new/edit 獨立頁面，並在 new/edit 頁面共用各自的 category_form、operation_form、system_form

並且，對 category_form，請參考舊版 (../hdwcp-main) 的 products/categories 的 new/edit 功能，要可以設定 category 的：
- 可搭配的 systems
- 可搭配的 operations
- 可搭配的 model_sizes
請用 check boxes 來實現上述項目的複選

相對的
在 system_form 裡，也要可以設定搭配的 categories
在 operation_form 裡，也要可以設定搭配的 categores

另也要補上
- model_sizes 的 CRUD 管理
- others(其他產品)的 CRUD 管理


請更新左側的主選單，加上其他產品和布料尺寸的連結選項
將以下產品管理頁面的 分類 filter，改成用 category code
- models
- colors
- fabrics
- category_options
- serials

# 價目表升級項目管理
請參考舊版 rails 專案的價目表管理(pricings/tables/:id) 下的功能，

將以下管理功能：
- 操作系統升級
- 產品系統升級
- 其他升級選項
migrate 至當前專案 (pricins/tables/:id)

請詳細研究舊版本後，規劃清楚後再進行
程式架構請實作成依 elixir phoenix 的風格與符合最佳實務
ui 也請依現有專案的方式做調整，使用 daisyui 和既有元件做配置
若有任何問題請提出來討論

# 訂單流程管理
參考舊版 rails 專案(../hdwcp) 中訂單的流程管理(/sales/orders/:id/*)
主要包括流程轉換將訂單從、草稿、提交、接受正式訂單、生產中、出貨等狀態之間的管理

請詳細研究舊版本後，規劃清楚後再進行
程式架構請實作成依 elixir phoenix 的風格與符合最佳實務
ui 也請依現有專案的方式做調整，使用 daisyui 和既有元件做配置
若有任何問題請提出來討論




# (WIP) 顯示 Dealer
- 要可以自訂顯示區域、地區、編號、名稱


# (WIP) Overall
實作 stepper UI
出貨作業
折抵申請作業
海外出貨包裝作業

訂單建立
一般商品
新增購買商品
獨立進行商品估價

修改
維修
樣品
其他


訂單流程管理
提交
接受
出貨、部分出貨
取消
刪除
暫停
手動編輯
手動調整

訂單瀏覽
出貨單瀏覽
海外文件瀏覽

經銷
合約管理
會計帳款管理
回饋試算
行銷活動

產品管理

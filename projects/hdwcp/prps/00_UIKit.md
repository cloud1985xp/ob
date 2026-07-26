

# 待修正
- 變更/回收繞過狀態機不寫歷程
- 部分出貨要可以選擇部分出貨的項目並設定出貨單
- 金額套用貨幣顯示方式
- 所有 filter card 裡的 分類篩選，統一用  pills 樣式 (而非 tab 樣式)

實作訂單商品明細功能
這個功能包括了既有的：
- 編輯訂單 -> 增加新的商品明細
- 編輯訂單 -> 編輯已建立的商品明細
以及與舊版不同，要增加一個：不用建立訂單，直接可以預然商品建立明細，做為估價查詢的功能

這三者都會有共通的行為：
- 先選擇商品分類
- 開始編輯商品內容來進行建立/更改或詢價
- 過程中使用者會填入商品所需的規格，系統會進行規格檢查，然後查詢價格資訊，產生查價結果預覽

商品
- 主分類: category
- 產品系統: system
- 操控/操作系統: operation
- 葉片/布料(主要、第一組)的:
	- 型號: model
	- 葉片/布料的顏色: color，可用關聯
	- 葉片/布料的顏色代碼: color_code 也允許直接輸入顏色代碼
	- 現貨 or 期貨: model_is_future = true = 期貨
		- 期貨時，會直接輸入顏色代碼
- 

技術實作上，我覺得可以建立一個產品的 struct
再設計模組來對這個資料結構做規格檢查，以及價格查詢

會有一些




# 實作訂單詳細功能
請參考舊版 rails 的專案，目錄位置：`../hdwcp` 裡對訂單操作的功能，即 /sales/orders/:id 下的功能
完整 migrate 所有功能至現在專案下，可重新設計頁面的排版以符合新版本的架構、設計原則與 ui 樣式。可用用 playwright-cli 等工具參考瀏覽 http://hdwcp.test/sales/orders/248079 畫面來理解原版本的功能

包括例如：
- 訂單資訊的呈現、排序
	- 狀態變更紀錄
	- 備忘錄筆記的瀏覽與建立
	- 出貨單紀錄等
- 訂單的狀態管理(拆單、轉待處理、手動出貨、取消、設為待料、複製、變更)
- 訂單的列印輸出

其中訂單的編輯、增加訂單商品明細(order detail) 的功能，可以先理解但先不實作，
這部分的功能我想要重新設計成不同於舊版的流程，所以這段需要與我再詳細討論

請先詳細了解舊版對指定訂單下的各種操作與功能
擬定完整的實作計畫，且可以分步驟來實現




調整訂單工作區 (sales/orders/workspace)
- 訂單排序依照狀態，先列出 formal，再列出 pending
- 加上側欄分類篩選，用當下訂單的經銷商經銷商作為分類，附上每個經銷商的訂單數，並用訂單數從多到少來排列側欄分類

對 orders index 的資料列增加「自訂欄位顯示」的功能
- 在 orders list 的 card 的標題右側，加上「齒輪」圖示，點擊後 dropdown 展開設定面版
	- 這個功能的 ui 做成 component，未來可以其他資源的列表，像是：
		- card 的標題支援右側 aside 區塊 或是 toolbar
		- 這個 toolbar / dropdown 的 component
- 可以自訂列表要顯示的欄位有哪些，包括
	- 顯示訂單列表時，依自訂的欄位將不需顯示的欄位隱藏
		- 可以用 css 來控制顯示/隱藏即可，參考 dealer_name 的 upref- 作法
	- 還可以自訂「經銷名稱」要不要顯示完整的名稱
		- 可選要不要顯示「地區」
		- 可選要不要顯示「城市」
		- 可選要不要顯示「負責業務」
- 目前可將 user preference 資訊存在 cookies 即可，然後放到 user 的 virtual fields
	- 送出更新可用 post request 然後重整整個頁面
	- 資料結構分別存放
		- dealer_name 的顯示偏好
		- orders list 的顯示偏好
		- 未來可以存其他資料的顯示偏好

請評估作法，若有問題或其他建議請提出討論

調整 dealer_name component
加上顯示 dealer 的負責業務的名稱
負責業務：saler_id foreign key 指向的 user
若沒有 saler 則顯示 `無業務`




# 實作 orders 其他列表頁


實作時部分需要參考舊專案的一些定義或邏輯
舊專案是 rails 的版本，位置：`/hdwcp`

一、訂單工作區
實作 /sales/orders/workspace 頁面，做為「訂單工作區」的列表
與 orders index 類似，但會將所有待處理的新進訂單 orders 一次全部列出
- 不需要 infinite scroll
- 不需要 filter

新進訂單的定義：狀態在 formal 或 pending 的訂單，
請參考 app/presenters/sales/order_workshop_presenter.rb 裡的 `Order.common.with_states([:formal, :pending])`

二、重作及退回訂單列表
實作 /sales/orders/rebuild 頁面，做為「重做與退回的訂單」列表
與 orders index 類似，但取得的資料限於「重做與退回」的範圍
參考舊專案 Sales::OrderRebuildListPresenter 來取得資料範圍定義

列表功能與 orders index 一致：
- 需要 infinite scroll
- 需要 filter

三、異常訂單列表
參考舊專案 Order.abnormal scope 取得資料範圍定義
其他功能與 orders index 一致：

四、報備訂單列表
參考舊專案 Sales::OrderFilingListPresenter 取得資料範圍定義
其他功能與 orders index 一致：

二、三、四 頁面基本行為與 index 一樣，只有取得的 orders 資料範圍不同
可以視情況重構優化現有的程式，方便重複使用及未來維護




請將 filter_bar 的樣式參照 claude design 範例的「資料列頁面 · 篩選器」做調整
先以 sales/orders index 頁為基準

- 現在叫 filter_bar，也許改叫 filter_card？
- 分成上下區塊
- 上區塊(header)
	- 標題「篩選訂單」
	- 副標題：目前篩選的符合數，若尚無篩選條件則顯示資料總數就好
	- 若有啟用水平排列分類，用 pills 選單呈現在這裡
		- 若是啟用側欄分類選單，就不顯示，改成下方的 card with sidebar 來顯示分類
- 下區塊(body)有底色，排列各篩選欄位
	- 加上 dealer(經銷商)篩選欄位
		- 請調整(覆寫) tom select 的樣式改成像 claude design 的樣式
	- 加上 order suffix code(用途碼) 的篩選欄位，下拉選單，
		- 選項有：
			- A: 樣本
			- S: 維修
			- C: 重做
			- R: 退回
			- D: 樣品
			- FD: 期貨樣品
			- P: 零件
			- F: 期貨
		- 允許空白(不限)
		- 選項顯示成例如: 「重做(C)」
	- 訂單日期篩選欄位
		- 套用 i18n 翻譯：
			- 1d -> 1天內
			- 3d -> 近3天
			- 7d -> 近7天
			- 15d -> 近15天
			- 1m -> 近1個月
			- 3mo -> 近3個月
		- 參考 claude design 的作法，
			- 點選「指定月」後，出現 年份 & 月份 的選單
			- 點選「指定年」後，出現 年份 的選單
			- 點選「自訂範圍」後，出現 開始日 -> 結束日 的欄位 
		- 可以指定日期條件的目標欄位：
			- 不限(預設=空白)
			- 訂單日期(date)
			- 預計到貨日(date_ship)
			- 出貨日(date_arrive)
			- 若在使用時只有傳入一個欄位名稱，就不用出現目標欄位的選項

請確保樣式和 claude design 範例一致
並且維持 component 的設計架構，方便其他頁面需的時候可以取用


orders index 的資料列表 
欄位順序請調整/修改成：
- 編號
- 類型
- 經銷商(= 原本的客戶)
- 牌價 (chage)
- 經銷價 (chage_sale)
- 案名(label)
- 訂單日期(date)
- 預定到貨日(date_arrive)
- 實際出貨日(date_ship)
- 狀態
不需要：
- 數量
- 刪除按鈕



我要重新設計整個專案的視覺與 ui 系統
請完整理解整個專案的內容、功能，歸納出所以要的視覺元件
整理成一份需求文件，讓我可以將文件提交給 claude design 完成：
- 視覺色彩設計系統的建立
	- 基於 tailwindcss + daisyui
- 各種 ui 元件的設計，
	- 包括一般常見的元件，如 sidebar、topnav、footer、header、breadcrumb、card、badge、tag、各種表單元件
	- 以及這個專案特別需要的元件
- 組裝各種主要的頁面
	- 基本版型 layout
	- 資料列表頁：含分類選單、篩選器與各種變體
	- 資料內容頁：顯示內容 properties / attributes

請先解決各功能的內容，判斷有哪些頁面、元件需要被設計
若有任何疑問或建議請提出討論

已請 claude code 完成新的設計，成果位置路徑：/Users/aaron.kuo/Downloads/ui
接下來請規劃將新的設計套用到專案：

一、先將新的設計規劃轉成 ai_docs/design 下的文件
並確保 claude.md 有正確參照，可依設計系統、概念、元件分成多個不同的檔案
現有的 ai_docs 有關的 design guidelines 也一併做整合
目的套用新設計並重整相關文件，要讓之後的 agent 都能從頭依這份設計文件來工作、開發功能

二、實作各元件的 ViewComponent
目前已有部分 UI ViewComponent 已實作，請依照設計文件套用樣式
若有不在設計文件中的的元件，也依照視覺系統修改樣式，並更新至文件中
若是文件中尚未實作的元件，則建立新的元件
目的是完整化 UI 元件庫，並且更新 storybook 成最新/完整版本的

三、開始將新的設計/視覺/介面/元件套用到現有功能
再拆分多個小階段來套用，可以讓每個階段獨立處理
推薦拆分為：
- app 主頁面 layout、sidebar、topnav、footer 等共用的介面
- app 端功能
	- sales
	- products
	- pricings
	- contacts
	- shippings
	- marketings
	- statistics
	- accountings
- admin 端各功能
- login 頁面
目的是讓整個專案都改套用新版本的設計

注意，此次所有修改都是視覺介面的調整，不該影響任何既有的功能
若有任何疑問或建議請提出討論


## 套用設計

有幾個在新設計中的新樣式，沒有被套用到
請確認 
1: 有確實更新至 ui 元件/css 樣式
2: 定義規範至文件
3: 更新至對應的頁面

包括新設計中的：

###  資料列頁面 · 篩選器

各種篩選器的樣式，要套入到各種資產的 index 頁篩選項樣式中

### 分類篩選版型

水平分類標籤 選單
應該要套用到像是  /products/models, /products/colors, /products/fabrics 或其他類似頁面

垂直側欄分類選單
應該要套用到像是 /marketings/coupons, /shippings/batches 或其他類似頁面

還有其他頁面與各種元件，請確實檢查新的 claude design 的設計有正確被到用到專案中




建立 Dealer Name 元件
DealerComponent .name
會 render dealer 的 地區+城市+編號+名稱
但前三個是
可傳入參數決定是否要 render
可以經由用戶偏好設定自訂是否要顯示

用戶自訂的部分，我偏向透過 css 去控制顯示隱藏
即 render location, city, number 時用 span 元素包起來+ class
然後依用戶的偏好設定在外在層容器套用 class 後，影響 dealer name 的樣式進而控制是否顯示 location, city number 等元素

另外還有參數可以控制 render的結果是否包含 link 可以連接到 dealer show 的畫面

主要用途會套用在資料列table row裡的 dealer 欄

我要為專案整理的 UI System，美化樣式設計並使用 phoenix_storybook(https://github.com/phenixdigital/phoenix_storybook) 建立 demo page

請幫我掃描現有功能下各頁面使用到的 UI 以及現有的 UI Components (lib/hdwcp_web/components/)，進行：
- 整理歸納元件
- 使用 ui-ux-pro-max 重新設計
- 建立 storybook 
- 整理 ui 相關規格文件在 ai_docs

## 需求
請完成以下需求及目標：

一、使用技術
- esbuild + tailwind + daisyui 為基底
- icon 系統使用： lucide
- 下拉選單套用 tom_select 
- 日期選單套用 flatpickr
- 可增加其他需要的 js 套件

二、設計全新元件化 UI 組件
請先掃描所有既有頁面，判斷出可抽出共用的 ui 元件，
UI 元件的設計不涉及業務邏輯，僅處理畫面顯示與操作互動行為

設計與 UI 元件包括：

### 基本常見：
一般網站 app 常見的元件，例如
1. Layout 共用的介面：包括左側的主選單(navigation)、上方的 topnavi、使用者個人選單等等
2. 各種表單 input 元件：例如日期選擇選單，下拉選單、選擇環境的下選單、輸入使用者ID的文字欄為等等
3. label、badge 等
4. 各頁面共用元件，例如：
	1. 麵包屑
		1. 麵包屑帶有下拉選項選單
	2. 頁首標題
	3. 頁面動作選單：resource index 頁面中的「新增 resource」
	4. 資料列選單：resource index 中 table row 對每筆資料的「瀏覽」、「編輯」、「更多動作」等
	5. 資料搜尋列

### 專案特定元件
定義一些本專案會用到的共用元件，例如
- 經銷商(Dealer) 選擇，下拉選單(base on tom_select)，可用 text 輸入篩選
	- 傳入經銷商清單做為選項，
	- 選項顯示為：經銷編號 + 名稱
	- 選項可以分組顯示，分組可以用包括：經銷類型、經銷所在地區
- 年份選擇器
- 月份選擇器
- 日期範圍
	- 開始~結束日期
	- 快速選擇的選項：提供例如 1天內、3天內、7天內…等選項
- 以「產品分類代碼」(Category) 為分類的篩選，類似 pills 的選項

每個 UI 元件：
- 若需要 javascript，來綁定行為，請為各自設計成 Hook，並確保可以重複地被使用
- 若需要 render html 可以包裝成 html component
- 請也檢查目前已經建立的元件，有需要調整的也一併歸納調整

目標請將上述元件整理成一個專案使用的元件庫，使用 storybook 來產出並管理
並建立完整的使用文件與設計原做，做為未來專案製作功能新頁面時，可直接參考引用的元件，且若有新增元件的需求，也能按照規範來建立。

三、整體樣式設計
並基於 tailwindcss 與 daisyui
請使用 ui-ux-pro-max 為整個 app 所需的樣式設計及 ui/ux 做整體性的優化，
符合現代化的操作介面與使用者體驗，

## 建立規範與文件
請將前述的新設計規範、ui 元件庫定義與說明等產出
都匯整建立成完整的文件與 guidelines / 指引，並且加入到 CLAUDE.md 
做為日後開發專案依照的準助

請用 brainstorm 來啟動整個規劃，任何不確定之處請與我討論，或給我專業的建議。
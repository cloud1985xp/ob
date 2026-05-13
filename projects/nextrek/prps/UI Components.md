
我要 migrate 整個專案的UI元件到新版本的前端技術 Stack。
使用 esbuild + stimulus + tailwindcss + daisyui + ViewComponent

請幫我掃描現有功能下各種使用用藥的UI。
整理並製作成在新版版本，前端技術stack下的元件。
需使用USB等等來實作。
並且整合用Loco LC來建立整體的。

## 需求
請完成以下需求及目標：

一、未來預計要摒棄舊的 sprocket，引入新版的 js/css library
- 將所有 coffee script 改寫為 es6 javascript 並用 esbuild 的架構風格
- 一些舊的 javascript class (大多數是為了Form 而設計)，可改寫為 stimulus 的 controller
- icon 系統增加 lucide，舊版先不移除，但新的設計都改用 lucide
- 導入 tom_select 來取代 select2 (目前新版本應該已經有了)

二、設計全新元件化 UI 組件
請先掃描所有舊版本既有頁面，判斷出可抽出共用的 ui 元件，
UI 元件的設計不涉及業務邏輯，僅處理畫面顯示與操作互動行為

例如包括
1. Layout 共用的介面：包括左側的主選單(navigation)、上方的 topnavi、使用者個人選單等等
2. 各種 input 元件：例如日期選單，下拉選單、選擇環境的下選單、輸入使用者ID的文字欄為等等
3. 各頁面共用元件，例如：
	1. 麵包屑
		1. 麵包屑帶有下拉選項選單
	2. 頁首標題
	3. 頁面動作選單：resource index 頁面中的「新增 resource」
	4. 資料列選單：resource index 中 table row 對每筆資料的「瀏覽」、「編輯」、「更多動作」等
	5. 資料搜尋列
	6. 唯讀/受限模式提示列：有些頁面會是唯讀模式 或 受限模式，需要有一個清楚的提示讓使用者知道，可以用 fixed-top 的方式置頂

每個 UI 元件：
- 若需要 javascript，來綁定行為，請為各自設計成 stimulus controller，並確保可以重複地被使用
- 若需要 render html 可寫成 rails helper function，或需要複雜邏輯來處理 ui 呈現邏輯，則可以包裝成 view component
- 請也檢查目前已經建立在新版本的元件，有需要調整的也一併歸納調整

ViewComponent：
目前已有部分已建立的 view component，使用於 端的功能
但可以不要理會目前已實作的部分，請另行製作需要的 view component，以 符合 best practice 以及考慮通用性來實作新的，包括像是：表格(table)、麵包屑(breadcrumb)…等各種需要獨立出來的元件

目標請將上述元件整理成一個專案使用的元件庫，並建立完整的使用文件與設計原做，做為未來專案製作功能新頁面時，可直接參考引用的元件，且若有新增元件的需求，也能按照規範來建立。

# Migrate TagifyWithApiSearch

請將舊版前端 stack 定義的 TagifyWithApiSearch 元件，改寫成在新版前端 stack 的 stimulus 元件


請先參考舊版本設計與用法
app/assets/javascripts/components/tagify_with_api_search.js.coffee
是做為包裝 tagify 的元件，來做為輸入 tags 或 contact 使用
主要用在兩種情境

一、資料編輯表單中填入資料的 input 模式
其中輸入 tags 會是可輸入複數個項目，輸入 contact 時只會輸入一個項目
input 模式可以從即有選項中選擇，也允許輸入不存在的項目來建立新資料

使用方式會是在對應表單(form) js class 中去啟用它，
並在表單行為事件中去綁定需要的動作，例如：

app/assets/javascripts/components/journal_form.js.coffee
app/assets/javascripts/accountings/bulk_form.js.coffee

二、在搜尋表單中設定條件的 search 模式
search 模式只允許從現有的選項選擇，不允許輸入建立新資料
大部分情況不論 tags 或 contacts 都允許複選，少數情況

並且是從 global.js 自動去對有 [data-toggle=tagify] 的元件啟用
參考 app/assets/javascripts/global.js.coffee:168

請根據上述參考資料
改寫成用 stimulus 包裝的新版本作法
新版本檔案放在
app/javascript/controllers/tagify_controller.js
(已有檔案，可以參考或是覆蓋)

全域的 js 進入點：
app/javascript/v3.js

我建議可以把 基本的 tagify 包裝成 tagify_controller
但另外把 tag select 和 contact select 分別定義在各自的 controller，讓它們去使用 tagify_controller
然後把 search 用的也再做成另一獨立的 controller

tagify_controller
tag_select_controller
contact_select_controller

並且可以的話，都用 stimulus 的 data-controller 的方式來啟用，
而不需要再另外透過 global.js 或從 form class 裡去啟用

用 data attribute 來傳入 config 上請維持原本設計，僅調整參數的命名格式成為 stimulus 的方式

請先將新版本的設計、規格、使用方式等釐清後，建立一份文件
放在 ai_docs/ui/ 目錄下，方便之後參考，包括使用的範例等等


## 請再次確認 TagifyWithApiSearch 元件

原舊版本的 TagifyWithApiSearch 有做了一些優化
請再次檢查確認是否有完整地 migrate 到新版本的各相關元件
包括：

- /app/javascript/controller/contact_select_controller.js
- /app/javascript/controller/tag_select_controller.js
- /app/javascript/controller/tagify_controller.js
- /app/javascript/controller/tagify_search_controller.js

請重新檢視各個使用情境
確保新版本 stimulus 的程式能符合需求

完成後記得更新 ai_docs/ui/tagify_stimulus_controllers.md 文件

# 04/18

## 重整標籤/對象的輸入元件

目前設計的 tags / contacts select 元件，有以下的需求
- 預設顯示 top 30 個 items，透過 data attributes 設定
	- 可以用 source option 指向另一個 dom 定義的 items，通常運用在 bulk input 的情況，有很多資料列都有 tags/contacts select，此時會共用相同的 option source
- 當使用者輸入 text 篩選時，會呼叫 api 查詢符合的 item，然後更新選項清單(whitelist)
- 可以用 data attribute 關閉 api 功能，只從那 30 items 中選擇 
	- 發生在 total items < 30 的情況，就不需要啟用 api search
- 允許輸入新的 item，表單送出後會建立新的 item(tag or contact)

但目前有個問題：
在 async request 的情況，使用者送出表單後，不會對表單頁面做 refresh，這時如果送出的 request 是有建立新 item 的情況，這個新 item 就不會出現在 data attributes 裡的 options
又如果這個問題發生在 bulk form 大量輸入的情況，新建立的 item 會需要更新到共同的 source options，然後需要同步到各個 tags/contacts select 欄位

我希望用更乾淨的方式解決這個問題
目前的想法是
建立獨立的 resources store，頁面載入時會先透過載入 (server render or api load) resources store
將 tags / contacts 的 top 30 items 存放在 store，未來若有其他需要的資源也可以放進來

tags 和 contacts select 則就是從 resources store 取得 top 30 items
而當有 tags 或 items 新增時
要觸發 resource stores 去 reload 資料
然後再觸發要求畫面上的 tags/contacts select 更新 whitelist

以上的想法，請幫我評估是否合適

除此之外，目前程式(包括 tag_select, contact_select, tagify_search) 是用 Tagify 這個套件為基底
請同時幫我評估若改成用 Tom Select 這個套件來實作是否可行

若有任何不清楚的問題，請與我討論
如果有更多適合的想法，也請提出建議

1. 同意，用 SSR 注入 top 30 + 暴露 refresh 方法
2. 只有「在 select 內輸入新名稱 → 表單送出 → 後端建立」
3. 允許較大的遷移，可以包括 view、新的 helper、甚至後端 controller、form 的處理，但只需要先套用在 journal imports 的功能，其他地方未來再陸續改用新版本
4. 要完整對等
5. 僅先處理 journal imports，這邊的分頁是用 turbo frame 實作，請確保在此情況下換頁後 select 會套用新的 options


# 2nd
1. 照你的建議放 draft_stages/show.html.erb，換句話說未來其他頁面若需要 preload resource 也是相同作法，請將這個紀錄到文件中
2. 若有需要可以將 JournalForm 改寫成另一個新的 class，例如 JournalForm::V3
3. 可以刪除


# 重整 Import Journals 功能中所有用到的元件

我們正在實作新版本的 import journals 功能
之前有一組用前端實作的參考頁面，包括
- 參考 ./tmp/journal-import-frontend 的程式碼
- 參考 http://localhost:5173/accountings/journal_draft 運行的結果
- 僅參照 form 表單用到的元件，樣式、互動行為
	- 但請忽略裡面的「收付原因」欄位，尚未實作
	- 不需沿用這邊的 js 程式碼，會改依現下專案的情況來實作
	- 不需理會任何與後端溝通的實作，這部分會依現有專案情況來實現

目前我們已經實作了新版本 import journals 的功能，但尚未完全符合需求
不過我想先將目前會用到的各種輸入欄位，建立成完整的元件庫
為它們設計包括需要的 js、helper、或後端 module 或建立使用原則與文件

已知欄位將包括：

- 日期輸入欄位
	- 會使用 date picker 的元件

- 金額輸入欄位
	- 會在欄位前方附上 $ 的圖示
	- 預計會套用 NumericInputController (app/javascript/controllers/numeric_input_controller.js)

- 收付原因欄位
	- 目前實作包括
		- SubjectMenuInputController
		- 與相關的 helper 來 render 所需的 dom
			- FormHelper#subject_menu_inputj
- 對象欄位
- 標籤欄位
- 檔案上傳元件

以下補充各欄位目前仍需要調整和確認的需求

## 實作「檔案上傳」元件

在 import journal 的資料列，每筆資料都有一個「上傳附件」欄位，允許上傳多個檔案，在進行實作前，我想先將「檔案上傳」建立成一個可重複使用的元件。可包括：
- 對應的 form_helper，來包裝 render input fields 
- 對應的 helper 或 partial，來包裝 modal 的 view
- 對應的 javascript controller(stimulus) 來包裝 uploader (dropzone) 的行為

需求
- 程式端用 form_helper 來 render input fields，可接受 form builder、field name 等參數，render 出對應的 html、button 和 attribute 來綁定 js
- 點擊 button 後跳出 modal，出現讓使用者進行上傳的介面
- modal 裡用 dropzone 套件
- 可以透過 data-attributes 來設定 dropzone 的一些 options，例如
	- 支援哪些檔案格式
	- 最多上傳幾個檔案
	- 單個檔案的容量大小
- 上傳檔案後，按下 modal 的確保按鈕，會將上傳的檔案實際輸入 input field 裡

設計和實作時要考慮到上述在 import journal 的使用情境，即同個畫面上會有許多資料列，每列可進行自己的上傳，不互相影響
 

參考：
- 舊版本的上傳檔案實作：
	- app/views/accountings/journal_drafts/_fields.html.erb
	- app/controllers/accountings/journal_drafts_controller.rb
		- 先將更新 attachment to journal_drafts
	- app/assets/javascripts/accountings/journal_draft_bulk_form.js.coffee
		- 僅參考，我的目標是把 uploader 做成可複用的 controller 元件，不限於綁定在 journal draft form

請先將 uploader component 完成後，建立規格文件在 ai_docs/ui/ 裡
再套用到 journal draft (draft stage) 的功能上

為維持 uploader 元件的獨立及可重複使用，
請設計最適合的功能邊界
若有其他責任需是要放到呼叫端(ex: draft stage list controller) 就請明確拆分工作，並將使用方式也寫入文件

## 優化「收付原因」輸入元件

### 欄位輸入的具體行為

收付原因輸入時，會讓使用者經過選單的方式，輸入一組 subject_id 與 menu_id 的選取

本質是提供選項讓使用者選擇，每個選項用 data attribute 來代表它的 subject_id 和 menu_id
- 輸入欄位處會顯示當前已選擇的 item name
	- 若當仍未有選擇的 item，則顯示 placeholder 或預設文字
- 使用者點擊輸入欄位處 (item name or placeholder 文字)，會展開選單(dropdown) 讓使用者選擇選項
- 當選項被選擇時，
	- 將該 item 的 subject_id、menu_id 值填入對應的 hidden input 欄位，
	- 在表單輸入欄位處填上該 item 的 name
- 選單的結構基本支援分層(向右展開下一層的選單)，基本上是兩層的選單，但也可以支援更多層，而前面的層數都只是分類，只有最末層的選項才是可被選擇的 item
### 既有程式實作
目前已有實作用於輸入「收付原因」的 input 欄位
請參考相關程式碼
- SubjectMenuInputController: app/javascript/controllers/subject_menu_input_controller.js
- 相關的 helper 來 render 所需的 dom
	- FormHelper#subject_menu_input, #render_nested_menus_v3 
- 相關的 view: 
``` 
app/views/shared/components/v3/_subject_menu_input.html.erb
與
app/views/shared/components/v3/subject_menu_input/*.html.erb
```

### 待完成的項目
目前尚未完成的部分：

一、增加輸入文字進行篩選
選單點擊展開時，輸入欄位處要變成一個文字欄位，
讓使用者可以經由輸入文字，來篩選選項
當進行篩選時，會只用一層的方式將符合的 item 列出，讓使用者選擇

二、整體 ui 美化
要為整個元件做樣式/ ui 上的美化，
請以 daisyui 的風格設計為主，符合現代化的 html/css 作法，且兼顧友善的 ui/ux

請先完整了解目前的程式碼與需求
再進行上述未完成的項目實作
並且請試著改善整體程式碼的品質


二、對象與標籤欄位






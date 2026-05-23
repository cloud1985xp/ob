我要為專案整理的 UI System，美化設計並使用 phoenix_storybook(https://github.com/phenixdigital/phoenix_storybook) 建立 demo page

請幫我掃描現有功能下各種使用到的 UI。

注意：
有一部分的功能 (v3 layout) 已經是開始採用新前端版本
沒有使用 v3 layout 的就是舊版本

## 需求
請完成以下需求及目標：

一、未來預計要摒棄舊的 sprocket，全面引入新版 (v) 的 js/css library
- 將所有 coffee script 改寫為 es6 javascript 並用 esbuild 的架構風格
- 一些舊的 javascript class (大多數是為了Form 而設計)，可改寫為 stimulus 的 controller
- icon 系統增加 lucide，舊版先不移除，但新的設計都改用 lucide
- 導入 tom_select 來取代 select2 (目前新版本應該已經有了)

二、設計全新元件化 UI 組件
請先掃描所有舊版本既有頁面，判斷出可抽出共用的 ui 元件，
UI 元件的設計不涉及業務邏輯，僅處理畫面顯示與操作互動行為

設計與 UI 元件包括：
1. Layout 共用的介面：包括左側的主選單(navigation)、上方的 topnavi、使用者個人選單等等
2. 各種 input 元件：例如日期選單，下拉選單、選擇環境的下選單、輸入使用者ID的文字欄為等等
3. 各頁面共用元件，例如：
	1. 麵包屑
		1. 麵包屑帶有下拉選項選單
	2. 頁首標題
	3. 頁面動作選單：resource index 頁面中的「新增 resource」
	4. 資料列選單：resource index 中 table row 對每筆資料的「瀏覽」、「編輯」、「更多動作」等
	5. 資料搜尋列

每個 UI 元件：
- 若需要 javascript，來綁定行為，請為各自設計成 stimulus controller，並確保可以重複地被使用
- 若需要 render html 可寫成 rails helper function，或需要複雜邏輯來處理 ui 呈現邏輯，則可以包裝成 view component
- 請也檢查目前已經建立在新版本的元件，有需要調整的也一併歸納調整

ViewComponent：
目前已有部分已建立的 view component，使用於產生 daisyui 的元件
可以不要理會目前已實作的部分，請另行製作需要的 view component，以 符合 best practice 以及考慮通用性來實作新的，包括像是：表格(table)、麵包屑(breadcrumb)…等各種需要獨立出來的元件

目標請將上述元件整理成一個專案使用的元件庫，使用 lookbook 來產出並管理
並建立完整的使用文件與設計原做，做為未來專案製作功能新頁面時，可直接參考引用的元件，且若有新增元件的需求，也能按照規範來建立。

三、整體樣式設計
新的設計請參照，這個路徑的(設計)專案
> /Users/aaron.kuo/projects/nextrek-tw

或是拜訪這個網址：
> http://localhost:5173/

但請不要直接使用這個專案的程式碼，請調整成符合這個專案的規則與結構
並基於 tailwindcss 與 daisyui
若有需要請使用 ui-ux-pro-max 為整個 app 所需的樣式設計及 ui/ux 做整體性的優化，
符合現代化的操作介面與使用者體驗，

## 建立規範與文件
請將前述的新設計規範、ui 元件庫定義與說明等產出
都匯整建立成完整的文件與 guidelines / 指引，並且加入到 CLAUDE.md 
做為日後開發專案依照的準助

請用 brainstorm 來啟動整個規劃，任何不確定之處請與我討論，或給我專業的建議。
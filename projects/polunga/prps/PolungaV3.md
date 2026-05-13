# Polunga v3 UI/UX Design

我想重新設計前端的所有頁面，改成用新版本前端 stack：esbuild + stimulus + tailwindcss + daisyui
## 範圍
目前已有套用到 admin 端的功能 (layout admin、路徑 admin 下的功能)，這部分先不要改動

要調整的範圍是 admin 以外的主要功能
並且不要影響到所有既有功能的運作。

## 需求
請完成以下需求及目標：

一、摒棄舊的 sprocket，引入新的 js/css library
- 將所有 coffee script 改寫為 es6 javascript 並用 esbuild 的架構風格
- 一些舊的 javascript class (大多數是為了Form 而設計)，可改寫為 stimulus 的 controller
- icon 系統增加 lucide，舊版先不移除，但新的設計都改用 lucide
- 導入 tom_select 來取代 select2

二、設計全新元件化 UI 組件
請先掃描所有舊版本既有頁面，判斷出可抽出共用的 ui 元件
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

ViewComponent：
目前已有部分已建立的 view component，使用於 admin 端的功能
但可以不要理會目前已實作的部分，請另行製作需要的 view component，以 符合 best practice 以及考慮通用性來實作新的，包括像是：表格(table)、麵包屑(breadcrumb)…等各種需要獨立出來的元件

目標請將上述元件整理成一個專案使用的元件庫，並建立完整的使用文件與設計原做，做為未來專案製作功能新頁面時，可直接參考引用的元件，且若有新增元件的需求，也能按照規範來建立。

三、設計全新的頁面基本 Layout 
包括

A. 整體畫面的 Layout，包括
- 左側主選單，支援兩層式架構，第一層選單帶有 icon
	- 選單可以被收合或展開
- 上方 topnavi，次要選單，可先放幾個 mock 選項，另包括使用者個人的下拉選單
- 主要 content body 的區域，需規劃好，並且允許以下主內容可能的情況能正常運作
	- 可能會有 fixed-top 的元素
	- 可能會有 fixed-bottom 的元素
	- 可能會有 float (如 dropdown menu、popup) 的元件
	- 可能會 modal 的元件

B. 內頁的基本 Layout
- 頁首 Hero section，頁面標題
- breadcrumb (麵包屑)
	- 註：有些頁面的 breadcrumb 的 item 會帶有可以點擊展開下拉選單的形式，常用在「可以切換環境」的功能下
- 頁面唯讀模式，有些頁面進入唯讀或受限模式時，要出現該提示
- 頁面的動作選單，應該統一按排在一致的位置，動作選項可規範最多 4 個動作，若超過就要用下拉選單來展開

C. 內頁一般資料表格的設計
- 表格要有一致的樣式設計
	- 表格的表頭，要可以支援 fixed-top 的行為(視窗向下捲動超出時，表頭會固定在畫面上方)
	- 特定類型欄位，例如 boolean 值，也用一致的 ui 呈現
	- 資料列的動作選單，也要一致的位置與樣式，可規範最多3個動作，若超過就要用下拉選單來展開
	- 有些表會需要支援「複選 -> 批次動作」的行為
- 資料表格的篩選器設計

D 內頁呈現資訊的設計
- 通常用於 show 頁面，用來呈現某項資源的屬性，例如文章的標題、分類、作者、內文…等，可用 dl, dt 或適合的結構

三、整體樣式設計
請重新設計整體樣式，基於 tailwindcss 與 daisyui
使用 ui-ux-pro-max 為整個 app (admin 除外) 設計新的 ui/ux 與 design，符合現代化的操作介面與使用者體驗，app 是內部後台的管理系統，介面盡量以簡潔清新為主，可帶有日式設計風格。

並將設計好的樣式、元件，套到現有的頁面，如列表、show 頁面，表單等等
若有頁面較為複雜不適合直接列用，可以先不處理，列入 todo 後面再詳細討論作法

四、確認需另外進行的部分
以下功能頁面，請先只列入參考，但此次不修改，請列入後續計畫來單獨討論進行
包括
- /operations/ 路徑下的頁面
- /campaigns/ 路徑下的頁面
- AnnouncementsController 相關功能
- BannersController 相關功能

這些頁面請在 scan 專案時列入考慮，裡面有可能也需要共用的要素，
但也有許多較複雜的行為，所以放到後面再另外進行處理

## 建立規範與文件
請將前述的新設計規範、ui 元件庫定義與說明等產出
都匯整建立成完整的文件與 guidelines / 指引，並且加入到 CLAUDE.md 
做為日後開發專案依照的準助


請用 brainstorm 來啟動整個規劃，任何不確定之處請與我討論，或給我專業的建議。

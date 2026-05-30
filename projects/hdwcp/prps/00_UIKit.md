
## 建立 Dealer Name 元件
ex: HdwcpWeb.Components.UI.Dealer.name (或建議命名方式)

會 render dealer 的 地區+城市+編號+名稱
但前三項(location, city, number) 是：
- 可傳入參數決定是否要 render
- 可以經由用戶偏好設定自訂是否要顯示

用戶自訂的部分，我偏向透過 css 去控制顯示隱藏
即 component 依然會 render location, city, number ，產生時會用 span 元素包起來 + 加上 class
然後整個頁面(全域)依用戶的偏好設定，在外層容器套用對應的 class ，來影響 dealer name component 內的元素樣式，進而控制是否顯示 location, city number 等元素

另外還有參數可以控制 render 

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
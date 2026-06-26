
Operation Issue Difference Show 頁面(ex: /operations/355/issues/7510/difference) 
中所 render 的 presenter:
> app/views/operation_issue_differences/_presenter.html.erb

原本畫面中的 tab 選單有 "editor" 項目，連到 edit 畫面  (ex: operations/355/issues/7510/difference/edit)
但因為 migrate 成 tab component 而消失了
請調整 tab component 讓它仍可以支援看似 tab item 但實際是連到獨立畫面的選項
然後把 editor 的連結加回來

調整 operation issues 頁面，例如：/operations/385/issues
在 operations/index.html.erb 頁面中的 issue 清單，應該要和 operations/list.html.erb 中一樣：
- column toggler 與 expand 的功能，放到 card toolbar 的位置
- description 與 resources 欄的 expand 展開/收合效果
- 但不需要 filter

若可以的話，將兩個 view 共用的 issues list 抽成 partial

調整 operation_issue_lists 頁面 (位於 app/views/operation_issues/list.html.erb) 的內容排版與 UI

description 欄的資料，樣式 v3-expandable 請調整以下效果
- 當展開時有簡單的動畫對顯示內容做展開/收合
- 收合狀態下，內容會有漸層的半透明覆蓋讓內容向下漸漸隱去的效果

將 filter 
> app/views/operation_issues/_filter.html.erb

的樣式與排版做美化
這個 filter 會有多個查詢條件
我想將「送出」與「 clear」 的選項安排在右側，僅佔適合內容的最小寬度，剩餘的空間都給左側的條件欄位，並請美化各個輸入欄位的樣式，包括
- 文字輸入
- 開始 - 結束時間的條件輸入

- 將 column-toggler 的項目，移到 CardComponent 的 title 右方 toolbar，做成一個可以展開下拉選單的按鈕，用設定的圖示來代表
- 按鈕展開下拉選單後，選單列出各個 column toggler



# 重整 v3 UI Branches
我要整理目前開發的變動的進行 branch 拆分與 commit
主要分成以下部分。

一、新版本 v3 fronted 基礎建設
branch: feature/rails8-fronted-base (現在的 branch)

內容包括新版本前端的基礎配置更新，
包括使用：tailwindcss + daisyUI + stimulus controller、更新 js library(例如 toast、tom-select)
以及導入 ViewComponent 與整合 SimpleForm
並且對 AI Agent 開發測試整合 tidewave 與 使用 playwright-cli SKILL、e2e 測試
另有一些是刪除不再使用的功能

二、實作各種 view compnoent 並導入 lookbook
branch: feature/rails8-fronted-uikit
實作各種 UI Component 元件、包括對應的 stimulus controller or js library 
更新 kaminari 分頁元件樣式
並建立 lookbook 做為 UIKit demo site

三、套用新版 UI/Fronted Stack 至實際功能
將 v3 的 fronted stack 套用至實際的功能，並也實作對應功能所需的 stimulus controller 或 js library
第三部分可以再依以下分類分成多個 branch 
- v3 application layout migration，branch: feature/rails8-fronted-migration-base
- 套用 admin side 相關功能，branch: feature/rails8-fronted-migration-admin
- 套用 app side (非 admin side)，branch: (branch: feature/rails8-fronted-migration-app)
  但以下功能頁面先排除，仍在修改確認中
	- /operations/ 路徑下的頁面
	- /campaigns/ 路徑下的頁面
	- AnnouncementsController 相關功能
	- BannersController 相關功能

請比較與bese Branch (feature/upgrade-rails) 的差異詳細理解上述的改動內容。
若有不再上述描述的範圍內，也請進行分析與分類。
若無法辨別分類，請先與我確認。
若完全與開發無關，的 untracked 變動，可以忽略(保留檔案)
確認分析之後進行 Commit 與 branch 的拆分，請讓各 branch/commit 有清楚乾淨的相依

照理最後的結果跟現有的內容要完全一樣，不去修改任何程式變動。

若有任何建議或問題，請提出討論




# 修正以下 v3 ui / component 的問題：

- v3 TableComponent 的標頭 sticky 效果
	- 請修正啟用 sticky 的效果，目前 sticky 的效果不正確，
	- 調整成可以用參數控制是否要套用 sticky，預設為關閉
- v3 的 sidebar menu, 在 collapse 收起的狀態下，menu item 的 submenu 會無法使用，應該要轉換成 hover 時會 popup submenu 在右方的行為

# Polunga v3 UI/UX Design

我調整 layout (v3) 使用的前端 stack
改成 esbuild +  tailwindcss + daisyui + stimulus，摒棄 sprockets
並安排將整個 app 原本使用 v1 的 layout 的功能
都改套用到 application layout (v3) 的版本
## 需求
請完成以下需求及目標：

一、改用 esbuild + tailwindcss + daisyui + stimulus
- 將 layouts/application 改用 esbuild 並引入新的 javascript + styles(css library)
- 將 flowbite 改成 daisyui
- 將所有 coffee script 改寫為 es6 javascript 並用 esbuild 的架構風格- 
- 一些舊的 javascript class (大多數是為了Form 而設計)，可改寫為 stimulus 的 controller
- icon 系統增加 lucide，舊版先不移除，但新的設計都改用 lucide
- 導入 tom_select 來取代 select2

二、Simple Form 整合：
請增加一組 simple form 的 wrapper，符合 tailwindcss + daisyui 的 form 樣式
並將這組 wrapper 設為預設，舊的頁版才仍使用舊版的 assets + wrapper(bootstrap)

三、設計全新元件化 UI 組件
將所有 ui 元件，改套用 tailwindcss + daisyui 的樣式
除了基本的常見元件庫之外
請先掃描所有舊版本既有頁面，判斷出可抽出共用的 ui 元件
UI 元件的設計不涉及業務邏輯，僅處理畫面顯示與操作互動行為

包括像是：
1. Layout 共用的介面：包括左側的主選單(navigation)、上方的 topnavi (navbar)、使用者個人選單等等
2. 各頁面共用元件，例如：
	1. 麵包屑
		1. 麵包屑帶有下拉選項選單
	2. 頁首標題
	3. 頁面動作選單：resource index 頁面中的「新增 resource」
	4. 資料列選單：resource index 中 table row 對每筆資料的「瀏覽」、「編輯」、「更多動作」等
	5. 資料搜尋列
	6. 常見的元件，例如：
		1. Tab
		2. Accordion / Collapse
		3. Menu / DropdownMenu
		4. Card 
		5. Pagination
	7. Feedback 類型元件，例如：
		1. Alert
		2. Loading
		3. Progress
		4. Tooltip
		5. Toast
	8. 唯讀/受限模式提示列：有些頁面會是唯讀模式 或 受限模式，需要有一個清楚的提示讓使用者知道，可以用 fixed-top 的方式置頂

每個 UI 元件：
- 若需要 javascript 來綁定行為，請為各自設計成 stimulus controller，並確保可以重複地被使用
- 若需要 render 複雜的 html 或需要複雜的處理邏輯，請包裝成 view component 

ViewComponent：
目前已有部分已建立的 view component，使用於 admin 端的功能
先可以忽略目前已實作的部分，請另行製作需要的 view component，以 符合 best practice 以及考慮通用性來實作新的，包括像是：表格(table)、麵包屑(breadcrumb)…等各種需要獨立出來的元件

目標請將上述元件整理成一個專案使用的元件庫，並建立完整的使用文件與設計原做，做為未來專案製作功能新頁面時，可直接參考引用的元件，且若有新增元件的需求，也能按照規範來建立。

三、設計全新的頁面基本 Layout 
包括：
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

四、整體樣式設計
請重新設計整體樣式，基於 tailwindcss 與 daisyui
使用 ui-ux-pro-max 為整個 app (admin 除外) 設計新的 ui/ux 與 design，符合現代化的操作介面與使用者體驗，app 是內部後台的管理系統，介面盡量以簡潔清新為主，可帶有日式設計風格。

並將設計好的樣式、元件，套到現有的頁面，如列表、show 頁面，表單等等
若有頁面較為複雜不適合直接列用，可以先不處理，列入 todo 後面再詳細討論作法

## 初步套用範圍
目前 v3 已有套用到 admin 端的功能 (layout admin、路徑 admin 下的功能)，
可以先從這部分進行調整，確認 esbuild + daisyui 
但不要影響到所有既有功能的運作。

接著將 app side (非 admin 端) 的功能也開始套用 v3 的版本
但以下功能頁面，請先只列入參考，但此次不修改，請列入後續計畫來單獨討論進行
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
並使用 lookbook，建立所有 ui 元件的元件庫 demo 頁面

任何不確定之處請與我討論，或給我專業的建議。

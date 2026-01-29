# 修改 Miles 處理 google_sheets Document 的作業流程

## 目標

Miles 的 document 功能
目的是將從 document 解析得到的 terms 加到 Package 裡

Document ~ Term 之間的關聯是：
- Document has many subject
- Subject has many entries
- Entry has many terms

(目前已經實作好這段資料的建立)

使用者的操作是會從指定的 Document 裡，挑選 term 加到 Package

> Term belongs to Package

挑選 Term 的操作方式會因 Document type 而不同

而當 Document 的類型(type) 是 Google Sheets (:google_sheets) 時
使用者會採用
直接選擇 subjects 後，將所選的 subjects 下的所有 terms 加到指定的 package 中

## 核心作業模組

在 Miles.Documents 模組中，建立一個方法 apportion，來完成

輸入 document、subject_ids、目標 package 後

將該 document 下對應的 subjects 下的 terms 全部關聯到目標 package

目標 package 有可能是

- 指定既有的 package_id: 直接更新 terms, subject 的關聯
- 新建立的 package: 先建立 package，再更新至 terms、subject 的關聯
- 新建立的 package 時欄位為：
	 - name
	 - date

請在在 subject 上增加一個 package_id 欄位來關聯該 subject 已被指定到哪個 package

如果在執行後，發現該 Document 的所有 subjects 都有被指定到 package
就會把 Document 的狀態(status)標記成 completed
註：請幫 Document status 的 enum 值加上 :completed

## 操作畫面需求

當使用者進到 DocumentLive.Show 畫面，且瀏覽的 document:

- status = processed (表式已經完成解析 subjects/entries/terms 資料)
- 且 type = :google_sheets

這時候會 render 這次要製作的功能頁面：讓使用者直接從挑選 subjects 的方式，將其下的 terms 加入目標 package

規劃操作頁面的流程如下

先將 subject 依目前所屬 package 分組，也就是會分成

- 尚未有 package ：表該 subject 還沒關聯到 package，即表式正在待處理中，可以被選擇
- 已有 package，已經被處理過，會顯示內容但不允許其他操作

使用者可以切換瀏覽各個分組，這邊可以用 tab 來呈現
因此 tab 的標題就會是「處理中」或 package 的名稱(ex: 分組1、分組2)

### 處理中的頁籤

處理中的頁籤內容區塊，最上方會有 package 的表單，
可以讓使用者「選擇目標 package」或「建立新的 package」

選擇目標 package，會有下拉選單供選擇，選項為目前專案「狀態為 pending」的 packages

選擇建立新的 package 時，會出現欄位：

- name: 輸入自訂名稱
- date: 輸入一個日期

兩者為必填欄位

內容區塊下方則是會顯示各個 subject 的內容
以垂直區塊的方式依 subject 的 scheme 排序顯示

每個 subject 標題都有 checkbox，讓使用者勾選表示要將這個 subject 加到對象 package
每個 subject 預設是收合的，點擊標題末端的圖示可以切換展開/收合內容
每個 subject 內容展開後高度是固定的，若超過高度會出現捲軸
每個 subject 內容若超過容器寬度，會出現水平捲軸
每個 subject 內容是用表格呈現：
- 每一列就是一個 entry
- 每一欄就是 entry 的每個 terms
- 可使用或參現有的 `MilesWeb.WikiHTML.wiki_section` component 來呈現這個表格的內容

程式碼參考：
```
subjects = Documents.list_document_subjects(document.id)

<MilesWeb.WikiHTML.wiki_section subject={subject} />
```

### 已處理= package 頁籤

即顯示該 package 下的各 subjects 的內容
這裡的 subjects 則也是用 tab 頁來顯示
每個 tab 內容，用 table 列出該 subject 的內容
- 每一列是一個 entry
- 每一欄就是 entry 的每個 terms

可以參考使用現有的 <.tab_items> Component 來處理
- apps/miles/lib/miles_web/controller/package_html/package_processed.htmlheex


## 處理流程

- 使用者在勾選要處理的 subjects
- 選擇目標 package (選擇或新增)
- 提交表單
	- 發送 request 至 /miles/documents/:document_id/apportion
	- 由 DocumentController apportion action 處理
	- 呼叫 Documents module 的 apportion 方法
	- 傳入 所選擇的 subject ids 與 目標 package 的參數
	- 完成將 terms 加到 package 的工作


# 修正處理中的頁籤

請對 document_google_sheets Component 裡的 處理中 的 tab 頁面
修正以下問題
- 每個 subject 的標題無法被勾選，點擊 checkobx 會先被 collapse 的效果影響，請修正
- 增加一個「全選」的選項，一次全選全部或取消全部，請參考 apps/miles/assets/js/controllers/toggleController.js 是否能直接使用
	- 但請不要修改 toggleController.js 的原有行為

# 允許把已處理的 subject 收回

在已處理的每個 package 頁籤中

如果 package 的狀態是 pending，那會允許使用者可以把這個 package 中的 subject 收回

在允許收回的 package 中
- 對每個 subject 區塊加上一個按鈕「從 package 移除」的按鈕
- 當使用者按下後，發送 request 至 DocumentController withdraw action 處理
- 呼叫 Documents module 的 withdraw 方法
- 將該 subject 的
	-  package 關聯移除
	- subject 下的 terms 的 package 關聯也移除

等於是該 subject 會回到處理中的狀態

# 增加測試

為這次的功能增加測試，主要包括

- Miles.Documents
- MilesWeb.DocumentController

若檔案的測試不完整，請補足它，提升覆蓋率






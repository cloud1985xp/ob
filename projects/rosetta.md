# 更新 Translation Source Admin

一、調整 show 頁面
目前進到 show 頁面預設就是連到 source string 這個子頁面
但載入 source string 會拖慢速度
我想改成預設是另一個簡單的 summary 頁，要瀏覽 source string 時再點擊 source string 的頁籤

二、增加 segmentation rules 管理
目前 show 頁面裡有 segmentation rules 的子頁面，會列出 rules
這邊要加上可以新增/修改/刪除 segmentation rule
請實作這項功能

追加修改：
三、優化 Translation Sources 列表
可參考目前 /subjects 的作法，主要包括
- 查詢 namespace + property 的作法，改成 string 欄位，並自動依「.」 判斷比對 namespace 或 namespace + property
	- 其他篩選條件保留
- 在 full name 旁加一個 edit 的 icon 連去 edit 頁面
	- 移除 view 和 edit 的按鈕	- 
- 移除 delete 按鈕，移到 show 頁面右上方的 page action，要有 confirmation 才執行 delete
- Enabled, Ingestion 等資訊合併到 Full Name 欄，獨立一行排列顯示
- Translation strategy 欄用和 /subjects 頁列表一樣的呈現方式 (共用 component)
	- 所以 pattern 欄會合併進 translation strategy
- Associated Glossary 維持現在呈現，要可以一眼看清有哪些，但改成依文字排序


# 重新設計 UI

我要搭配 claude design 來重新設計整個 application 的視覺與介面
請基於以下需求幫我完成委託 claude design 進行作業

## 基本 layout 調整
將主選單移到上方 navbar，選項用 dropdown 向下展開
有些功能在進入後，才會再採用有左側欄的頁面 layout (後面說明)

## 重整選單
調整選單結構如下：
- Dashboard
- Database
	- Subjects
	- Masterdata
	- Glossary
- Operation
	- Source Documents
	- Packages
	- Export
	- Import
- Tools
	- Reviews
	- Corrections
	- Translation Batches
- Management (原本的 settings)
	- Trainings
	- Translation Sources
	- Source Strings
	- Kotoba Models
	- LLM Usage
- Dev (另增加的項目，放仍在開發測試中的功能)
	- Agents
	- Evaluations
	- Search

## 重新設計 UI / Layout / Components
基於前述的調整，我需要整合用 claude code 來重新設計整個專案的設計
包括 配色視覺定義、基礎元件、各項功能項面、共用元件等

我列出幾個明確主題的需求：

### 一、基礎元件
包括像是選單、breadcrumb、page action buttons、row action buttons
各種表單元件、篩選器等

### 二、兩階段資訊顯示的頁面
像是 Masterdata, Glossary 這類功能，會有「先選擇 namespace」，再顯示列表的兩層
我想做成有左側 sidebar 選單先選擇 namespace，右方顯示資料列表：

- 左側列表有 text field 可以快速篩選 namespace
- 右方列表畫面，有 breadcrumb、page header、資料 table
- 有篩選/搜尋功能

這樣的頁面 layout 需要被設計

### 三、關鍵元件
有一些屬於這個 app 的重要 ui 元件，會在多個地方使用，我特別提出來：
#### TextContent
用來顯示 翻譯(包括原文) 文字
同時有單行或多行
顯示要完全維持原本文字的格式，不自動換行
也要避免掉因 lint format 造成 html 內容的起始空白
用到的地方很多，包括
- glossary/masterdata 的資料列裡的各語言文字內容
- export 的結果，也會顯示各語言的內容，也要替換成統一的組件
- source document，parse 完後的結果(原文)，也屬於文字內容
- packages 顯示的原文和譯文
- translation batch / corrections 的內容
- translation source 下的 source strings 或直接瀏覽 source strings，顯示的內容

目前現有許多不同的實作方式，像是：
- package_translation_card 裡的顯示內容
- glossary_live/index.html.heex 裡直接輸出的寫法
- 以及套用 .whitespace-pre-wrap 樣式的作法

此外，要對這個組件加上「可以點擊直接複製內文」的功能
#### TranslationSource (Subject) Card
目前沒有這個元件，會需要增加
主要是有許多頁面會同時列出多個 translation source 下的資料的巢狀資訊
例如在 package 裡，有多個 translation sources 的資料列
目前是用類似 tab 頁來呈現
但使用者在這裡會需要可以觀看關於 translation source 的資訊，主要包括它的 translation strategy 的內容，用哪種 strategy、用哪種 model、有哪些 pattern、關聯哪些 glossaries
(可參考 /subjects 裡的資料列的資訊)
所以我想定義 SubjectCard，可以在 translation source 被點擊或滑過時，show 出 subject card

### JobLog 
多個情境會有「長時間執行的背景工作」會讓使用等候，並在畫面上印出 log 紀錄
這個目前應該已經有共用元件，請考慮為它優化視覺設計


### 四、特別功能頁
有幾個功能的資訊比較複雜，例如 packages、reviews
請依這個些功能現有的內容、操作需求，產生要 claude design 進行設計的規劃
並設計獨立的頁面版型給我確認

### 五、基本的 CRUD 
Settings (Management) 下的功能，基本會有
- 列表頁 (帶有搜尋/篩選)
- 瀏覽頁
- 新增/編輯的表單頁

## 其他注意
- dev 下的功能 (agent、evaluation、search)，可先不特別設計，先套用基本的 CRUD 的基本樣式即可
- 此次目標只做視覺/UI的改善，切記不可以影響所有現有的功能運作
- 可以擅用 ui-ux-pro-max 的 skill 來強化你的設計規劃

## 目標
請詳細了解現有專案的功能、常見ui元件 pattern
建立一份可以讓 claude design 進行完整提案的規格，
並指引我向 claude design 合作進行這項作業

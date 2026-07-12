---
tags:
  - nextrek
  - project
---
# 0712
暫存區的草稿操作改成以「Import 批次」來處理

進入到資金帳戶暫存區 (draft_stages#show) 時
列出的草稿資料 (journal_drafts) 改以匯入批次 (import) 來處理：
- 依作業中的 import 分批處理 journal drafts
- 一進入畫面預設先用最新的 import 來取出
- 在 header 下方，增加一列作為 import 批次的區塊：
	- 批次選單，用下拉選單 (dropdown)，
		- 每個選項的格式：「 {icon} + {上傳日期} 上傳（{pending count} 筆）」:
			- icon 用 lucide 的 `layers` 圖示
			- 文字例如：2026-05-14 上傳（8 筆）
		- 列出所作業中 import 作為選項，點擊後切換成目標批次
			- 選項從舊到新排序
			- 當前的選項有 active 效果
	- 批次選單未展開的狀態，是顯示當前的選項，
	- 批次選單後方有一個垃圾桶的圖示，用來將當前的 import 整批刪除
		- 點擊會跳出 SweetAlert 供確認
			- sweet alert title：確認刪除 
			- sweet alert content：確定要刪除整批資料嗎？此操作無法復原。
		- 確認後送出，
			- 會將該筆 import 與其相關的 import detail 和 journal drafts 全數刪除
			- 並導回 (redirect) draft_stages#show 畫面
- 改為依 import 批次後，原本的篩選 (query)，也要以該批次為條件範圍下做篩選
- 


# 0701
## 增加資金帳戶匯入批次上限
為了要限制在資金帳戶暫存區進行上傳時(ex: /accountings/draft_stages/:id/import)
當使用者送出上傳表單時，進行檢查，目前已經存在且仍在作業中的的上傳批次(Accountings::Import) 數，不得超過5筆：

- 若作業中的批次已有5筆，會阻止進行上傳，並出現訊息「已達上傳批次上限 5 批，請將先前批次資料建成交易或刪除」的錯誤訊息 (用現有元件整點 toast 訊息效果)
- 需要對 import (Accountings::Import class) 增加一個 pending_count 欄位來代表它還有多少 journal_drafts 仍未處理
	- 若 pending_count 大於 0 即代表該批 import 仍在作業中
- 在資金帳戶暫存區 (/accountings/draft_stages/:id) 對資料列 (journal_draft) 作操作時，包括單筆的建立/刪除、選取多筆的建立/刪除，在執行完動作時都要更新對應 import 的 pending_count
- 注意.. 
	- 只有 import 的 state 是 completed 才列入有效的 import，也就是必須是有效的 import 才算入上限5筆的檢查
	- 目前 import has_many details 但 always 只取用第一筆 detail

請先了解並檢查現有相關的架構和程式，評估規劃作法
若有任何問題或建議請先提出討論

檢查上限5筆，應該要包含正在上傳中的 import，不僅限於上傳完成
以免使用者若短時間連續上傳，最終有可能超過5筆
所以計算是否達上限 5 筆時，應該要僅排除 failed state，這樣合理嗎

# 0627

調整在資金帳戶暫存區頁 (ex: /accountings/draft_stages/:id) 裡的篩選功能
在「新增篩選條件」裡的收付日期，要改成用 date range 的型式
點擊輸入欄位要出現 date range picker (datepickr 應該有支援)
需確認/修改 server 端執行的 query 可以接收 date range 傳來的參數做條件查詢
但要確保不破壞原本既有的 query 功能

選擇後一樣會在「篩選」旁邊顯示目前的條件內容，例如：「收付日期: 2026-06-03 - 2026-07-07」

在資金帳戶暫存區頁 (ex: /accountings/draft_stages/:id) 裡
參考 `_row.html` 裡的金額(amount)、手續費(fee)輸入欄位
將  `_selection_bar.html.erb` 裡的 amount, fee 欄位
也採用同樣的 Ui::Forms 的 Component，且：
- 套用一樣的 maxlength 設定 
- 維持 selection bar 原本的外觀樣式
- 不需要 prefix
- 不要影響原本的功能

# 0625 修正
請優化修正 ContactSelectComponent 與 TagSelectComponent
包括對應的 js：
- app/javascript/controllers/tag_select_controller.js
- app/javascript/controllers/contact_select_controller.js
- app/javascript/controllers/tom_select_base_controller.js

目前主要使用在新版交易匯入功能的資金帳戶/交易暫存區頁 (accountings/draft_stages/:id)
裡的標籤輸入與對象輸入的欄位中(未來也會在其他輸入表單中使用)

兩者有大部分有著共同的行為，差別僅在於可允許輸入/選擇的資料量(max_tags) ：
ContactSelect 通常為 1，而 TagSelect 上限為 5

現有的 tom_select 有限制 item(tag) 文字長度上限(30)
當使用者想要建立超過上限的tag時，會無法完成tag建立
但我想改成允許建立，但會把超過文字長度上限的部分自動捨去

主要修正針對「從剪貼簿貼上的方式輸入」，
當剪貼簿複製時：
- 可能會是從來源複製多行的內容(例如從 excel 多個儲存格)，要把每行內容都視為獨立的 字串，來比對候選清單中符合的 item
- 可能複製的內容是用 tab (\t) 來區分多筆字串，貼上輸入時要拆分成多個字串，來比對候選清單中符合的 item
- 但上述仍受 max_tags 限制，即若拆分後有超過 max_tag 上限數量時，超過的部分就自動捨去
- 但若 max_tags 為1，貼上後(多餘的會捨去)的內容不會自動變成 item，會仍維持在輸入狀態，用該文字篩選候選清單後，讓使用者選擇
- 在啟用 api search 模式的情況下，因為貼上的多個字串有可能不在目前的候選清單中，所以會需要將所有輸入的字串，用 「,」合併後當參數送給 api，取得符合的候選清單，再變成選取的 item。

請修正符合上述的使用操作情況，並盡量簡化對的 js 修改，降低程式複雜度
若有任何問題或建議，請與我討論

# 0624 修正

上傳明細頁 (/accountings/draft_stages/:id/import)
右方的「上傳檔案說明」的第2項「已上傳筆數：」調整內容如下：

```
已匯入批次：{N} 批（每個資金帳戶上限：5 批） 
```

N = 用該資金帳戶現有的 Accountings::Import 資料量來計算

並確保:
上傳產生的 journal drafts 資料在建立成 journal 或被刪除，要檢查來源的 import 是否

上傳明細頁 (/accountings/draft_stages/:id/) 裡的 selection bar


當收付原因 / 對象 / 標籤 / 營業稅 / 附註 / 手續費 / 憑證日期欄位，套用完後，已選擇/輸入的資料應該要被清空還原

上傳明細頁 (/accountings/draft_stages/:id/import) 裡的「篩選」操作與功能
請參考
http://localhost:5173/accountings/journal_draft
來實作：

- 「搜尋附註內文字」和「收付日期」篩選條件欄位，在 blur 時就會觸發套用篩選動作
- 套用「搜尋附註內文字」作為條件時，旁邊要出現目前輸入的文字條件內容
- 套用「收付日期」作為條件時：
	- 要出現 datepickr 供選擇 (請使用現有的元件)
	- 套用條件後，旁邊要出現目前輸入的日期條件內容
- 旁邊出現的條件內容，按下「x」就會刪除條件 (更新篩選結果)

篩選的查詢，請參考舊版

JournalDraftsController#load_draft_data 的作法
但請不要完全照抄，實作時考慮效能與優良的程式架構來作整合&改寫
但查詢行為仍維持一致

上傳明細頁 (/accountings/draft_stages/:id/import) 裡的 selection bar
裡面的「對象」點擊展開的容器，若按下 「x」關閉會變成送出 request
應該要正確關閉展開的元素材對(像「標籤」或「收付原因」的行為就是正確的)，請修正

上傳明細頁 (/accountings/draft_stages/:id/import) 裡的 selection bar 的展開
會超出 main content area (main 元素或 .content-wrapper) 的整體寬度
應要讓寬度在 main content area 內，但高度、距離視窗下方的位置不變
## 優化 SubjectMenu Component

在上傳明細頁 (/accountings/draft_stages/:id/import) 中
頁面裡(包括資料列 與 selection bar) 的 SubjectMenu，做為「收付原因」下拉選單，位於畫面底部時，仍往下展開，導致選項被 viewport 遮住、無法選取。 

問題情境： 
1. 單筆列的「收付原因」下拉選單：當該列位在畫面底部時，點開下拉只看得到前幾個選項，下方選項被截斷 
2. selection bar（貼在畫面底部）的「一次套用收付原因」下拉選單：點開後下拉超出畫面底部，大部分選項看不到、選不到 

預期行為： 
- 「收付原因」下拉選單該，依據觸發點與 viewport 邊界的可用空間，自動判斷向上或向下展開： 
- 下方空間足夠 → 向下展開 - 下方空間不足、上方空間足夠 → 向上展開 - selection bar 因為固定在畫面底部，其「收付原因」下拉預設就該向上展開

另請參考舊版本的 subject menu，如首頁 (/dashboard) 的 journal form 裡，
將舊版本 subject menu 的樣式：第一層選項會有左邊線顏色的樣式，migrate 至 v3 SubjectMenu Component 中

# 0621 修正

交易匯入總覽頁(/accountings/draft_stages)的「最新上傳」區塊裡列出的資金帳戶的上傳資料數，與下方「所有暫存區」資金帳戶清單的資料筆數不一致，請修正：

- 應為一致的數字，請確認使用一致的資料來源，原為 draft_stage 上的快取數字
- 請確認在交易暫存區頁 (accountings/draft_stages/:id) 的各種操作，有確實正確地更新快取數字，包括
	- 對單筆資料做建立或刪除
	- 對多筆選取的資料批次建立或刪除
- 該數字的定義為該資金帳所有所有 draft_journals 資料，未被處理(未完成建立或刪除)的資料總數

## 上傳明細頁 (/accountings/draft_stages/:id/import)

上傳表單裡的上傳檔案預覽區塊，要放在 dropzone 上傳區的下方，而非裡面
上傳檔案預覽的元素和內容，要靠左對齊

## 調整資金帳戶/交易暫存區頁 (accountings/draft_stages/:id) 

請修改資料列的日期輸入欄位(dealt_on, audited_on)，將 date-pick migrate 成使用 UI::Forms::DatePickerComponent 元件，並且：
- 維持原本資料列中的 date-pick 操作行為正常運作
- 若有需要調整 UI::Forms::DatePickerComponent 元件(包括 date_pick_controller JS)，請拆成獨立的 commit
- 請檢查 date-pick 跳出的日曆選單的樣式顯示正常

請修改資料列中的「附註(remark)」欄位，限制可輸入文字的長度為 300 字
# 0529 修正
## 上傳明細頁 (/accountings/draft_stages/:id/import)


請修改資料列的日期輸入欄位(dealt_on, audited_on)

上傳表單中的「套用收付原因」選項行為仍然不正確
下方的「收款原因」和「付款原因」欄位，要隨著「套用收付原因」選項切換
- 當選擇「不套用」(預設)時，下方的「收款原因」和「付款原因」欄位要被隱藏
- 當選擇「套用」時，下方的「收款原因」和「付款原因」欄位要出現

請參考使用現有的 simple_toggle_controller 來實現
這個 js controller 提供基本的綁定點擊觸發「顯示」或「隱藏」指定對象元素的行為
若這個 js controller 無法實現上述需求，請優化加強這個 controller 來達成
但保持這個 controller 通用性，讓其他情景也能用這個 js 來控制類似的行為

- 當收付原因選擇「套用」時，要出現下方的 subject-selects 區塊讓使用者選擇
	- 之前的修改(4f942465d4e05fcd1e1ee2530fdbdba4ddd93d47) 不正確

請修改資料列的上傳附件功能 (dropzone_upload_controller)
- 目前已有 dropzone_controller，應該有共同類似的部分，是否可以抽出共用的部分做繼承
- 將 dropzone_uploader 改名叫 attachments_upload_controller
- 上傳完檔案，原本資料列的「上傳附件」按鈕，要變成顯示「已上傳 n 個附件」
- 上傳完檔案，再次點擊「已上傳 n 個附件」跳出來的 modal 裡的 dropzone 要仍顯示已上傳的檔案資訊，而且可以刪除任一個已上傳的附件
可以參考 http://localhost:5173/accountings/journal_draft 裡的相關模擬畫面

# 0526 修正

修正以下問題：

## 資金帳戶頁 (/accountings/draft_stages)
- 請移除頁面上方的麵包屑

## 上傳明細頁 (/accountings/draft_stages/:id/import)
- 當收付原因選擇「套用」時，要出現下方的 subject-selects 區塊讓使用者選擇
	- 之前的修改(4f942465d4e05fcd1e1ee2530fdbdba4ddd93d47) 不正確
- 表單最下方的「取消」按鈕，點擊的行為不正確，應該回到該資金帳戶 (/accountings/draft_stages/:id)
- 上傳檔案的 dropzone 裡的 preview container 的內容應該要置中，現在是被設為：`grid grid-cols-2 gap-2`

## 資金帳戶/交易暫存區頁 (accountings/draft_stages/:id) 
- 標題「交易暫存區」應該為 「交易暫存區 - {資金帳戶名稱}」
- 分頁(pagination) 的樣式仍有問題，active 的那個項目的樣式不正確，請確保符合 daisyui 的樣式

# 0514 修正

- 關閉 dark mode，即使用戶設定環境為 dark mode，不會有任何作用
- 點擊「上傳明細」，會出現「全部刪除」menu，且位置應該是要在「其他功能」下方
- 一開始載入時，會顯示 selection bar，請改為隱藏
- 分頁(pagination) 的樣式跑版，請修正套用 daisyui 的版本
- selection bar 在做批次套用時，面板裡不需要有「全部刪除」鈕
- selection bar 的位置，有時候會超出畫面，應該在顯示時，無論什麼情況都在距離畫面下方固定的位置
- 當類型被設為「移轉」時，營業稅欄位應 disabled
- 進入資金帳戶的上傳頁面時，頁面標題，應為「上傳檔案 - {資金帳戶名稱}」
- 資金帳戶的上傳頁面，在上傳檔案後，出現的 file 區塊要置中
	- file 區塊內的文字，要靠左對齊
- 在資金帳戶的上傳頁面，表單裡的，「套用收付原因」，選擇「不套用」時，下方的兩個收款、付款原因的欄位需隱藏，不顯示。
- 上傳檔案成功後，回到 draft_stages 首頁，最新上傳區塊，要出現最新上傳的資金帳戶明細相關資訊，詳情可參考：http://localhost:5173/accountings/journal_draft_home
- 在 draft_stages 首頁，列出的資金帳戶暫存區，僅限於資金帳戶類型是「銀行存款」(kind = bank) 類型的

# 調整前後端運作架構
在繼續補足 Journal Draft Import 的功能細節前
我想先針對目前已實作的部分先做調整
## 建置共用 API
我想先建置一些取得群組常用資源的 API，資源包括
	- tags
	- contacts
	- subject menu
- 這三項資源是許多情境都會用到，所以事先規劃成獨立未來也可重複使用的 API

我希望可以同時有分別三種 resources 的 api，例如
/api/tags - 取得標籤列表
/api/contacts - 取得聯絡人清單
/api/menu - 取得會計科目選單
/api/accounts - 取得資金帳戶清單

但也可以有一支 api 透過參數來一次取得多種資源，例如

/api/resources?tags=1&contacts=1&menu=1

## 重整前端組件
目前的前端 js 設計太過僵化，請適度做重構，提高使用彈性並降低耦合
這裡先列舉幾個未來可能也會需要考慮到的功能，希望在未來的開發上也能依循/複用現在開發出來的前端元件，功能包括：

- 單一筆 Journal 輸入與編輯表單，也會有 accounts 、subject menu、tags、contacts 等輸入元件
- 搜尋的篩選，也會有 subject、tag、contacts 等輸入條件
- 交易移轉的表單，同時有多組 (移出 與 移入) 的 account 選單、多組 subject 

而這次 import 功能裡有使用到幾個 input 元件，包括 
- tag 選單
- contacts 選單
- subject 選單，
目前是寫死的，需要做出以下調整
- 選單的選項是依當下使用者所在帳本(group)呼叫 api 取得的
- 這幾個 input 的 javascript/html/css 應被設計成可重複使用的組件
- 介面上的一些文字、屬性應可被參數化，例如 placeholder 的文字，可以動態透過像是 data-attribute 來定義(例如考慮 i18n，由 server render 屬性)

如果可以，把這些組件抽出用 Stimulus 來做綁定，但
- 保留目前設計的動態效果(例如繼續使用 toastr)
- 維持目前的設計樣式，放到獨立的對應 css 檔案來管理樣式

如果有其他建議或可評估的方案，也請提出來討論

# 改善&重構 Journal Draft 的前端程式
好，接下來請開始將新版的 journal drafts 功能，替換整合成使用新設計的 select 元件，可以重構 journal drafts 的前端程式

# 建立 mock api 測試

我想製作可以讓前端工程師方便工作/測試的 mock api，
可以用 swagger 之類的解決方案嗎？
等於同時建立 api 文件以及 mock api 供測試

請評估適合的解決方案


請將 journal_drafts_controller 的功能的頁面中
Selection Bar 裡的「對象」與「標籤」的 selector，都換成 tagify 的版本

繼續調整 select 相關的前端元件，完成以下需求

一、優化 tagify_controller.js
請參考舊版的 TagifyWithApiSearch.js
檢查並調整 tagify_controller.js 成支援跟 TagifyWithApiSearch.js 一樣的 data-attributes 選項，包括：

- 可設定 apiUrl
- 可設定是否啟用 api search: useApiSearch: true/false
- 可設定最多多少選項: maxTags
- 從 data-attributes 設定選項清單: whitelist 
- 若有 data-existed-values 就用 data-existed-values 判斷已選取的項目，若無則從 input 本身的 value

二、調整  tag_select_controller.js 與 contac_select_controller.js
請將基於 tom_select_base_controller.js 版本的 tag_selector_controller.js 和 contact_selector_controller.js
調整成像 tagify_controller 的功能，包括：

- 仍然基於 tom_select，但
- 支援 api 查詢
- 用 data-attribute 控制是否要啟用 api 查詢
- 和 tagify 版本用同樣的 data-attributes 來控制
	- 可設定 apiUrl
	- 可設定是否啟用 api search: useApiSearch: true/false
	- 可設定最多多少選項: maxTags
	- 從 data-attributes 設定選項清單: whitelist 
	- 若有 data-existed-values 就用 data-existed-values 判斷已選取的項目，若無則從 input 本身的 value

請參考以下 commits:
- 8f672fab946a6332c4ecdcf4c14589149948ec21
- 192bdce6114cc7bb27fe7049b421fa327967bbba
裡的修改
將 journal_drafts_controller 的功能的頁面中
Selection Bar 裡的「對象」與「標籤」的選項也加上對應的 data-attributes 
可能會需要在  HasGroupAssets 裡調整產生給新版 (stimulus) 版本的 input_data


ex:
```
#tags_input_data(:stimulus)
#contacts_input_data(:stimulus)
```


目的是為了讓在 Selection Bar 的對象&標籤 input 中，要使用從 data attributes 傳入的設定，包括
- whitelist options
- apiUrl 與 useApiSearch 等設定


# 建立「收款原因」 input 元件

我要新的前端 stack 架構下建立「收款原因」的 input 共用元件
參考舊版前端 stack 的下的設計，migrate 成新版架構的元件，且易於各處表單可重複使用

「收款原因」的元件可命名為 subject_menu

主要行為是依當下 group 所擁有 menu 資料，render 出階層(2層)式的選單
- 第一層是選項分類
- 第二層是會被選取的目標選項
- 第一層分類被 hover 時，展開第二層選單
	- 第二層選單仍會帶有該分類標題

同時這個選單會帶有一個 text field 可以讓使用者用文字輸入的方式，篩選選項
- 當有輸入文字時進行篩選，會將符合的選項直接用第二層選單的樣式展開出現

請參考：

- `app/views/accountings/journals/_form.html.erb`  中的 `data-role="subject-dropdown-menu"` 的 DOM 元素，了解 html 結構
- `assets/javascript/components/subject_menu.js.coffee` 了解 javascript 的設計邏輯
- `app/assets/javascripts/components/journal_form.js.coffee` 中：
  ```
  @subjectMenu = new SubjectMenu @dom.find('[data-role=subject-dropdown-menu]')...
  ```
  等程式碼，理解該元件如何被使用的情境

## 實作內容
將產生 html 結構的部分製作成 FormHelper#subjet_menu_input 方法，傳入所需的參數(包括 form_builder 物件與 subject menu 的資料)，來產生 menu html

搭配新版前端的元件，例如 subject_menu_input_controller.js 裡實現對應的前端互動行為

期望的使用方式：

ex:
```ruby
<% simple_form_for(some_form) do |f| %>

  <%= subjet_menu_input(f, current_group, :subject_id) %>

<% end %>
```

其中 
- f 是 form_builder 
- current_group 是當前的群組帳本，會帶有 #fetch_menu_cache 方法取得 menu 資料
- :subject_id 則是這次對 form build 所要呼叫 input field 的名稱

請研究清楚後規劃實作方法，確認後開始進行
若資訊不夠清楚，可再多參考其他相關檔案的程式碼
對於前端互動行為若有不妥善之處，也提出建議討論如果修改

# 套用收付原因至 journal_drafts 功能

請將新的收付原因元件(subject_menu)，套用至 journal_drafts_controller 的功能的頁面中，包括

- Row Template 中的 Reason 欄位
- Selection Bar 中的 bulk-input-reason 欄位

正確的用法會需要用 simple_form_for 產一個 form builder，所以 controller 中應該要定義一個 form object 來給 form_for 使用

請參考 Accountings::JournalDraftsLegacyController#index 裡的 index 的作法 (包括 #load_draft_data 方法)
若有需要可以先用假資料來建立一個 form object


# 實作 JournalDraftForm



# 製作 Dropzone Controller

用 stimulus 包裝 Dropzone 套件來製作一個可公用的 dropzone 前端元件

用新版 stack 來建立 app/javascript/controller/dropzone_controller.js

預期使用方法

```
<div data-controller="dropzone">
  <input name="file" 
    data-dropzone-target="input" 
    data-dropzone-preview-template-value="[data-role=dropzone-template]"
    data-dropzone-previews-contaienr-value="[data-role=dropzone-container]"
    />
</div>

<div data-role="dropzone-template">
  ... 給 dropzone 使用的進度模板內容
</div>

<div data-role="dropzone-container">
  ... 給 dropzone 顯示上傳區的容器內容
</div>
```

- 用該 input 欄位的 name 做為檔案上傳的 input name
- 用 preview-template 和 preview-container 這兩個 value，來分別指向 dropzone 所需要的 previewTemplate 和 previewsContainer 的 dom selector，在 dropzone 被初始化前，會先用這兩個 selector 找到對應的 dom，傳給 dropzone
- dropzone_controller 本身內建兩組預設參數，分別代表 import 與 attachment 模式
	- import 預設參數：
		- maxFiles: 1,
		- maxFilesize: 10,
		- acceptedFiles: ".csv,.xlsx,.xls",
	- attachment 預設參數：
		- maxFiles: 4,
		- maxFilesize: 5,
		- acceptedFiles: ".jpg,.jpeg,.png,.pdf,.zip,.rar",
- 用 data attribute(value) 來決定要用哪種模式，預設是 attachment
- DropZone 的 config 都可以再用 input 元素的 data-attribute(value) 來設定




# 修正 Subject Menu Input 樣式
請調整 FormHelper#subjet_menu_input 產生出來的 subjet menu
應該要長得像附檔圖片中的樣式
而且目前有以下的問題：

- 當 input 在畫面靠底部，空間不夠顯示時，選單應該向上方展開，而非擠開下方畫面
	- 同理，在靠畫面右方空間不夠時，要向左方展開
- 整個 dropdown menu 應該要浮現在其他元素之上，目前有 z-index 的問題，會被其他元素 overflow 狀態下裁切掉
- 最末層的選項要帶有 subject code

請詳細參考舊版元件


- `app/views/accountings/journals/_form.html.erb`  中的 `data-role="subject-dropdown-menu"` 的 DOM 元素，了解 html 結構
- `assets/javascript/components/subject_menu.js.coffee` 了解 javascript 的設計邏輯
- `app/assets/javascripts/components/journal_form.js.coffee` 中：
  ```
  @subjectMenu = new SubjectMenu @dom.find('[data-role=subject-dropdown-menu]')...
  ```
  等程式碼，理解該元件如何被使用的情境

附圖樣式：
![[Pasted image 20260210201502.png]]

# 新版本的交易匯入功能

我正在實作新版本的交易匯入功能
請比較與 feature/t2646-selection-api 的差異了解目前的實作狀況
並參考以下相關的程式碼了解舊版本的功能

```
apps/controllers/accountings/journal_drafts_controller.rb
apps/controllers/accountings/import_uploads_controller.rb
```

新版本的交易匯入主要有以下變動：

新版本在操作邏輯上以「資金池暫存區」的基礎
原本匯入的所有交易草稿(JournalDraft)資料，將依照所屬的資金帳戶(Accountings::Account) 分組
讓使用者在處理交易匯入時，都先決定要在哪個資料池暫存區操作

因此流程調整為 

列出所有資金池暫存區
使用者先選擇資金池暫存區
- 若選擇進行上傳，會進入該資金池暫存區的上傳表單
	- 選擇檔案上傳後，上傳的草稿就歸在該資金池暫存區下
- 若進入瀏覽，進入該資金池暫存區的草稿清單，進行管理，包括
	- 對單筆草稿編輯資料，存為正式交易
	- 刪除草稿紀錄
	- 選擇複數草稿編輯資料，整批存為正式交易


操作優化，新版本在草稿管理的頁面，改用了 hotwire 的 turbo
以及前端 stack 改用新版本 (esbuild + stimulus 為主)，並對一些操作介面做了調整

目前已完成基本的新版 routes 配置與 controller 規劃：

```
/draft_stages 做為新的入口，列出所有資金池暫網區
/draft_stages/:draft_stage_id 進入特定資金池暫存區
/draft_stages/:draft_stage_id/upload 在指定的資金池暫存區進行檔案上傳
```

目前已完成
- 在 draft_stage_uploads_controller 進行上傳檔案建立 journal_drafts 
- 在 draft_stages#show 的部分功能，包括
	- 列出 journal drafts
		- 使用 turbo_frame
		- 實現換頁
	- 產生 form tag 來包裝各筆 journal draft 的資料列 (rows)

請接續完成以下目標 
- draft_stages#show 裡的各種操作行為，包括
	- 在表單上對單筆的 journal draft 調整資料 
	- 將單筆的 journal draft 資料送出存成正式的 journal
	- 將單筆的 journal draft 資料進行刪除
	- 在表單上勾選多筆的 journal draft 批次調整資料
		- 新版本將批次編輯的輸入項設計在 selection_bar 中
	- 將勾選多筆的 journal drafts 送出存成正式的 journal
		- 預計是由 journal_draft_bulks_controller 負責，但沿用 JournalDraftForm 來處理資料寫入

請參考並舊版的 JournalDraftForm 改寫成 DraftStageForm 來實現後端對資料的建立，包括
- 當勾選多筆資料，從 selection bar 按下「一次建立」時，將表單使用 turbo frame 的方式送出，DraftStagesController#create 接到 request 使用 DraftStageForm 將 JournalDraft 轉成 Journal 的動作
- 同理，當勾選多筆資料，從 selection bar 按下「全部刪除」時，將表單用 turbo frame 的方式送出，送到 DraftStagesController#destroy，來處理刪除
- 當對單筆(row)資料按下「建立交易」時，透過 js，僅取得該單一 row 的資料送出表單，一樣送用 turbo frame 到 DraftStages controller，由 DraftStageForm 將該筆 JournalDraft 轉成 Journal
- 同理，若是單筆 row 按下「刪除資料」，一樣用 js 僅取得單一 row 的資料然後送出表單，由 DraftStagesController#destroy 資料

DraftStagesController 在 DraftStageForm 處理完筆資料後，會需要將畫面上處理過的 journal draft 資料 row 從畫面上移除，並且補上同等數量下 n 筆 journal draft 資料列到表單 list 裡


JournalDraft 轉成 Journal 的行為必須維持原有的批次建立的作法與規則，
請確實參照舊版本 JournalDraftForm 裡的邏輯來改寫 DraftStageForm

如果有舊版的 js 的部分需要從舊版本 migrate 到新版本，請寫在 draft_stage_list_controller.js 


請一步一步進行規劃開發
先將目前已實作的部分釐清，以及了解舊版本的規格
並且要確認有哪些是舊版本存在的功能，但在新版本還未實現的 ，這部分要特別列成 檢查清單
然後再規劃及進行


# 修正

請修正以下問題

- DraftStage list 中每一個資料列(row)，當被勾選時，要自動被展開(expand)
- selection bar 套用「收款原因」時，沒有正確更新對像資料列的 「收付款原因」，應該要：
	- 正確更新每列的 subject_id 與 menu_id
	- 而且要觸發各列的 subjet_menu input 顯示成對應的選項

Part II

請修正以下問題
- 當交易資料列的交易類型(kind) 被設為移轉(exchange)時，要把憑證日期(audited_on)和發票號碼(receipt_number)的欄位設成 disabled
- Selection Bar 中
	- 各個輸入欄位，必須對應的 checkbox 勾選時才啟用
	- 收款原因選項的輸入欄位，必須要 list 中所勾選的資料列每一列的類型是相同的(收款或付款)，才允許使用

Part III
請修正以下問題

- Selection Bar 中，收付原因欄位
	- 該欄位的名稱，維持「收付原因」即可，不需要因選擇的資料列是收款或付款而變更名稱
	- 該欄位預設應該就是 disable 的狀態

20260408

請根據剛才的 dev-note 接續開發

主要以先確保以下功能正確運作為目標，包括

一、匯入檔案成 journal drafts
- 從選定的 draft stage 對應的資金池進入 import 檔案頁面
- 進行檔案上傳
- 上傳後順利 import 成為 journal drafts 
	- 確保產生的 journal draft 是屬於對應的資金帳戶 (Accountings::Account) 的

二、對 DraftStage 下的 journal draft 進行操作
- 進入 draft stage 的頁面，載入 journal drafts 列表
	- 列表的換頁功能要正常運作
-  可對單筆的 journal draft 各欄位屬性做設定
- 可對單筆的 journal draft 完成 save 動作
- 可對單筆的 journal draft 做刪除動作
- 可勾選多筆 journal draft，會出現 selection bar
	- 可從 selection bar 設定欄位屬性，一次套用到勾選中的 journal draft
	- 可一次將已勾選的 journal draft 全部 save
	- 可一次將已勾選的 journal draft 全部刪除

功能的架構、流程以及前端的部分
應該都已經實作了，請維持不要改動
主要是對上述的資料編輯、寫入等行為
請做詳細的檢查，確保能正常運作

其他注意事項：
- 前端的部署還有待改進，這部分我們之後再改
- 未 commit 的檔案應該都是用不到的，請忽略

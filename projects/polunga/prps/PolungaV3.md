# Fixlist
```
   Job-state (V3): create app/views/shared/_job_state_v3.html.erb (daisyUI card + data-controller="poll" data-poll-mode-value="job-state" + [data-attr=state] + [data-role=logs] — the existing poll controller job-state mode drives it
   unchanged, see app/javascript/controllers/poll_controller.js). Render it in the image _forms via the existing JobStatePresenter data: render "shared/job_state_v3", presenter: JobStatePresenter.new(form.job_state, text: "…"). Do NOT
   touch shared/_job_state.html.erb — still used by v1 announcement_translation_reviews, operation_patches, customer/account_cases.
   
   
   - Legacy coffee (drive_image, remote-image-input, signed-url-preview) stays in the v1 bundle (still used by v1 pages via shared/forms/_remote_image_input + others); we only stop relying on it here — don't edit the v1 JS manifest.
   - Do NOT delete shared/forms/_remote_image_input.html.erb (still consumed by v1 announcement_mission_campaign_translations).
```


```
ApplicationResource.find_by(name: 'Settings::DownloadPresenter')&.destroy

  - Infra: the location /cable {…} block in config/deploy/nginx.conf:69-79 and the commented config.action_cable.* lines in three env files now proxy/configure nothing.
  - app/controllers/settings/authentications_controller.rb — a second unrouted settings controller found while mapping; same dead-code family, not in scope.

```


# 整合 CS Tool

請將 cstool 專案下 (/Users/aaron.kuo/aktsk/ishin-tw/ishin-tool-cs
) 的功能
migrate 進現下這個專案中，並套用 v3 的元件與樣式
註：cstool 專案是從本專案之前拆出的分支

需要整合的 cs tool 功能包括：
- /tool/users
- /tool/users/search
- /tool/payments/googleplay
- /tool/payments/webstore
- /tool/user_items/
等路徑的頁面和頁面

另注意 cstool 整合過來的功能
- 不需要有切換 ServerStage 的機制，永遠使用固定的 Globalization.default_stage
- 功能內容都以 cstool 的版本為主
- 原本放在舊 cstool 專案是在 Tool:: Namespace 下，migrate 到現下專案的 Customer:: Namespace 下

請先詳細理解 cstool 的功能、評估完整的整合計畫再執行
若有任何建議或問題提出與我討論

Users Search 表單的欄位需進行以下修正
- - Purchase Condition 的 product_id 欄位，tom select 展開的選單顯示不正確，會被截斷在 input 欄位的範圍內，有點像是 overflow or z-index 的問題
- Device brand 的欄位也要套用 tom_select，且確保展開的選單也要正常顯示
- Purchase Conditions 的區塊(card)，要在 Purchase condition 欄位 (radio) 被選擇「不是 None」的時後才出現啟用

purchase condition  仍有以下問題：
- product_id 欄位，應該也要是 select field(tom_select)，列出 product 清單供使用者選捏
- Purchased between 的 checkbox 沒有勾選擇，要把 duration_range 的欄位 disable 掉

# 優化 Dockerfile

請嚐試優化 Dockerfile 的內容，尤其是考慮在使用 buildx 來同時 build linux/arm64 與 linux/amd64 的情況，並且：

- 設法減少最終產出 image 的大小
- 適當的 cache layer
- 優化執行 bundle 的指令，提高重試次數與超時時間、降低平行下載數





Announcement/Banner Images Sync 各語言欄位要固定 min 寬，超出畫面啟用水平捲軸
- 加 sticky header & 凍結側欄
- 加上點圖放大瀏覽

## 修正 operation issue 編輯的 document mode 

在 OperationIssue#edit 使用 document mode 時
在觸發 issue_document_form_controller 的 renderFields 時
取得的 html 內容，是來自 app/views/operation_issues/document.html.erb 的內容
這個 html 在 server 端 render 的結果會有一層 form 包覆
但 renderFields 應該只取用 form 裡面的 `<div data-role="document-fields">` 裡的內容就好
不然會變成 form 裡面再包一層 form 造成表單欄位無法正確送出


# 導入 datetimepicker
請在專案目前的前端 stack (v3) 裡導入 datetimepicker 套件
和其他元件類似的作法，用一個 stimulus controller 來包裝、啟用 datetimepicker，支援到:
- format: 'YYYY-MM-DD HH:mm'
- stepping: 60

需求
- 完成 js 前端的配置
- 套用 datetimepicker 在以下功能：
	- apologies 的 form 表單裡的 start_at, end_at 欄位 `app/views/master/apologies/_form.html.erb`
	- operation issue 的 form 表單裡的 begins_at, ends_at 欄位 `app/views/operation_issues/_form.html.erb`




樣式
- [x] Delete Button in Danger Zone 因為全紅所以看不清楚
	- http://v1.polunga.test:3000/master/gogeta/announcements/107050
- [x] 切換顯示語言(複選)的 ui 需優化，跟 edit 按鈕太相似
	- http://v1.polunga.test:3000/master/gogeta/announcements/107045

- [x] Topnav 切換主語言的選單

# 重新製作 dashboard
在 root (/) 畫面裡重新製作 dashboard
目前的 utility link 請刪除
改成排列各項功能的 widget 區塊，
功能 widget 包括
- operation 功能，有三個 widgets:
	- 顯示目前正 release 的 operation，顯示摘要內容
	- 顯示即將釋出的 operation，顯示摘要內容
	- 顯示接下來近期 release 的 operation，僅標題+日期
- statistics
	- 顯示前一日( +8 Timezone) 的 kpi 總覽，用數字 card 顯示
		- Total Revenue, 和前一日變化百分比
		- DAU, 和前一日變化百分比
		- PDAU, 和前一日變化百分比
	- 前一日過去七天的圖表
- announcement
	- 顯示接下來釋出的 5 篇 announcement，用 default stage 撈取資料
- campaign
	- 顯示接下來釋出的 3 則 campaign，日期、名稱與包含的 deployment (operation) days
- tournament
	- 顯示目前正在進行  tournament 和當前的 mission 內容，
- GDPR case
	- 顯示接下來 pending 的 case，包括 expired 日期與 user_ids
	- 

並符合以下需求
- 每個 widget 我希望是用 async load 的方式，以降低載入頁面所需的時間
- widget 內容若為內則顯示 empty state


每個 widget 還要是視權限 ability#can?(:read, Resource) 來判斷畫面上能不能顯示該 widget
(可參考 sidebar 裡對各項目的權限判斷規則)

若有任何問題與建議，請提出討論


## Table Component
美化 table list，包括：
- announcements#index
- banners#index
- apologies#index
- campaigns#index
- operation_issue_categories#index
- operations#released
- budokai_cheaters#index

可以逐項分批完成，先完整一個讓我確認後，再繼續其他項

修改重點：
- 將 table 格式參照 claude design 的設計，且支援 dark mode
- 若有 Filter ，將 filter 移出 box(card) 外，並參照 claude design 的設計
- 若有自訂欄位的選項，參照 claude design 的設計
- 務必確保原本的功能正常不被破壞

並將上述對應的調整列為設計原則，未來實作新功能時要按照這個設計

### Bulk Action in Table
優化 bulk action
請參照 claude design 來實作 bulk action 的行為，在
- announcements#index 
- banners#index
頁面中，當勾選多筆資料，會出現 bulk action 的選項 「sync」與「duplicate」


所有有 WithReadonlyContext 的頁面
包括引用 ActsAsAnnounceableController 的 controllers, 與 apologies_controller, apology_users_controller, announcement_users_controller 等
都應該在判斷在 readonly_context? 的情況下，在 inner page layout 出現 readonly_banner 提示

## 優化 announcements 列表
在 announcements#index 的 table 列表中，優化資料列的顯示，請參考 claude design 的 operation issues 列表：

- 將 is_individual_limit, is_sticky ... 等  is_xxx 等欄位，合併成一個欄位叫 features，每筆資料用 tag(badge) 的方式來呈現，只顯示 true 的項目，像是 `is_display_title` `is_display_home`
- Platform 欄位的內容現在是陣列 (enum)，也調整顯示成 tag(badge)
- Conditions, New Appear At, Link To 等欄位，用和參考頁面中額外展開的方式來呈現
- category 欄位也用 tag(badge) 包裝，類似參考頁面的樣式
- 將 start_at、end_at 欄位移到 title 後面
- en, fr ... 等六語言的連結，合併一個欄位裡排列，每個連結用 secondary button 的樣式

## 優化 announcement 瀏覽頁樣式

請對 announcements#show 頁面版式和樣式設計做調整：
### Header
- 調整 page actions: 
	- 只有 edit 是用 primary 樣式，其他用 secondary
	- 按鈕調整排序：「Edit」「Sync Images」「Sync」「Translation Review」然後才是「History」「Copy」
	- 按鈕名稱調整：
		- 「Sync」改成「Sync to Stage」
		- 「Copy」改成「Copy to Stage」
- 切換語言 locales 的介面，跟 primary 按鈕太過相似，請重新設計
	- 可改成「齒輪」圖示，點擊展開後，出現像 columns 的開關，每個語言用 toggler 切換開關
	- 可以是都選取開/關後，再一次送出更新切換要顯示的語言
	- 這個介面其他頁面也會共用，可以一起修改套用
	- 可以的話放在 page action 最後面的位置

#### 增加 sub header (或用更好的命名)
在 header 下方另增加一個區塊，做為 sub header，分成左右兩欄：
- 左方放 announcement 的重要屬性資訊：
	- id
	- title
	- start_at - end_at
	- banner filename
- 右方：
	- 將原本在 announcement 裡的各語言翻譯連結 (announcement_translation_path) 改放到這裡
	- 另加一個回到頂端的 icon 連結，點擊後會將畫面捲動回到最上方
- 對這個 sub header 加上 fixed top 效果
- 請設計 sub header 的樣式，建立規範，未來其他頁面若有 sub header 也依順這個設計

### 內容的部分

內容分成以下數個 section
- Announcement
- Announcement Bodies
- Preview
- Danger Zone

每個 section 有自已的 section heading
- 左側是 heading 名稱
- 右則可允許放 tool buttons (optional)
#### Announcement Section
分成左右兩欄，整體內容高度依照右側欄，左側欄的內容若超過右側欄，內容會出現捲軸
- 右側欄
	- 填滿剩餘寬度
	- 一樣是放六語言的內容瀏覽：title、banner、summary
		- 每個顯示語言的內容欄寬要相等，不論該語言實際是否有內容
		- 六語言內容超出整欄容器寬度時，出現水平捲軸
		- 內容靠上對齊
- 左欄
	- 固定寬度
	- 顯示該筆 announcement 的屬性資訊 (properties)，有多種呈現方式：
		- 不需顯示(已搬到 sub header)
			- start at
			- end at
		- 不需要 property name:
			- available locales 直接用 tag/badge 顯示
			- platforms: 直接用 tag/badge 顯示
			- category : 直接用 tag/badge 顯示
			- layout type: 直接用 tag/badge 顯示
		- boolean 類型的，直接用 property name: icon 來顯示, 例如
			- O Individual Limit
			- X Public
		- 用 property name 、property value 分兩行顯示
			- new appear at
			- link to
			- platform countries
	- 部分行為會跟 announcements#index 裡 table list 很相似，請適當地重構，可將一些 hardcode 搬到 IshinServer::Announcement model class 中

#### Announcement Bodies Section

一樣分成左右欄
左欄：
- 固定寬度
- 做為 bodies 資料的清單列表 (list)，依序列出每筆 body 的 id (靠左) 、drive image file name (靠右)
- 每筆 body id 是可以被點擊的 link，連到右側欄對應 body 的 錨點
- 高度不超過畫面高，內容若超出會出現垂直捲軸

右側：
- 填滿剩餘寬度
- 列出每筆 body 的實際內容 (和現狀類似)
	-  調整樣式，每一篇 body 是一個獨立有 box/card 樣式
	- 有該 body 的錨點
		- 讓左側欄的連結點擊時會捲動畫面到對應 body 的錨點
		- 當畫面捲動到 body 錨點到畫面中上位置時，左側欄對應的
	- 顯示該 body 的 properties：
	  id, layout type, start at ~ end_at, mission_campaign_id；用水平並列上(field header)下(value)呈現，ex:

| id  | layout_type | start_at - end_at | mission_id |
| --- | ----------- | ----------------- | ---------- |
| val | val         | val               | val        |
|     |             |                   |            |
- 每筆 body 顯示各語言的翻譯內容
	- 每個顯示語言的內容欄寬要相等，不論該語言實際是否有內容
	- 六語言內容超出整欄容器寬度時，出現水平捲軸
	- 內容靠上對齊

#### Preview Section
切換 preview 語言的選項調整成「像 tab」的樣式 (但實際行為不變，仍是傳送 request 再 render 內容)

preview 的內容會有兩個版本「default」與「指定時間」
將這兩個版本改成左右並列 (各佔 50%)

#### Danger Zone Section
在 danger zone 裡的刪除按鈕樣式會因為同樣紅底而無法分配，將它改成白底色字的按鈕

## 附註
上述修改應只調整樣式排版與設計，不影響任何功能
會需要幾個新的元件設計，例如
- sub header
- section heading
- properties 與各種呈現的變體
請先評估規劃好這些元件，並實際成可以用的 ui component 且制訂使用規範
設計的部分請維持現代化、清楚友善的使用者體驗，且支援 dark mode 為原則

若有任何問題與建議，請提出討論


請進行以下修改：
- sub header 右側的 語言翻譯 link 項目，要依照該 announcement 的 available_locales 決定有哪些語言，而非 display locales
- 第一個 section (announcement) 與上方的 sub-header 太近，增加一點上間距
- Announcement 的  TranslationTableComponent 需要白色背景，可以用 card 包起來
- TranslationTableComponent 裡的各語言內容要等寬，可設固定 width = 480px
- Announcement Bodies 的左側 list，
	- 加上流水號
	- drive image file 改成: 若有值的話顯示 icon (代表有圖)，不用顯示 file name
- Announcement Bodies 的右側資料內容的 properties 的部分:
	- 刪除 schedule，不用顯示
	- 加上 drive image file 
	- 其他都改成用 badge 的方式 (不需要 property name)，例如：
	  `2000001433` `Large banner` `Mission Campaign Id: 12345` `someImage.png`
		- 若沒有 mission campaign id 就不顯示
		- 若沒有 drive image file 就不顯示

請進行以下修改：
- Sub Header 裡
	- schedule 改成用 icon (日曆或時鐘)
	- banner 改成跟 announcement bodeis 裡的 drive image file 一樣的樣式 (icon + badge)
		- 並且對這個樣式加上「點擊複製」的行為：請用現有的 clipboard controller js 來實現
	- 將 title 用標題  heading (h2 or h3) 包起來獨立成第一行，較大的字體，不需要 TITLE 標籤
	- 其他 property 放在第二行
	- 確保右側欄(各語言翻譯選項、回到頂端) 有足夠的空間並排在右側

請進行以下修改：
- Header 裡的「History」「Copy to Stage」改成用「…」dropdown menu 展開選項
- Header 裡的其他按鈕 (Edit)之外，改用 secondary
- Announcement properties 中，pair 呈現的 value，若 value 是空值，用一個 EMPTY badge 表示
- Announcement Bodies 裡的左側欄，調整成跟 Announcement 的左側欄一樣寬
- Sub-header 裡的 drive banner file 和 Announcement bodies 裡的 drive image file，它們的 click copy 行為都沒有效，請修正


## 優化 Banner 瀏覽頁樣式
請參考 Announcement 瀏覽頁做類似的調整
- Header 的 page actions、breadcrumb
- 增加 sub-header，fixed top 效果，
- 對主要內容(banner)拆分左欄 properties 與右欄 translation table
- 調整 danger zone

## 優化 Banner 列表頁樣式
也參考 Announcement 列表頁做類似的調整
欄位有一些不同，請按照：
- 用 banner 的 description 代替原本 title 的位置
- 用 banner 的 place 代替 category 的位置
- conditions、link_to、priority 放到展開的區塊內

請將目前工作至今的頁面修改內容，歸納成頁面設計、版型、樣式的原則
更新至 ai_docs 下的文件中，做為未來製作新功能或修改時的參考準則
同時也包括確認目前 ui components 的使用方法，更新至 ai_docs 中 

## 優化 Resource 列表頁樣式
請參考 Announcement#index 列表頁做以下調整：
- 調整 filter 的版型和樣式
	- filter 區塊獨立在 box/card 上方
	- 調整樣式，包括 add filter (add extra condition) 
- 將「translation check」調整跟 announcements 的 bulk action 一樣的處理方式
	- 勾選後才出現 bulk action 按鈕
- Table 內的資料列內容不需調整

將 resources#index 頁裡的 table:
- 對標頭(thead) 加上 fixed top 的效果
- 對第二欄加上 fixed 的效果 = 水平滑動時前兩欄固定
	- 參考 TranslationTableComponent 裡的效果

類似的情況，在 resources#show 、 resources#check 畫面裡的 table
- 對標頭(thead) 加上 fixed top 的效果
- 對第一欄開始加上 fixed 的效果

在 Resources#show 的 table view 模式下加以下功能：
在 columns 旁多一個 jump to 的按鈕
按下後，會從右側展開一個 aside 區塊
區塊裡列出當下 resource presenter 的所有顯示欄位清單 (不需多 locales)
當點擊對應的欄位
會將 table 裡的內容水平捲動至對應的欄位 (第一個語言的位置)


在 Resources#show 與 Resources#check 都加以下功能

對每一個欄位，檢查是不是各筆資料的各語言都是空值
如果是空值，將該欄位的 table column 加上標記
在頁面的 card 標是右方 toolbar 加上一個按鈕，可以切換 顯示/隱藏 這些「被標記全是空值」的欄位
# 0713

我想要重新優化整個網站的 ui 設計，尤其是視覺設計
請幫我匯整整個網站的 ui 介面與功能，建立成一份能交給 claude design 做為設計需求的說明文件
包括整個系統的功能定位、頁面架構、版型種類
以及需要設計的元件清單和各元件行為描述、可能的變體
不僅是參考 ui component，還要檢視目前各功能，因為有一部分尚未被建立成 ui component

結果請放在 ai_docs/design.md 裡
目標是可以用這個文件讓 claude design 來完成整個 ui 系統的規劃與設計
並且也可作為未來開發新功能的參考

若有任何疑問或建議請提出跟我討論

我已請 claude design 重新設計網站，產出的 spec 位置 /Users/aaron.kuo/Downloads/design_handoff_ishin_ui_system
，請依照這份設計套用到現有的專案，

我發現這份 spec 有一些網站架構與專案現狀不符，所以請以視覺設計為主要優先，包括
- 主要的配色系統、token
- 整體 layout 的配色、ui
- 基本 component 的樣式
確保不要影響到現有的任何功能
其他無法套用的部分，請先紀錄下來我們後續再討論要如何調整
若有任何疑問或建議請提出跟我討論

- Topbar 的樣試調整成跟 claude design 的版本，包括
	- sidemenu toggler 的樣式
	- 搜尋 bar
	- brand name 的 logo「維」的字樣，但改成用 lucide 的 「paw」icon
	- 不需要 stage 切換的選單
	- 不需要切換 theme (faithful / refined) 的選項
- Breadcrumb 裡的 item 高度要依 center 對齊，包括 item 是 dropdown 選單的情況也要維持對齊
- side menu 主選單的 submenu 選項美化，在次選項前也加上 icon  或用統一的圖示來美化

修正 Announcement Translation 編輯畫面
(ex: /master/:stage/announcements/:id/translations/edit?locale=en)

調整 breadcrumb，應該要 with server stage 下拉選單的樣式
- 參考 announcements#show 的畫面

調整 preview 的 panel
- preview panel 的 fix top 位置要再下方一點
- preview panel 要可以被 collapse 收合/展開，預設是收合
- preview 的內容中，.validation_message 的樣式未被套用，應該要是紅底白字而且有閃爍的效果


將 announcements 和 banners 的基本管理功能，migrate 成 v3
包括
- banners_controller
- banner_translations_controller
- announcements_controller
- anouncement_banners_controller

兩者的表單(form) 裡會有一些類似的行為，尤其是圖片檔案的處理，即：
- announcement#banner
- announcement_body#image
- banner#image
這部分會有共用的 js 行為，若有需要可以抽成 Form Component 並整合對應的 stimulus controller

請現符合 v3 的現代 js 與 rails 並考慮重構讓程式碼讓程式更加簡潔穩固





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

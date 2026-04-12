我想將 一般業務功能 與 Admin 相關功能折分開來，使用不同的 layout

系統整體來說分成三部分

1. 公開的頁面
2. 一般業務功能
3. Admin 限定功能

公開頁面基本上不在此次調整範圍，先維持原狀

## 修改內容：
 
Admin 的功能為  /settings/ 下所有功能
請將非 admin 的功能，改成套用 app layout
另外要將 admin 與 app 的 layout，都調整成：

- 左側欄是主選單
	- 主選單可以被收合，會用 js 紀錄選單被收合，以確保切換畫面會維持選單收合/展開的狀態
	- 主選單最上方有一個漢堡選單圖示，來切換展開/收合左側欄
	- 主選單內容會在 admin 與 app layout 有各自的選項
	- 主選單可以有兩層
		- 第一層的項目會帶有 icon，第二層沒有 icon
		- 若第一層選單是帶有第二層子項目時，第一層選項被擊點是展開子選單
	- 主選單在展開狀態時
		- 第二層選單會被收合，在點擊父選項時下向展開子選單
	- 主選單在收合狀態時：
		- 第一層選單會用 icon 顯示
		- 若第一層的選項是帶有第二層選項，會在滑過父選項時，用 popup 在右方的方式顯示第二層選單
- 最上方加上 topnav
	- topnav 的最右方的常見的選項，包括顯示當前登入使用者，登出選項
		- 若為使用者具有 admin role，就會有進入 admin 模式的選項
	- topnav 的最左方
		- 放置企業的品牌 logo 圖示
		- 會有一個快速搜尋的搜尋列，功能之後再實作

先請把基本的 layout 調整好，並套用現代化的設計，請參考常見的 saas 平台的設計
並用 esbuild + tailwindcss + daisyui 完成製作
必須確認操作介面清楚簡潔、使用者體驗良好，且有 responsive 設計


修正以下問題：

- 主選單在收合狀態下，當滑鼠 hover 第一層選單項目時，要 popup 子選單，但目前子選單 popup 出來會因為 overflow 的問題被 layout 主(main) 容器蓋掉，像是被隱藏在下方，可能是 overflow 或 index 的問題，請修整 (app layout 與 admin layout 都有同樣的問題)
- 主選單的展開或收合狀態並沒有被正確紀錄下來，切換畫面或 refresh 頁面，它都會還原成展開的狀態，請修正
- app layout 的 topnav 的 right side 選單應該要靠向最右方



接續以下調整：

- 請將 layout (包括 app 與 admin) 中選單的文字，都改套用 i18n 處理
	- 請先確保專案正確配置了 i18n 機制，請用 elixir/phoenix 實務上常見的作法來實作 i18n
	- i18n 須支援繁體中文(zh_tw) 與 英文(en) 兩種語言
	- 預設語言為 zh_tw
- 幫我把 zh_tw 和 en 的語系檔都建立好，內容都先寫英文沒關係
- 關於開發時使用 i18n 語系檔時的編寫規則，請幫我建立一份 guidelines 文件放在 ai_docs/guidelines/ 路徑下，並更新 CLAUDE.md 做為 guidelines 參考指引

我希望 layout 裡的主選單，第一層的選項，也會有依當前瀏覽的畫面而有 active 的樣式效果

# Settings 功能的 index 畫面採用 infinite scroll

請將 admin 的各項功能(settings 下的各項 crud 基本功能)，
在列表畫面，都調整成 infinite scroll 的方式來載入內容
並且實作基本的搜尋篩選功能

請將這個機制實作從可以用共用 macro，讓各個功能可以重複套用，
並設計成可透過參數來設定，例如可搜尋篩選的欄位，或是其他實務上的作法

請在進行修改前，先更新 ai_docs/ui/crud-pattern.md 裡的規格


## 修正 settings 功能

目前在 settings 下的任一個功能，在 index 的 table 列表中，一旦點擊了任一列(而非 edit button)觸發了 modal 顯示編輯 dialogue 之後，畫面左側的主選單就會失去作用：點擊第一層選單無法展開第二層子選單。

請檢查並修正這個問題


## Settings 功能的 index 畫面支援分類篩選

CRUD 的 index 畫面，支援分類篩選
除了原本的用 term 的 search 查找
我要加上一個分類選單可以快速依分類來篩選，
不同的對象資料會是用不同的分類

介面上我希望可以做成想 tab 的樣式：
- 但不是真的 tab，它不需要載入各 tab-content 的內容，只是用 tab 來表示目前正在瀏覽的分類
- 預設是顯示全部，所以第一個 tab 選項是 all
- tab 選單可以是在上方，也可以是左側，請讓我可以透過參數設定來決定哪一種版型
另外：
- 不是每個功能都需要分類篩選
- 不同功能的分類會是不同的資料模式，例如 users 會用 role 來篩選，blog 會用 blog_category，

請這個機制加到需要調整的共用 macro 模組，如 HdwcpWeb.InfiniteListLive , Hdwcp.Pagination 等 
並更新 ai_docs/ui/crud-pattern.md 裡的說明文件

然後將這個機制套用到 settings 下 users 與 blog

## Settings 功能的 index 畫面的 UI 優化

我要讓 CRUD index 畫面的 filter ui，包括 term search 與 category filter(如果有使用的話)，在畫面捲動超出範圍時，有 fixed top 固定的效果
請同時考量 filter layout 是 top 和 left 的情況


# Settings 功能的 show 頁面

請將 admin 的各項功能(settings 下的各種 resources)
都增加獨立的 show 頁面
要有獨立的 Liveview 模組例如 BlogLive.Show 以及獨立的 html template file (blog_live/show.html.heex)
show 頁面的內容先放上該 resource 的基本資訊即可
詳細的內容與 ui 我們之後再各自實作

並更新 ai_docs/ui/crud-pattern.md 裡的說明文件

# Settings 功能的 show/new/edit 採用獨立畫面

請將 admin 的各項功能(settings 下的各項 crud 基本功能)
1. 將點擊 table row 時連結到 #show 的畫面
2. 在新增與編輯的畫面，都調整成 render 獨立的畫面，而非用 modal dialog 的型式，
	1. 分別有獨立的 Liveview 模組，例如 BlogLive.New 和 BlogLive.Edit
	2. 及獨立的 html template file, ex: blog_live/new.html.heex, blog_live/edit.html.heex
	3. 在 new 和 edit 中
		1. 在頁面加上各自的頁首標頭(header)
		2. 再覆用原本的 form_component

並更新 ai_docs/ui/crud-pattern.md 裡的說明文件

# Migrate Role 的管理功能

目前從舊版本(rails)系統 migrate 過來的 role settings 的功能並不完整
請使用 rails-migration skill ，再次詳細解析舊版本(../hdwcp-main) 的功能實作，
並改寫成 elixir/phoenix/liveview 的版本，
在設計架構上請改用 elixir 實務的作法

補充 role 的管理(新增/編輯)功能應該要包括：
- 可以新增/修改 role 的 (has_many) privileges
- 每筆 role privilege 
	- 設定允許的 resource(module_name)
		- 不同的 resource(module_name) 會有不同的 actions
		- actions 是透過 config/privilege_action.yml 來定義的
	- 可以勾選複數個 permitted actions
	- 也可以設定成 all actions

先請將重新理解後的 role settings 規格建立成文件，放在 ai_docs/specs/ 下
確認後再開始進行程式撰寫




## 修正 Blog Settings

settings 下的 blog 管理功能 #index 頁套用的 filter 功能無法正確運作
出現以下錯誤

```
** (Ecto.QueryError) lib/hdwcp/pagination.ex:58: field `blog_category_id` in `where` does not exist in schema Hdwcp.Settings.Blog in query:

from b0 in Hdwcp.Settings.Blog,
  where: b0.blog_category_id == ^1,
  order_by: [desc: b0.id],
  limit: ^20,
  offset: ^0,
  select: b0,
  preload: [:category]
```

請檢查是否是套用 macro 時傳入的參數有錯誤
還是 macro 的實作需要調整？

如果 macro 的實作有問題需要調整，請修正並更新 ai_docs/ui/crud-pattern.md 裡的說明


# 重整功能路徑與 settings 選單

有一些功能應該要改放在 settings 下
包括：
- product_codes 功能，在選單請放在 maintenance 下
- exchange_rates 功能
請確保上述功能被搬到 settings 下
從功能(curd) 都有符合跟其他 settings 功能一致的作法，參考 ai_docs/ui/crud-pattern.md 


另外還需要增加以下資產的 settings 功能
- DealerType 管理(CRUD)，可用 rails-migration skill 參考舊專案的內容


調整 admin layout 的選單
請將 maintenance 與 operation 兩個項目合併，名稱叫 operation，但 icon 用目前 maintenance 的 icon

對更多 settings 功能啟用分類 filter
- repair_items ：啟用 RepairCategory 做為分類 filter，layout 用 :left
- evaluation_items ：啟用 RepairCategory 做為分類 filter，layout 用 :left
- product_codes: 啟用 Category 做為分類 filter，layout 用 :top


## 優化 InfiniteScrollList 

請針對以下需求進行功能加強和優化

- 支援可以指定排序的欄位，包括
	- 可指定預設排序欄位，若無指定則用 id 
	- 可指定預設排序方式，若無指定則用 desc
	- 可透過點擊列表 table 的標頭欄，來指定當下的排序欄位
	- 可透過重複點擊標題欄，來切換排序方式 (asc or desc)
	- 標頭欄要有對應的 icon 來標示正在排序中與排序方向
	- 可以用參數設定不允許作為排序用的欄位

請確保 settings 各項功能，在加上這個優化後，功能都能正常運作


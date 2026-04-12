我想要引入更新 Form UI，包括

- 對於 form 以及各種 input 欄位，套用 daisyui 的樣式
- 對 select 的選單，我想引入 tom_select

我預估是要調整 CoreComponent 各種 input component 的 class，按照 daisyui 的慣例
請幫我確認評估這個作法並提出建議

確認後再開始執行

執行前，將整理好的的 Form UI 實作方式
建立在 ai_docs/guidelines/form_ui.md 做為未來的 guidelines


## Date Range Filter

我想設計一個 Date Range Filter 的 component
方便使用在各種依日期篩選輸入 input 的情境
包括像是
- 可被 InfinateScrollList 功能整合使用
- 可在一般 form 表單裡使用，
- 可以與 Phoenix.HTML.FomField 整合使用

可以支援多種選擇日期的方式，例如

- 直接輸入 starts_on 、ends_on 參數，用 datepicker 元件
- 直接提供單一年份選擇，對應背後的 starts_on、ends_on
- 直接提供單一月份選擇，對應背後的 statts_on、ends_on
- 選擇 n 個月，用相對現在月數來計算 
	- 例如 1 個月內、3 個月內
- 選擇 n 天，用相對現在的天數來計算
	- 例如 30天內、90 天內

在呼叫 component 時
可以透過參數去控制提供哪些日期選擇方式，
component 會產生對應的 html ui，搭配需要的 js、hook

請規劃了解這個 component 的需求
並整理成一份這個 ui 元件設計規格和使用方式，加到 ai_docs/guidelines/form_ui.md 裡
實作這個元件，然後整合到 InfiniteScrollList

整合到 InfiniteScrollList 時
因為當下目標 schema 可以有多個 date field 可以當作 date range 條件的欄位，所以也要可以讓
- 定義有哪些 date field
- 接受參數來決定實際 query 時要用個 date field 做 range filter

最後將這個套用到 shippings 下的 delivery_orders
並確保功能可以正常運作

請修正以下 Date Range Filter 的問題
- Year+Month 的選單怪怪的，month 應該只要 1~12月就好，year 請用近 n 年來產生，n 預設是 5，可用參數傳入來決定
- 預設的天數調整成 1天、3天、7天、15天、1個月、3個月
- 切換 date_field 的功能似乎無法正常運作，例如切換成 created_at，再選擇 1個月，date_field 會跳回原本的，請修正

請再修正 month 的選單
只列出當年份過去12 個月就好


## Dealer Input
類似 Date Range Filter 的用途
請實作一個用來 dealer(經銷商) 選取或篩選的 input component
一樣可以用在像是查詢條件、form 或是與 FormField 整合的情況

並且要能支援用文字輸入框來過濾選項名稱，可用 tom_selector

經銷商的選項清單需滿足以下需求
- 經銷商的名單是可被設定的 scope，預設是當前 active 狀態的經銷商，可傳入自訂的 query 查詢
	- active 應該是要看 dealer 當前的合約狀態，若不清楚定義，可先用簡單的方式定義，只要保留未來方便修改就好
- 選項名稱顯示是用 經銷商編號(number) + 名稱(label)來組成
	- 例如 88001 大成公司
- tom_select 應該可以自訂它的呈現選項方式，請將 component 設計成支援用 component 的 slot 來決定 option template，而預設就是單純顯示 number + label


請規畫完整後，將規格與使用方式
更新成 form_ui.md 文件







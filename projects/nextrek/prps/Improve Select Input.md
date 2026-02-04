
目前的標籤(tag)和對象(contact) input(select) 欄位
在用戶帳本(group)有大量資料(tag/contact)的情況下會很慢，server 端在 render 頁面時
會需要將所有的 options 輸出在 html

我想進行優化改善
計畫有幾個目標 

一、建立 GroupAssets Cache
對用戶帳本(group)來說，很多種屬於該帳本的資源，我在這稱它們為 assets
例如 tags、contacts，甚至是屬於 group 的設定值等等
這些資料經常會在每個頁面需要被讀取，而且不會經常變數
我想建它們建立快取的機制，
- 快取在 memcache
- 封裝在 group assets 物件中

透過如以下方法來取得資料

```
current_group.group_assets.all(:tags)
current_group.group_assets.all(:contacts)
或

current_group.group_assets.tags
current_group.group_assets.contacts
```

而當有資料變動需要更新(清除)快取時

```
current_group.group_assets.expire!(:tags)
current_group.group_assets.expire!(:contacts)
```

除此之外，對某些 assets，會提供最近/常使用的 50 筆資料的版本
(50 請定義成常數，方便日後管理修改)

```
current_group.group_assets.recent(:tags)
current_group.group_assets.recent(:contacts)
```

還有提供 total 資料量的查詢 (一樣將數量快取下來)

```
current_group.group_assets.total(:tags)
current_group.group_assets.total(:contacts)
```

補充
- 取得的資料無論是 all 或著是 recent，預設都是照名稱排序
- group_assets 只提供讀取資料，沒有更新或新增資料的機制，只有清除重建快取
- 快取只存必要的資料結構，例如 id 和 name 的 hash，可評估是否要 serialize/deserialize 成對應物件

二、更新 Tags/Contacts API 支援查詢和使用快取

目前專案已有基本的 api 實作，像是 

```
api/v1/tags_controller.rb
api/v1/contacts_controller.rb
```

為這兩次 api 做修改

- 接受 limit 參數來決定要回傳的資料量
- 接受 term 參數來對名稱做 like %term% 條件的篩選
- 預設是用 group_assets 中的 cache 資料查詢
- 接受 nocache 參數來改成真的對 db 下 query 查詢

三、優化所有 tags 和 contacts 的 input 欄位

這部分要開始完成本次修改的主要目標，分成3個步驟

A. 首先將各個 tags/contacts 的 input (應該都是 select)的 option 選項
從原本讀出全部選項，改成只列出
- 從 group_assets 取得 recent 的資料，加上
- 目前表單已經選取/輸入的選項(如果有的話)
	- 請也從 group_assets 中的快取資料來讀出已選取的項目

這樣可以加快 server render 頁面的速度

B. 請設計可共用的 tags / contacts 選擇器的前端元件，滿足以下功能

- 判斷 group_asset 的 total 量是否超過 recent 的數量 (ex: 50)，若未超過，就用一般的行為，server render page 時等於是會 render 所有選項
	- 可將判斷結果放該 input/select 的 data-attribute，來代表是用哪種模式
- 如果判斷結果 total 量是超過的，那就會啟用 api 查詢的模式，此時 server render page 只會 render recent 的選項，而當使用者輸入文字查詢時，就會呼叫 api 搭配 term 參數來查詢；來動態更新 select 的選項
	- 為避免輸入反應過快造成 api 頻繁呼叫，請記得實作 debounce 的機制

注意：
- 請確認 tags 與 contacts 兩種組件的行為，如果有必要，可以分開製作成兩個元件；但每個元件是可以重複使用在各頁面/表單的 tags 與 contacts 的 input/select 欄位
- 因為是修改既有功能，請用 legacy 版的前端 stack 來製作元件

C. 將新組件套用至各個頁面的 tags/contacts input

已知目前有以下頁面可能會需要套用，請確認並確保它們能如預期運作：
```
- 總覽頁 /dashboard
- 交易建立頁 /accountings/journals/new
- 編輯交易頁 /accountings/journals/journal_id/edit#accountings-journal-form_journal_id
- 完成交易頁 /accountings/journals/journal_id/edit#accountings-journal-supplement-form_journal_id
- 複製交易頁 /accountings/journals/journal_id/edit#tab-copy
- 交易明細頁 /accountings/journals/init
- 批次建立頁 /accountings/journals/bulk
- 批次編輯頁 /accountings/batches/init
- 交易匯入 /accountings/journal_drafts
- 發票列表 /accountings/journals/invoice_init
- 應收款 /accountings/journal_uncompleteds/account_receivables
- 應付款 /accountings/journal_uncompleteds/account_payables
- 暫收款 /accountings/journal_uncompleteds/temporary_receipts
- 暫付款 /accountings/journal_uncompleteds/temporary_payments
- 全部 /accountings/journal_uncompleteds
- 逾期表 (應收款) /accountings/journal_uncompleteds/overdue_collection
- 逾期表 (應付款) /accountings/journal_uncompleteds/overdue_payment
- 帳齡表 (應收款) /accountings/journal_uncompleteds/aging_ar
- 帳齡表 (應付款) /accountings/journal_uncompleteds/aging_ap
- 交易明細 (對象) /contacts/contact_id
- 交易明細 (標籤) /tags/tag_id
- 新增傳票 /vouchers/new
- 編輯傳票 /accountings/vouchers/voucher_id/edit
- 複製傳票複製 /accountings/vouchers/voucher_id/edit
- 新增常用傳票 /accountings/voucher_templates/new
- 複製常用傳票 /accountings/voucher_templates/template_id/edit
- 會計項目明細 /accountings/journals/subject_init
- 會計項目明細 - 搜尋後 /accountings/journals/subjects/subject_id
- 標籤對象損益表 /accountings/reports/show/tag_contact_analysis/tag_contact_income
- 標籤對象綜合損益表 /accountings/reports/show/tag_contact_analysis/tag_analyses
- 標籤對象經營現金流量表 /accountings/reports/show/tag_contact_analysis/tag_contact_cash_flow_statement
```

如果有發現任何頁面因為有不同的行為導致元件必須修改，保留成 todo 項目，我們之後再調整，但不要影響它現有的運作

請將此項功能的開發調整，先整理成文件，更新在 @ai_docs/ 目錄下
然後確認後開始進行開發


原因應該是：
- api 回應資料過多
- render option內容過多

我想調整成
- async load 載入時，對 api 傳送 limit 值，即只 render 最多 limit 數個選項
- 這個 limit 值取自 select input 上的 data attribute，讓它可依視情境決定不同的 limit 值
- 並且，當使用者輸入時 ，會再呼叫 api 去用名稱做 like %term% 查詢，來動態更新選項
	- 要做 debounce 避免太頻繁呼叫
	- 可能要修改 api 的部分

並更新元件並重新檢查所有修改的地方，如果有不同行為可以設計成不同的元件
但請維持設計風格與使用方法的一致性，並盡量重複設計好的使用元件
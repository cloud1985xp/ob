
# Migrate TagifyWithApiSearch

請將舊版前端 stack 定義的 TagifyWithApiSearch 元件，改寫成在新版前端 stack 的 stimulus 元件


請先參考舊版本設計與用法
app/assets/javascripts/components/tagify_with_api_search.js.coffee
是做為包裝 tagify 的元件，來做為輸入 tags 或 contact 使用
主要用在兩種情境

一、資料編輯表單中填入資料的 input 模式
其中輸入 tags 會是可輸入複數個項目，輸入 contact 時只會輸入一個項目
input 模式可以從即有選項中選擇，也允許輸入不存在的項目來建立新資料

使用方式會是在對應表單(form) js class 中去啟用它，
並在表單行為事件中去綁定需要的動作，例如：

app/assets/javascripts/components/journal_form.js.coffee
app/assets/javascripts/accountings/bulk_form.js.coffee

二、在搜尋表單中設定條件的 search 模式
search 模式只允許從現有的選項選擇，不允許輸入建立新資料
大部分情況不論 tags 或 contacts 都允許複選，少數情況

並且是從 global.js 自動去對有 [data-toggle=tagify] 的元件啟用
參考 app/assets/javascripts/global.js.coffee:168

請根據上述參考資料
改寫成用 stimulus 包裝的新版本作法
新版本檔案放在
app/javascript/controllers/tagify_controller.js
(已有檔案，可以參考或是覆蓋)

全域的 js 進入點：
app/javascript/v3.js

我建議可以把 基本的 tagify 包裝成 tagify_controller
但另外把 tag select 和 contact select 分別定義在各自的 controller，讓它們去使用 tagify_controller
然後把 search 用的也再做成另一獨立的 controller

tagify_controller
tag_select_controller
contact_select_controller

並且可以的話，都用 stimulus 的 data-controller 的方式來啟用，
而不需要再另外透過 global.js 或從 form class 裡去啟用

用 data attribute 來傳入 config 上請維持原本設計，僅調整參數的命名格式成為 stimulus 的方式

請先將新版本的設計、規格、使用方式等釐清後，建立一份文件
放在 ai_docs/ui/ 目錄下，方便之後參考，包括使用的範例等等


## 請再次確認 TagifyWithApiSearch 元件

原舊版本的 TagifyWithApiSearch 有做了一些優化
請再次檢查確認是否有完整地 migrate 到新版本的各相關元件
包括：

- /app/javascript/controller/contact_select_controller.js
- /app/javascript/controller/tag_select_controller.js
- /app/javascript/controller/tagify_controller.js
- /app/javascript/controller/tagify_search_controller.js

請重新檢視各個使用情境
確保新版本 stimulus 的程式能符合需求

完成後記得更新 ai_docs/ui/tagify_stimulus_controllers.md 文件
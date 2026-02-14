---
tags:
  - nextrek
  - project
---
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
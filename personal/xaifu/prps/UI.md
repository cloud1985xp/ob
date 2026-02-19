# Improve Selection

實作 Components 以及前端 javascript 來優化表單中的 select 下拉選單，包括
- prompt 的 select input
- subject 的 select input

目的
- 可以支援 text search
- 自訂選項的顯示

## 實作內容

### 實作 Components

請將
- prompt 的 select input
- subject 的 select input

做成可重複使用的元件放在 `lib/xaifu_web/components/form_component.ex` 下，
實作包括：

```
Xaifu.FormComponent.dropdown_select
Xaifu.FormComponent.prompt_select
Xaifu.FormComponent.subject_select
```

主要功能實作在 dropdown_select 作為基底
而 prompt_select 與 subject_select 主要是決定 options 清單以及 option 的呈現方式，
用 slot 的方式傳入 dropdown_select

### Dropdown Select 

- 允許傳入 Form Field 做為 input name，這是這個 select 元件真正的 input field 名稱，做為表單送出時的參數，可以參考 CoreComponents 的 input 元件來接受參數
	- dropdown select 會產生一個 hidden field 來做為這個參數
- 接受 slot 來做為 render option 的呈現方式
- 接受 options 來做為 dropdown 的選項，用 slot 來依序 render 每個選項
- 允許傳入 value 來做為目前選單的值，從 options 中找對應 id 的值
	- 並一樣用 slot 來 render 為目前選擇中的項目
- 產生一個 text field 來做為 searchable 的輸入，搭配 javascript 來做篩選

### Prompt Select
主要負責
- 接受 form filed input 參傳，做為 dropdown select 的參數
- 決定 options 清單，可以接受
	- project + category 參數，去撈出與 project 相關的 prompts
		- 使用 Prompts.list_prompts 搭配條件
	- prompts 清單，直接用這些 prompts 做為 options
- 定義 prompt 的 option 顯示的 slot 樣式，內容包括：
	- prompt 的名稱
	- prompt 的 text_en
	- prompt 的最近 5 張 generated_images 的 thumb，以水平排列
然後呼叫 dropdown select 來 render 元件

### Subject select
主要負責
- 接受 form filed input 參傳，做為 dropdown select 的參數
- 決定 options 清單，可以接受
	- project 參數，去撈出與 project 相關的 subject
		- 參考 Projects.list_subjects_with_images
	- subjects 清單，直接用這些 subjects 做為 options
- 定義 subject 的 option 顯示的 slot 樣式，內容包括：
	- subject 的名稱
	- subject 的描述
	- subject 的圖片
然後呼叫 dropdown select 來 render 元件

### Option Slot 的規範
- 每個 option 的 html 要有一個 data-attribute，來代表 searchable 的名稱，做為 text search 時對比對目標
- 每個 option 的 html 要有一個 data-attributes 來代表 id value，做為被選中時要傳入 hidden input filed 參數的值

### 前端 Javascript / CSS 實作要點
- 請配合 daisy ui 的 dropdown 樣式
	- 用 focus 來觸發顯示 dropdown content
- 用 stimulus 實作 js，定義一個 DropdownSelectController
	- 當點擊 dropdown menu 時，隱藏當前 option 的現式，改顯示 text_field 且 focus 輸入欄位
	- 當 text field 有輸入值時，去比對每個 option slot 的 data attribute，來決定是否顯示/隱藏 option
	- 當沒有任何符合的內容時，顯示提示訊息
	- 當 text field 清空時，還原顯示所有
	- 當點擊 option 時
		- 將該 option 的 id value 設定成 hidden field 的值
		- 將該 option 的 slot 樣式替換成 dropdown 當前的選擇中的項目顯示
	- 當 blur 事件/離開 dropdown menu / text field 時，會將 text field 隱藏，顯示選擇中的項目

## 更新 GenerationLive.FormComponent
上述元件/js 完成後
請將編使 Generation 使用的 FormComponent 表單中的 subject 與 prompt 欄位，替換成製作好的 subject_select 及 prompt_select 元件

然後在 prompt_select 和 subject_select 中呼叫 select_dropdown

select_dropdown 會接受：傳入的 option slot 樣式來 render


而 prompt_select 與 subject_select 則分別負責
- 決定使用的 options
	- 可傳入 options (= prompts / subjects )
	- 可傳入 project，用 project 去撈出
- 定義 prompt/subject 的 options slot 的呈現方式

subject 的 options 顯示
- subject 的名稱
- subject 的描述
- subject 的圖片
	- 參考 Projects.list_subjects_with_images








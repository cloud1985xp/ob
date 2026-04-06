
提示詞解析
輸入圖片網址、檔案或貼上
使用 grok API 取得提示詞
使用 comfyui 取得標籤
使用 grok API 過濾標籤：服裝顏色、場景
串預覽的功能流程


支援指定尺寸的功能



# (wip)從 Generation Show 快建更新

我要在 Generation Show 的畫面，直接可以快速調整 Generation 並執行
修改需求如下：

- 將現在顯示 Input Configuration 調整成一個表單，
- 表單裡照目前顯示每個 Generation Input，但每個都有一個編輯圖示，點擊後可以設定該 generation input，包括
	- 可以直接從下拉選單選擇同 category 的 prompt
	- 同時也有一個 textarea 可以直接輸入新的 prompt text
	- 也可以設定 amount 
- 表單按下送出後，處理的邏輯如下：
	- 如果有輸入 textarea 內容，會新建立一個新的 prompt，在對應的 category 下
	- 然後將對應的 generation input 改成使用這個新建立的 prompt
	- 

# (tbd)調整 Subject Index 
將 Subject 改成類似表格呈現，但仍用 grid 實作
每一列是一個 subject

調整 Projects.list_subjects_with_images 的 query：
- 目前已可透過 opts 傳入一個 target workflow，預設為第一個，來 preload 對應的 generations
	- 要把該 workflow 的 workflow input 和 prompt category 也載入
	- 要把對應的 generations 的 generation input 也載入


調整列表的呈現
欄位包括
- subject name
- subject description
- 顯示一張 generation 的 image
- 當前 workflow 的各 prompt category 名稱
在顯示每一列時，
- 顯示該 subject name 與 description
- 

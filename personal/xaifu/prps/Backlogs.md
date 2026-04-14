
- 圖生提示詞解析功能
	- 輸入圖片網址、檔案或貼上
	- 使用 grok API 取得提示詞
	- 使用 comfyui 取得標籤
	- 使用 grok API 過濾標籤：服裝顏色、場景
	- 串預覽的功能流程

- Prompt 優化
	- 複製 prompt 的功能，方便複製->微調 or 改尺寸
		- 自動幫 prompt 命名
	- 加上 rank 或置頂、like？
	- 建立提示詞後，繼續在同分類下建立下一個
	- 瀏=覽提示詞，在同一分類下 navi
- 刪除 generated_image (同時要刪除圖片原檔與各 version 圖檔)
- 修正 Generation duplicate 的功能
- Subject 優化
	- 設定 rank 或 置頂
	- 可從 Subject 建立屬於該 Subject 的 Generation
- Generation 列表優化
	- 顯示 recent 圖片預覽
	- 用 subject 篩選或排列
	- 用 workflow 篩選或排列

- 批次建立 prompt
	- 用 llm 用一段描述，批次生成多組提示詞，編修確認後批次建立
	- 直接輸入多行文字，每行是一個 prompt

- [x] 把 尺寸移到 prompt level
	- 優先於 generation 的 size
	- 若多個 prompt 都有設定 size 時，
		- 在 prompt 加上 position，用 last position 的
- [x] ~~支援指定尺寸的功能~~



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


- 圖生提示詞解析功能
	- 輸入圖片網址、檔案或貼上
	- 使用 grok API 取得提示詞
	- 使用 comfyui 取得標籤
	- 使用 grok API 過濾標籤：服裝顏色、場景
	- 串預覽的功能流程

- 利用 LLM 自動幫 prompt 命名 / 翻譯
- 刪除 generated_image (同時要刪除圖片原檔與各 version 圖檔)
- 修正 Generation duplicate 的功能
- Subject 優化
	- [x] 設定 rank 或 置頂 -> likes
	- [x] 可從 Subject 建立屬於該 Subject 的 Generation
- Generation 列表優化
	- 顯示 recent 圖片預覽
	- 用 subject 篩選或排列
	- 用 workflow 篩選或排列

- 批次建立 prompt
	- 用 llm 用一段描述，批次生成多組提示詞，編修確認後批次建立
	- [x] 直接輸入多行文字，每行是一個 prompt

- [x] 把 尺寸移到 prompt level
	- 優先於 generation 的 size
	- 若多個 prompt 都有設定 size 時，
		- 在 prompt 加上 position，用 last position 的
- [x] ~~支援指定尺寸的功能~~


## 實作 LLM PromptHelper

我要實作一個 PromptHelper ，利用 LLM 來實現以下功能

1. 依傳入的情境和需求描述，產生多組做為 comfy ui 文生圖的 prompt
2. 依一張上傳的圖片或 url，請 LLM 解讀圖片產生該張圖的提示詞(圖生文)
3. 依傳入一段用來文生圖的 prompt 文字，請 LLM 將文字摘要成一段標題

其中功能 1 和 2 都是產生提示詞
每組提示詞的產生都統一為以下的格式輸出

- title: 標題(中文，不超過30字)
- summary: 摘要說明這個主題的內容(中文，不超過 150字)：
- prompt_zh: 實際的提示詞內容中文版
- prompt_en: 實際的提示詞內容英文版

但功能 1 是產生多組結果，功能2 則是一組結果

目前預計是使用的 LLM 是用 grok api 服候
程式庫使用 ReqLLM，並使用 struct response 來確保 API 回傳資料的格式

使用情境舉例：
功能一
讓 LLM 依傳入的情境和需求指示，來產生多組的文生圖 prompt
傳入的內容例如：

你是一個中國武俠劇的專業編導，請設計一段劍客英雄救美的橋段
包含30個分鏡畫面，來描述這段劇情脈絡
請將這30組分鏡，整理成適用於 comfyui 的標籤式文生圖提示詞
提示詞內容需包括角色的動作、服裝與場景
每一組都需要包含：
- title: 標題(中文，不超過30字)
- summary: 這段分鏡中文說明
- prompt_zh: 生成這段分鏡圖片的中文版提示詞內容
- prompt_en: 生成這段分鏡圖片的英文版提示詞內容

功能二
是用圖生文的方式，要求 LLM 產生格式化結果
- title: 標題(中文，不超過30字)
- summary: 這段圖片內容的中文說明
- prompt_zh: 作為中文版提示詞內容
- prompt_en: 作為英文版提示詞內容

輸入可能是
- 由使用者上傳一張圖片檔案
- 由使用者提供一個圖片的網址
- 使用者直接從剪貼簿貼上的 blob 資料

功能三
單由傳入一段 prompt 的中文或英文內容，讓 llm summarize 成一個 50 字內的中文標籤

請幫我規劃並實作出這個 PrompHelper 模組的功能
並且保留未來可替換成別的 LLM provider (例如 openai) 的彈性


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

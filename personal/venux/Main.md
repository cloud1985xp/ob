這是一個整合 comfyui 生成圖片與提示詞的管理系統

技術選擇
- Elixir + Phoenix + PostgreSQL
- Liveview + TailwindCSS4 + Lucide Icon + esbuild 

使用 phoenix auth 實作簡易的 user 登入

## 主要服務

這個系統主要讓使用者管理並使執行「ComfyUI 圖片生成的工作流」

- 管理生成圖片的工作流 workflow，與生成的紀錄 (generation)
- 管理提示詞
- 挑選與編輯提示詞後呼叫外部服務(comfyui service) 的進行生成(generation)
- 管理生成的圖片
- 管理專案與主題：每次的生成紀錄(generation)生成可被歸在某個主題下，主題則屬於專案
- 標籤系統，來方更查找各種資源，包括提示詞、圖片、主題，或之後其他資源也可被套用標籤
	- 請使用 Elixir/Phoenix 成熟的 tag 系統來實現
- 檔案上傳至雲端服務，生成的圖片會再上傳到 AWS S3 存放
	- 請使用與 Ecto 整合良好的 library 來實現

## 資料模型結構

所有 id 都採用 uuid 格式

### User
使用者，經由 Phoenix auth 產生的 user 的基本資料表

### Project
專案：user 可以建立多個專案，所有資源是歸屬於專案下
欄位有：
- name: 專案名稱
- description: 專案描述
- owner_id: belongs to user

Has many
- 

### PromptCategory
提示詞類型
欄位有：
- name: 類型名稱
- project_id: belongs to project
- editor_id: belongs to user

### Prompt 
會被用在執行 workflow 時輸入的提示詞，每個提示詞：
- 是由一串文字組成，而且同時會有中、英文版本
- 會屬於某一個提示詞分類
- 可以有多個標籤(tags)，方便使用標籤系統
欄位：
- project_id: belongs_to project
- editor_id: belongs_to user，誰編輯的
- text_en: 提示詞的英文內容
- text_zh: 提示詞的中文內容
- category_id: belongs to prompt_category
- remark: 備註欄文字

Has many
- workflow_inputs
- workflows through workflow_inputs
- prompt_changes

### PromptChange
另外要建立 prompt_changes 資料表
- id
- prompt_id: belongs to prompt
- project_id
- editor_id
- text_en
- text_zh
- category_id
- remark
- event
在建立 migration 時，請同時定義 postgresql 的 trigger，當 Prompt 的 text_en 或 text_zh 欄位有變動時
就會將舊資料紀錄到 prompt_changes 裡，為了儲存變動前的歷史紀錄

### Workflow
工作流，對應會被傳至 comfyui 服務的工作流/檔案
每個工作流會定義有多個可被輸入的參數，是讓使用者在每次實際執行時可自訂輸入的的提示詞

- project: belongs to project
- creator: belongs to user
- title: 自訂的標題
- description: workflow 的說明文字
- content: json 格式，即 comfyui 的 workflow json 檔案內容

Has many:
- workflow_inputs
- generations

### Workflow Input
- workflow: belongs to workflow
- identifier: 文字，在 workflow json 裡對應的 input 參數名稱
- prompt_category_id: belongs_to prompt_category，表示這個 input 是哪種類型的 input
- prompt_id: belongs to prompt，代表預設的 prompt input，是 optional 的

### Generation
每用工作流執行一次生成，就會建立一筆 Generation 的紀錄
有點類似： Workflow 是 template, Generation 是產生的 instance
為了可以重複執行生成，未來可以實作 re-run generation 的功能

欄位有：
- project_id: belongs_to project
- workflow_id: belongs_to workflow
- trigger: string 紀錄觸發的來源，例如 user 的 email，或者某個 event 名稱
- parameters: json，紀錄實際執行時傳入的各種 input 的參數內容

Has many
- generated_images
### Generated Image



Subject
WorkflowSubject

功能架構

帳號系統與專案機制
- 建立基本的 User 登入機制
- Project based， 每個 user 可以建立多個 project，所有可操作的資源都是歸於 project 下
	- 每個 user 都會有一個預設專案




基本管理功能

工作流管理：workflow 的 CRUD 管理
提示詞管理：prompt 的 CRUD 管理
提示詞類型的管理：prompt_category 的 CRUD 管理
主題的管理：subject 的 CRUD 管理

進階功能

圖片管理：
主題：subject

資料架構

管理 workflow



提示詞組


一、產生提示詞作為 comfyui workflow 輸入參數發送給 comfyui service 啟動圖片生成任務，任務完成後將生成的圖片存回系統

二、輸入圖片網址或檔案，呼叫外部工具解析圖片的生成提示詞，存入系統


## 從提示詞生成圖片

先從輸入的提示詞建立一個生成作業，一個生成作業可以執行多次來生成多張圖片

生成作業是一個 template
template 會定義好包括：

- 標題名稱
- 使用的 workflow json
- Workflow 的內容描述
- 多個提示詞片段
	- 每個提示詞片段會有
		- 類型
		- 指定的提示詞組
		- workflow 中的對應參數名稱
	- 如果片段已有指定的提示詞組，則執行時就會預設在這個片段使用指定的詞組
	- 如果片段沒有指定的提示詞組，
		- 則執行接受傳入的提示詞組
		- 否則就亂數從同類型的提示詞組中挑選
- 自訂附加正向提示詞
- 自訂附加負向提示詞
- 預設審查等級

例如 generation template
有3個段片，
片段1 ：髮型相貌，未指定
片段2：服裝，未指定
片段3：動作情境，未指定

表示使用這個 template 時，會需要三個 片段 input
每個片段 input 是一個提示詞組

提示詞組會有
- 標題
- 提示詞內容
- 類型

例如

- 標題：髮型一
- 類型：髮型相貌
- 提示詞內容：cute face, energetic smile, long flowing black hair, half-rim glasses

- 標題：休閒造型一
- 類型：服裝
- 提示詞內容：oversized yellow hoodie with hazard-tap drawstrings, blue cargo skirt

提示詞組可以被 tag，方便篩選查找


例如 Kana template
片段1: 髮型相貌 -> 指向 kana 人物髮型相貌片段
片段2: 

對 Kana template 的使用情境

1 -> 呼叫 template，讓服裝、動作隨機
2 -> 呼叫 template，同時傳入服裝片段 id
3 -> 呼叫 template，傳入自訂服裝片段原文
4-> 從一張參考圖圖生文，解析取出服裝/動作，呼叫 template，傳入服裝/動作原文
5-> 從當日人物心情/個性/情境生文，生成服裝/動作提示詞，呼叫 template ...
6 -> 複製 Kana 生成另一個新的 Mika template, 把髮型相貌、服裝片段換成另一組

每生成一張圖片，紀錄
- 來自哪個 template
- 各片段的
	- 文字原文
	- 片段來源關聯(optional)
- 圖片檔案
- 可被 tag

從提示詞片段(組)做圖片查找
例如選擇 雙馬尾 + 休閒服 兩種 tag，找到相關的 提示詞組，再用提示詞組撈出所有產生過關聯的圖片



每次的提示詞由多個片段組成
例如：

：包括風格、品質、

提示詞由多個提示詞



提示詞管理

- 提示詞由多個提示詞組組合而成
- 提示詞組由多個提示詞字
- 提示詞字的內容
  - 可為多個單字組成，中間可有空白，但不可有半形逗號(,)，例如：long hair、
  - 同時會有中英文對照，分成兩個欄位

提示詞分類

多個提

PromptGroup


可以隨機生成


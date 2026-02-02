Xaifu 這是一個整合 comfyui 生成圖片與提示詞的管理系統
以下為開發需求與規格
請幫我分析並規劃系統架構，擬定開發計劃向我確認後，建立文件並且執行

## 技術選擇
- Elixir + Phoenix + PostgreSQL
- Liveview + TailwindCSS4 + Lucide Icon + esbuild 
- 使用 phoenix auth 實作簡易的 user 登入
- 使用 Oban 處理非同步任務

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
- prompt_categories
- prompts
- workflows
- generations
- subjects
- generated_images

需紀錄 user 當下的 session 正在使用哪一個 project
### PromptCategory
提示詞類型
欄位有：
- name: 類型名稱
- font_color: 文字代表色 rgb string
- background_color: 背景代表色 rgb string
- project_id: belongs to project
- editor_id: belongs to user

每產生一個 project，預設會建立一個 default 的 prompt category

### Prompt 
會被用在執行 workflow 時輸入的提示詞，每個提示詞：
- 是由一串文字組成，而且同時會有中、英文版本
- 會屬於某一個提示詞分類
- 可以有多個標籤(tags)，方便使用標籤系統
欄位：
- project_id: belongs_to project
- editor_id: belongs_to user，誰編輯的
- title: 標題名稱
- text_en: 提示詞的英文內容
- text_zh: 提示詞的中文內容
- category_id: belongs to prompt_category
- remark: 備註欄文字

Has many
- workflow_inputs
- workflows through workflow_inputs
- prompt_changes

可以被 tag
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

- project_id: belongs to project
- creator_id: belongs to user
- title: 自訂的標題
- description: workflow 的說明文字
- content: json 格式，即 comfyui 的 workflow json 檔案內容

Has many:
- workflow_inputs
- generations

可以被 tag
### Workflow Input
- workflow: belongs to workflow
- identifier: 文字，在 workflow json 裡對應的 input 參數名稱；必填且在同個 workflow 下必須唯一
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
- generation_inputs: json，紀錄實際執行時傳入的各種 input 的參數內容，用 embed_many 實現
- subject_id: belongs to subject
Has many
- generated_images
Embed many
- generation_inputs

generation_input 的結構

- identifier: 對應 workflow_input 的 identifier
- prompt_id: 實際選擇的 prompt_id 值

```
[
  {"identifier": identifier1, "prompt_id": prompt_id1},
  {"identifier": identifier2, "prompt_id": prompt_id2},
  {"identifier": identifier3, "prompt_id": prompt_id3}
]
```
### Generated Image
- project_id: belongs to project
- generation_id: belongs to generation
- workflow_id: belongs to workflow
- subject_id: belongs to subject(optional)
- file:  實際檔案資訊，(視使用的處理檔案上傳的 library 而定)

可以被 tag

### GeneratedImagePrompt
Generated Image 與 Prompt 之間是 many to many 的關聯
當 generation 產生 generated_images 時，會將 generation 的 parameters 裡紀錄的所有 prompt_ids 跟 generated_image 建立關聯

### Subject
- project_id: belongs to project
- name: 名稱
- type: enum，目前有 [character, scene], 預設為 character

### WorkflowSubject
Workflow 與 subject 之間是 many to many, through workflow_subjects

## 功能與程式架構

### 帳號系統與專案機制
- 建立基本的 User 登入機制
- Project based， 每個 user 可以建立多個 project，所有可操作的資源都是歸於 project 下
	- 每個 user 都會有一個 default 專案

### Context Module 程式架構
程式的架構請依不同的功能領域定義成不同的 context module，來區分不同業務領域
context module 內定義了該領域業務目標會被呼叫的的主要 functions 


以下為目前規劃的 context/domain 模組

```
Xaifu.Projects
Xaifu.Prompts
Xaifu.Workflows
Xaifu.Generations
Xaifu.Tags
Xaifu.Images
```


Projects 模組
- 管理 project 與 Subject

Prompts 模組
- 管理 Promp 與 PromptCategory

Workflows 模組
- 管理 workflow 與 workflow input

Generations 模組
- 管理 Generation
- 實際執行 Generation
- 與 ComfyUI API Service 互動

Tags 模組
- 管理 tag

Images 模組
- 管理 Generated Image

一般在 context module 路徑下會再定義：
- 該 context 下用到的 schema module
- 該 context 下的 submodule：如果 context module 的工作太過複雜，會將業務抽到 submodule，以維持程式可讀性與維護性
- submodule 只會被其所屬的 context module 呼叫使用
- context module 可以被別的 module 呼叫調用，但不能直接呼叫別的 context module 下的 submodule

以 Projects 為例

```
Xaifu.Projects -> 為 context module
Xaifu.Projects.Project -> Project Schema 定義
Xaifu.Projects.Subject -> Subject Schema 定義
```

以 Generations 為例

```
Xaifu.Generations
Xaifu.Generations.Generation -> Generation Schema 定義
Xaifu.Generations.External.ComfyUI -> 提供與 ComfyUI 服務互動的 API
```


### 基本管理功能
建立對 Project 下各項基本資源的 CRUD 功能，包括：
- Subject 主題管理
- Prompt 提示詞與類型管理
- Workflow 工作流管理
- Generation 生成紀錄的管理
- Image (GeneratedImage) 圖片的管理
#### Subject 管理
基本的 CRUD：
- 輸入 name: 用 text 欄位
- 選擇 type: 用 radio buttons
#### Prompt 管理
同時包含 PromptCategory 與 Prompt 的管理
##### 主畫面：
- 採用左側選單介面，選項有「全部 prompt」以及各個 prompt category；右邊主內容就是 prompt 的列表
- 依目前選擇的分類(或全部) 列出對應的 Prompts
- 新增 prompt category：
	- 左側欄有個 「+」可以點擊跳出新增 prompt category 的表單
		- 輸入 name 欄位，
		- 有一個隨機色彩選擇器，亂數產生文字&背景顏色的現呈，代表 font/background color 的值
			- 類似常見的為 label 設定顏色的方式
- 編輯 prompt category：
	 - 左側選單每個 prompt category 未端有編輯的圖示，點擊後跳出表單供編輯
 - Prompt 列表：
	 - 右方主內容會列出 Promp 列表，每個 Prompt 的： 
		- prompt category 名稱
	- 中文內容、
	- 英文內容
	- 備註
	- 對這筆 prompt 的動作選單：
		- 編輯：點擊後進入編輯畫面
		- 瀏覽：點擊後進入瀏覽畫面

##### 新增 Prompt 
Prompt 管理主畫面的動作選單，有「新增 prompt」的選項，
點擊後進入新增 prompt 的頁面，
##### Prompt 表單
- 用下拉選單挑選 PromptCategory
- 用 textarea 輸入中文內容(text_zh)，旁邊要有一個「複製」的圖示，點擊後將中文內容複製到剪簿
- 用 textarea 輸入英文內容(text_en)，旁邊要有一個「複製」的圖示，點擊後將英文內容複製到剪簿
- 用 textarea 輸入備註(remark)

##### 編輯 Prompt
- 與新增 Prompt 共用表單，但編輯時不能修改 PromptCategory

##### 瀏覽 prompt 的頁面
- 列出 prompt 的各欄位資料
- 會列出 prompt_change 的清單

#### Workflow 管理
列表：
- 會同時列出 workflow 資訊以及它所設定的 workflow_inputs 的 prompt category 名稱

新增/編輯表單：
- title: 文字欄位輸入
- description: textarea 輸入
- 選擇 Subjects，用 multiple select 選單選擇 subjects
- content: 提供兩種方式輸入：
	- textarea 直接貼入 json
	- 上傳 json 檔案，讀入內容存入 content 欄位
- 設定 workflow_inputs 關聯
	- 可以增加多組 workflow_input，每組 input 有
	- identifier: text 欄位
	- 用 select 選單選擇 prompt_category
	- 用 select 選單選擇 prompt
		- 當切換 prompt_category 時，會跟著動態更新 prompt 的選項
		- 允許不選擇，保留空白
瀏覽頁
- 顯示 workflow 的欄位
- 列出該 workflow 下的 generations
#### Generation 管理
較為複雜，分以下部分說明

##### 列表頁：
- 列出 Project 的所有 Workflow
- 可用 Workflow 來篩選 Generation

##### 新增 Generation 
- 必須先選擇 workflow，選擇後，依 workflow 定義的 workflow input 產生對應數目的 generation input fields
- 每組 generation input 會：
	- 帶入對應 workflow_input 的 identifier (不可修改)
	- 產生 select 選單讓使用者選擇 prompt
		- 選項會依對應的 workflow input 設定的 prompt_category，篩選對應分類下的 prompt 供選擇

##### 編輯 Generation
- 不可以再更改 Workflow
- 只能調整 generation inputs 的設定

##### 瀏覽 generation
- 列出 generated images

#### Image 管理

列表頁
- 列出 Project 的所有 Generated Image
- 可用 workflow、subjects、tags 篩選

## 業務功能
系統主要的業務功能是從建立的 Generation 去呼叫 ComfyUI 進行圖片生成，然後將生成的圖片資訊取回建立 Generated Image 資料

這部分的詳細邏輯我們先不實作，

以下提供預想的資料範例

假設已有三個 PrompCategory
PromptCategory:
1. name: 髮型
2. name: 服裝
3. name: 動作

另有五組 prompt (僅以 text en 為例)
prompt1
- category: 髮型
- text_en: red_hair, long_hair
- title: 紅色長髮

prompt2
- category: 髮型
- text_en: sliver write long hair, high ponytail
- title: 銀色高馬尾

prompt3
- category: 服裝
- text_en: wearing a cozy oversized beige cable-knit sweater and a long soft grey skirt
- title: 休閒服飾

prompt4
- category: 服裝
- text_en: office lady attire
- title: 辦公套裝

prompt5
- category: 動作
- text_en: sitting on wooden brench
- title: 坐在木椅上

建立一個 Workflow
Workflow
- title: 人物設定
- content: .... json 資料
- workflow inputs 有三個
	- input1
		- identifier: node_1_input1_text
		- prompt_id: prompt1.id
	- input2
		- identifier: node_3_input5_text
		- prompt_id: prompt4.id
	- input3
		- identifier: node_14_input4_text
		- prompt_id: 不指定

使用 workflow 進行一次 Generation
對三個 input 分別
input1: 不指定，即沿用 workflow的 prompt1
input2: 不指定，即沿用 workflow 的 prompt4
input3: 指定為 prompt5

即最後存入的 generation_inputs 為：

```
[
  { "identifier": "node_1_input1_text", "prompt_id": <prompt1.id> }, 
  { "identifier": "node_3_input5_text", "prompt_id": <prompt4.id> },
  { "identifier": "node_14_input4_text", "prompt_id": <prompt5.id> }
]
```



## UI 組件與互動設計

### 選單
#### 頁面動作選單
例如進到列表頁，會有「新增xxx」的選項，這個選項所在的選單稱「頁面動作選單」
例如進到瀏覽頁面，對該對象的「編輯」、「刪除」這些選項的選單，即這個頁面的動作選單

在各頁面出現的頁面動作選單，應設計在相同版面位置，並統一樣式與互動效果

#### 列表動作選單
在列表中，會列出多項資源，例如多個 prompt，每一列的 prompt 都會有可以操作的選單，例如「編輯」「瀏覽」，稱為「列表動作選單」 

列表選單每個選項請用「icon」+「文字」來表示，並統一樣式與互動效果
 
### 列表頁
- 列表頁請實作成 infinite scroll
- 列表盡可能抽出成組件共用，例如在 Generations 的列表，可以在瀏覽 Workflow 時顯示該 Workflow 的 Generations 時共用同個列表

### 其他互動元件
- 請使用 tom_select 來優化下拉選單組件
- stimulus 來建立共同的前端互動元件，包括
	- 點擊後將目標內容複製到剪貼簿的元件


以上為目前的需求規劃，請分析研究後，建立一份開發計畫與我討論
尚不完整的地方可以留在之後再定義開發，或暫時用假資料代替
並請把分析規劃的結果與開發計畫，存在 docs/ 目錄下

確認計畫沒問題後，就開始實作開發
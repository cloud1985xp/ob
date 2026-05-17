# Generation Images Management Mode




# Collection 功能

新增 Collection 的功能，來將 Generated Images 收錄進 Collection

## 主要開發項目：

### 一、從 generation 生成 image 建立/加入 collection
在使用 generation 生成圖片(run process)時
可以同時設定要將生成的圖片，建立(或加到現有的) collection
這時建立的 collection 會跟該 generation 有一樣的 subject
並建立關聯：
- collection belongs to subject
- collection belongs to generation
- collection has many prompts

Collection has many prompts (CollectionPrompt) 會紀錄
當下 generation 用的 inputs 資訊，包括每個 input 的：

- prompt_category_id 關聯
- prompt_id 關聯 (若沒有就留空)，以及
- prompt_title
- prompt_text (當下用的 text)

建立(如果 genertion 時選擇建立 collection) 好後
generation 後續生成的 image，也都會被加到這個 collection

在 Generation Show 頁面
增加 Run with Collection 表單
可以選擇：
- 「加到現有 collection」 -> 下拉選單
	- 實作 collection select 元件，與 prompt select、subject select 類似
	- collection select 的選項，要依當下 generation 選擇的 subject 連動只列出該 subject 的 collections
- 「建立新的 collection」出現簡易的表單
	- 出現 title 欄位，但允許空白(後端自動變成 timestamp)
	- 可選擇 nsfw
按下送出後開始執行 run process with collection

註：
- 仍然可以只 run generation process 但不使用 collection 功能
- 每個 image 可以被加到多個不同的 collection，這部分之後再實作
- generation 生成的圖片，會依生成的順序加入 collection (記錄成 position)

主要欄位說明：
- 標題，預設用 timestamp (yyyymmdd-hhiiss)
- 標籤(labels)，輸入時用 `,` 間隔，可以用 labels 當條件來篩選查詢
- nsfw: 一個 boolean 值標記是否為 nsfw，預設 false
- cover_image: 封面圖，關聯到 generated_image，當作 collection 代表圖片，允許空值
	- 在 generation 同時建立成 collection 時，會將第一張 image 設為封面
	- 若 generation 時只是加入現有 collection 時，不會改動 collection cover image

### 二、collection 功能

基本頁面
- /app/collections 會列出所有 collection
	- infinite scroll 載入
	- 依 likes 數 desc 排序
	- 可篩選：
		- 用 subject (subject select)
		- 用 title 輸入文字，模糊比對
		- 用 labels 
			- 可輸入文字，可多個用 `,` 間隔) 篩選
			- 或有預定義常用 labels 作複選
		- 用 nsfw (true / false toggler)

- /app/collections/:id 瀏覽 collection
	- 基本頁面跟 generation 類似，列出 collection 資訊
		- 用 ImageGallery 陳列 collection images，依 position 排列
		- 側邊有 image detail panel 跟可以使用 full view 模式
	- 另外，show 頁面的左側加上清單，列出同 subject 下的 collections，方便切換瀏覽
		- 實作 collection card 元件
			- 顯示封面，若沒有封面空 placeholder 區塊代替
	- Collection 可被 like / dislike，請使用即有模組實現功能
- /app/collections/:id/edit 與 /app/collections/new
	- 實作編輯與新增 collection 的功能，獨立頁面，但共用 CollectionForm，輸入欄位：
		- subject(required): 用 subject select 元件
		- generation (optional)
		- title
		- labels
		- nsfw
		- likes

註：
若在 image detail panel 或 full view 執行刪除 image，行為仍維持跟原本一樣：
- 把 generated_image 資料刪除
- 把該 image 與 collection 的關聯也一併刪除

目前僅實作從 generation 來將 image 加入 collection 的情境
未來會再支援直接操作 image 加入 collection，或從 collection 中移除 image 的情境

### 三、調整 subject show
在瀏覽 subject 的頁面，加上列出關聯的 collection，複用 collection card component

## 資料表規劃
collections
- belongs to subject (required)
- belongs to generation (optional)
- title: string
- description: text
- cover_image_id: reference generated_image，allow null
	- generated image 若有被關聯到 collection 的 cover_image，則不允許刪除
		- 先僅作 constraint 保護，不做檢查和 ui 顯示
- likes: integer
- labels: array of string, 可當作 search 條件
- nsfw: boolean

collection_images
關聯 image to collections
- collection_id
- generated_image_id
- position

collection_prompts
關聯 Collection 與當下 generation 生成時的 Prompts 資訊
- collection_id: (Collection has many prompts)
- prompt_category_id: belongs_to prompt_category, not null
- prompt_id: belongs_to prompt, allow null
- prompt_title: 當下 prompt title
- prompt_text: 當下 prompt text(en)


## 執行目標

請先計劃再開始實作
且確保既有的功能不被影響
並盡量使用現有的 module、元件，或重構再利用
維持一慣的 ui 和操作介面的友善

若有任何不確定，請與我討論或提出建議

追加以下修改：
- 在 Generation Show 頁面把 Run with Collection 的區塊，往上移到 Generation 基本資訊的下方，也就是 Input Configuration 區塊之上
- 在 app layout 的主選單，加上 "Collections" 的連結項目，放在 Subjects 下，並加上適合的 icon


# Prompt category 選單

將所有出現 prompt_category 的 select 選單
都改成 group option 的結構：
先用 prompt_category_group 分組、排序(by prompt_category_groups.position)
每組再排列 prompt_categories

請先將它做成一個可重複使用的 form component (可以 form field 整合)
然後把有用到 prompt category select 的地方都替換成這個 component

包括：
- PromptForm
- GenerationForm 裡的每個 GenerationInput
- Image2Text Record 裡的 Create prompt from record
- Image2Text Batch 裡 Create Prompts 的功能
請再檢查還有沒有其他地方也需要替換

請 Prompt Category Select 

FormComponent 的 category_select 元件
在表單編輯時，不會正確地選中 current option，請檢查並修正

# TODO
- 
- collection 功能
- 支援設定步數
- 支援設定 lora
- 手機版
	- 閱讀
	- 生圖
	- app?

- (wip) 圖生提示詞解析功能
	- 輸入圖片網址、檔案或貼上
	- 使用 grok API 取得提示詞
	- 使用 comfyui 取得標籤
	- 使用 grok API 過濾標籤：服裝顏色、場景
	- 串預覽的功能流程

- (wip) 利用 LLM 自動幫 prompt 命名 / 翻譯
- [x] 刪除 generated_image (同時要刪除圖片原檔與各 version 圖檔)
- [x] 修正 Generation duplicate 的功能
- Subject 優化
	- [x] 設定 rank 或 置頂 -> likes
	- [x] 可從 Subject 建立屬於該 Subject 的 Generation
- Generation 列表優化
	- [x] 顯示 recent 圖片預覽
	- [x] 用 subject 篩選或排列
	- [x] 用 workflow 篩選或排列

- 批次建立 prompt
	- 用 llm 用一段描述，批次生成多組提示詞，編修確認後批次建立
	- [x] 直接輸入多行文字，每行是一個 prompt

- [x] 把 尺寸移到 prompt level
	- 優先於 generation 的 size
	- 若多個 prompt 都有設定 size 時，
		- 在 prompt 加上 position，用 last position 的
- [x] ~~支援指定尺寸的功能~~

Workflow management
- 要可以設定 latent identifier, prompt identifier, filename identifier

# Full View 播放模式
在 Full View 模式下新增自動播放功能
可選擇：
- 播放方向，連續播放上一張 or 下一張
	- 可用鍵盤啟動連續播放：
		- h: 啟動連續播放上一張
		- l: 啟動連續播放下一張
		- s: 停止
	- 也要有 ui 可以用點選圖示的方式來啟動 (並在旁邊帶有熱鍵提示)
- 播放速度，間隔多久(ms) 播放，用一個 slider 讓滑鼠來控制
- 仍然可以手動切換上/下一張
	- 維持可用原本的鍵盤控制
	- 手動切換後就停止連續播放
另外加上可以用熱鍵：
- 方向鍵「↑」來代表按 like，
- 方向鍵「↓」來代表按 dislike

請做以下調整
一：
在 Image Full View 模式下，使用者切換上一張/下一張圖片時，要記下使用者的切換動作是「上一張」還是「下一張」
然後當刪除圖片時，預設不要關閉 full view，
而是依照剛才紀錄使用者切換動作，繼續切到下一張圖片(或上一張圖片)
- 若沒有下一張(或上一張)圖片時，才關閉 full view

二：
當有圖片被刪除時，也要從畫面上的 image list 把對應的 image 移除
# 刪除圖片
加上以下功能修改：
- 在側邊欄展開瀏覽 image detail 的面板裡面加上刪除按鈕，按下刪除經過確認後，會將該筆 generated_image 刪除。
	- 必須確保刪除資料，同時也要刪除實體(s3)的檔案。
- 對 image detail 進入 full view 模式，在 full view 模式可以按下鍵盤 "d"  鍵刪除圖片。
	- 一樣要經過確認後，才執行刪除
	- 一樣要確保刪除遠端的檔案



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


# 實作 comfyui 圖生文功能

新增「圖生文」功能，在 /img2text

會分成上下兩個表單

第一個表單
左方有一個輸入圖片的區塊
使用者可以透以下方式來輸入圖片：
- 由使用者上傳一張圖片檔案
- 由使用者提供一個圖片的網址
- 使用者直接從剪貼簿貼上的 blob 資料

使用者輸入後
右方會先出現該張圖片的預覽

送出後，會透過與 comfyui 整合來進行圖生文，我理解的實作流程會是：

先將圖片內容，呼叫 comfyui 上傳成檔案 asset，取得該 asset 的 id
利用預先定義好的工作流 json，將 asset id 置入 json
用工作流 json，呼叫 comfyui api 執行，得到回傳的資料

預先定義的工作流 json 請先用常數定義，
以及給定替換 asset id 的 node name(identifier)，我之後會再修改 

第二個表單
左測有一個 textarea 欄位和一個 workflow 的選單
workflows 的選單可以選擇 workflow
第一個表單圖片產生的 text，會放進畫面中這個表單的 textarea

按下「預覽」按鈕，會用所選擇的 workflow + textarea 的內容，進行生成圖片的預覽
產生的圖片的顯示在第二個表單右側的區塊



請先幫我把整個流程架構規劃好，缺少的部分我再補上


# 擴展圖文生文功能

一、建立圖生文資料紀錄
我想將圖生文的紀錄用開資料表紀錄下來
即 form1 的行為會被存下來
資料表名稱 image2text_records

紀錄包括
- image：將貼上(或上傳、或 url取得)的原始圖片存下來(上傳到s3)
	- version 包括原圖與縮圖 (thumb)，縮圖按照原比例，縮至寬最多320px
- comfyui_filename：來源圖片在 comfyui upload 之後的檔名
- content: img2text 之後得到的文字
- state，狀態
	- saved: 上傳至 s3 完成
	- uploaded: 上傳至 comfyui 完成
	- converted: img2text 完成
- remark: 文字備註

二、更新現有 img2text 功能
將現有的圖生文功能 form1，修改成會把執行結果存進 (一) 建立的資料表

三、增加 img2text 紀錄列表
預設依 inserted 時間 desc 排序，列出所有 image2text_records
用 infinite scroll 載入更多
列表顯示縮圖、狀態、content、文字備註

四、image2text show 畫面
(三) 的列表可以點擊進入 show 畫面
show 畫面顯示原圖，以及可編展 remark
同時也顯示原本 img2text 的 form2 (請抽成重複使用的元件)，
表單裡填入當下這筆 image2text_record 的 content，可進行 generation preview


請先幫我把整個流程架構規劃好，若有不確定之處請與我討論

# 批次 img2text 功能
建立 image2text_batches 資料表，來存放批次
每個 batch 可以有多筆 image2text_records
image2text_record belongs_to batch，但允許不屬於任何 batch

batch 的欄位有
- title: 標題，若使用者沒填就自動用 timestamp 命名
- state: 狀態，pending、saving、ready、processing、completed
- remark: 備註文字

使用者可以建立批次
- 在批次裡上傳(增加)多個圖片，來建立 image2text_records
- 增加 records 後，先把每筆 records 的來源圖片存下，狀態為 saved
- 然後執行批次，把所有 saved 的 records，執行 img2text，即經過 uploaded -> converted 的階段
	- 進行批次 img2text 處理會需要長時間，應該要用 oban 處理
	- 進行多個檔案上傳，因為會需要處理大量圖檔 blob 資料，我覺得不適合用 oban，用同個 process 開 task 或其它適合的方式處理

## 操作流程：
- 使用者先建立批次
- 建立後，進入批次圖片上傳表單
- 使用者上傳多個圖片
- 送出表單後，
	- 批次狀態變為 saving
	- 會將圖片建立成多筆 image2text_records
	- 完成後狀態變為 ready
- 使用者會進到 show 畫面，show 畫面列出各筆 image2text_records
- image2text_records 的排列用檔名排序
- 可按下 process 開始執行
	- 執行時狀態變為 processing
	- 執行過程中，不能再進到圖片上傳表單 = 不能再增加圖片
	- 執行完成後，
		- 狀態變為 completed
		- 可以再進到圖片上傳表單再增加圖片

可以在增加更多圖片後，再次按下 process 執行
每次執行都只會處理狀態為 :saved 的 records，進行 img2text 轉換

## 主要頁面：

Batch Index
- Infinite Scroll 載入
- 預設用 inserted_at desc 排序
- 可用 title 搜尋
- 顯示檔名、狀態與備註文字，可建立、編輯批次
- 可點擊進入 show 畫面
- 可點擊直接進入 upload 畫面

Batch Show
- 列出 records
	- 複用 app/img2text/records/ 的列表，呈現 縮圖、狀態、content、文字備註
		- 點擊 record 進到對應的 app/img2text/records/:id show 畫面
- 有按鈕點擊進入 upload 畫面

Batch Upload
- 有表單可以進行多張圖片檔案上傳
## Route 規劃：
/app/img2text/batches 會列出 batches 的清單
/app/img2text/batches/:id  batch show 畫面
/app/img2text/batches/:id/upload 該 batch 的圖片上傳表單畫面 


請先幫整個流程架構規劃好，若有不確定之處請與我討論
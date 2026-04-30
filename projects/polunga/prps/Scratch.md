# 優化 Scratch Conversation 

我有以下問題，請確認是否合理，並提出修改建議
- 使用者送出文字訊息後，是否應該就要先寫入 conversation，而不是等 agent 處理完才存入？
- converation 在 agent 進行處理時，希望有更明顯的「處理中」的提示
	- 可以直接在顯示 assistant 的對話泡泡「…」的樣式，來代表 agent 正在處理中
	- 或參考現有 gemini、chatgpt、claude 等的處理方式
- 現在有時候是輪到 user (agent 已經處理完且回應，狀態非 processing)的情況，但畫面上仍出現 assistant 的對話泡泡「…」，請檢查並修正
- Agent(ClaudeCode / Gemini) 處理時間往往很長，有什麼辦法增加 log 或除錯的方式，知道 process 仍在運作處理中？

# Setup Workspace 的 skill

在 Agent 初始化 setup_workspace，加入複製 skills 的行為

Skills 的來源基本路徑為  priv/agent-plugins 下的：

/{category_name}/{plugin_name}/skills/{skill_name}

我希望有一個清單(先定義成常數)來管理要複製 skill 有哪些

在 setup_workspace ，會將清單中的 skills，複製到 provider 對應的 skills 目錄下

例如
provider = claude_code

要複製的 skill ：
category = "server"
plugin_name = "ishin-engineer-assistant"
skill_name = "diagnose-dev-error"

就會將 

priv/agent-plugins/server/ishin-engineer-assistant/skills/diagnose-dev-error

複製到

workspace/.claude/skills/diagnose-dev-error



# 修正 Template 管理功能
目前 Template 管理(CRUD)有以下問題
- 在編輯(edit) template 時，每按下 add fields 時，在觸發 validate 後，畫面上原有的 fields 資料會消失
- 每個 field 的排序被擠壓，使得每個欄寬都被縮到最小，請調整成填滿整列的樣式，讓每個欄位都有適當的寬度分配
- 每個 field 在選擇 type 為 Selection 時應該要出現 textarea 讓使用者輸入 
- 在 template 增加多個 fields 後，再對 field 按下刪除，只能刪除一次，之後再對其他 field 按刪除，會變沒有作用
請詳細檢查問題原因，並進行修正

# Scratch 線上工程師助理 LLM Agent
目前專案已經有實作 Agent 機制，能夠：
- 接受使用者輸入的訊息
- 建立 conversation
- 將使用者的問題，當作提示詞，傳入啟動的 agent
	- agent provider 可以是 claude code 或 gemini
- agent 處理完後，將結果回覆給使用者
- 繼續後續的 conversation

Agent 的運行已確認可行

現在要實作整個功能面的設計，請協助我規劃並實作整個網站的需求

## 目標

這個功能名稱叫做 Scratch ，是這個系統目前主要的功能之一
(未來還會增加其他功能)，用 LLM 實現 Agent 來執行/回答用戶的託委 or 問題 

當使用者輸入工作上遇到、需要委託工程師處理的問題，會到系統上與 Agent 展開對話，提出要處理的問題

Agent 即扮演工程師助理的角色，會依照用戶的問題，進行處理，並將結果回覆，或是向用戶收集更多問題的資訊，再進行處理

Agent 的問題處理能力我們先忽略，我們會設計 skill system 來擴展 agent 的能力，後續再加強擴充

## 實作內容
### 一、增加 template 功能
當用戶要展開對話時，可以先從 template 挑選 request 模板
選擇了模板，會出現表單，依模板設定要求用戶輸入對應所需的參數，以及問題內容

不同的模板需要的輸入參數不同，這部分可透過 request 模板的管理功能來設定

模板還包括了內建的前置提示詞，當使用者送出問題後，系統會將前置提示詞，搭配模版的參數，再組合使用者的問題，當作內容傳送給 agent 處理

使用者也選擇可以不使用模板功能，直接輸入問題
系統僅將使用者輸入的文字當作 prompt 傳給 agent

### 二、規劃設計整體 layout 與 ui

整體動線還是經由首頁 ->  Scratch 
所以還是要設計獨立的首頁與 layout，只是目前只有 Scratch 一個功能，可以用類似 desktop

進入到 Scratch 的功能可以設計全新的 layout，可以類似大部分主流 AI ChatBot 的介面，例如 Gemini, OpenAI, Claude 等

左側欄是過去的對話主題的歷史紀錄，可以點擊在右方主畫面開啟對話內容，也請將對話的介面重新美化設計

因為有 template 的功能，可以在新增對話時先引導使用者從現有 template 中選擇，或不使用 template

另外希望可以用 template 分類來瀏覽對話紀錄的介面
可以是列出對話的清單，然後可以選擇用 template 分組顯示，或是用 filter 來用 template 篩選對話紀錄

### 三、RequestTemplate 系統

#### Request Template 資料結構
定義 request template 的欄位包話
- name: 標題
- description: 說明該 template 用途
- fields: 用 json 存下該 template 需要使用者輸入的參數
	- 可以用 embed_many 來實現 
- prompt: 屬於 template 的前置 prompt 內文

每個欄位可能要包括
- identifier: 代號，例如 date、player_id，用來對應可能出現在 prompt 內文要用到的參數
- label: 欄位的顯示名稱
- type: 欄位類型
- required: 是否必填

欄位類型可以是
- date: 代表是要使用者輸入一個日期
- datetime range: 要使用者輸入時間範圍，
- selection; 需搭配定義有哪些選項，可以用 textarea 輸入，每行代表一個 option
- string: 單行的文字

#### Request Template 表單顯示
當使用者選擇用 template 來開啟對話時，
會 render 出表單讓使用者輸入參數，不同 type 的欄位會有不同的輸入介面

date: 出現日期選單
datetime range: 會出現一組介面，讓使用者可以
1、直接選擇 30分鐘內、1 小時內、2小時內、3 小時內
2、或自訂輸入 hh:mm:ss 表示自這個時間內
3、或自訂輸入選擇 start_at 與 end_at 的 yyyy/mm/dd hh:ii:ss 範圍

selection: 出現下拉選單供選擇
#### Request Template 管理
要有一個可以對 template 做 CRUD 管理的功能
其中 New/Edit template 時，實作一個共用的 form component
並且 form component 要支援管理 template fields 的功能


請完整規劃上述需求的程式架構與頁面設計，擬定開發計畫
與我討論確認後，再開始進行
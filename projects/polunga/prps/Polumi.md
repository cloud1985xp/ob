# 建立與 AI Agent 互動功能

我要建立一個功能讓使用者可以建立與 AI Agent 的對話，在對話中與 AI Agent 訊息往來

## 資料表設計
建立兩張表 Conversation, Message 來分別儲存對話 session 與對話的歷史訊息

- conversation belongs to user(user_id)
- conversation has many messages

因為本專案沒有真的 user table，
user id 等資訊是由 auth token 中的取得
所以 conversation 只要紀錄 user_id 值，不用設定 db 關聯

Conversation 基本欄位
- id: integer, primary key
- user_id: integer
- session_id: UUID 格式
- title: 對話標題
- model: 對話使用的 LLM 模型

Message
- id: integer, primary key
- conversation_id: belongs to Conversation
- role: user or system
- content

## 基本功能

製作基本的介面，讓使用者可以
- 列出 conversation 列表
- 新增 conversation
	- 進入 conversaion 後，列出訊息紀錄
	- 輸入訊息紀錄，等候 agent 回應
		- 目前先設計成在 agent 回應前不可再輸入訊息
- 刪除  conversation

## Agent 的實作
當使用者進入 conversation 時，會啟動 Agent

首先會初始化工作區：
先用 conversation session_id 作為識別
得到一個工作區(workspace) 路徑： /tmp/polumi/{session_id}/

1. 檢查 workspace 目錄是否存在，不存在則建立
2. 將 priv/polumi/* 所有資料複製到工作區下
3. 將 messages 資料整理成對話紀錄的格式，存放到 {workspace}/chat.history，若檔案已存在則覆蓋

當使用者提交訊息時
1. 畫面顯示處理中的提示
2. 將此次的使用者的訊息，存在對話紀錄的格式，附加到 {workspace}/chat.history 檔案末端
3. 執行 command line 指令 
	1. 且 command line 會先切換到 workspace 路徑下才執行
	2. 執行 claude -m {gemini-2.5-flash} --yolo -p "{message}" 
	3. 等候 cli 執行結果
4. cli 回傳結果，
	1. 將此次的 agent 的訊息，存成對話紀錄的格式，附加到 {workspace}/chat.history 檔案末端
	2. 將 agent 的訊息結果顯示在畫面

Conversation 用  Liveview 實作
每個 Agent 會是一個 GenServer process 獨立運作
透過 session_id 作為 conersation 的識別，用 PubSub 推送 agent 執行 cli 的結果回 LiveView

對話紀錄的格式，請用方便 Agent memory 理解的格式
暫時先不考慮內容長度的問題
但要保留機制未來我們要可以對歷史紀錄做摘要的機制
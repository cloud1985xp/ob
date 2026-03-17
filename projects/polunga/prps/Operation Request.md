
製作讓使用者提交 營運需求案件的功能
並能將案件加到批次，讓工程師依批次去處理案件的內容

一個批次會有多個案件
實務上，個批次作業的執行時間其所包含的多個案件中，expration 日期最近的
代表必須在該日期把對應的案件都處理掉

建立資料表

OperationRequestCase: 每一個被提出的案件
OperationRequestBatch: 案件被歸入的批次

欄位

OperationRequestCase
- batch_id: belongs_to OperationRequestBatch,
	- 可以是空值，代表暫時未歸在任何批次下
- creator_id: belongs to User, 代表建立/提出案件的 user
- parameters: text 欄位，會用來存序列化的資料，未來可依不同案件存不同的參數
- description: text 欄位，用來存放案件描述資訊，純文字
- issued_on: date, 提出日
- expired_on: date, 到期日

OperationRequestBatch
- title: string 欄位，該批次作業的標題
- state: integer, 狀態，會用 ActiveRecord enum 來實現
- processed_on: date 欄位，批次作業預計執行的日期

建立 model class

OperationRequestCase
對 paramers 做 store，目前設定裡面要存放一個 player_id，來代表處理案件的玩家 id

OperationRequestBatch
- state 用 enum 來存放狀態，包括 "pending", "processing", "completed"

實作提出案件的功能

- 位於 /requests，
- OperationRequestsController
- 進入畫面會顯示表單，讓使用者建立新的 OperationRequestCase
- 實作 OperationRequestForm

OperationRequestForm
是用來建立或編輯 OperationhRequestCase

表單提供以下欄位
- expiraton_on: 讓使用者輸入到期日期
- description: 讓使用者輸入案件描述
- player_id: 讓使用者輸入 玩家 id，會序列化存入 parameters[:player_id]


但會再分成以下兩種情況：

一、如果當下系統內有未完成(pending)的 Batch
表單會有 batch selection 讓使用者在建立 case 時選擇要歸在哪個 batch 下
- 選項為所有 pending 的 batch
OperationRequestForm save 時將新建立的 Case 歸屬在該 Batch 下

二、如果當下系統內沒有任何未完成的 Batch，
則表單不需出現 batch 選項，僅出現文字提示：建立後會自動產生一個批次作業
OperationRequestForm save 時會在建立 Case 時同時建立一個新的批次
並自動為該批次
- title 以新建立的 case 的 expired_on 日期來命名，例如 "2026/03/30 工作批次"
- 將 processed_on 設定成跟 Case expired_on 一樣的日期


通知系統
實作一個通知機制，目前是用 slack 發送通知
並將 slack 的訊息的發送頻道、通知的訊息內文要 mention 的對象，都設計成可以被設的組態

在以下事件發生時會要發送通知，且不同事件可以設定不同的

- 頻道
- mention 對象(可多位)
- 訊息內文

事件包括：
- 當有 case 被建立的時候
- 當有 batch 的狀態從 pending -> processing 時
- 當有 batch 的狀態從 processing-> completed 時

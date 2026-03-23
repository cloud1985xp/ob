# 營運案件管理與通知系統

製作讓使用者提交「營運需求案件的功能」
並且有「案件批次」的概念，會將案件加到批次，讓工程師可以事後依批次去處理案件的內容

一個批次會有多個案件
實務上，一個批次作業的執行時間為其所包含的多個案件中，到期日(expired_on) 最近的日期
代表必須在該日期把該批次的的案件都處理掉

## 建立資料表

OperationRequestCase: 每一個被提出的案件
OperationRequestBatch: 案件被歸入的批次

欄位：

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

## 建立 model class

OperationRequestCase
對 paramers 做 store，目前設定裡面要存放一個 player_id，來代表處理案件的玩家 id

OperationRequestBatch
- state 用 enum 來存放狀態，包括 "pending", "processing", "completed"

## 功能實作

### 案件管理功能

- 位於 /requests，
- OperationRequestsController
	- 實作 OperationRequestForm
- 可以瀏覽目前未完成(pending) 的批次作業清單
- 可以讓使用者建立新的 OperationRequestCase- 

主畫面

顯示 batch 清單：
- 預設只顯示 pending 的批次作業
	- 同時顯示包括所含的 case list，用 table 呈現，欄位包括 case 的
		- expired_on
		- issued_on
		- parameters
		- description
- 批次作業清單用 processed on 升冪排序
- 若當前沒有未完成的批次作業則顯示 blank state

主畫面同時顯示表單(OperationRequestForm) 讓使用者建立新的 case
實作成 OperationRequestForm class來封裝行為
用來建立或編輯 OperationhRequestCase

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

理論上同時間只會有一個 pending 的 batch 可供選擇，因為 batch 是隨著 case 一起建立，
目前沒有其他方式可以另外建立 batch


### 通知功能

實作一個通知機制，當事件(主題)發生時，發送通知訊息
目前是用 slack 發送通知，保留以後擴充其他通知管道

請將 slack 訊息的
- 發送頻道、
- 通知的訊息內文
- 要 mention 的對象(可多位)

都設計成可以被設定的組態
而且且不同事件可以有不同的設定

且因應案件管理功能，定義在以下事件發生時會要發送通知
事件包括：
- 當有 case 被建立的時候
- 當有 batch 的狀態從 pending -> processing 時
- 當有 batch 的狀態從 processing-> completed 時

對應的頻道、訊息內文、mention 對象，預設都先用暫時的假資料，我之後再修改預設值



# 修改 OperationRequestForm

請 OperationRequestForm 增加 validation
檢查 OperationRequestCase 的 expired_on 不可以小於所屬 OperationRequestBatch 的 processed_on 時間

# 增加 OperatioRequest 功能

一、「處理」和「完成」OperationRequestBatch

- 對 operation_request_batches table 增加欄位：
	- processed_by: 關聯到 user
	- processed_at: timestamp 
	- completed_by: 關聯到 user
	- completed_at:: timestamp 類型
- 對 Controller 增加 process 與 complete action，對參數 params[:id] 指定對應 id 的 OperationRequestBatch 進行
	- process 的動作
		- 請實作成 OperationRequestBatch#process(user)
			- 將 state 改成 processed，
			- 將 processed_by 設為傳進來的 user
			- 將 processed_at 設為當下時間
	- complete 的動作
		- 請實作成 OperationRequestBatch#complete(user)
			- 將 state 改成 completed，
			- 將 completed_by 設為傳進來的 user
			- 將 completed_at 設為當下時間
- 對 index action 增加接受 params[:state] 參數，來篩選對應 state 的 OperationRequestBatch
	- 預設仍顯示 pending 的 batches
	- 在 index 畫面加上選項來篩選 pending 、processing 、completed or all batches
- 對 index 列表中的 batches，每一個都加上 action buttons，包括
	- process: 對該 batch 執行 #process
	- complete: 對該 batch 執行 #complete


二、實作 #show 畫面
在 index 列表可以點擊  batch 進入 #show 畫面，瀏覽該 batch 的資料
除了顯示基本資料、關聯的 cases 之外，另外顯示工作指引內容

工作指引的內容，直接沿用目前的 Admin GDPR 功能，
請參考 Admin::GdprController 與 Admin::GdprPlan

用 OperationRequestBatch 下各 cases 的 player_id，來組成 GdprPlan 所需的 user_ids
用 processed_on 的日期轉成時間，來做為傳入 GdprPlan 的當下時間。

在畫面上顯示和. Admin::GdprController#index 一樣的內容

在 #show 畫面一樣有 action buttions，來對 batch 執行 process 和 complete 的 action

# 實作 Case 編輯

實作 requests/:id/edit，對應 OperationRequestsController#edit, #update action 來進行對 OperationRequestCase 的資料編輯
一樣使用 OperationRequestForm 來實現編輯功能

並在 OperationRequestForm 裡考量以下情況：
- 當有 OperationRequestCase 的 expired_on 日期有調整時，要重新評估調整所屬的 OperationRequestBatch 的 procssed_on 是否要被更新
- 當有 OperationRequestCase 所屬的 batch 有變動時，變動前的 Batch 與變動後的 Batch 的 procssed_on 都需要評估是否要更新

原則是  OperationRequestBatch 的 processed_on 必須是 has_many cases 中最小的 expired_on 日期

在 OperationRequestsController#index 的畫面上，對每個 batch 裡的 cases 加上 edit 的連結，連到 requets/:id/edit 來進行編輯

修正：在 OperationRequestForm 裡 save 時，若 batch 或 previous batch 的 processed_on 有變動時，對應的 title 也要跟著調整

# 實作 Case 刪除

實作 DELETE requests/:id 對應 OperationRequestsController#destroy action 來對
OperationRequestCase 的資料刪除

並在刪除後，一樣要觸發對該 case 所屬的 OperationRequestBatch 做調整，包括：

- 如果刪除後該 Batch 下沒有任何的 case，那該 batch 也跟著被刪除
- 否則，要檢查該 batch 的 processed_on 和 title 是否要被更新

在 #index 的列表中，一樣對 case 加上 "delete" 的連結，發送 DELETE request 到 /requests/:id 來執行刪除，刪除連結要加上 data-confirm 的訊息做確認


進行以下調整：

1、在 #index 畫面中，每個 batch 裡的 case，請用 expired_on 升幕排序
2、如果 batch 的狀態是 :pending，才會允許刪除裡面的 case



# 產生文件

請將目前實作且調整後的 OperationRequest 整體功能
彙整成規格文件，放在 ai_docs/operation_request_system.md 裡
做為日後理解功能規格和設計的依據


請檢查並確認 OperationRequesgt 相關的 spec 測試
確保 spec 內容符合規格與程式現狀，若有需要請修正測試檔

包括：
 - spec/models/operation_request_batch_spec.rb
 - spec/models/operation_request_case_spec.rb
 - spec/forms/operation_request_form_spec.rb
 - spec/services/operation_request_notification_service_spec.rb

請把 OperationRequestNotification 讀取的設定檔 `config/operation_request_notifications.yml`
改成可以依  Rails.env 有不同的設定，包括 development、test、staging、production 環境



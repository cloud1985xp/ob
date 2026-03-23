
# 用戶帳號案件檢查功能

## 背景

用戶帳號案檢查功能，負責以下工作
- 去指定的 googlesheet 上取得資料
	- 從指定的 row number 開始處理資料
	- 取得 user_id(player_id)，預設是在 F 欄
	- 取得 toggler 欄位，是一個 checkbox 預設是在 M 欄
- 對取得 rows data 檢查， 只留下有 toggler 是有 checked 的 (value=true)
- 對這些 data 裡的每個 user_id，到系統資料庫查找對應的 transfer_code
	- 用 user_id + transfer_code，產生一個定型文案
	- 將這個文案寫回 google sheet 上對應的 replay_content 欄位，預設是 L 欄

googlesheet 端的存取目前已經完成了，在 GoogleSheets::CustomerAccountCase class

## 實作需求
現在要實作的部分是
是讓使用者啟動檢查的作業，

路徑位於
/customer/account_cases/
請在 routes 增加一個 namespace "customer"，在下面定義 account_cases resources
應該只會有 index 和 create 兩個 action

- index 顯示表單讓使用者填寫設定
- create 接受參數、儲存並執行檢查

在畫面上提供一個表單，讓使用者輸入

- googlesheet 網址 / id
- 想要讀取資料的起始列 number，預設值用 GoogleSheets::CustomerAccountCase::DEFAULT_START_ROW_NUMBER
- 對應的各欄位
	- player_id
	- toggler
	- reply_content
	- 預設用 GoogleSheets::CustomerAccountCase::DEFAULT_COLUMNS 裡的資料
- 文案的內文，預設定

使用者提交表單後，controller 會將收到的參數，經過 model class Customer::AccountCaseCheck 寫入，這是一個類似 singleton instance，將傳入的參數寫進唯一個 instance，資料存在對應的 redis value 欄位，目前已經定義完成

Customer::AccountCaseCheck 裡已有部分實作

請接續補完 attribute= 與 save 的行為

controller 在 #create action 將 model save 成功後
會執行檢查作業
這部分要用 background process，請用 active job (sidekiq) 來實現
請參考其他地方類似的作業

建立 AccountCaseCheckJob
在 Job 中取得 model instance ，再執行 #perform method

Customer::AccountCaseCheck#perform 部分目前已經完成
唯 sheet_options 的部分，必須改採用使用者輸入/儲存的值
若沒有值就用原本預設值

## 畫面
實作 #index 的畫面，需包括

- form 表單，讓使用者輸入 AccountCaseCheck 的各參數
- 目前 GoogleSheets::CustomerAccountCase 的 data，不需要重 load，直接顯示 redis 裡現有的資料，若資料已過期就用 blank state，告知目前沒有資料

當使用者送出表單，job 會進入 enqueue 或執行中的狀態
要將畫面使用 job state 顯示目前的狀況
請參考其他地方整合 job state 的作法
## 參考程式碼 

- app/models/google_sheet/customer_account_case.rb
- app/models/customer/account_case_check.rb

Job State 
- app/models/job_state.rb
- app/models/operation_patch.rb
- app/presenters/operation_patch_presenter.rb
- app/presenters/job_state_presenter.rb
- app/views/operation_patches/_presenter.html.erb
- app/presenters/job_state_presenter.rb


請將 #index 的表單 form 裡改成用 simple_form_for 的寫法

請參照目前的修改檔案，包括：

- app/controllers/customer/account_cases_controller.rb
- app/jobs/account_case_check_job.rb
- app/models/customer/account_case_check.rb
- app/models/google_sheet/customer_account_case.rb
- app/views/customer/account_cases/index.html.erb

將 Customer Account Case Check 的整體功能
理解功能及行為後
將結果匯整成規格設明文件
放在 ai_docs/customer_account_case_check_system.md 裡
做為日後理解功能規格和設計的依據

請根據目前的理解與規格，對以下檔案程式實作 rspec 測試

 - app/models/customer/account_case_check.rb
- app/models/google_sheet/customer_account_case.rb
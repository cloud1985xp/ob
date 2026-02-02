# 檢查 Job 是否仍在 Sidekiq 處理中

目前有一 JobState 統一封裝 background job 的狀態管理
其背後是用 sidekiq 執行真正的作業

在 JobState#enqueue 方法中目前已有將 sidekiq (provider) job id 紀錄下來
即當執行 enqueue 後，表示已將 job 正確送入 sidekiq queue 中進行，而取得了 job_id

## 問題：
worker process(sidekiq) 有可能因為特殊原因(例如記憶體不足)，而直接被 system kill (例如 container 直接被關閉)
而造成 Sidekiq 異常中止，沒有正常回收該 job，導致當 work process 重啟後，JobState 仍認為 job 還在被 sidekiq 處理，但實際上 sidekiq 並沒有真的在繼續處理中

## 優化的部分 

一、在 job state 中增加一個方法
 從紀錄的 provider_job_id 去查詢 sidekiq，得知該 job 是否仍在正常處理中，
 例如：有真的在 queue 等候或真的有 worker 正在處理中)

ex:
```
JobState#handling?
```


二、更新 JobStatePresenter
在 presenter 顯示時，增加判斷 job_state 是否仍在 handling? 
若否，則 render 的 partial 畫面會顯示訊息類似：

```
This job has been killed in accident, please restart it or contact system manager
```


但請不要影響「正常狀態」下的顯示邏輯
也就是如果 job state 有正常被完成，那它不應該受 handling? 的檢查影響，因為正常完成的 job，它本來就不會繼續存在 sidekiq 處理佇列中

## 對應參考檔案

- JobState: app/models/job_state.rb
- JobStatePresenter:
	- app/presenters/job_state_presenter.rb
	- app/views/shared/_job_state.html.erb
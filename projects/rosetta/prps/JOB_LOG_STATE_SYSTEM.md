
# 優化 Background Job 訊息紀錄機制

目前系統有許多使用 Oban 執行的背景工作

但通常 Oban Job 長時間運行，希望建立一個共同的機執在 job 執行過程的 process，可以將目前的進度 訊息 broadcast 出來，讓使用者在 liveview 頁面上可以看到 log 訊息

目前已有部分地方有實作類似的機制，但我想優化它，使它方便應用到其他各處的 Oban Worker / Background Process

現有的機制請參考

> Rosetta.Translations.Imports.process
> Rosetta.WorkerLog
> RosettaWeb.ImportLive.Show

例如：
```
{:ok, logger} = WorkerLog.create("import", import.id)
worker_log(logger, "Start importing duckdb strings")
```
建立 logger，並用 worker_log 方法來紀錄訊息

而呼叫其他 module 時，必須把 logger 當作參數傳送

例如：
```
opts = [
	....
	output: {logger, WorkerLog.Broadcaster}
]

ImportDuckdbStrings.execute(opts)
```

請先參考現有的程式來了解情境
## 修改需求

我想讓使用方式可以簡潔一些，並且盡量使用 elixir 本身的機制來實作，目前計畫

- Worker 使用內部 Telemetry 發送自定義事件
- 寫一個專門的模組監聽事件，再由它負責做 PubSub.broadcast
- Liveview 訂閱 PubSub 收到訊息後更新在畫面上

請規劃並實作這套機制，並且最用在目前的 Imports.process (包括 ImportDuckdbStrings ) 的使用情境



# 處理 Oban Job 非預期中斷問題

問題：
Oban Job process 有可能因為非遇期的原因被中斷，例如 container 佔用過多資源而被強制 kill，會造成正在執行的 job 停止，但 Oban 並未回收它，導致就算 oban process 重啟後，該 job 在 db table 裡該 job 還是執行中的狀態，但實際上沒有任何 process 在處理它


請規劃適合的解決方案，提出來跟我討論
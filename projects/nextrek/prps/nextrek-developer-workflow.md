

schedule 每週一~五，早上八點

請用 github mcp 前往專案 nextrek，列出目前 projects issues，assignee 是我的 project issue 且狀態是開發中的

對每一個 issue
使用 issue tracker 了解 issue 的內容與目前進度

發送 slack 將目前收集的 issue 狀況，整理後彙報到 slack 


# Nextrek Development Team

這是為了用 LLM AI Agent 打造一個 Nextrek 產品的開發團隊 

## Nextrek 專案背景

用 Ruby/Rails 開發的一套 saas 產品
資料庫用 PostgreSQL
採用 git/github 做版本控制
專案管理追從使用 github 上的 projects

Github Repo: github.com/ataxtrek/nextrek
本地的專案目錄位於 /Users/aaron.kuo/projects/nextrek

## 基本工作流程

### 一、理解並彙整目前工作項目
追蹤 Github projects 上屬於我(assignee=cloud1985xp) 且狀態是開發中的 issues

可用 issue id 做為識別

對每個 issue 進行內容與最新的問題理解、消化後，建立成最新的 todo
- 若判斷沒有新的問題，則不需建立 todo
整理好後的 todo 內容，先彙成一份報告，發送到 slack 頻道

每次消化後的理解必須匯整並且紀錄下來，
作為下一次追蹤理解時，判斷同個 issue 是否有新的後續問題或內容

#### 特別的需求
- issue 白名單：可以定義一份白名單，名單中的 issue(id) 代表不需要由 agent 處理

### 二、依當前的 todo，每個 issue 啟動 agent 進行開發

開發 agent 進入到本地的專案目錄開發
多個不同 issue，可利用 claude worktree 的功能分別開發

需要有定義一個 git branch 命名的規則
若是新 issue ，會 checkout 新的 branch 開發
若是既有 issue 的後續修改，用相同的 branch 接續開發

專案目錄下建立自己的 CLAUDE.md 與文件，來作為開發時的運行規則
每個 issue 開發修改完成，將該次修改的內容 push 至 github remote
並回覆進度到對應的 project issue 上

### 三、回報各項工作處理進度
當各項 issue 的 todo 事項被完成後
將工作成果整理至 done 的目錄下，並將 todo 清除
另外彙一份成果報告，發送到 slack 

## 專案開發的 agent

專案目錄下可有自己的 CLAUDE.md 與文件，來作為開發時的運行規則
請同時也規劃設計這邊的運作模式，以順利讓團隊中的各個 agent 合作

## 運行與啟動
預計這組 agent 團隊會每天(一~五)持續運行
起初可能是透過手動呼叫啟動，未來希望可以用自動排程
基本上是每日為單位，所以 todo/done 或一些 history、log 的紀錄資料，可以用日期為路徑來區隔

## 參數化設定
整個工作流程應該會要有一些可設定的參數被管理，方便將工作流運行在不同地方

目前我已經可能會需要可以參數化的設定：
- slack 通知的 channel
- slack token or webhook
- 本地的專案路徑
- github 的 repo

請再仔細計畫補充還有別的參數，並且建立一個設定參數的方法

# 整體目標
請依據上面提出的工作需求，設計整個工作流程的代理團隊
請一步一步指導我或，建立 CLAUDE.md 作為 Schema 檔案以及規劃任何所需的文件與配置
並且深入應用 AI Agent 的各項機制來完善整個代理團隊，且讓它是一個可持續強化、增強能力的服務

包括像是：
設計所需的 agent 或 skill 來完成整個開發工作，例如

- github issue tracker
- nextrek rails developer

建議安裝/建立的 skill 來提高工作的品質，例如
- 提高 rails 相關開發技能
- 提高/定義 commit 內容品質

若有任何問題或不確定之處，我們可以持續討論規劃
一步一步完成


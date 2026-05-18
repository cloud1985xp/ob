來做個 Ishin Engineer 分身吧

藍圖：想像它能做什麼、怎麼做

啟動時/定時，檢查 todo -> 需要整理一份屬於它的工作清單 (從 youtrack 產生？)

我要設計一個以 LLM Agent 來代理我的日常工作的工作流
請與我討論後，設計出這個工作流的 CLAUDE.md Schema 
以及規劃出可能需要建立的 skill

目標滿足以下的工作行為、任務項目與需求

## 工作任務內容
- 多個不同專案的開發項目 issue，可能是依需求功能開發，或修正 bug
- 執行應用程式部署
- 執行一些 Jenkins pipeline
- 多種維運伺服器的 devops 任務
- 定期 merge 新版本程式並解衝突
- 調查 jenkins 上 job 執行失敗的原因

## 知識管理
不同的任務、問題，可能會需要參考不同的程式碼或文件
要先對問題作判斷，例如
- 屬於 masterdata 資料面的問題
- 屬於 server 程式碼的問題
- 屬顧 client 
- 屬於調查 log 方面的任務

有關知識庫、任務判斷的迴路，要可以被維護、累積
而且 Agent 自己也會將遇過修正的經驗收集回到知識庫


## 工作流：

每日(早上)執行
有一個 agent，先將各項工作來源，收集成一份本日工作事項
並且擬定工作計畫，讓我先 review，我可以追加、刪除或調整項目
並且討論區分哪些項目要由 agent 代理執行
產生最後的工作計畫

工作項目來源可以包括
- 程式開發項目：來自多個不同 GitHub repository，被有特定 label 或篩選條件的 issues
- 查詢我的 Google Calendar，收集一些特定名稱的事件，如會議、進行 server 部署
- 其他我可能透過另外製作自動化工具收集的待辦事項清單

收集好工作項目後，整理成結構化的資料，例如分類、需要的 skill、工作優先度之類的，是否能交由 agent 執行等等

然後建立 agent 的工作計畫
- 找出確定可由 agent 執行的項目
- 規劃 sub agents 來進行工作
	- 規劃作法、目標、驗收方式
- 擬定 plan 好紀錄下來，並傳送到 slack 指定的 channel

由我確認計畫沒問題後
開始執行
這時照計畫啟動 subagents 執行的對應負責的項目
subagent 開始解決工作項目，過程中會把所有紀錄在指定的 daily work note 檔案中

subagent 過程中若遇到任何問題，會停止下來向我確認
向我確認得到解答或修正指示時，會紀錄下修正的內容與知識在指定的目錄，作為下次工作的參考

subagent 工作完成後，會將結果紀錄在指定的 daily work note
最後主 agent 將所有 subagent 的工作結果紀錄，匯整成當日的報告


每日(下班前)執行復盤
grill on me 詢問今日的工作復盤
一、詢問一些固定的問題，例如
- 是否有開早會
- 是否有開版更組會議

例行項目的 reference 清單

| 名稱  | 預設時間 | 預設分類 |
| --- | ---- | ---- |
| 早會  | 0.5h | 例行會議 |


二、review 今日執行項目
針對今日執行的項目(包括我 與 agent 的項目) 逐一進行 review
包括：
摘要工作進度或成果
填入花費的時間

三、紀錄工作關鍵改善項目
有什麼工作是勞務性質(toil)、有改善空間可以自動化或是委派給 agent 執行的？

有哪些要特別追蹤的類型工作有出現，有沒有確實記錄下來，包括工作時數

定義特別追蹤類型清單


四、將所有項目匯整成資料庫
後產生一份當日的工作成果報告


五、更新工時紀錄表
在指定的 googlesheet 上，填寫工時紀錄表
包括：工作項目名稱、花費時間
並且選擇工作項目的分類
工作項目分類，在 googlesheet 上的欄位的選單中選擇適合的分類
可以參考過去的選擇，以及用一分 reference 文件來說明該如何選擇

六、參數設定
整個工作流中有一些數值、資料，應該要被規劃可設定的參數
例如會用到的：
- GOOGLE_OAUTH_REFRESH_TOKEN
- SLACK_WEBHOOK_URL
- USER_NAME
等等

定義在 CONFIG.md，並不會進入 git 版本控管(提供 CONFIG.example.md)




## 每月工作
- 確認 gcp / server cost


待處理/需要解決的問題
- 可解決的

- 不確定能否解決的

# Discussions
- 技術規格？工作流程？什麼東西要放進 AGENTS.md / CLAUDE.md
- CLAUDE.md vs SessionStart Hook

## Workflow Orchestration  
  
### 1. Plan Mode Default  
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)  
- If something goes sideways, STOP and re-plan immediately - don't keep pushing  
- Use plan mode for verification steps, not just building  
- Write detailed specs upfront to reduce ambiguity  
  
### 2. Subagent Strategy  
- Use subagents liberally to keep main context window clean  
- Offload research, exploration, and parallel analysis to subagents  
- For complex problems, throw more compute at it via subagents  
- One tack per subagent for focused execution  
  
### 3. Self-Improvement Loop  
- After ANY correction from the user: update `tasks/lessons.md` with the pattern  
- Write rules for yourself that prevent the same mistake  
- Ruthlessly iterate on these lessons until mistake rate drops  
- Review lessons at session start for relevant project  
  
### 4. Verification Before Done  
- Never mark a task complete without proving it works  
- Diff behavior between main and your changes when relevant  
- Ask yourself: "Would a staff engineer approve this?"  
- Run tests, check logs, demonstrate correctness  
  
### 5. Demand Elegance (Balanced)  
- For non-trivial changes: pause and ask "is there a more elegant way?"  
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"  
- Skip this for simple, obvious fixes - don't over-engineer  
- Challenge your own work before presenting it  
  
### 6. Autonomous Bug Fixing  
- When given a bug report: just fix it. Don't ask for hand-holding  
- Point at logs, errors, failing tests - then resolve them  
- Zero context switching required from the user  
- Go fix failing CI tests without being told how  
  
## Task Management  
  
1. **Plan First**: Write plan to `tasks/todo.md` with checkable items  
2. **Verify Plan**: Check in before starting implementation  
3. **Track Progress**: Mark items complete as you go  
4. **Explain Changes**: High-level summary at each step  
5. **Document Results**: Add review section to `tasks/todo.md`  
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections  
  
## Core Principles  
  
- **Simplicity First**: Make every change as simple as possible. Impact minimal code.  
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.  
- **Minimat Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

Hooks
SessionStart
PreToolUse
PermissionRequest

claude --worktree
搭配 WorktreeCreate hook



/branch

/btw

claude --bare
claude --add-dir

or 
/add-dir

or
additionalDirectories in settings.json

claude --agent=<name>

to use .claude/agents/name.md

/voice

/loop
/schedule




來做個 Ishin Engineer 分身吧

藍圖：想像它能做什麼、怎麼做

啟動時/定時，檢查 todo -> 需要整理一份屬於它的工作清單 (從 youtrack 產生？)


TODO:
- 增強 debug knowledge 能力
	- 可能要參考最近的 程式 變動


增加新的 skill: sprint-meeting
我要用這個 skill 來進行 task planning
來安排團隊的 next sprint 的工作方向與項目
讓 agent 可以協助我與團隊擬定計畫

包括：
1. 確認接下來的 sprint 編號(ex: Sprint 74)、日期(預設長度是隔日開始的 2週)
2. 目前正在處理的 version merge 有哪些
	- 各別的版本，在接下來的 sprint 日期範圍+1週內，有什麼重大 milestone 要達成
		- 收集進工作計畫
		- 建立至 ishin-server github issue
3. 檢查下 sprint 範圍內，是否有要處理的 CS GDPR 安件
4. 檢查下個 sprint 範圍內，是否有重要需要 scale 的 event


# Report Server Diff

請用在 server 分類目錄下的 server-engineer-assistant plugin 下建立一個新的 skill: `tw-ishin-generating-server-diff`

用述是為了對指定的 ishin-server repo branch 進行 git diff 並將結果產生 diff report
實際情境會是直接在專案目錄下，運行這個 skill
專案目錄位置，例如：/Users/aaron.kuo/aktsk/ishin-tw/ishin-server

關鍵參數為:
- target branch，例如 merge/v4.6.0-gl ，預設為 develop
- base branch，例如 merge/v6.2.0-gl ，預設為 master

比較 diff 行為包括

- 先用 git diff --name-only 找出 diff 檔案清單
- 用以下參考規則對 diff 檔案目錄作 priority 分類

包括：

| Prioity | Directories                                                        | explain? | Remark                           |
| ------- | ------------------------------------------------------------------ | -------- | -------------------------------- |
| -1      | db/fixtures, config/globalization                                  | no       | 可完全忽略                            |
| 1       | apps/models, apps/controllers, db/migrations, config/jp, config/gl | yes      | 主程式、主設定和 migration               |
| 2       | config                                                             | yes      | 其他 config                        |
| 3       | apps/views,                                                        | no       | view                             |
| 4       | lib/tasks, db, .github                                             | yes      | github workflows, task 與其他 db 檔案 |
| 5       | lib/check_masters, admin/, admin_lib, admin_init_lib, app/assets   | no       | admin 與 check_master tool 相關     |
| 6       | spec                                                               | no       | spec                             |
| 7       | 其他檔案                                                               | no       |                                  |
將上述規則製作成可維護的 skill reference，讓未來可以持續調整更新

依 priority 分類後
將各 diff 檔案，製作成一個 html 網頁，可以像 github 一樣瀏覽 diff 樣式
可以將需要的 diff 樣式頁面產生製作成 script 
頁面樣式要可以：

- 用 priority 先分類，每個分類要簡述包含的檔案(即上述 reference 的目錄規則)
- 瀏覽 priority 分類，會顯示 file 的 differences，分類下也會展開 file 清單，點擊指定的 file 會移動頁面捲軸到該 file 的 diff 顯示區塊

建議可將 diff 分類結果先存到一個暫存目錄
再將該目錄路徑當作參數傳給 script 寫好的程式來生成 html 網頁
這樣我們可以用同一份結果來不斷測試 網頁生成的效果，後續討論與修改

此外，對 explain = yes 的分類，請分析理解這些程式碼的變動內容，產生程式變動的報告：
- 先將相關的變動程式內容分組，
- 依分組產生分析報告
- 將報告的路徑當作生成 html 的參數，當執行生成 html 網頁時，將對應的報告內容也放進生成的 html 網頁中
	- 允許不給報告路徑的參數，即略過嵌入報告的部分


請詳細理解需求，製定實作計劃
並用 /skill-creator ，將這個 skill 建立出來
若有任何不確定的問題或建議，請提出討論



# Collect-Scheduled Task

我想優化 collect-tasks skill 中從 custom markdown todo 抓取任務的功能
可以在 markdown 中有一個區塊是定義排程式的工作
讓每次在進行 collect-tasks 時會去檢查當天有沒有符合排程設定的任務項目要進行
例如每個月的第一天(workday)，執行某項任務(例如 gcp cost report)

請將這個機制加進 collect-tasks skill，
並在目前的 custom_todo_path file( ~/Documents/today-todo.md) 加上排程任務的範例

# 解決 version merger 上遊更新問題

version merger 的上遊 branch (upstream_branch)，有可能是指向 branch 或 tag name
而且 tag 也有可能被更新(指向更新的 commit)，目前檢查 upstream 的變動，似乎不會追蹤到更動後的 tag，請檢查並修正對應的步驟，讓：
- orchestrator 在檢查 no-ops 前，能先確保 upstream 是最新的
- agent 自己單獨在執行時，能先確保 upstream 是最新的

# 優化 Tool Engineer

我要確保 Tool Engineer Agent
在收到 dispatch task 會依循以下的行為：

當工作任務來源是 github issue 且 label 有 `feature` 時
先用 gh cli 從 github 上取得 task 的需求內容時，
- 先查看本地對該 issue 的 history 紀錄，了解目前對該 issue 處理的狀況
	- 最後處理的時間
	- 是否已建立 pr
- 查看 issue 與 pr(如果已建立) 自上次處理之後的 comments 討論，
	- 若沒有 history 與 pr 紀錄，就從 issue 取得所有內容
- 從上述取得的內容得知 task 資訊，包括
	- base branch
	- 需求與目標
	- issue 的 labels 是否有 `epic`

從 task 對應的 repo 路徑
- 從 base branch 建立 worktree 作為工作空間
- 進到 worktree 工作空間
	- 載入該工作空間下的 CLAUDE.md 在該專案下進行開發
	- 如果 labels 包含 `epic` 時，agent 必須使用 `/brainstorming` skill 進行開發
	- 若沒有，則主動依需求內容判斷是否要用 `plan mode` 

當工作任務來源不是 github issue 時，則先暫停，詢問 user 該如何處理

請將上述行為更新到 tool-engineer agent ，如果需要的話，也更新對應的 knowledge



in agent-plugins

# 增加 monthly GCP cost report Skill

我需要在每個月初(例如六月)時執行，
檢視上個月份(例如五月)的 gcp 帳單
並且再跟前一個月(例如四月)比較各項費用的變化量，以及判斷變化量是否合理
綜合成一個摘要報告

可將每個月的處理結果，存成 history data，方便每個月與之前的資料做比較

帳單會包括多個專案(project id)
- ishin-168508
- ishin-tw-dev
- ishin-tool-165703
- rosetta-284004

每個專案各自做與前一個月的分析比較與摘要報告

除此之外
要對 ishin-168508 專案的 BigQuery 用量做進一步的檢查

包括：
一、從帳單資料
統計每日的費用，是否有特別異常的變化

二、直接從 BigQuery JOBS_BY_PROJECT 檢查用量
參考以下 SQL
```
SELECT ROUND(SUM(total_bytes_processed)/1024/1024/1024, 2) AS processed_GB,

ROUND((SUM(total_bytes_processed)/1024/1024/1024/1024)*5, 2) AS USD,

SUM(total_bytes_processed) AS total_bytes, COUNT(*) AS total_queries

FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_PROJECT

WHERE
DATE(creation_time) BETWEEN "2026-04-01" AND "2026-04-30"
AND project_id = 'ishin-168508'

GROUP BY user_email
ORDER BY total_bytes desc
```

來統計每個 user 的使用量
並比較過去(最多三個月)的紀錄，看使用量是否有異狀

請先評估所需的工具指令，例如 gcloud、bq 是否能完成上述目標
並規劃將整個流程製作成一個 skill： gcp-cost-report


從 BigQuery JOBS_BY_PROJECT 的部分
除了專案 ishin-168508 之外
請也加上 project_id = "ishin-tool-165703" 



# 改良 version merge Agent
我要強化 version merge agent 的工作流
讓 version merge 時，可以支援同 repo 但處理多個版本
而且可以使用 git worktree 來建立目標工作路徑

在處理 version merge 工作時，有可能同一個 repo (ex: ishin-server) 在進行多個版本
例如同時處理 v6.4.0 與 v6.4.5，所以：

version_merge_targets 裡要變成可以定義兩組 ishin-server repo merge 工作的內容
例如：

```
version_merge_targets:
  - name: ishin-server
    work_path: /Users/aaron.kuo/aktsk/ishin-tw/ishin-server-develop
    upstream_remote: jp
    upstream_branch: v6.4.0
    tw_base: develop
    tw_branch: merge/v6.4.0-gl
    target_version: v6.4.0
    forward_merge_branches: [gogeta]
  - name: ishin-server
    worktree_path: /Users/aaron.kuo/aktsk/ishin-tw/ishin-server-worktree
    upstream_remote: jp
    upstream_branch: pilaf
    tw_base: merge/v6.4.0-gl
    tw_branch: merge/v6.4.5-gl
    target_version: v6.4.5
    forward_merge_branches: [krillin]
```

並且加上：
一、可設定 tw_base branch
用來表示這個 version merge 在 tw 本地側的 base branch
例如 v6.4.0 是基於 develop，
所以一開始在 checkout 建立工作 branch 時，要從 develop checkout 出 merge/v6.4.0-gl
而上述的例子 v6.4.5 是基於 merge/v6.4.0-gl checkout 出 merge/v6.4.5-gl

- agent 每次開始 merge upstream version 前，會先檢查 base branch 有沒有新內容，有的話會先將 base branch 先 merge 進工作 branch，解衝突，再開始 (呼叫 potara skill) 進行 version merge
- base branch 有可能會變動，例如過了一個月，merge/v6.4.0-gl 已完成 merge 進 develop branch後，v6.4.5 的 base branch 就會被改成 develop，那 agent 開始 merge version 前，就變成檢查 develop(更改後的 base branch) 是否有新內容
- 只有一開始是從 base branch checkout 出 version merge 工作用的 branch，後續就是一直檢查 base branch 有無新內容要 merge 進 工作 branch

二、支援用 worktree 模式
我希望在  version_merge_targets 裡，可以設定 worktree_path 來啟用 worktree 模式
當該次 version merge target 有設定 worktree_path 時，
agent 就會用 worktree 的模式來建立/決定該次 version merge 的工作空間

決定目標路徑： {worktree_path}/{target_version} 
例如：/Users/aaron.kuo/aktsk/ishin-tw/ishin-server-worktree/v6.4.5

agent 工作時先檢查 worktree 的工作路徑是否存在
若不存在，要先建立 worktree 工作路徑
從 {tw_base} branch checkout 建立 {tw_branch} 在 worktree 目標工作路徑

若已存在，就直接到 worktree 工作路徑下繼續 merge 工作

- work_path 和 worktree_path 只會設定其中一項

請幫我更新 version merge agent 的 schema，加上上述的行為能力
且要確保原本的既有能力仍正確運行
若有任何疑問或建議請和我討論確認

我注意到一件事
同個 repo 的多個 version merge 可能會有相依性
例如 v6.4.0 會是 v6.4.5 的 base branch，所以 merge 工作必須要有序順性
例如先處理 v6.4.0 後再處理 v6.4.5 (要merge base)
所以在做 agent dispatch 時，同個 repo 的多個 version_merge_targets
- 要依照 version_merge_targets 裡的順序
- 不能平行 dispatch (如果未來使用平行的方式委派工作給 agents 的話)
這部分可能要紀錄在 orchestrator 的行為裡

# 區分 version-merge and tooling

我想將工作項目(task domain) `server` , `version-merge` 做進一步補充

一、server -> tooling、tool development
工具程式的開發/維護，包括像是 rosetta、ishin-tool、polunga 等 repo
未來可能還會增加，

可能改叫 tooling 或 tool-development

二、version merge
這部分主要都是 ishin 這個專案相關的工作，包括
版本的更新，會去 merge 上遊(另一個 jp 版本的 repo)的程式
像是 ishin-server、ishin-analysis、ishin-cookbooks 的 repo 都會是這類的工作對象 repo

對於 version merge 的對象 repo，也要有獨立的 repo 路徑，定義在 CONFIG.md 裡
在處理 version merge 的 knowledge 裡，
agent 應該依要處理的對象(server 、analysis or cookbook) 不同
依照 CONFIG.md 的設定到不同的 repo 工作





要工作內容是

## 增加 REPO_LIST 在 CONFIG.md 裡

整個工作流會使用到多個 repositores
我想直接在 CONFIG.md 裡定義好清單，包括

repo name, alias names, local path

這樣在工作流中或 CONFIG.md 裡的其他地方，
可以直接用 repo name 或 alias 來代表就好

## 收集程式開發紀錄
我想在每日復盤階段
增加一個動作來收集我的程式開發工作行為紀錄
經由：
- neovim 編輯器的 log
- 各個 repo 的 commit 紀錄
收集這些資訊，來匯整今當日的工作紀錄
並做為 `Stage 2 — Review Today's Items` 內容的一部分

請把這個「收集程式開發工作行為」也做成一個 skill

並會從 CONFIG.md 裡來定義：
1、需要收集的 repo 清單
2、neovim 的 log 位置

結果產生一份 report，
並讓在 retrospective skill 裡也去參考這份資料

## 確認 update_worksheet Skill

請檢查並修改 update-worksheet 的 Skill
是否符和以下期望的行為來寫入 worksheet

我已設定了 CONFIG.md
一、請試試看是否能正確讀到我的 sheet

二、請閱讀目前的 worksheet 結構：
- 將目前有選取過的「分類」(B)，資料，更新到 references/sheet-categories.md 裡，這些是目前常用到的項目與相關的工內容

三、確認 skill 寫入的行為
當工作紀錄資料整理好後，skill 要進行更新時：
- 從 A 欄(日期)最後一列空白開始填寫當日的工作紀錄
- 會從即有的分類選擇適合的項目
- 這個寫入的行為，應該可以製作成 skill 的 scripts 腳本，方便直接呼叫執行





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




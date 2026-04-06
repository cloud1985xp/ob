---
title: "Claude Code 創始者 Boris Cherny 親自示範每天在用的 15 個功能，從排程自動化到說話寫程式 - INSIDE"
source: "https://www.inside.com.tw/article/40974-claude-code-boris-cherny-hidden-features-voice-scheduling-workflow-2026"
author:
  - "[[INSIDE 硬塞的網路趨勢觀察]]"
published: 2026-03-30
created: 2026-04-04
description: "Anthropic 旗下 AI 程式工具 Claude Code 的負責人 Boris Cherny，公開 15 項 Claude Code 進階功能，重點涵蓋跨裝置遠端作業、排程自動化與大規模並行處理。#人工智慧,開發者,自動化,跨裝置,AI 代理人,Claude Code (claude-code-boris-cherny-hidden-features-voice-scheduling-workflow-2026)"
tags:
  - "clippings"
---
Anthropic 旗下 AI 程式工具 Claude Code 的負責人 Boris Cherny，今日以連串推文形式公開 15 項他個人最倚重、卻普遍被低估的功能。推文發布後數小時內達到逾 56 萬次瀏覽，在開發者社群掀起廣泛討論；Cherny 直言，這些功能早已內建於產品中，多數開發者只是不知道它們存在。

### 跨裝置連續工作，不在電腦前也能下令

Cherny 打破了許多人對 Claude Code 是「終端機工具」的既定印象，第一個提出的功能便是行動 app。他表示，個人相當大比例的程式修改直接從 iPhone 完成，無需打開筆電。使用者只需下載 iOS 或 Android 版的 Claude app，切換至左側 Code 分頁，即可存取雲端工作階段。

跨裝置的能力遠不止於此， `--teleport` （或工作階段中輸入 `/teleport` ）可將雲端工作階段直接拉回本機繼續執行； `/remote-control` 則讓使用者從手機或任何瀏覽器即時操控本機正在進行的工作階段。Cherny 透露，他已在 `/config` 中開啟「Enable Remote Control for all sessions」，確保每個工作階段都預設可以遠端接手。

Cowork Dispatch 則是他另一個每日必用的工具，Dispatch 作為 Claude Desktop app 的安全遠端控制介面，可在使用者授權的前提下調用 MCP 連接器（模型上下文協定伺服器）、操控瀏覽器，乃至直接操作電腦。Cherny 說，不在寫程式的時候，他就在用 Dispatch 處理 Slack 訊息、管理電子郵件與整理檔案。

三個功能疊加起來，Claude Code 的產品定位已悄然轉移；它不再只是一台電腦上的終端機工具，而是一套可在多裝置間持續運行的自主代理人平台。

### 排程、並行、自動驗證，讓 Claude 在你不在時繼續工作

排程功能是 Cherny 本人點名「Claude Code 最強功能之一」的一組指令。 `/loop` 與 `/schedule` 讓使用者設定 Claude 以固定間隔自動執行任務，最長可排程一週。Cherny 自己就在本機持續跑著多個 loop： `/loop 5m /babysit` 每 5 分鐘執行一次，自動處理 code review 回饋與 PR rebase； `/loop 30m /slack-feedback` 則每 30 分鐘自動將 Slack 上的使用者回饋整理成 PR。他說，這類「把工作流打包成 loop」的思維，是 Claude Code 最值得深度探索的方向之一。

hooks 讓開發者在 Claude Code 工作週期的特定節點注入確定性邏輯，Cherny 給出了四個具體用途：

- 工作階段啟動時（SessionStart），自動將專案上下文載入，讓 Claude 每次開工都有完整的背景資訊，無需重複說明。
- 每次模型執行 bash 指令前（PreToolUse），自動記錄操作日誌，完整追蹤 Claude 的每一個動作。
- 遇到需要使用者確認的授權請求時（PermissionRequest），自動將通知轉發至 WhatsApp，讓開發者直接在手機上批准或拒絕，不必守在電腦前。
- Claude 停下來等待時（Stop），自動 poke 它繼續執行，讓工作流程不中斷。最後這個用法的意義尤其深遠：開發者幾乎可以讓 Claude 在完全無人看管的狀態下持續推進任務。

並行能力方面，Claude Code 對 git worktrees 提供原生深度支援，透過 `claude --worktree` 指令（或 Claude Desktop app 中的 worktree 選項），使用者可在同一個 repo 中開啟多個完全獨立的工作樹，讓每個 Claude 代理人在自己的環境中作業、互不干擾。Cherny 表示自己隨時都有幾十個 Claude 在跑，worktrees 是實現這件事的核心基礎設施。非 git 版本控制系統的使用者，也可透過 `WorktreeCreate hook` 自訂 worktree 建立邏輯。

大規模任務則交給今年 2 月底才正式推出的 `/batch` 。這個指令會先對使用者提問以釐清需求，再自動拆解工作並同時開啟數十、數百甚至數千個 worktree 代理人並行執行。Cherny 給出的典型場景是大型程式碼遷移，例如直接下指令「 `/batch migrate from jest to vite` 」，讓 Claude 自行協調整個遷移過程，無需人工逐檔處理。

### 工具清單直指開發者痛點，每一項都已可用

Chrome 擴充套件的核心邏輯，Cherny 以一個工程師的比喻說明：如果你請一個工程師建網站，但不讓他開瀏覽器，成果會好嗎？大概不會。給他瀏覽器，他就會不斷測試、迭代，直到結果正確，而Claude Code 的 Chrome 擴充套件（目前為 beta 版）正是把這個驗證能力還給 Claude，讓它能直接操控瀏覽器、讀取 console log、測試 UI 互動。

Claude Desktop app 則在同樣邏輯下，進一步整合了自動啟動 web server 並以內建瀏覽器測試的能力，讓本機開發的完整流程都可以在 Claude 中閉環完成。CLI 或 VS Code 使用者也可透過 Chrome 擴充套件達成類似效果。

`/branch` （或 CLI 的 `claude --resume  --fork-session` ）解決了「改壞了回不去」的心理負擔，讓使用者從同一工作節點分叉出不同嘗試方向，隨時可切回原本主線。 `/btw` 則讓使用者在代理人仍在執行任務時，插入快速旁支問答而不中斷主工作流程。

`--bare` 旗標針對 SDK 整合場景。以 `-p` 模式或 TypeScript、Python SDK 非互動式呼叫時，加上 `--bare` 可跳過 CLAUDE.md、設定檔、MCP 等的自動載入，Cherny 說這可將 SDK 啟動時間縮短最多 10 倍，對批次處理與 CI 流程有實質影響。

`--add-dir` 解決的是跨 repo 協作的摩擦。Cherny 的習慣是在主要 repo 啟動 Claude，再以 `--add-dir` （或工作階段中的 `/add-dir` ）加入其他 repo 的目錄。他強調，這個指令不只是讓 Claude「看到」其他目錄，還同步賦予它在該目錄內操作的完整權限。對團隊而言，可將 `additionalDirectories` 寫入共用的 settings.json，讓所有成員啟動時自動載入相同配置。

`--agent` 旗標允許開發者為 Claude Code 套入自訂系統提示與工具限制，建立行為完全不同的子代理人。使用方式是在 `.claude/agents` 目錄中定義代理人的 Markdown 檔案，再以 `claude --agent=<名稱>` 啟動。Cherny 示範的 `ReadOnly` 代理人僅開放讀取工具、無法編輯檔案或執行 bash 指令，適合需要安全審查的場景。

最後是第 15 項語音輸入（ `/voice` ）。Cherny 親口確認，他日常以說話為主、鍵盤為輔。在 CLI 執行 `/voice` 後按住空白鍵，或點擊 Desktop 版的語音按鈕即可啟用；iPhone 使用者則可在系統設定中開啟聽寫功能達到同樣效果。

### 這份清單的背後，是一個正在轉型的開發工具

Cherny 在 3 月 9 日公開的數據 [顯示](https://x.com/bcherny/status/2031089411820228645) ，Anthropic 內部工程師今年以來的程式碼產出已成長逾 200%，而 Code Review 功能正是因為「審查速度跟不上產出速度」才被開發出來。稍早，研究機構 SemiAnalysis 估計 Claude Code 已佔 GitHub 公開提交量的 4%，並預測年底可能突破 20%。

此次主動整理這份功能指南，顯示 Anthropic 正有意識地引導使用者從「會用 Claude Code」升級到「真正發揮其自動化潛能」。從行動端遠端控制到數千個並行代理人、從排程自動化到語音驅動的工作流，15 項功能合在一起描繪的，是一套以 AI 代理人為核心的新工作作業系統。

責任編輯：Claire

核稿編輯：Mia

本文初稿由 INSIDE 使用 AI 協助編撰，並經人工審校確認； **加入 INSIDE 會員，獨享 INSIDE 科技趨勢電子報，** [**點擊立刻成為會員**](https://member.inkmaginecms.com/?console_id=1&client_id=9da69691-a221-4a2b-945c-c845c9999070&redirect_uri=https%3A%2F%2Fwww.inside.com.tw%2Fmember%2Fprocess) **！**

延伸閱讀：

- [AI 真的能幫你做決定？Claude Code 推出 Auto 模式自動判斷風險，不用一直按「允許」](https://www.inside.com.tw/article/40934-claude-code-auto-mode-permission-classifier-research-preview)
- [Claude Dispatch 完整攻略：六個場景、三大優勢、跟龍蝦的關鍵差異](https://www.inside.com.tw/article/40891-assign-tasks-to-claude-from-anywhere-in-cowork)
- [Claude Code 創辦人：用 AI vibe coding 仍有極限](https://www.inside.com.tw/article/40308-claude-code-creator-vibe-coding-limits)
---
title: "Claude Code 創始者 Boris Cherny 親自示範每天在用的 15 個功能"
type: source
tags: [claude-code, features, automation, mobile, parallel, voice]
sources: [2026-03-30_boris-cherny-15-features.md]
updated: 2026-04-06
---

# Boris Cherny 親示每天在用的 15 個 Claude Code 功能

- **來源**：[INSIDE 硬塞的網路趨勢觀察](https://www.inside.com.tw/article/40974-claude-code-boris-cherny-hidden-features-voice-scheduling-workflow-2026)
- **作者**：INSIDE 硬塞的網路趨勢觀察
- **發布日期**：2026-03-30

## 核心摘要

[[Boris Cherny]] 透過連串推文公開 15 個「普遍被低估」的 Claude Code 功能，推文數小時內達到 56 萬次瀏覽。這份清單揭示了 Claude Code 已從「終端機工具」演化為「多裝置自主代理人平台」——可在無人監管下持續工作。

## 15 項功能分類整理

### 跨裝置連續工作

- **行動 App**：iOS/Android Claude app 左側 Code 分頁，可直接從 iPhone 完成程式修改
- **`--teleport` / `/teleport`**：將雲端工作階段拉回本機繼續執行
- **`/remote-control`**：從手機或任何瀏覽器即時操控本機工作階段
- **Cowork Dispatch**：Claude Desktop app 的安全遠端控制介面，可調用 MCP 連接器、操控瀏覽器、操作電腦

### 排程與自動化

- **`/loop` 與 `/schedule`**：設定 Claude 以固定間隔自動執行任務，最長可排程一週
  - Cherny 實例：`/loop 5m /babysit`（每 5 分鐘處理 code review 與 PR rebase）
  - Cherny 實例：`/loop 30m /slack-feedback`（每 30 分鐘整理 Slack 回饋成 PR）
- **Hooks**：在工作週期特定節點注入確定性邏輯
  - `SessionStart`：自動載入專案上下文
  - `PreToolUse`：自動記錄每個 bash 操作日誌
  - `PermissionRequest`：授權請求自動轉發至 WhatsApp
  - `Stop`：自動戳 Claude 繼續執行（實現完全無人監管）

### 並行處理

- **Git Worktrees（`claude --worktree`）**：在同一 repo 開啟多個獨立工作樹，Cherny 表示自己隨時有幾十個 Claude 在跑
- **`/batch`**（2026-02 推出）：自動拆解工作並同時開啟數十至數千個 worktree 代理人，典型場景是大型程式碼遷移（如 `/batch migrate from jest to vite`）

### 開發工具

- **Chrome 擴充套件（beta）**：讓 Claude 直接操控瀏覽器、讀取 console log、測試 UI 互動
- **`/branch`（`--resume --fork-session`）**：從同一節點分叉不同嘗試方向，隨時切回主線
- **`/btw`**：代理人執行中插入快速旁支問答，不中斷主工作流
- **`--bare` 旗標**：SDK 整合時跳過 CLAUDE.md 等自動載入，啟動時間縮短最多 10 倍
- **`--add-dir`（`/add-dir`）**：加入其他 repo 目錄並賦予完整操作權限
- **`--agent` 旗標**：套入自訂系統提示建立行為不同的子代理人（`.claude/agents/` 目錄）

### 語音輸入

- **`/voice`**：按住空白鍵說話；Cherny 確認日常以說話為主、鍵盤為輔

## 關鍵數據

- Anthropic 內部工程師 2026 年以來程式碼產出成長逾 **200%**
- Claude Code 已佔 GitHub 公開提交量的 **4%**，SemiAnalysis 預測年底可能突破 **20%**

## 提及的實體

- [[Boris Cherny]] — 功能展示者，Claude Code 負責人
- [[Claude Code]] — 工具本體
- [[Anthropic]] — 開發公司
- [[Cowork Dispatch]] — Claude Desktop app 的遠端控制介面

## 提及的概念

- [[排程自動化（Loop & Schedule）]]
- [[Hooks 機制]]
- [[Worktrees 並行處理]]
- [[子代理策略]]
- [[批次處理（/batch）]]

---
title: "Claude Code"
type: entity
tags: [tool, anthropic, ai, coding-assistant]
sources: [2026-02-24_claude-code-workflow-detools, 2026-03-30_boris-cherny-15-features, 2025-06-09_how-anthropic-teams-use-claude-code]
updated: 2026-04-06
---

# Claude Code

[[Anthropic]] 開發的 AI 程式工具，最初定位為終端機 CLI 工具，現已演化為**多裝置自主代理人平台**。

## 使用介面

- **CLI（終端機）**：最原始的使用方式
- **Claude Desktop App**：整合瀏覽器測試、worktree 管理、Dispatch 遠端控制
- **VS Code 擴充套件**
- **iOS / Android App**：左側 Code 分頁，支援雲端工作階段
- **Chrome 擴充套件（beta）**：讓 Claude 直接操控瀏覽器

## 核心功能分類

### 配置與記憶

- `CLAUDE.md`：放在專案根目錄，定義 Claude 的行為規範與專案規則；見 [[CLAUDE.md 最佳實踐]]

### 排程與自動化

- `/loop`、`/schedule`：定時自動執行任務，最長一週；見 [[排程自動化（Loop & Schedule）]]
- `Hooks`：在工作週期特定節點注入確定性邏輯；見 [[Hooks 機制]]

### 並行處理

- `--worktree`（`claude --worktree`）：在同一 repo 開多個獨立工作樹
- `/batch`（2026-02 推出）：自動拆解並同時開啟數千個代理人並行；見 [[Worktrees 並行處理]]

### 跨裝置

- `--teleport` / `/teleport`：將雲端 session 拉回本機
- `/remote-control`：從任何裝置遠端操控本機 session
- Cowork [[Dispatch]]：Claude Desktop 的安全遠端控制介面

### 代理人架構

- `--agent` 旗標：套入自訂系統提示，建立行為不同的子代理人（定義於 `.claude/agents/`）
- `--add-dir`：加入其他 repo 目錄並賦予操作權限
- `--bare`：SDK 非互動式呼叫時跳過配置載入，啟動加速最多 10 倍

### 工作流輔助

- `/branch`（`--resume --fork-session`）：從節點分叉不同嘗試
- `/btw`：執行中插入旁支問答不中斷主任務
- `/voice`：語音輸入

## 市場數據（2026-03）

- 佔 GitHub 公開提交量：**4%**
- SemiAnalysis 預測年底：**20%**
- Anthropic 內部工程師產出成長：**200%+**

## 出現在以下來源

- [[sources/2026-02-24_claude-code-workflow-detools]]
- [[sources/2026-03-30_boris-cherny-15-features]]
- [[sources/2025-06-09_how-anthropic-teams-use-claude-code]]

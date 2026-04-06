---
title: "CLAUDE.md 最佳實踐"
type: concept
tags: [claude-code, configuration, workflow, best-practices]
sources: [2026-02-24_claude-code-workflow-detools, 2025-06-09_how-anthropic-teams-use-claude-code]
updated: 2026-04-06
---

# CLAUDE.md 最佳實踐

`CLAUDE.md` 是放置於專案根目錄的設定檔，作用是給 [[Claude Code]] 的「長期記憶」——定義行為規範、工作流程、以及專案相關的所有規則。每次啟動新 session 時 Claude 會自動讀取它。

## 核心功能

- 跨 session 保持一致的工作標準
- 讓 Claude 記住過去犯的錯誤（透過 `tasks/lessons.md`）
- 定義任務管理流程（plan mode、驗證流程等）
- 說明使用者角色與偏好（特別是非技術用戶）

## [[Boris Cherny]] 建議的內容框架

### Workflow Orchestration（工作流程編排）

1. **Plan Node Default** — 非瑣碎任務一律先進 plan mode
2. **Subagent Strategy** — 大量使用子代理保持主 context 乾淨；見 [[子代理策略]]
3. **Self-Improvement Loop** — 被糾正後立即更新 `tasks/lessons.md`；見 [[自我改善迴圈]]
4. **Verification Before Done** — 未證實可運行前不標記完成
5. **Demand Elegance** — 停下來問「有沒有更優雅的方式？」
6. **Autonomous Bug Fixing** — 直接修 bug，不需要手把手

### Task Management（任務管理）

- 計畫寫入 `tasks/todo.md`（含可勾選項目）
- 執行中標記進度
- 完成後加 review 段落
- 糾正後更新 `tasks/lessons.md`

### Core Principles（核心原則）

- Simplicity First、No Laziness、Minimal Impact

## 非技術用戶的使用方式

根據 [[sources/2025-06-09_how-anthropic-teams-use-claude-code]]，Anthropic 設計團隊的做法：
> 在 CLAUDE.md 中說明：「我是設計師、程式經驗有限，需要詳細說明與小步驟變更」

這能讓 Claude 自動調整解釋深度和操作粒度。

## 相關概念

- [[子代理策略]]
- [[自我改善迴圈]]
- [[計畫模式（Plan Mode）]]

## 出現在以下來源

- [[sources/2026-02-24_claude-code-workflow-detools]] — 完整框架定義
- [[sources/2025-06-09_how-anthropic-teams-use-claude-code]] — 設計師使用案例

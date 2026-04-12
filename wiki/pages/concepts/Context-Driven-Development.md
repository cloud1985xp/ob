---
title: "Context-Driven Development"
type: concept
tags: [ai-tools, workflow, context, planning, claude-code, gemini-cli]
sources: [2025-12-17_conductor-gemini-cli, 2026-02-24_claude-code-workflow-detools]
updated: 2026-04-07
---

# Context-Driven Development

把 AI 代理人的「記憶」與「專案規範」**持久化到檔案系統**，確保每次 session 都有一致的上下文，而非依賴易消失的聊天記錄。

## 兩種實作的比較

| 面向 | CLAUDE.md（Anthropic） | Conductor（Google） |
|------|----------------------|---------------------|
| 平台 | [[Claude Code]] | [[Gemini CLI]] |
| 核心文件 | `CLAUDE.md` | specs + plan Markdown 檔案 |
| 計畫模式 | Plan Node Default | `/conductor:newTrack` |
| 進度追蹤 | `tasks/todo.md` | `plan.md` 勾選 |
| 上下文範疇 | 行為規範 + 任務管理 | 產品目標 + 技術棧 + 工作流 |

## 核心哲學

> 「Failing to plan is planning to fail.」
> 在 AI 時代，我們常直接跳入實作，而不先建立對目標的清晰認識。

把上下文作為**受管理的成品**（managed artifact）——與程式碼並存於 repo，讓 repo 成為唯一的真相來源。

## 這個 Wiki 本身就是 Context-Driven Development 的應用

- `CLAUDE.md` = Schema（行為規範）
- `index.md` = 目錄（讓 Agent 快速找到相關頁面）
- `log.md` = 歷史紀錄
- `pages/` = 持久化的知識成品

## 相關概念

- [[CLAUDE.md 最佳實踐]]
- [[AI 代理人工作流]]

## 出現在以下來源

- [[sources/2025-12-17_conductor-gemini-cli]] — Google 的實作
- [[sources/2026-02-24_claude-code-workflow-detools]] — Anthropic 的實作

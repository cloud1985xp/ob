---
title: "Hooks 機制"
type: concept
tags: [claude-code, hooks, automation, deterministic]
sources: [2026-03-30_boris-cherny-15-features]
updated: 2026-04-06
---

# Hooks 機制

[[Claude Code]] 允許開發者在工作週期的特定節點注入「確定性邏輯」的機制。與 LLM 的不確定性不同，Hooks 是固定執行的腳本，確保某些行為一定發生。

## 四個主要 Hook 節點

| Hook 節點 | 觸發時機 | Cherny 建議用途 |
|-----------|---------|----------------|
| `SessionStart` | 工作階段啟動時 | 自動載入專案上下文，無需每次重複說明 |
| `PreToolUse` | 每次執行 bash 指令前 | 自動記錄操作日誌，追蹤 Claude 的每個動作 |
| `PermissionRequest` | 遇到需確認的授權請求時 | 自動轉發通知至 WhatsApp，手機批准或拒絕 |
| `Stop` | Claude 停下來等待時 | 自動戳 Claude 繼續執行，實現完全無人監管 |

## 關鍵洞察：`Stop` Hook 的意義

> `Stop` hook 讓開發者幾乎可以讓 Claude 在**完全無人看管**的狀態下持續推進任務。

這個設計配合 [[排程自動化（Loop & Schedule）]]，能建立真正的「無人值守工作流」。

## `PermissionRequest` Hook 的實用場景

將 Claude 請求授權的通知轉發至 WhatsApp，讓開發者可以：
- 不守在電腦前
- 直接在手機上批准或拒絕 Claude 的操作
- 維持安全性的同時不犧牲移動性

## 相關概念

- [[排程自動化（Loop & Schedule）]]
- [[Worktrees 並行處理]]

## 出現在以下來源

- [[sources/2026-03-30_boris-cherny-15-features]]

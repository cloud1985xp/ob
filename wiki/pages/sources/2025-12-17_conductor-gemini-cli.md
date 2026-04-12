---
title: "Conductor: Introducing context-driven development for Gemini CLI"
type: source
tags: [ai-tools, gemini-cli, context-driven, workflow, google]
sources: [2025-12-17_conductor-gemini-cli.md]
updated: 2026-04-07
---

# Conductor：Gemini CLI 的 Context-Driven Development

- **來源**：[Google Developers Blog](https://developers.googleblog.com/conductor-introducing-context-driven-development-for-gemini-cli/)
- **作者**：Keith Ballinger、Jay Kornder、Sherzat Aitbayev（Google）
- **發布日期**：2025-12-17

## 核心摘要

Conductor 是 [[Gemini CLI]] 的擴充套件，實作「Context-Driven Development」——把專案上下文從聊天視窗移到程式碼庫中的持久化 Markdown 檔案，讓 AI 代理人有一致的專案記憶與指引。

> ⚠️ 此概念與 [[CLAUDE.md 最佳實踐]] 高度相似，是 Google 對同一問題的解法。

## 核心哲學

**「Failing to plan is planning to fail」**（Benjamin Franklin）

與其讓 AI 直接跳入實作，不如先把意圖正式化（formalize intent）。把上下文作為「managed artifact」管理，讓 repo 成為唯一的真相來源。

## 三步工作流程

### 1. Establish Context（`/conductor:setup`）
定義三個核心組件：
- **Product**：使用者、產品目標、高層功能
- **Tech Stack**：語言、資料庫、框架偏好
- **Workflow**：團隊工作方式（例如 TDD）

### 2. Specify & Plan（`/conductor:newTrack`）
建立一個「track」（高層工作單元）：
- **Specs**：詳細需求——我們在建什麼？為什麼？
- **Plan**：可執行的 to-do 清單（Phases > Tasks > Sub-tasks）

### 3. Implement（`/conductor:implement`）
代理人根據 `plan.md` 逐步執行，勾選完成項目。**狀態存在檔案中，可以暫停再繼續**，甚至在中途編輯計畫。

## 與 CLAUDE.md 模式的比較

| 面向 | Conductor（Google） | CLAUDE.md（Anthropic） |
|------|--------------------|-----------------------|
| 平台 | Gemini CLI | Claude Code |
| 上下文存儲 | 結構化 Markdown（specs + plan） | CLAUDE.md + tasks/todo.md |
| 計畫模式 | 建立 track 前強制規劃 | Plan Node Default |
| 進度追蹤 | plan.md 勾選 | tasks/todo.md |

兩者都是「Context-Driven Development」的實作，核心思想相同：**把 AI 的記憶和規範持久化到檔案系統**。

## 提及的實體

- [[Gemini CLI]] — Google 的 AI 程式工具
- [[Google]] — 開發公司

## 提及的概念

- [[Context-Driven Development]]
- [[CLAUDE.md 最佳實踐]]
- [[AI 代理人工作流]]

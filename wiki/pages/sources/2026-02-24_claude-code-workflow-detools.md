---
title: "Claude Code 之父分享的內部工作流：一份 CLAUDE.md 讓你晉升 10 倍工程師"
type: source
tags: [claude-code, workflow, CLAUDE.md, best-practices]
sources: [2026-02-24_claude-code-workflow-detools.md]
updated: 2026-04-06
---

# Claude Code 之父分享的內部工作流

- **來源**：[DeTools 工具翼零](https://tools.wingzero.tw/article/sn/3788)
- **作者**：DeTools 工具翼零
- **發布日期**：2026-02-24

## 核心摘要

本文整理了 [[Boris Cherny]]（Claude Code 創始者）親授的 CLAUDE.md 終極設定模板，核心概念是透過 CLAUDE.md 賦予 Claude Code「記憶」與「行為規範」，讓 AI 在跨 session 間保持一致的工作標準，並能不斷自我修正與進化。

## 關鍵洞察

- **計畫優先**：任何非瑣碎任務（3 步以上）都要先進入 plan mode，遇到問題立即重新規劃，不要硬衝
- **子代理策略**：大量使用 subagents 保持主 context window 乾淨；複雜問題就投入更多 subagents
- **自我改善迴圈**：每次被使用者糾正後，立即更新 `tasks/lessons.md`，讓錯誤只發生一次
- **完成前驗證**：任務未被證實可運行之前不能標記完成；問自己「資深工程師會 approve 這個嗎？」
- **優雅要求**：非瑣碎改動要停下來問「有沒有更優雅的方式？」，但簡單修正不過度設計
- **自主 Bug 修復**：收到 bug report 就直接修，不問怎麼做；指向 log、錯誤、失敗測試後自行解決

## 任務管理六步驟

1. 計畫寫入 `tasks/todo.md`，含可勾選項目
2. 開始實作前先確認計畫
3. 進行中標記完成項目
4. 每步說明高層次變動
5. 完成後在 `tasks/todo.md` 加入 review 段落
6. 被糾正後更新 `tasks/lessons.md`

## 三大核心原則

- **Simplicity First**：每個變動盡可能簡單，影響最小的程式碼
- **No Laziness**：找根本原因，不臨時修補，維持資深工程師標準
- **Minimal Impact**：只動必要的東西，避免引入新 bug

## 提及的實體

- [[Boris Cherny]] — Claude Code 創始者
- [[Claude Code]] — Anthropic 的 AI 程式工具
- [[Anthropic]] — 開發 Claude 的公司

## 提及的概念

- [[CLAUDE.md 最佳實踐]]
- [[子代理策略]]
- [[自我改善迴圈]]
- [[計畫模式（Plan Mode）]]

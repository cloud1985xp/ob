---
title: "《How Anthropic teams use Claude Code》的中文翻譯"
type: source
tags: [claude-code, anthropic, marketing, design, workflow, non-technical]
sources: [2025-06-09_how-anthropic-teams-use-claude-code.md]
updated: 2026-04-06
---

# How Anthropic Teams Use Claude Code（中譯）

- **來源**：[vocus.cc 小滑](https://vocus.cc/article/684628d8fd8978000117c744)
- **原始文件**：Anthropic 官方內部指南 PDF
- **發布日期**：2025-06-09
- **備註**：本文為部分章節翻譯，作者表示有空會補完其餘章節

## 核心摘要

[[Anthropic]] 官方整理的內部工作指南，揭示非技術團隊（行銷、設計）如何實際運用 [[Claude Code]] 達到 2–10 倍的效率提升。核心模式是：**將具 API 介面的重複工作流程，拆解為專屬子代理後自動化**。

## 成長行銷團隊

### 主要用途

- **Google Ads 廣告文案自動生成**：處理含數百條素材的 CSV，識別表現不佳的項目後生成新版本（標題 30 字元、描述 90 字元的限制下）；使用「標題代理 + 描述代理」雙子代理協作，數分鐘內產出數百條
- **Figma 插件批量創意生產**：自動識別版位框架、批量替換標題描述，單批生成 100 條變體，原本數小時縮短為不到半秒
- **Meta Ads MCP 伺服器**：與 Meta Ads API 串接，在 Claude Desktop 中直接查詢廣告成效，無需切換平台
- **記憶系統 + 提示工程**：記錄每輪廣告測試的假設與結果，下一輪自動帶入歷史數據，建立持續自我優化的測試框架

### 效益數據

- 廣告文案撰寫：**2 小時 → 15 分鐘**
- 創意產出：提升 **10 倍**

### 最佳秘訣

1. 優先找「具 API 介面的重複任務」作為切入點
2. 複雜工作流拆分為多個單一職責的子代理（方便偵錯）
3. 先用 Claude.ai 腦力激盪、規劃提示，再交給 Claude Code 執行；逐步執行而非一次到位

## 產品設計團隊

### 主要用途

- **前端視覺微調**：設計師直接在 Claude Code 實作字體/顏色/間距等改動，無需工程師中介
- **GitHub Actions 自動建單**：提交描述需求的 issue，Claude 自動生成程式解法
- **Figma mockup → 互動式原型**：貼入截圖，立即生成可互動原型
- **邊界情境分析**：用 Claude Code 繪製錯誤狀態圖、邏輯流程圖，提前識別邊界情況
- **跨團隊法務合規協作**：大規模文案更新（如全庫刪除「research preview」）從一週縮短為兩次 30 分鐘會議

### 效益數據

- 執行速度提升：**2–3 倍**
- 設計師的 Claude Code 使用佔工作時間：**80%**

### 最佳秘訣

1. 請工程師協助完成初始設定（非技術人員上手門檻高，但一旦配置好效果顯著）
2. 在 CLAUDE.md 中說明自己的角色與需求（如「我是設計師、需要詳細說明」）
3. `Command+V` 直接貼入設計截圖，Claude 擅長「讀圖生碼」

## 提及的實體

- [[Anthropic]] — 本指南的發布方
- [[Claude Code]] — 工具本體

## 提及的概念

- [[子代理策略]]
- [[CLAUDE.md 最佳實踐]]
- [[MCP 伺服器整合]]
- [[非技術用戶使用 Claude Code]]

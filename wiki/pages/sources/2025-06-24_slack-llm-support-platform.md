---
title: "From Alert Fatigue to AI Conversations: Building a Slack-Native LLM Support Platform"
type: source
tags: [ai, mcp, slack, observability, kubernetes, llm-ops]
sources: [2025-06-24_slack-llm-support-platform.md]
updated: 2026-04-07
---

# 從告警疲勞到 AI 對話：打造 Slack 原生 LLM 支援平台

- **來源**：[Medium - Yen Chuang](https://medium.com/@yenchuang/from-alert-fatigue-to-ai-conversations-building-a-slack-native-observability-platform-dc039de86ae6)
- **作者**：Yen Chuang
- **發布日期**：2025-06-24

## 核心摘要

在 Slack 中整合 LLM + MCP，讓 AI Bot 成為 Kubernetes / Grafana 的第一線除錯助手，平台工程師不再是知識瓶頸，開發者可以自助解決大多數問題。

## 問題

- 平台工程師擁有寶貴的專業知識，卻不自覺地成為守門人
- 開發者缺乏初步除錯能力，tickets 缺少關鍵上下文
- Alert fatigue：工程師淹沒在無數通知中

## 解決方案架構

```
Slack（使用者介面）
    ↓
slack-mcp-client（通用橋接器）
    ↓
Multiple MCP Servers（工具整合）
    ↓
Target Systems（K8s, Grafana, Jira 等）
```

## 真實場景示範

### 場景一：Pod 故障
```
開發者：@pe-support-bot my app is failing, can you help?
Bot：Sure. Could you provide the namespace?
Bot：發現 ImagePullBackOff，根本原因是...
成本：10 美分
```

### 場景二：效能尖峰
開發者提供 Grafana panel UID → Bot 自動識別：
- 問題部署
- 資源限制導致頻繁 restart
- 提供具體操作建議

## 安全最佳實踐

- MCP Bot 只賦予**最小必要權限**（diagnostics only）
- 不能修改 IaC 或將敏感憑證暴露到 Slack 頻道
- Secure by design

## Human-in-the-Loop 模型

- AI 提供第一線診斷
- 需要升級時，人工接收**完整的 AI 對話上下文**
- 在 Slack Thread 中公開協作 → 整個工程團隊都在學習

## 效益

- 降低 MTTR（Mean Time To Recovery）
- 知識共享：開發者親眼看到進階除錯過程
- 完整審計軌跡

## 與 [[MCP 伺服器整合]] 的關聯

本文與 [[sources/2025-06-09_how-anthropic-teams-use-claude-code]] 的 Meta Ads MCP 伺服器案例形成對比：
- Anthropic 團隊用 MCP 連接廣告平台
- 本文用 MCP 連接 K8s / Grafana
- 兩者都是「用 MCP 讓 LLM 直接查詢外部系統」的模式

## 提及的實體

- [[Kubernetes]] — 目標系統
- [[Grafana]] — 監控工具

## 提及的概念

- [[MCP 伺服器整合]]
- [[LLM 可觀測性平台]]
- [[Human-in-the-Loop AI]]

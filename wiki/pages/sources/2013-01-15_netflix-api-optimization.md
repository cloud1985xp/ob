---
title: "Optimizing the Netflix API"
type: source
tags: [system-design, api, netflix, reactive-programming, distributed-systems]
sources: [2013-01-15_netflix-api-optimization.md]
updated: 2026-04-07
---

# 最佳化 Netflix API

- **來源**：[Netflix Technology Blog](https://netflixtechblog.com/optimizing-the-netflix-api-5c9ac715cf19)
- **作者**：Ben Christensen（Netflix API Team）
- **發布日期**：2013-01-15

## 核心摘要

Netflix API 重新設計，解決**Chatty API**問題（每個 UI 呈現需要多次 REST 調用），改為**以裝置為單位的聚合端點**，搭配伺服器端並行執行，大幅提升效能。

## 問題：Chatty REST API

原始設計：每個 REST 端點只返回部分功能，UI 需要組合多個請求才能呈現一個畫面。問題：
- WAN 延遲被乘以請求次數
- 每個裝置端（電視、手機、Web）的需求不同，通用 API 不夠高效

## 解決方案架構

**五個設計目標**：
1. 減少 Chattiness（多請求→單一聚合請求）
2. 分散式 API 開發（每個客戶端團隊自己管理端點）
3. 降低部署風險（多團隊快速迭代）
4. 支援多語言（JS、Objective-C、Java、Ruby 等）
5. 分散式運維

**三大架構決策**：
- **動態多語言 JVM Runtime**：端點用 Groovy 動態定義，不需重新部署整個 API
- **全異步服務層**：底層服務調用全部非阻塞
- **Reactive Programming Model**（仿 Rx Observables）：用函數式風格組合異步回調

## 關鍵元件

- **Dynamic Endpoints**：端點程式碼存在 Cassandra 叢集，可以即時更新和 Canary
- **Hystrix**（斷路器）：隔離服務層，處理大量網路呼叫中不可避免的失敗
- **API Service Layer**：抽象所有後端服務，端點訪問「功能」而非「系統」

## 洞察：伺服器端並行

> 「多個網路請求在同一網路上執行，比從裝置執行更高效。而且不需要讓每個工程師成為低階並發程式設計專家。」

## 歷史意義

這篇文章（2013 年）是早期 **BFF（Backend for Frontend）模式**的重要案例，也是 **Reactive Programming** 進入後端的里程碑之一。Hystrix 後來成為 Java 微服務生態系的標配。

## 提及的實體

- [[Netflix]] — 案例主體

## 提及的概念

- [[BFF 模式（Backend for Frontend）]]
- [[Reactive Programming]]
- [[API 設計模式]]
- [[斷路器模式（Circuit Breaker）]]

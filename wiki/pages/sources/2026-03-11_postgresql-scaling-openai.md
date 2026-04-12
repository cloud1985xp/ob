---
title: "擴展 PostgreSQL 以支援 8 億名 ChatGPT 使用者"
type: source
tags: [postgresql, database, scaling, openai, chatgpt, system-design]
sources: [2026-03-11_postgresql-scaling-openai.md]
updated: 2026-04-07
---

# OpenAI 如何將 PostgreSQL 擴展到支援 8 億使用者

- **來源**：[OpenAI Engineering Blog](https://openai.com/zh-Hant/index/scaling-postgresql/)
- **作者**：Bohan Zhang（OpenAI）
- **發布日期**：2026-03-11

## 核心摘要

OpenAI 用**單一主節點 PostgreSQL + 近 50 個讀取副本**，在不分片的情況下支援 ChatGPT 的讀取密集型工作負載（每秒數百萬次查詢）。關鍵是嚴格的工程紀律和多層最佳化，而非架構革命。

## 規模數據

- 使用者：8 億
- 過去一年負載成長：**10 倍以上**
- QPS：每秒數百萬次查詢
- 可用性：**99.999%**
- P99 客戶端延遲：**低雙位數毫秒**
- 讀取副本：全球多區域近 **50 個**

## 核心架構決策

**單一主節點 + 多讀取副本**（未分片）
- 理由：工作負載以**讀取為主**；寫入密集型工作負載已移至 Azure CosmosDB
- 分片的代價：需要修改數百個應用程式端點，耗時數月甚至數年

## 五大挑戰與解法

### 1. 寫入壓力（單一寫入器）
- **問題**：MVCC 在高寫入負載下會造成顯著的寫入放大、表格膨脹
- **解法**：
  - 可分片的寫入密集工作負載移至 CosmosDB
  - 應用層減少不必要的寫入（修正重複寫入 bug、延遲寫入）
  - 回填資料時嚴格限速

### 2. 高成本查詢
- **問題**：12 表格 JOIN 的查詢曾是 SEV 的罪魁禍首
- **解法**：
  - 避免複雜 JOIN；必要時分解到應用層處理
  - 仔細審查 ORM 生成的 SQL
  - 設定 `idle_in_transaction_session_timeout` 防止 autovacuum 阻塞

### 3. 單點故障（主節點）
- **解法**：HA 模式 + 熱備援副本（持續同步，隨時可接管）

### 4. 工作負載隔離
- 近 50 個讀取副本分散於多個地理區域
- 正在測試**級聯複製**（Cascading Replication）：中繼副本轉發 WAL，讓副本數可擴展到 100+

### 5. 突發流量保護
- 多層限速：應用層、連線池管理器、代理、查詢層
- ORM 層增強支援限速
- 避免重試間隔過短（防止重試風暴）

## Schema 變更的嚴格紀律

- 只允許輕量級 Schema 變更（不觸發整表重寫）
- 5 秒超時限制
- 新功能需要新表？→ 必須用 CosmosDB，不能用現有 PostgreSQL
- 回填時嚴格限速（可能需要超過一週，但確保穩定性）

## 關鍵洞察

> PostgreSQL MVCC 的設計使它在讀取密集型工作負載下擴展性很好，但在寫入密集型工作負載下效率較低（整列複製、死元組、表格膨脹）。

## 提及的實體

- [[OpenAI]] — 案例主體
- [[PostgreSQL]] — 資料庫

## 提及的概念

- [[PostgreSQL 大規模擴展策略]]
- [[資料庫讀寫分離架構]]
- [[MVCC（多版本並發控制）]]

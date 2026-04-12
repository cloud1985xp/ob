---
title: "PostgreSQL 大規模擴展策略"
type: concept
tags: [postgresql, database, scaling, read-replicas, sharding, performance]
sources: [2026-03-11_postgresql-scaling-openai]
updated: 2026-04-07
---

# PostgreSQL 大規模擴展策略

[[OpenAI]] 用單一主節點 PostgreSQL 支援 8 億 ChatGPT 用戶的實戰經驗，揭示讀取密集型工作負載下 PostgreSQL 的擴展上限與策略。

## 核心架構：讀寫分離

```
[單一主節點（所有寫入）]
         ↓ WAL 串流
[~50 個讀取副本（分散全球多區域）]
```

- 所有讀取盡可能路由到副本
- 寫入事務中的讀取保留在主節點
- 使用 HA 模式 + 熱備援（主節點故障可快速切換）

## PostgreSQL 的天然限制：MVCC

**問題**：MVCC 在高寫入負載下效率低
- 更新一列 → 整列複製建立新版本（寫入放大）
- 查詢需掃描多個版本（讀取放大）
- 產生死元組 → 表格膨脹 → autovacuum 壓力

**解法**：把寫入密集型工作負載移至 Azure CosmosDB（分片系統）

## 多層防護策略

### 查詢層
- 避免複雜 JOIN（12 表 JOIN 曾引發 SEV）
- 審查 ORM 生成的 SQL
- 設定 `idle_in_transaction_session_timeout`

### 限速層（多層次）
- 應用層
- 連線池管理器（PgBouncer）
- 代理層
- 查詢層（可封鎖特定查詢 digest）

### Schema 變更紀律
- 只允許輕量級 Schema 變更（5 秒超時）
- 新表→必須用 CosmosDB，不能加進 PostgreSQL
- 回填：嚴格限速（可能超過一週）

## 惡性循環的防止

```
負載增加 → 延遲增加 → 請求逾時 → 重試 → 更多負載 → 崩潰
```

解法：限速 + 快取 + 讀寫分離 → 在循環啟動前截斷它

## 未來方向

- **級聯複製（Cascading Replication）**：中繼副本轉發 WAL，可擴展至 100+ 副本
- PostgreSQL 分片（長期選項）

## 洞察：分片的真實成本

分片看似簡單，實際上需要修改**數百個應用程式端點**，可能耗時數月至數年。在讀取密集型場景下，讀寫分離 + 嚴格工程紀律往往已足夠。

## 出現在以下來源

- [[sources/2026-03-11_postgresql-scaling-openai]]

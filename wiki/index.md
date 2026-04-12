# Wiki 索引

> 由 Agent 自動維護，每次 ingest 後更新。
> **主題**：AI 開發工具 / 軟體架構 / Elixir 生態 / 系統設計 / 基礎設施
> 最後更新：2026-04-07

---

## 來源摘要（Sources）

### AI 開發工具

| 頁面 | 摘要 | 日期 |
|------|------|------|
| [[sources/2026-02-24_claude-code-workflow-detools]] | Boris Cherny 親授 CLAUDE.md 工作流框架，6 大核心引擎 | 2026-02-24 |
| [[sources/2026-03-30_boris-cherny-15-features]] | 15 個被低估的 Claude Code 進階功能：跨裝置、排程、並行、語音 | 2026-03-30 |
| [[sources/2025-06-09_how-anthropic-teams-use-claude-code]] | Anthropic 內部指南（部分），行銷與設計團隊的實際用法 | 2025-06-09 |
| [[sources/2025-12-17_conductor-gemini-cli]] | Google Conductor：Gemini CLI 的 Context-Driven Development | 2025-12-17 |
| [[sources/2025-06-24_slack-llm-support-platform]] | Slack + LLM + MCP：AI 第一線除錯支援平台的架構與實作 | 2025-06-24 |

### 軟體架構

| 頁面 | 摘要 | 日期 |
|------|------|------|
| [[sources/2017-09-28_clean-architecture]] | Clean Architecture 整合 Hexagonal/Onion，並非革命，而是整理 | 2017-09-28 |
| [[sources/2018-10-24_hexagonal-architecture]] | Hexagonal Architecture（Ports & Adapters）：輸入/輸出在邊緣，核心可測 | 2018-10-24 |
| [[sources/2020-07-16_onion-architecture]] | Onion Architecture：依賴向內，透過 contract 解耦每一層 | 2020-07-16 |
| [[sources/2024-08-22_strangler-fig]] | Strangler Fig：漸進式取代遺留系統，而非大爆炸式重寫 | 2024-08-22 |

### Elixir / Phoenix 生態

| 頁面 | 摘要 | 日期 |
|------|------|------|
| [[sources/2023-11-09_liveview-async-assigns]] | LiveView 非典型 async assigns：即時串流中間訊息而非等待最終結果 | 2023-11-09 |
| [[sources/2023-12-04_liveview-leaflet-12k-markers]] | 12,000+ 地圖標記：async_stream 並行查詢 + Canvas renderer 避免 DOM 爆炸 | 2023-12-04 |
| [[sources/2025-03-25_cyanview-elixir-superbowl]] | Cyanview：9 人團隊用 Elixir 支援超級盃 200+ 攝影機，靠 Supervision Trees | 2025-03-25 |
| [[sources/2019-04-29_k8s-erlang-vm]] | K8s 與 Erlang VM 互補：叢集層 vs 應用層的容錯，各司其職 | ~2019 |

### 系統設計 / 效能

| 頁面 | 摘要 | 日期 |
|------|------|------|
| [[sources/2013-01-15_netflix-api-optimization]] | Netflix API：Chatty REST → 聚合端點 + Reactive Programming + Hystrix | 2013-01-15 |
| [[sources/2020-02-27_discord-go-to-rust]] | Discord Go→Rust：Go GC 每 2 分鐘掃描 LRU 快取，Rust 所有權立即釋放 | 2020-02-27 |
| [[sources/2026-03-11_postgresql-scaling-openai]] | OpenAI PostgreSQL：單主節點 + 50 副本支援 8 億用戶，嚴格工程紀律 | 2026-03-11 |

### 工具 / 其他

| 頁面 | 摘要 | 日期 |
|------|------|------|
| [[sources/2021-05-16_htop-explained]] | htop 完整欄位解析：CPU 顏色、LA、VIRT/RES/SHR、Process State | 2021-05-16 |
| [[sources/2018-05-04_information-architecture]] | 資訊架構（IA）：Labeling / Organization / Navigation / Search System | 2018-05-04 |

---

## 實體（Entities）

### 人物

| 頁面 | 說明 |
|------|------|
| [[entities/Boris-Cherny]] | Claude Code 創始者與負責人，Anthropic |
| [[entities/Martin-Fowler]] | 架構與設計模式權威，ThoughtWorks，提出 Strangler Fig |
| [[entities/José-Valim]] | Elixir 創始者，Dashbit 創辦人 |
| [[entities/Uncle-Bob]] | Robert C. Martin，Clean Architecture / SOLID 提出者 |

### 工具 / 框架 / 平台

| 頁面 | 說明 |
|------|------|
| [[entities/Anthropic]] | 開發 Claude 系列模型的 AI 安全公司 |
| [[entities/Claude-Code]] | Anthropic 的 AI 程式工具 / 自主代理人平台 |
| [[entities/Elixir]] | 函數式語言，建立在 Erlang VM，以並行與容錯著稱 |

---

## 概念（Concepts）

### AI 工具與工作流

| 頁面 | 說明 |
|------|------|
| [[concepts/CLAUDE.md-最佳實踐]] | CLAUDE.md 的作用、內容框架與非技術用戶的使用方式 |
| [[concepts/Context-Driven-Development]] | 把 AI 記憶持久化到檔案系統，Google Conductor vs Anthropic CLAUDE.md |
| [[concepts/子代理策略]] | 將複雜任務拆解給多個專屬子代理（三篇 Claude Code 來源的共同核心） |
| [[concepts/排程自動化]] | `/loop` 與 `/schedule` 的用法與 Cherny 的實際排程範例 |
| [[concepts/Hooks-機制]] | 在工作週期節點注入確定性邏輯，實現完全無人監管的工作流 |
| [[concepts/Worktrees-並行處理]] | git worktrees 的並行架構與 `/batch` 的大規模應用 |

### 軟體架構

| 頁面 | 說明 |
|------|------|
| [[concepts/軟體架構模式]] | Clean / Hexagonal / Onion / Strangler Fig 的關係與對照 |
| [[concepts/Strangler-Fig-遺留系統現代化]] | 漸進式取代遺留系統的四大步驟與核心概念 |

### Elixir 生態

| 頁面 | 說明 |
|------|------|
| [[concepts/Elixir-進程模型]] | Actor Model、PID 傳遞、Supervision Trees 的實際應用 |

### 系統設計 / 效能

| 頁面 | 說明 |
|------|------|
| [[concepts/GC-vs-所有權模型]] | Discord Go→Rust 案例：GC 掃描 vs 所有權立即釋放的效能差異 |
| [[concepts/PostgreSQL-大規模擴展策略]] | OpenAI 單主節點 + 50 副本的架構決策、MVCC 限制與多層防護 |

---

## 合成分析（Synthesis）

*尚未有合成頁面。*

---

## 統計

- 來源數：18
- 實體頁：7
- 概念頁：11
- 合成頁：0

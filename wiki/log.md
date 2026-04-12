# 操作日誌

> append-only 記錄。快速搜尋最近操作：`grep "^## \[" log.md | tail -10`

---

## [2026-04-06] update | Wiki 系統初始化
- 建立目錄結構：inbox/、archive/、pages/sources/、pages/entities/、pages/concepts/、pages/synthesis/
- 建立 CLAUDE.md（Schema v1.0）
- 建立 index.md（空索引）
- 建立 log.md（本檔）
- Inbox 中現有待處理來源：
  - `Claude Code 之父分享的內部工作流：一份 CLAUDE.md 讓你晉升 10 倍工程師`
  - `Claude Code 創始者 Boris Cherny 親自示範每天在用的 15 個功能`
  - `《How Anthropic teams use Claude Code》的中文翻譯`

## [2026-04-06] ingest | Claude Code 三篇來源首次批次消化
- 消化來源：
  1. `Claude Code 之父分享的內部工作流` (DeTools, 2026-02-24)
  2. `Boris Cherny 親示 15 個 Claude Code 功能` (INSIDE, 2026-03-30)
  3. `How Anthropic teams use Claude Code 中譯` (小滑, 2025-06-09)
- 新建頁面（11 頁）：
  - sources/2026-02-24_claude-code-workflow-detools.md
  - sources/2026-03-30_boris-cherny-15-features.md
  - sources/2025-06-09_how-anthropic-teams-use-claude-code.md
  - entities/Boris-Cherny.md、entities/Claude-Code.md、entities/Anthropic.md
  - concepts/CLAUDE.md-最佳實踐.md、concepts/子代理策略.md
  - concepts/排程自動化.md、concepts/Hooks-機制.md、concepts/Worktrees-並行處理.md
- 更新：index.md（統計：3 來源、3 實體、5 概念）
- 移動：3 個原始檔從 inbox/ → archive/

## [2026-04-07] update | Wiki 主題擴展為技術知識庫
- 更新 CLAUDE.md：新增 Wiki 主題範疇（AI 工具 / 架構 / Elixir / 系統設計 / 基礎設施）

## [2026-04-07] ingest | 15 篇技術文章批次消化
- 消化來源（15 篇）：
  - 架構：Clean Architecture、Hexagonal Architecture、Onion Architecture、Strangler Fig
  - Elixir：LiveView Async Assigns、Cyanview 超級盃案例、K8s+Erlang VM、LiveView+Leaflet.js
  - 系統設計：Netflix API 優化、Discord Go→Rust、PostgreSQL 擴展到 8 億用戶
  - AI 工具：Conductor Gemini CLI、Slack LLM 支援平台
  - 工具：htop 欄位解析、資訊架構 MIX2018
- 新建頁面（24 頁）：
  - 15 個 source 頁（pages/sources/）
  - entity 頁：Martin-Fowler、José-Valim、Uncle-Bob、Elixir
  - concept 頁：軟體架構模式、Strangler-Fig、Context-Driven-Development、Elixir-進程模型、GC-vs-所有權模型、PostgreSQL-大規模擴展策略
- 更新：index.md（統計：18 來源、7 實體、11 概念）
- 移動：15 個原始檔從 inbox/ → archive/（archive 共 18 個檔案）

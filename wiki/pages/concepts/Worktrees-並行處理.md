---
title: "Worktrees 並行處理"
type: concept
tags: [claude-code, git, worktree, parallel, batch]
sources: [2026-03-30_boris-cherny-15-features]
updated: 2026-04-06
---

# Worktrees 並行處理

[[Claude Code]] 對 git worktrees 的原生支援，讓多個 Claude 代理人能在同一 repo 中**完全獨立地並行工作**。

## 核心機制

Git worktree 讓同一個 repo 的不同分支在獨立的目錄中同時存在。Claude Code 利用這個特性，讓每個代理人在自己的 worktree 中作業、互不干擾。

## 使用方式

### 手動 Worktree

```bash
claude --worktree
```

或在 Claude Desktop App 中選擇 worktree 選項。

### 批次自動化（`/batch`）

2026 年 2 月底推出，適合大規模並行任務：

```
/batch migrate from jest to vite
```

流程：
1. Claude 先向使用者提問，釐清需求
2. 自動拆解工作
3. 同時開啟數十、數百甚至**數千個** worktree 代理人並行執行

## [[Boris Cherny]] 的使用規模

> 「我隨時都有幾十個 Claude 在跑，worktrees 是實現這件事的核心基礎設施。」

## 非 git 使用者

可透過 `WorktreeCreate hook` 自訂 worktree 建立邏輯，適用於非 git 版本控制系統。

## 典型應用場景

- 大型程式碼遷移（framework 切換）
- 大規模 API 介面重構
- 批量測試生成
- 多個 bug fix 並行推進

## 相關概念

- [[子代理策略]]
- [[排程自動化（Loop & Schedule）]]

## 出現在以下來源

- [[sources/2026-03-30_boris-cherny-15-features]]

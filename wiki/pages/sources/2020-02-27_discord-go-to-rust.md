---
title: "Discord 為何從 Go 轉為使用 Rust"
type: source
tags: [rust, go, performance, memory-management, discord, case-study]
sources: [2020-02-27_discord-go-to-rust.md]
updated: 2026-04-07
---

# Discord 為何從 Go 切換到 Rust

- **來源**：[Discord Blog](https://discord.com/blog/why-discord-is-switching-from-go-to-rust)
- **作者**：Jesse Howarth（Discord 基礎架構首席工程師）
- **發布日期**：2020-02-27

## 核心摘要

Discord 將 Read States 服務從 Go 重寫為 Rust，消除了每 2 分鐘一次的延遲峰值。根本原因：**Go 的垃圾回收器（GC）強制每 2 分鐘掃描整個 LRU 快取**，而 Rust 沒有 GC，記憶體在不需要時立即釋放。

## Read States 服務背景

- 唯一功能：追蹤你閱讀過的頻道和訊息
- 數十億個 Read State（每位使用者每個頻道一個）
- 每個快取有數千萬個 Read State、每秒數十萬次更新
- 後端用 Cassandra 持久化

## Go 的問題

**症狀**：每約 2 分鐘，延遲和 CPU 突然飆升

**根本原因**：
1. Go [強制每 2 分鐘執行一次 GC](https://github.com/golang/go/blob/895b7c85addfffe19b66d8ca71c31799d6e55990/src/runtime/proc.go#L4481-L4486)，無論記憶體增長如何
2. GC 需要**掃描整個 LRU 快取**確認記憶體是否可以回收
3. LRU 快取越大，GC 掃描越慢，延遲峰值越嚴重

**嘗試的解法（失敗）**：
- 調整 GC 百分比 → 無效（分配速率太低，不會觸發更頻繁的 GC）
- 縮小 LRU 快取 → 減少 GC 延遲，但增加快取未命中率

## Rust 的解法

- **沒有 GC**：記憶體在不再需要時**立即**釋放
- 從 LRU 快取淘汰時，記憶體立刻釋放，不用等 GC 掃描

## 結果數據

**基礎版本（僅粗略轉譯，最少最佳化）**：
- 延遲與 Go 相當，**但沒有延遲峰值**
- 即使沒有深度最佳化，Rust 已超越大量調整過的 Go

**最佳化後**：
- 平均延遲：毫秒 → **微秒**級
- CPU 和記憶體全面優於 Go

**最佳化措施**：
1. LRU 快取的 HashMap → BTreeMap（記憶體佈局更優）
2. 更換為現代 Rust 並行指標庫
3. 減少記憶體副本

**額外彩蛋**：升級 tokio 0.2 後，CPU 再次免費下降。

## 關鍵洞察

> 「即便如此，Rust 還是能夠超越**經過大量人工調整的 Go 版本**。這充分證明了與需要深入調整的 Go 相比，使用 Rust 編寫高效率的程式是件多麼容易的事。」

## 提及的實體

- [[Discord]] — 案例主體
- [[Rust]] — 目標語言
- [[Go]] — 被替換的語言

## 提及的概念

- [[GC vs 所有權模型的記憶體管理]]
- [[語言效能比較]]

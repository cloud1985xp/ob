---
title: "你一定用過 htop，但你有看懂每個欄位嗎？"
type: source
tags: [linux, htop, system-monitoring, performance, tools]
sources: [2021-05-16_htop-explained.md]
updated: 2026-04-07
---

# htop 完整欄位解析

- **來源**：[Starbugs Weekly - Larry Lu](https://medium.com/starbugs/do-you-understand-htop-ffb72b3d5629)
- **作者**：Larry Lu
- **發布日期**：2021-05-16

## 核心摘要

htop 所有欄位的完整解釋，從 CPU 顏色編碼到 Process State、從 VIRT/RES/SHR 的差異到 Time+ 的意義。除錯效能問題的必備參考。

## CPU 顏色編碼

- **紅色**：Kernel thread（系統調度、記憶體管理，最高優先權）
- **綠色**：Normal priority thread（一般使用者程式）
- **藍色**：Low priority thread（優先權最低，系統資源緊張時最先被殺）

## Memory 顏色編碼

- **綠色**：Process 佔用的記憶體（瀏覽器、VSCode 等）
- **藍色**：Buffer pages（儲存 metadata，如 `ls -l` 的目錄資訊）
- **橘色**：Cache pages（儲存檔案內容，如 `cat` 讀取的內容）

> **重要**：記憶體使用量並非越低越好——閒置記憶體被系統拿來當 buffer/cache，加速讀取。「記憶體清理大師」只會降低效能。

## Load Average（LA）

三個數字：最近 1、5、15 分鐘平均有多少個 thread 需要 CPU。

- LA < 1：電腦幾乎閒置
- LA = 1～2：正常使用（上網、聽音樂）
- LA > 邏輯核心數：CPU 飽和，有任務在排隊

**診斷用法**：
- 程式慢但 LA 低 → 程式沒善用多核心，或瓶頸在 IO
- LA 高 → 改善演算法或換更快 CPU

## PRI & NI（優先權）

- **PRI**：系統決定，無法自行修改，數字越小越優先
- **NI（Nice）**：使用者可以用 `renice` 調整（-20 最高，19 最低）
- 注意：系統不一定會遵守 Nice 值

## VIRT / RES / SHR（記憶體）

| 欄位 | 意義 |
|------|------|
| VIRT | Process 可以存取的記憶體總和（包含未實際使用的） |
| RES | 物理上實際佔用的記憶體（VIRT >> RES 很正常） |
| SHR | 可與其他 process 共用的記憶體（glibc、read-only 檔案） |

**實務**：通常只看 RES，VIRT 不用擔心。

## Process State（S 欄位）

| State | 意義 |
|-------|------|
| **R** | Running 或在 running queue 等待 |
| **S** | Sleeping，等待事件（如使用者輸入）|
| **D** | Disk sleep，等待 IO（IO 瓶頸的指標）|

## CPU% vs Time+

- **CPU%**：短期（3秒窗口）的 CPU 使用率，適合即時找出暴衝元兇
- **Time+**：累計 CPU time，適合找出長期最耗 CPU 的程式

## 提及的概念

- [[Linux 系統監控]]
- [[效能診斷工具]]

---
title: "bliki: Strangler Fig"
type: source
tags: [architecture, legacy, modernization, patterns, martin-fowler]
sources: [2024-08-22_strangler-fig.md]
updated: 2026-04-07
---

# Strangler Fig（絞殺榕）模式

- **來源**：[Martin Fowler's bliki](https://martinfowler.com/bliki/StranglerFigApplication.html)
- **作者**：[[Martin Fowler]]
- **發布日期**：2024-08-22（最新更新版）

## 核心摘要

Strangler Fig 是 [[Martin Fowler]] 2001 年在澳洲雨林看到絞殺榕樹後提出的比喻：**漸進式取代遺留系統，而非一次性重寫**。新代碼從邊緣開始生長，逐步接管舊系統功能，直到舊系統可以退役。

## 為什麼不能一次性重寫

- 重寫耗時，使用者無法等待新功能
- 舊系統行為難以完整規格化
- 大量行為其實是不需要的，重建是浪費

## 四大活動

1. **理解目標**：明確希望達成的結果，在各利害關係人之間對齊
2. **拆解問題**：找出系統的接縫（Seams），識別可以獨立替換的部分
3. **交付各部分**：逐步替換小元件，降低風險，提早實現價值
4. **組織變革**：技術的漸進替換需要伴隨開發文化的變革（Conway's Law）

## 關鍵概念

- **接縫（Seams）**：可以插入切點、讓系統被分割的邊界
- **過渡架構（Transitional Architecture）**：允許新舊系統共存的暫時性架構——雖然之後會拆掉，但降低風險的價值大於成本
- **漸進式投資與回報**：每個小部分替換後即可開始獲益，無需等待全部完成

## 與其他模式的關係

- 需要識別 **Legacy Seam** 才能實施
- 過渡期間使用的是 **Transitional Architecture**
- 需要配合 **Conway's Law** 進行組織調整

## 提及的實體

- [[Martin Fowler]] — 模式提出者

## 提及的概念

- [[Strangler Fig — 遺留系統現代化]]
- [[軟體架構模式]]

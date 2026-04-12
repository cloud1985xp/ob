---
title: "Onion Architecture 🧅"
type: source
tags: [architecture, onion-architecture, ddd, dependency-inversion]
sources: [2020-07-16_onion-architecture.md]
updated: 2026-04-07
---

# Onion Architecture（洋蔥架構）

- **來源**：[dev.to](https://dev.to/barrymcauley/onion-architecture-3fgl)
- **作者**：Barry McAuley
- **發布日期**：2020-07-16

## 核心摘要

Onion Architecture 由 Jeffrey Palermo 提出，核心概念：**業務邏輯完全不依賴外部關注點**（資料庫選擇、UI 框架等）。所有依賴方向從外向內，透過 contract（介面）解耦每一層。

## 層次結構（由外到內）

```
[ Infrastructure（DB、External APIs）]
[ Tests（Unit、Integration、E2E）   ]
[ User Interface                    ]
         ↓（依賴方向）
[ Application Services（Transport） ]
         ↓
[ Domain Services（業務邏輯）       ]
         ↓
[ Domain Model（高層資料物件）      ]
```

- **外三層**（Infrastructure、Tests、UI）：經常變動，與業務邏輯分離
- **Application Services**：定義服務能做什麼的 contracts
- **Domain Services**：業務邏輯的主體（把 A 變成 B）
- **Domain Model**：高層資料物件的表示

## 核心原則：Contract 的力量

```
"Externalise your dependencies and decouple them through contracts."
```

實際範例：系統從 NoSQL 換成 SQL 資料庫時，只要新 Schema 滿足 Domain Model 的 contract，業務邏輯完全不需要改動。

## 與其他架構的關係

- 與 [[Hexagonal Architecture]] 的差異：Hexagonal 把 Ports 在外圈，Onion 把 Domain Model 在最內圈——**幾乎是相同概念的不同表達**
- [[Clean Architecture]] 明確整合了 Onion 的層次（Application Services = Use Cases、Domain Model = Entities）

## 適用場景

- 大型應用程式、多工程師協作
- **不適合**：小型應用程式（額外的抽象成本 > 收益）

## 提及的概念

- [[Onion Architecture]]
- [[軟體架構模式]]
- [[依賴反轉原則（DIP）]]

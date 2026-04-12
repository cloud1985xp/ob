---
title: "Clean Architecture: Standing on the shoulders of giants"
type: source
tags: [architecture, clean-architecture, ddd, solid, uncle-bob]
sources: [2017-09-28_clean-architecture.md]
updated: 2026-04-07
---

# Clean Architecture：站在巨人的肩膀上

- **來源**：[herbertograca.com](https://herbertograca.com/2017/09/28/clean-architecture-standing-on-the-shoulders-of-giants/)
- **作者**：hgraca
- **發布日期**：2017-09-28

## 核心摘要

[[Uncle Bob]]（Robert C. Martin）2012 年提出的 Clean Architecture，**並非革命性的新概念**，而是整合了 [[Hexagonal Architecture]]、[[Onion Architecture]]、EBI 的既有智慧，並加以明確化與標準化。

## 三種架構的共同點

| 特性 | Hexagonal | Onion | Clean |
|------|-----------|-------|-------|
| 工具/框架獨立 | ✓ | ✓ | ✓ |
| 交付機制獨立 | ✓ | ✓ | ✓ |
| 可獨立測試 | ✓ | ✓ | ✓ |
| 依賴方向（向內） | 隱含 | 明確 | 明確 |

## Clean Architecture 的層次

由外到內：
1. **Frameworks & Drivers**（UI、DB、外部工具）
2. **Interface Adapters**（Controllers、Presenters、Gateways）
3. **Application Business Rules**（Use Cases）
4. **Enterprise Business Rules**（Entities）

**核心原則**：內層對外層一無所知；跨邊界傳遞資料時，使用最方便內層的格式。

## Clean Architecture 新增了什麼

在 Hexagonal + Onion 的基礎上，Clean Architecture 整合了 **MVC 的控制流** 概念，並明確說明 Controller → Interactor → Presenter 的資料流動方式。

## 作者的評價

> 我不會說 Clean Architecture 是革命性的，但它是極其重要的作品：它復甦了被遺忘的概念，並告訴我們這些概念如何組合在一起。

類比：Robert C. Martin 是軟體開發界的 Isaac Newton——重力一直在那，Newton 只是把它明確化了。

## 提及的實體

- [[Uncle Bob]]（Robert C. Martin）— 提出者
- [[Martin Fowler]] — 上下文中提及

## 提及的概念

- [[Clean Architecture]]
- [[Hexagonal Architecture]]
- [[Onion Architecture]]
- [[軟體架構模式]]
- [[依賴反轉原則（DIP）]]

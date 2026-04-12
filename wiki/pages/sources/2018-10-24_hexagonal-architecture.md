---
title: "Hexagonal Architecture: What Is It and How Does It Work?"
type: source
tags: [architecture, hexagonal, ports-and-adapters, testing, alistair-cockburn]
sources: [2018-10-24_hexagonal-architecture.md]
updated: 2026-04-07
---

# Hexagonal Architecture（六角形架構 / Ports & Adapters）

- **來源**：[DZone](https://dzone.com/articles/hexagonal-architecture-what-is-it-and-how-does-it)
- **作者**：Phil Vuollet
- **發布日期**：2018-10-24

## 核心摘要

Hexagonal Architecture（由 Alistair Cockburn 2005 年提出，本名「Ports and Adapters」）的核心思想：**將輸入/輸出放在應用程式邊緣，讓核心邏輯與外部關注點完全解耦**。最主要的動機是**可測試性**——不需要外部依賴就能測試應用程式核心。

## 三個層次

```
[外部系統（DB、HTTP、UI、Message Bus）]
        ↓
  [Adapters（具體實作）]
        ↓
  [Ports（Interfaces/抽象）]
        ↓
  [Application Core（業務邏輯）]
```

- **Port（端口）**：介面/抽象，定義核心如何與外界通訊
- **Adapter（適配器）**：Port 的具體實作，負責翻譯訊息到實際的 I/O
- **Core**：純業務邏輯，對外部一無所知

## 關鍵好處

1. **可測試性**：用 FakeRepo 替換 DatabaseRepo，不需要真實 DB
2. **可替換性**：換資料庫、換 UI 框架，核心邏輯不需改動
3. **UI 也可以替換**：這是 Hexagonal 與 Layered Architecture 的關鍵差異

## 程式碼示例（C#）

```csharp
// Port（介面）
interface IUserRepo { void Save(UserData userData); }

// Core（只知道 Port）
class UserAdmin {
    private readonly IUserRepo _userRepo;
    public void Save() { _userRepo.Save(_userData); }
}

// Adapter（具體實作）
class UserDatabaseRepository : IUserRepo { ... }
class UserHttpRepository : IUserRepo { ... }

// 測試用 Adapter
class FakeUserRepo : IUserRepo { ... }
```

## 與其他架構的關係

- [[Onion Architecture]] 和 [[Clean Architecture]] 都繼承了 Hexagonal 的「依賴向內」原則
- [[Clean Architecture]] 的文章明確指出，三者在工具獨立性、依賴方向、可測試性上完全一致

## 提及的實體

- [[Uncle Bob]]（相關概念提及）

## 提及的概念

- [[Hexagonal Architecture]]
- [[軟體架構模式]]
- [[依賴反轉原則（DIP）]]

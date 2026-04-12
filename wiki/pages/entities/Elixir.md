---
title: "Elixir"
type: entity
tags: [language, functional, erlang-vm, concurrent, distributed]
sources: [2023-11-09_liveview-async-assigns, 2025-03-25_cyanview-elixir-superbowl, 2019-04-29_k8s-erlang-vm, 2023-12-04_liveview-leaflet-12k-markers]
updated: 2026-04-07
---

# Elixir

由 [[José Valim]] 創造（2011 年）的動態函數式程式語言，建立在 [[Erlang VM]] 之上。

## 核心特性

- **並行性**：基於 Actor Model，每個進程獨立、輕量（數百萬個進程同時運行）
- **容錯性**：OTP Supervision Trees，局部失敗不影響整體
- **分散式**：Distributed Erlang，跨節點直接傳遞訊息
- **Pattern Matching**：強大的模式匹配語法
- **二進位處理**：位元級編碼/解碼，適合協議實作

## 主要框架

- **[[Phoenix LiveView]]**：全棧框架，WebSocket 驅動的即時 UI，無需前端框架
- **Ecto**：資料庫查詢與 schema 定義
- **Broadway**：資料處理 pipeline

## 適用場景（來自 Wiki 來源）

| 場景 | 來源 |
|------|------|
| 廣播/嵌入式裝置控制（Cyanview） | [[sources/2025-03-25_cyanview-elixir-superbowl]] |
| 12,000+ 地圖標記的即時串流 | [[sources/2023-12-04_liveview-leaflet-12k-markers]] |
| LiveView 異步操作 | [[sources/2023-11-09_liveview-async-assigns]] |
| K8s 環境下的大型 pod 部署 | [[sources/2019-04-29_k8s-erlang-vm]] |

## 出現在以下來源

- [[sources/2023-11-09_liveview-async-assigns]]
- [[sources/2025-03-25_cyanview-elixir-superbowl]]
- [[sources/2019-04-29_k8s-erlang-vm]]
- [[sources/2023-12-04_liveview-leaflet-12k-markers]]

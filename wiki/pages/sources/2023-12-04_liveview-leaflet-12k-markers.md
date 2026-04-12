---
title: "Performance optimization: 12,000+ markers with Elixir, LiveView, and Leaflet.js"
type: source
tags: [elixir, liveview, performance, leaflet, maps, concurrency]
sources: [2023-12-04_liveview-leaflet-12k-markers.md]
updated: 2026-04-07
---

# Elixir + LiveView + Leaflet.js：12,000+ 地圖標記的效能最佳化

- **來源**：[dev.to - aziz abdullaev](https://dev.to/azyzz/performance-optimization-when-adding-12000-markers-to-the-map-that-renders-fast-with-elixir-liveview-and-leafletjs-54pf)
- **作者**：aziz abdullaev
- **發布日期**：2023-12-04

## 核心摘要

在 LiveView + Leaflet.js 中渲染 12,000+ 地圖標記的實戰最佳化，核心技巧：**伺服器端用 Elixir Task.async_stream 並行分批查詢 + 客戶端用 Canvas renderer 避免 DOM 節點爆炸**。

## 問題

- 一次查詢 12,300 筆資料（187ms）→ 傳到 LiveView → 前端渲染
- 結果：瀏覽器凍結 10+ 秒
- 根本原因：JS 在客戶端運行，一次添加 12,000 個 SVG 節點到 DOM 代價極高

## 兩層最佳化

### 伺服器層：Task.async_stream 並行批次查詢

```elixir
def stream_projects(pid, batch_size \\ 2000, total_records \\ 14000) do
  batches = div(total_records, batch_size)
  Task.async_stream(0..(batches - 1), fn batch ->
    # 每個 batch 並行查詢，完成後 send 給 LiveView PID
    send(pid, {:data_received, result})
  end)
  |> Stream.run()
end
```

- 原本：187ms 一次查全部
- 最佳化後：66ms，7 個並行批次（每批 2,000 筆）
- 使用 LiveView 是進程 PID 的特性，讓異步任務可以回傳資料

### 客戶端層：Canvas renderer

```javascript
// 關鍵：用 Canvas 而非預設的 SVG
let myRenderer = L.canvas({ padding: 0.5 });
L.circleMarker([lat, lng], { renderer: myRenderer }).addTo(map);
```

- 預設 SVG：每個 marker 建立一個 DOM 節點 → 12,000 個節點 → 崩潰
- Canvas renderer：所有 marker 畫在同一個 canvas 元素 → DOM 節點數固定

## 資料傳遞流程

```
DB → Task.async_stream（並行）→ send(pid) → handle_info → push_event → JS Hook
```

`push_event` + `handleEvent` 是 LiveView 推送資料到 JS Hook 的標準方式。

## 關鍵洞察

LiveView 是進程，有自己的 PID，這讓「異步任務把結果傳回 LiveView」變得極自然——Elixir 的進程模型與 LiveView 的架構完美結合。

## 提及的實體

- [[Elixir]] — 伺服器語言
- [[Phoenix LiveView]] — 框架

## 提及的概念

- [[LiveView 異步操作]]
- [[Elixir 進程模型]]
- [[LiveView JS Interop（Hooks）]]

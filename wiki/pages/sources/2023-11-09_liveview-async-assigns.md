---
title: "Abusing LiveView's new Async Assigns Feature"
type: source
tags: [elixir, phoenix, liveview, async, concurrency]
sources: [2023-11-09_liveview-async-assigns.md]
updated: 2026-04-07
---

# LiveView 非標準 Async Assigns 用法

- **來源**：[Fly.io Phoenix Files](https://fly.io/phoenix-files/abusing-liveview-new-async-assigns-feature/)
- **作者**：Annie Ruygt（Fly.io）
- **發布日期**：2023-11-09

## 核心摘要

[[Phoenix LiveView]] v0.20 引入了 `async operations`（`start_async/3`、`assign_async/3`、`handle_async/3`），本文展示一個非典型用法：**不等待最終結果，而是即時接收異步進程串流的中間訊息**（類似 ChatGPT 的 streaming text 效果）。

## 問題背景

標準 async assigns 設計用來「在 mount 時載入資料」，但本文需求不同：
- 由**使用者互動**觸發（不是在 mount）
- 需要**即時串流中間訊息**，而不是等待最終結果

## 解決方案架構

```elixir
# 1. 啟動 async task，用 closure 傳遞 LiveView PID
def start_test_task(socket) do
  live_view_pid = self()
  socket
  |> assign(:async_result, AsyncResult.loading())
  |> start_async(:running_task, fn ->
    Enum.each(1..5, fn n ->
      Process.sleep(1_000)
      send(live_view_pid, {:task_message, "Chunk #{n}"})
    end)
    :ok  # 返回小而受控的值
  end)
end

# 2. handle_async 處理完成/失敗/崩潰三種狀態
def handle_async(:running_task, {:ok, :ok}, socket)     # 成功
def handle_async(:running_task, {:ok, {:error, r}}, s)  # 函數返回錯誤
def handle_async(:running_task, {:exit, reason}, socket) # 進程崩潰

# 3. 取消
def handle_event("cancel", _params, socket) do
  socket |> cancel_async(:running_task) |> assign(:async_result, %AsyncResult{})
end
```

## 關鍵洞察

- **`%AsyncResult{}`** 不只用來存結果，也可以用來追蹤 loading 狀態（即使不關心最終值）
- **`start_async/3` vs `assign_async/3`**：後者適合 mount 時載入；前者提供更細粒度的啟動/停止控制
- async assigns 封裝了：啟動異步 task、進程連結、trap exits、取消 task 等所有樣板程式碼

## 設計目標實現

- UI 進程保持不阻塞
- LiveView 關閉時，async 進程自動停止（linked process）
- async 進程崩潰**不會**影響 LiveView UI
- 支援取消執行中的任務
- 重視中間副作用而非最終結果

## 提及的實體

- [[Phoenix LiveView]] — 核心框架
- [[Elixir]] — 程式語言

## 提及的概念

- [[LiveView 異步操作]]
- [[Elixir 進程模型]]

---
title: "Elixir 進程模型"
type: concept
tags: [elixir, erlang, otp, concurrency, actor-model, fault-tolerance]
sources: [2023-11-09_liveview-async-assigns, 2025-03-25_cyanview-elixir-superbowl, 2019-04-29_k8s-erlang-vm, 2023-12-04_liveview-leaflet-12k-markers]
updated: 2026-04-07
---

# Elixir 進程模型

[[Elixir]] / [[Erlang VM]] 的核心執行模型，基於 Actor Model：每個**進程**是獨立、輕量的執行單元，透過訊息傳遞（message passing）通訊。

## 核心特性

- **隔離性**：進程間不共享記憶體，一個進程崩潰不影響其他進程
- **輕量**：可同時存在數百萬個進程（不是 OS 執行緒）
- **PID**：每個進程有唯一 ID，可以接收訊息
- **訊息傳遞**：`send(pid, message)` / `receive do ... end`

## OTP Supervision Trees

進程可以被 Supervisor 管理：
- 子進程崩潰時，Supervisor 自動重啟
- 可以設定策略（`:one_for_one`、`:rest_for_one`）
- 實現「Let it crash」哲學——與其防禦性編碼，不如讓崩潰快速發生並自動恢復

## 在 Wiki 來源中的實際應用

### LiveView 的 PID 傳遞模式
來自 [[sources/2023-11-09_liveview-async-assigns]] 和 [[sources/2023-12-04_liveview-leaflet-12k-markers]]：

```elixir
# LiveView 本身是一個進程，有 PID
live_view_pid = self()

# 異步任務透過 closure 捕捉 PID，完成後回傳資料
Task.async(fn ->
  # 做一些工作...
  send(live_view_pid, {:data_received, result})
end)
```

這是 Elixir 進程模型最優雅的體現之一。

### Cyanview 的容錯設計
來自 [[sources/2025-03-25_cyanview-elixir-superbowl]]：
> 「如果一個攝影機連線有 blip，其他一切繼續運作——這就是 Supervision Trees 提供關鍵優勢的地方。」

### K8s 的互補
來自 [[sources/2019-04-29_k8s-erlang-vm]]：
- K8s：叢集層的容錯（節點崩潰）
- Erlang Supervisor：應用層的容錯（局部資源失敗）
- 兩者運作在不同抽象層次，互補

## 相關概念

- [[OTP Supervision Trees]]
- [[K8s 與 Erlang VM 的互補關係]]
- [[LiveView 異步操作]]

## 出現在以下來源

- [[sources/2023-11-09_liveview-async-assigns]]
- [[sources/2025-03-25_cyanview-elixir-superbowl]]
- [[sources/2019-04-29_k8s-erlang-vm]]
- [[sources/2023-12-04_liveview-leaflet-12k-markers]]

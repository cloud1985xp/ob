---
title: "Kubernetes and the Erlang VM: orchestration on the large and the small"
type: source
tags: [kubernetes, elixir, erlang, distributed-systems, infrastructure]
sources: [2019-04-29_k8s-erlang-vm.md]
updated: 2026-04-07
---

# Kubernetes 與 Erlang VM：大尺度與小尺度的編排

- **來源**：[Dashbit Blog](https://dashbit.co/blog/kubernetes-and-the-erlang-vm-orchestration-on-the-large-and-the-small)
- **作者**：[[José Valim]]（Elixir 創始者，Dashbit 創辦人）
- **發布日期**：約 2019-04（確切日期未標注）

## 核心摘要

K8s 和 Erlang VM 都有「自我修復」、「水平擴展」等關鍵字，但它們**運作在不同的抽象層次**，大多數情況下是互補而非競爭。

> 「K8s 之於 OTP，如同區域容錯移轉之於 K8s。它們在不同的抽象層次運作。OTP 允許在**單一實例內**處理局部故障，這是 K8s 做不到的。」— Fred Hebert

## 互補關係逐項分析

### 自我修復（Self-healing）

| 層次 | 工具 | 範疇 |
|------|------|------|
| 叢集層 | Kubernetes | 節點/容器崩潰 |
| 應用層 | Erlang/Elixir Supervisors | 進程/連線/資源局部失敗 |

**關鍵洞察**：K8s 健康檢查只能偵測整體故障。部分資料庫連線偶發失敗？K8s 看不到，Erlang Supervisor 可以處理。

### 服務發現

K8s 提供 service discovery → 配合 [libcluster](https://github.com/bitwalker/libcluster) → **自動化** Distributed Erlang 節點連接，無需手動配置。這是兩者理想的協作點。

### Distributed Erlang 的優勢場景

同質系統（同一服務的多個副本）間直接廣播狀態（例如追蹤哪些用戶在同一聊天室），無需 RPC 協議、序列化、連線管理。這是 Phoenix Channels/PubSub 高效的原因。

### 部署：Rolling Update vs Hot Code Swapping

兩者衝突：不可變容器 vs 熱更新。**建議**：大多數 Elixir 應用不需要熱更新，使用 blue-green/canary 部署即可。熱更新的實際成本（狀態遷移測試）往往被低估。

### Pod 資源配置的關鍵提醒

其他語言常把一個大節點分成多個小 pod。**Erlang VM 例外**：建議給 Elixir 應用分配**大 pod**，因為 Erlang VM 擅長同時管理 CPU 和 I/O 並發，小 pod 反而浪費其能力。

## 提及的實體

- [[José Valim]] — 作者，Elixir 創始者
- [[Erlang VM]] — 核心平台
- [[Kubernetes]] — 叢集編排工具
- [[Elixir]] — 語言

## 提及的概念

- [[Elixir 進程模型]]
- [[OTP Supervision Trees]]
- [[K8s 與 Erlang VM 的互補關係]]

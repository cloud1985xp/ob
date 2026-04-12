---
title: "Cyanview: Coordinating Super Bowl's visual fidelity with Elixir"
type: source
tags: [elixir, case-study, embedded, broadcast, reliability]
sources: [2025-03-25_cyanview-elixir-superbowl.md]
updated: 2026-04-07
---

# Cyanview：用 Elixir 協調超級盃的視覺一致性

- **來源**：[Elixir Language Blog](https://elixir-lang.org/blog/2025/03/25/cyanview-elixir-case/)
- **作者**：Lars Wikman
- **發布日期**：2025-03-25

## 核心摘要

比利時小公司 Cyanview（9 人團隊）用 [[Elixir]] 打造了攝影機遠端控制系統，被奧運、超級盃、NFL、NBA 等頂級直播活動採用。本文是 Elixir 在**嵌入式 + 即時廣播**場景下的真實案例。

## 業務背景

- **Camera Shading**：直播活動中調整每台攝影機的顏色、曝光，確保視覺一致性
- 200+ 台攝影機同時協調；每台都不能失敗
- 系統部署在 Olympics、Super Bowl、NFL、NBA、ESPN、Amazon，甚至巴黎時裝週

## 為什麼選 Elixir

- Erlang VM 天生為「透過網路可靠協調數百萬裝置」而設計
- Elixir 對**二進位資料的位元級編碼/解碼**提供實用工具（整合各種專有攝影機協議）
- **快速迭代**：能在短時間內原型化新功能並在客戶現場驗證

## 關鍵技術決策

- 使用 IP 網路而非傳統 RF / 串列連接→可支援跨洲遠端製播
- 裝置間使用**自定義 MQTT 協議**通訊
- 單一 RCP 上 100+ 台攝影機無問題
- 系統架構：Yocto Linux + Elixir + C（低階色彩科學）+ FPGA
- Controller Web UI 已用 **Phoenix LiveView** 實作，在低規嵌入式 Linux 上表現優異

## 核心洞察：Elixir 的容錯性

> 「如果一個攝影機連線有問題、協議有 bug 或實體連線斷開，其他一切繼續運作是極其重要的。這就是 Elixir 的 supervision trees 提供關鍵優勢的地方。」— David Bourgeois（創辦人）

## 奧運北京案例

Panasonic PTZ 攝影機協議不是為網路設計的——需要精確時序和多條訊息。Cyanview 在北京裝置旁放 RCP，在巴黎透過 IP 控制，完美解決問題。

## 團隊規模效率

9 人團隊支援全球最大直播活動，沒有任何行銷，純靠口碑擴散。

## 提及的實體

- [[Elixir]] — 核心語言
- [[Phoenix LiveView]] — Controller UI
- [[Erlang VM]] — 底層平台

## 提及的概念

- [[Elixir 進程模型]]
- [[OTP Supervision Trees]]
- [[Elixir 在嵌入式場景的應用]]

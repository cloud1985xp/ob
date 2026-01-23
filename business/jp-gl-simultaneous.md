---
tags:
  - business
  - planning
created: 2024-01-01
updated: 2025-01-23
status: active
---

2024 Sep, Character
- 09, 10, 12 (as buffer) next anniversary
- Character and its related things are all synchronous
  - related events/quests/items
- Campaign(?) could be still separated
  - ex: login bonus, package, stone sales

- one repository, multiple binaries (?)
JP Operation Flow
- decided by 4 months before




DeploymentFlow
Tool

# Issue

・要merge日本PR最花時間的地方是哪裡？(之前有聽說要處理conflict的地方，是不是這部分也有關聯？
- Conflicts 其實還好，最多1-2天應該就可以解決
  - 但有時比較棘手的是 rspec 的調整，主要是gl規格可能不同造成的 (非多語系，既有 spec 我們是跑日文)，像是時區？


・現在有透過jp-tw-server頻道積極共享情報，是否還有不足之處？or希望如何改善？

・PR(https://github.aktsk.dev/ishin-tw/ishin-server/pull/1755)有看到差異，想確認是否有想跟日本討論怎麼做比較好之處？

Globalization Implementation / Gasha Step
Field Length
Fixture
Memory Issue
- Option1: 整個重練，找出需要多語系的 api，只針對它們做多語化
- Option2: 維持現狀，僅用 feature flag 來控制 Globalizaton

・除此之外還有哪邊有差異，針對這些差異要採取什麼樣的方針之類的，希望可以討論類似這樣的內容
- Announcement
- Maintenance
- AdminTool
- config
- active_admin
  - some utilities added for gl QA team but not sure if they still using it
- GeoIp Restriction for IAP Store
- Some in process cached data need to support multilingual


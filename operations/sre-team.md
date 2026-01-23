---
tags:
  - operations
  - sre
created: 2024-01-01
updated: 2025-01-23
status: active
---


## 2024 2nd Half
兩位 mikoto se 釋出，基本是一定會進 ishin
其他專案沒有 se 的 seat

所以我這邊要安排個合理的人力配置
有效運用這些在 ishin 的人力
一部分的比例仍會是會 common / 跨專案上 (要比以前更多)
一部分是用來提高 ishin 品質

要安排事情 ishin 需要

更多人力可以加強在 code quality
tooling automation
減少每個人的 context switch，可以更專心

Rosetta neon / ishin
Masterdata workflow neon / ishin
AI translation
AI application
prophet
Starfish

Ruby or Elixir
全面走向 ruby
or
打造平台工程，任意語言開發部署上平台

對 ISHIN 的影響
- 將 polunga 走向微服務化，各項功能用不同語言開發
- 懸宕的機能開始加快導入
	- masterdata 製作流程 workflow
	- rosetta 翻譯管理
	- 更多時間跟 neon/hokusai 做同步化
目前意料外的事：
- v5.21, v5.22 幾乎平行在做，直接佔掉2個人力，而且問題會互相影響

ISHIN 的配置
施策：讓 Jeff 變成有能力做版更
若全同步情況下，常態化會同時有兩個版本 overlap 在進行版更
ex: v5.20.2 + (v5.21.0 & v5.22.0 )
這樣需要有另一位至少像 tsungyi 可獨立負責一個版本，
- 儘管直接佔滿一個人的時間
- 頂多另外開發一些小功具、jenkins

可以將 Lyla 更多時間釋出，著重在
- 統籌維運工作，
- 工具調整
- infra 變動追蹤 <- 因為


Jeff 的狀況
ruby 不熟悉，
但 jeff 對事情的溝通及當握度很高
對適合對問題研究解決方案
也許研究 ai 來做維運強化是不錯的方向

Chuck 的狀況
原目標是主要處理 infra 工作，但需求最大的 mikoto 結束了
加上他又是無法獨立運作，做事情等於需要另一個人花工時來 cover 他的

Akash 的狀況
來到 ISHIN 應該會輕鬆很多
不需要扛溝通
可以專注在開發和解決問題


如果真的要減少一個 seat

理想中的
應該是把全部翻譯進 db
這邊開發一個系統，介於 lqa 製作端 與 application 置入端
這樣 lqa 可以維持一貫的流程
應該 application 端怎麼接資料，都是去讀整系統的流程




Team I
Lyla
Tsungyi
Chuck

Team M
Akash
Jeff
Aaron

### 2022-23

Team I
Albin
Newbie
Matt
Lyla

Team M
Akash
Aaron
Jerry
Matt


期望能以不再參加 ishin 例會的形式
所以主動觀察&發現問題的責任會落在 Lyla 身上
察覺問題，來跟我討論，擬定對策
或者說要建立好 monitor 機制，能觀察到大部分現在發生的問題
ex:
infra alarm, loading
quality
error happen
velocity / productivity 下降、工作排程追蹤
toil 增加
tool reliability
tool / workflow improvement

# 談談對面試工程師期望的特質
  正直、紀律性
最忌諱隱暱不通知
bad smell 的察覺意識
報相聯
各種我不知道、我不確定、workaround、走捷徑、不合理的設計

# 先訂 sli
建立 sli monitoring 的方法
再追求目標 slo

# 錯誤分級
P0, P1, P2, P3
連動 oncall


問題的類型
- Function 有問題
  - Server / Client Function
- Master data / assets 有問題
- Loading 造成(效能)問題
- 第三方，AWS, Apple, Google 問題
- 其他情況

影響的層級
- CS來信的問題
- 影響版更測試
- 影響送審
- 影響下一次的 release
- 影響外部，直接影響使用者

處理問題可能要花費的時間
- 既定的解決問題，例如重開、重部署
- 已知方向，不確定要花多久，例如 query 需要時間
- 不知方向，不知要花多久，樂觀/悲觀


是否為層級X以上
是否需要多位人員投入
是否需要全部人停下手邊的事情來投入


模擬 unicorn 被 kill 的情境
模擬 deploy replace task 時 發生 auto scaling


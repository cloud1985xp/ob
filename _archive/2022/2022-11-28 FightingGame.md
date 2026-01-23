---
tags:
  - archive
  - 2022
created: 2022-01-01
updated: 2025-01-23
status: archived
---

還記得 EVO Moment #37 帶來的感動嗎

先來複習一下

上次分享 Nakama Server 時，介紹了 tick 的概念
那時曾有討論到格鬥遊戲的設計
曾經推測，招試的設計是以 frame-based，而且為了處理延遲，推論會有
前搖、後搖的設計，並可以做為延遲的容許值


2k23 的 Jordan Legendary


看了 Youtube 上的分析與練習

以及快打旋風六的展示
更讓我確言了這一點



Combat System Design


Balance
Character Attribute
- Input
- Animation
- Meter

Skill(Action) Parameters

八方向搖捍

Term
Poke
硬直，霸體
受身
攻擊判斷
counter,
反擊
無敵判定、無敵幀
Hitbox, Hurtbox
確反
立回
目押
取消
派生(v)(n)



State Machine
招式參數
招式派生





https://www.facebook.com/legacy/notes/505497396132702/
例如：
SF4的昇龍拳，隆的發生3F，肯的是4F，沙卡特是5F
所以隆的昇龍拳最快性能最好
KOF XIIICM草薙的琴月被擋住是-7(聽說的應該啦)，對手只要用發生7F以下的招式就可以確反了(所以隆，肯，沙卡特的昇龍拳是可以確反的(跑錯棚了吧?))
SSF4AE楊的EX絕招步伐(236PP)對方擋住是+1，這代表著不只是沒有破綻，亂動還有可能被對方攻擊

F數除了有助於判斷敵我的優勢與破綻，目押連段的成立與否及其難度也是依據F數來判斷
例如：SF4隆的下中手擊中對手之後+5F(擊中對手後隆還有5格的時間可先動)
昇龍拳發生3F，可以目押且有2F的容許失誤時間(2F目押)
下中手發生4F，可以目押且有1F的容許失誤時間(1F目押)
下重腳發生5F，可以目押但沒有任何容許失誤時間(0F目押)
立重手發生8F，不能目押
所以下中手打到對手之後只要是發生時間小於或等於5F的招式都可以造成目押連段
反之發生時間超過5F的招式就不能與下中手構成目押連段了

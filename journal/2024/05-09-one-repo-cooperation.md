---
tags:
  - journal
  - 2024
created: 2024-01-01
updated: 2025-01-23
status: active
---


這份文件主要說明接下來 one-repo，GL SE team 期望要如何和 JP 一起使用 ishin-jp/ishin-server repo 協同作業

# 目標：
希望接下來將開始與日方共同 ishin-jp/ishin-server 這個 Github Repo，`僅用不同的 Git 分支來區分日版與國際版` ，並且，`最終期望`日版與國際版的分支 `在程式碼的部分是 100% 相同的`，差異的部分僅限於由 masterdata 產生的 fixture 與 i18 locales 檔案，這也是我們需要區分分支的唯一目的

我們把達到最終期望分成兩個階段

第一階段
GL 可以開始與 jp 同樣在 ishin-jp/ishin-server 上作業
但這階段仍有部分的 one-repo PR 尚末全部 merge 進 jp 的主分支，其中包括
- 先前發給 jp 有關國際版的各項功能調整
- 後續在各版本更新期間，發現需要的修正

因此，在這個階段期間，國際版會有暫時的分支，來當作 gl 版的 base branch (後面說明)

第二階段
前述的所有 pr 都已 merge 主分支
所以基本上國際版不會再有自己的程式分支
任何國際版需要的修改，應該都走和日版原本的工作流程 (也是未來想要了解的)

而 5.21 這版本我們希望先進入到第一個階段

# Git Branch 的使用

## GL 所理解的 JP Workflow
首先是我們對日版 branch 的使用的理解 (若有錯誤請指正)

我們大概的理解是未來開發中的新版本有各自的 branch 及測試環境
(在此先建立共同的詞彙，jp 的各版本開發 branch，下面我們先會用 `feature branch` 來稱呼它們，包括像是 freeza、pilaf、shenlong..等)

而營運上，則是用 pre -> pre2 -> staging  的順序以及測試資料內容並釋出到 production




## GL 國際版現在的 branch 作業流程

*可能需要先知道前提*
> 國際版的 masterdata 是以 csv 的形式存放在另一個 repo：ishin-tw/ishin-master，主要由 planner 使用以及實行版本控管


主要的分支 develop - master - stable
- develop：版更分支，即下一次版本更新的程式版本
- master：營運分支，現在營運中版本，接下來即將釋出的內容，等同 staging 環境的內容，
- stable：最新已釋出的版本，等於現在 production 環境的內容

其他的次要分支則是作為測試 masterdata 使用的 branch，我們也是用七龍珠的角色名稱來命名
它們都是從主分支流 (develop or master) 切出來的 ，在這些分支上不會直接有程式碼的變動，所有變動是來自 planner/qa 們想要測試的 masterdata 內容

基於 develop 的資料測試分支
- gogeta: 版更 QA 將要測試的 masterdata 製作進這個 branch，進行部署&測試

基於 master 的資料測試分支
 - bulma/chaozu/beerus：Planner與營運QA將未來要釋出的 masterdata 置入這些 branch 部署測試。(除了這三個也還有其他，但視 planner 們的需要來使用)

當完成測試後，planner 會將已在 bulma/chaozu/beerus 上測試好的 masterdata，置入到 master branch，部署至 staging 複測。

最終，當 staging 的內容順利釋出到 production 後，會將 master merge into stable branch

簡單來說
- 最主要的分支流就是 develop -> master -> stable
- 而所有的資料測試 branch (gogeta/bulma/chaozu/beerus) 都是隨時可以刪除重建的 (因為有 ishin-master 作為資料來源)

# JPxGL One-Repo 之後的流程


首先是第一階段

global-develop 分支：做為國際版下個要釋出版本的工作分支，它追蹤來自 develop 或是 feature branch (ex: freeza、shenlong，視現在正在哪個 branch 開發而定)。

在第一階段，有部分尚未完整 merge 進主分支 的 onerepo PR
GL Team 就會先把它們 merge 進 global-develop 

然後 gogeta 同樣就變成從 glolal-develop 切出，只有 masterdata 資料測試的變動



第二階段
當 onerepo 相關的 pr 都順利 merge 進 jp/ishin-server 主分支 (develop) 之後 
前述的 global-develop 理論上就不再需要了

所以

gogeta 就是直接從 feature branch 或是 develop branch 切(checkout out) 出來


## 在各個階段修正可能的作法

## 可能的影響

- 在 repo 中增加 GL 的 main stream branch
	- global-develop (第一階段，第二階段後刪除)
	- global-master
- 在 repo 中增加 GL 環境測試 branch
	- chaozu/bulma/beerus/gogeta ...etc，
	- 這些 branch 在測試期間會有大量的 masterdata 變動，也許 CI 可以略過？
- 其他(大家一起幫忙想想)

## 其他討論

- hotfix workflow




所以國際版與日版的差異，應該只限於 masterdata 更新產生的資料，包括
- db/fixtures 下的 .rb 資料 (國際版放置在 en/zh/ko/fr/de/es directory 下)
- config/globalization/locales 下的多語系資料 (日版不需要這些資料)

除此之外 application 的程式，
- config/gl 下的設定檔

為何選擇 v5.21
因為排程上 v5.20 - v5.21 之間有較長的準備時間，相較於 v5.21-v5.22 只有約兩週的時間

但沒有一定要在 v5.22 完全地進入第二階段，

# 國際版現在的流程


gl team 認至的日版現在的流程


gl team 提出未來整合在一起的流程




Next Action (TBD)

- GL Team 會在 ishin-jp/ishin-server 下從 v5.21 checkout 成 global-develop
- 將尚未完全 merge 的 onerepo PR，merge 進 global-develop
	- 以及一些在 gl v5.21 可能需要的在調整，base on global-develop 修改
- 從 global-develop checkout 出 gogeta branch
- 讓 GL QA team 用 gogeta 進行版更測試
- 後續任何有關 v5.21 的修正，GL team 會
	- merge into global-develop -> gogeta

等進入維運測試
- 從 global-develop checkout 出 global-master
- 從 global-master checkout 出 bulma/chaozu/beerus 給 GL Planner/QA 測試
	- 確認該次要 release 的 masterdata 測試完成後，將 masterdata commit 進 global-master，部署至 staging 測試
版更釋出當日
- 從 staging release to production





具體來說

I. JP Code OneRepo
II. AdminTool to Polunga Tool
III. Cooperation with JP SE Team


# Phase I
- 國際版啟用 one repo 版本，使用 jp repo 操作更版及維運
- one repo 仍有部分 PR 尚未合併進 jp 主分支

## 版更作業
- 增加 develop-onerepo 分支
	- 基於 develop 分支，將未合併的 onerepo PR合併至此分支
	- 切出 gogeta branch
	- gogeta branch 用於 seed 國際版 fixture 進行版更功能測試

### 已 FF 才開始版更測試
簡單，但大部分不會這麼晚才開始
- develop merge into develop-onerepo
- develop-onerepo merge into gogeta
- qa 在 gogeta 上測試直到 CF

### 當仍未 FF 就要進行測試時
大部分是這種情況
- develop-onerepo 追蹤日版開發分支 (ex: freeza)，merge 最新變動
- develop-onerepo merge into gogeta
- qa 在 gogeta 上測試直到 CF

## UPG 測試
不確定日版有無這段
global-staging -> upg + seed upg (before) fixture
develop-onerepo -> upg + seed upg (after) fixture

## CF 後，開始維運測試
- develop merge into develop-onerepo
- develop-onerepo
	- merge into (or checkout to recreate) bulma/chaozu/besrus 
	- merge into global-staging


## Staging 測試
planner 將資料 update 進 global-staging
部署至 staging 測試

## Release
從 staging copy image to production & 釋出
global-staging merge into global-master，打 tag


## 需要 bugfix / hotfix 時

Q0 若日版&國際版都未釋出，需要 hotfix



Q1 日版會修進 pre OR？
- 這樣 gl 沒有任何是 upstream 追 pre 的，若追的話會變成連日本的 fixture 也進來 = 時常有新變動，假設 gl (bulma/chaozu/beerus) 是追 develop，跟 pre 比較像平行的時間線


Q2 若日版已經釋出，需要 hotfix，國際版還在版更測試


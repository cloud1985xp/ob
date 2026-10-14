
我已調整了 26-27 下半年的 server cost 預估

- 上半年的 on-demand 費用(美元)已經發生，我將後台看到的帳單數字列入 AWS Real(M欄)以綠色標記
- 下半年的 RI 是新一期的費用，原先預估是用前一年的 RI 來計算，但今年已採買新的方案，故金額做了調整 (每月¥10,496,040 ->¥8,396,832)，但這是從 RI 採買的清單日方寫的報價金額來推算的，不完全等於會計實際收到的帳單
- 下半年的 OnDemand 預估依新的 DAU 為 base 做了調整，以藍色字標記
	- 今年 RI 購買減少，有一部分的費用會移到 ondemand
		- Budokai 機器 -> 目前仍未關閉，會有額外費用
		- RDS DataUsage 的成本，過去是用 RI 購買，今年我們調整模式買 OnDemand 方案
- 10月、11月、12 月，DAU 約 350k: 
	- 預估ondemand 費用會落在 50k + 55k 
	- 唯 10 月因為延後導入 RI 所以有一筆較大額外費用(即之前討論的方案C)
- 旺季 1月、2月、3月，DAU 450k~600k
	- 需加開 RDS 升級 writer 機器，參考今年 GDCP 做了調整

整體來說 (以下含稅 、匯率150)
- 上半年: 原預估費用約 154.6m，實際約 128.4m , -16.9%
- 下半年: 原預估 167.1m，目前下修至約 132.1m, -20.9%

若只看 AWS 的部分
- 上半年: 原預估 145.8m，實際約 120.1m, -17.6%
- 下年年: 原預估 157.0m，目前下修至約 121.4m, -22.6%


# PromoCard Preview Dev
https://github.com/aktsk-pjt-ishin/tw-ishin-devops/blob/feature/ISHINTW-15490-promo-chara-info/web/promo-chara-info/BACKEND_API_SPEC.md

資料來源

- jp csv

支援多語言？
- 若遇到該語言沒有的資料，要可以輸入？



目前的 card-promo 是依據以下路徑這個參考專案實作的
路徑：/Users/aaron.kuo/aktsk/ishin-tw/ishin-devops 
裡的 `web/promo-chara-info` 目錄裡的程式

但當初實作的版本是參考路徑的 `bcab9fc60bd0f8b9cb2155e342cea6b00290cef1` commit 的這個版本實作的
而該專案最近又進行了修改與調整

請先詳細了解該專案後來的修改內容 (即 bcab9fc60bd0f8b9cb2155e342cea6b00290cef1..HEAD)
並分析評估後，將變動整合進當前專案的 card-promo 功能之中

# Potara

## Skill of Merging Server Bots

我要在 potara plugin 中新增一個新的 still
請參考波特裡的合併 server cookbooks 的 skill (tw-ishin-merging-server-cookbooks) 與合併 analysis code (tw-ishin-merging-server-analysis) 的 skill
製作另一個類似也是在進行 merge git repo 上游版本的skill
大致流程會和 merge-server-analyisis 一樣，但更精簡。

名稱: tw-ishin-merging-hubots

這個 repo 所 merge 的程式碼，主要是以 type scripts 為主。
- merge 後不需要執行額外的任務，例如Rake指令。
- 一樣需要產生對應的Note
- 一樣需要有自己的 Reference 資料做為 domain knowledge 與 iron rules

大致流程：
- 決定工作 branch 名(merge/{target_version})
- 決定上游 remote 和 branch，預設 remote=jp branch=master
- 決定本地 base branch，預設為 develop
- 檢查目標工作 branch 是否存在，若不存在則從 base branch(develop) checkout merge/{target_version}
- 若已存在，在工作 branch 進行 merge
- 將上游 branch merge 進工作 branch
- 依 reference 中提的原則解衝突(如果有衝突的話)

若想要理解目前現有的 repo 內容，可參考
> /Users/aaron.kuo/aktsk/ishin-tw/ishin-bots

請幫我規劃這個 skill 的 schema 與，整個資料結構
讓 agent 可以用這個 skill 完成該 repo 的 upstream code merge 工作

若有任何問題請與我討論

## Skill of Merging Server Cookbooks

我要在 potara plugin 中新增一個新的 still
請參考波特裡的合併 server code 的 skill (tw-ishin-merging-server-code) 與合併 analysis code (tw-ishin-merging-server-analysis) 的 skill
製作另一個類似也是在進行 merge git repo 上游版本的skill
大致流程會和 merge-server-analyisis 一樣，比 merge-server-code 精簡。

名稱: tw-ishin-merging-cookbooks

這個 repo 所 merge 的程式碼，主要是以 ruby + shell scripts 為主。
- merge 後不需要執行額外的任務，例如Rake指令。
- 一樣需要產生對應的Note
- 一樣需要有自己的 Reference 資料做為 domain knowledge 與 iron rules

大致流程：
- 決定工作 branch 名(merge/{target_version})
- 決定上游 remote 和 branch，預設 remote=jp branch=master
- 決定本地 base branch，預設為 develop
- 檢查目標工作 branch 是否存在，若不存在則從 base branch(develop) checkout merge/{target_version}
- 若已存在，在工作 branch 進行 merge
- 將上游 branch merge 進工作 branch
- 依 reference 中提的原則解衝突(如果有衝突的話)

若想要理解目前現有的 repo 內容，可參考
> /Users/aaron.kuo/aktsk/ishin-tw/ishin-cookbooks

請幫我規劃這個 skill 的 schema 與，整個資料結構
讓 agent 可以用這個 skill 完成 ishin-cookbooks repo 的 upstream code merge 工作

若有任何問題請與我討論

## Skill of Merging Server Analysis

我要在 potara plugin 中新增一個新的 still
請參考波特裡的合併 server code 的 skill (tw-ishin-merging-server-code)
製作另一個類似也是在進行 merge git repo 上游版本的skill
大致流程一樣，但比較精簡。

名稱: tw-ishin-merging-analysis

這個 repo 所 merge 的程式碼，主要是以 shell script 為主。
- merge 後不需要執行額外的任務，例如Rake指令。
- 一樣需要產生對應的Note
- 一樣需要有自己的 Reference 資料做為 domain knowledge 與 iron rules

大致流程：
- 決定工作 branch 名(merge/{target_version})
- 決定上游 remote 和 branch，預設 remote=jp branch=master
- 決定本地 base branch，預設為 develop
- 檢查目標工作 branch 是否存在，若不存在則從 base branch(develop) checkout merge/{target_version}
- 若已存在，在工作 branch 進行 merge
- 將上游 branch merge 進工作 branch
- 依 reference 中提的原則解衝突(如果有衝突的話)

若想要理解目前現有的 repo 內容，可參考
> /Users/aaron.kuo/aktsk/ishin-tw/ishin-analysis

請幫我規劃這個 skill 的 schema 與，整個資料結構
讓 agent 可以用這個 skill 完成 ishin-analysis repo 的 upstream code merge 工作

若有任何問題請與我討論


## PSD 字形大量轉換
https://github.com/psd-tools/psd-tools
https://psd-tools.readthedocs.io/en/latest/

To build ISHIN Code KM
https://github.com/safishamsi/graphify
# Env Tool 

https://github.com/jdx/mise

https://github.com/zhongwencool/observer_cli


Alternative of btop
https://github.com/nicolargo/glances

nvim with agentic workflow
https://github.com/olimorris/codecompanion.nvim

nvim with MCP
https://github.com/ravitemer/mcphub.nvim

AutoResearch
https://github.com/karpathy/autoresearch

Skills

Elixir 
https://github.com/oliver-kriska/claude-elixir-phoenix

SDD
https://github.com/garrytan/gstack/blob/main/docs/skills.md
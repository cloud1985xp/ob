
# PromoCard Preview Dev
https://github.com/aktsk-pjt-ishin/tw-ishin-devops/blob/feature/ISHINTW-15490-promo-chara-info/web/promo-chara-info/BACKEND_API_SPEC.md

資料來源

- jp csv

支援多語言？
- 若遇到該語言沒有的資料，要可以輸入？


# Potara

## Merging Server Analysis

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
讓 agent 可以用這個 skill 完成 ishin-analysis repo 的 up
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
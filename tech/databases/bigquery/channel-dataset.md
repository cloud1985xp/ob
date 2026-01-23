---
tags:
  - bigquery
  - database
  - gcp
created: 2024-01-01
updated: 2025-01-23
status: active
---


Channel
Channel::Setting

-> channel_setting.json, gl_channel_setting.json
Channel::Manager


klass.globalized? -> 這個方法失效
因為實際上 initializer 執行 #globalize_active_record_model 都無效了
因為所有 klass 都不是 extends ActiveRecord::Base
造成它們都沒有真的執行 klass.globalize
所以 klass 自身不會被標記 .globalized?

影響
在生成 csv2ruby , csv2i18n 時
是會依靠 .globalized? 來判斷 klass(ar_class) 是否有多語化來決定行為

可能解法
讓 .globalized 仍能正常運作

其他影響
- 沒有被執行到 klass.globalize 還有什麼副作用？
	- Tool Schema 工具會被影響，需要呼叫 klass.globalized_fields
	- globak:language.rake 會被影響
- 最近做的 csv2i18n 排除 client only field 是否會影響


raw field
在 globalize_simple_master_model! 中沒有區分一般 global field 或 global raw field

需確認
也許在轉 json 讀入時就直接與 object 型式讀入了
-> 而非以前 i18n 讀入時是字串，還需要做 serialize


#{field}__#{local} 的 reader method 移除
-> 應該沒有影響，那個只是方便除錯用k的


Dataset 載入多語言的關鍵調查

Channel::Setting 的 config (json) 檔中會標記 data_sources 的 loader 是否啟用 globalize
當 settings 在 build_data_sources 時，遇發現該 loader 有 globalize flag 時
會對 loader 執行 Channel::Setting#globalize(loader)

其中會對 loader options 加上 globalize_proc


## 其他待研究

- 現有多語言運作，包括 masterdata.rake 能否正常運作 (csv to translation json)
- Dataset 如何做到載入 diff 製作 future / scheduled translation 資料？
	- 包括如何產生 diff
	- 如何 apply diff
- Timezonize Function
- and More..?

ClientAssets, ClientDatabase, TutorialAsset 處理
多語言 = topic (ja 也被視為一種 topic)




## SimpleMaster
Table, OndemandTable
Loader, QueryLoader, MarshalLoader,
Dataset, PersonalDataset, RequestStoreDataset
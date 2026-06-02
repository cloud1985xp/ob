
# 修正 FullTranslation 裡如何決定 Term.translation_state

Term.translation_state，增加一種 enum 值為 :all_formatted

若執行翻譯沒有得到結果，應設為 :untranslated
若執行翻譯有結果，要看每個語言是在哪個stage 完成翻譯的

一般狀況下(沒有 segments)
若所有語言都是在 canonical stage 完成，設為 :all_existed
若有任何語言是在 model stage 完成，設為 :auto_translated
若所有語言都是在 pattern stage 完成，設為 :all_formatted
其他情況設為 :partially_existed


如果是有 segments，先用以下規則決定該筆單語言 parent 整體的代表 stage
- 若所有 segments 都是在 canonical stage 翻譯完成，視為 canonical stage
- 若有任一 segment 是在 model stage 翻譯完成，視為 model_stage
- 其他情況則視為 pattern stage
	- 等於沒有任何經過 model stage，但也不全是 canonical stage，

用上述規則，來決定 FullTranslation 在得到翻譯結果，如果決定 translation_state 寫回 term

# 整合 Miles Package 與 Rosetta TranslationBatch

一、優化 TranslationWorkerV2
TranslationWorkerV2 裡有許多處理的邏輯，
是不是應該要拆出到另外的 module, ex: Rosetta.Translations.Batches
Worker 自身應該只要負責讓包裝 async process (retryable) 的角色，
而出的 module 可以單獨被執行運作


# 修正 Translations 和 Translations.Batches 用法
目前啟動點 Translations.enqueue_v2_batch 的行為
是傳入 translation source，對它的 source_strings 建立成 batch 然後啟動(enqueue job) worker

但我需要的 TranslationBatch 功能是傳入 namespace + property + 要翻譯的 strings
這些 string 通常是新要翻譯的資料，而非現有的 source_string

或許要改成
Rosetta.Translations.Batches 有
create
- 傳入 namespace + property + strings
- 用 namespace + property 尋找 translation source，若沒有則 return :error
- 傳入的 strings 每筆有 id(來源的 identifier) + content (要翻譯的內容)
來建立：TranslationBatch + TranslationBatchString

process (= 現在的 run)
- 傳入 batch_id
- 進行 TranslationBatch 的實際工作
	- 檢查狀態、resume、執行 translation、persistent strings 等等

enqueue
- 傳入 batch id
- 建立 oban worker job
	- job 裡呼叫 process(run)

當是要翻譯 source_strings 裡的 content，再包一個 function 是
- 傳 translation source (maybe 加上 query scope 條件)
	- 取出該 translation source 的 source_strings
	- 轉成 id + string
- 呼叫 create batch / run batch 來實現

二、整合 Miles
原本的 Miles Packages module 中 #process 與 #process_ai_translation
實作了舊版的處理「文字轉換」、「查找 db 中既有翻譯」、「(分開地)執行 ai 翻譯」

此次修改是要將它兩個動作合併，並改成用 Rosetta Transaltion.Batches 來啟動
TranslationBatch 與 TranslatorV2 來完整整個「轉換+翻譯」的操作

預計新增 Packages.process_full 方法來實現
- 取出需要處理的 term，整理成 strings (id + content)
- 呼叫 Rosetta API 提供呼叫 Batches 的方法，
	- 傳入 namespace + property + strings (id + content) 
- 建立 TranslationBatch 並啟動
- 等候 batch 執行完成，得到結果
	- 將結果寫回對應的 term 的 translations
		- 寫回時，若回傳的翻譯資料，是有經過 segmentation


首先 Rosetta 端覺得也要做出以下對應修正

> 詳見 github issue 上


而 Miles.Packages.process_full 就直接呼叫 module 並等候完成
Miles 端的 async 管理則由它自己的 worker 來啟動




## 調整 TranslatorV2

當執行 translate 時若傳入的 subject(translation_source)，是有 segmentation (rule)s 時
需要把傳入的 subject，當作 root subject 放進 context 中來傳遞

承上，在 KotobaProvider 中，會需要：

一：上傳 input 時的目標 gs 路徑

要改成
- namespace，若有 root subject 要使用 root subject 的 namespace
- property，若有 root subject 要使用 root subject 的 property

來組成 gs 的目標路徑，格式調整為：
```
"gs://#{config.gcs_bucket}/generate/#{job_id}/input/#{namespace}.{property}/#{namespace}.{property}.ja.jsonl"
```


二、在呼叫 jenkins 執行翻譯 job 時
傳送 trigger job 的參數，要調整：
- namespace，若有 root subject 要使用 root subject 的 namespace
- property，若有 root subject 要使用 root subject 的 property

三、下載 jenkins 執行好的翻譯結果

一樣要改成
- namespace，若有 root subject 要使用 root subject 的 namespace
- property，若有 root subject 要使用 root subject 的 property

每個語言下載的路徑調整為：
```
"gs://#{gcs_bucket}/generate/#{job_id}/output/#{namespace}.{property}/#{namespace}.{property}.ja-#{lang}.jsonl"
```

請確認上述需求，理解後擬定修改的計畫再進行

## 調整 Trainings 的 Build Data 方法

當遇到 translation source 是有 segmentation rules 時，
會將所有子(拆分後得到的) translation sources 的 build data 後合併

我希望，仍維持有一個製作單一 translation source data 行為的方法 (即現在的 build_data)

但有另一個入口，是會判斷傳入的 translation source
檢查是否為有 segmentations ，若是需要拆分子 translation source，
會分別呼叫 build_data
後再將得到的資料合併，合併時要注意
- content、glossary 的各語言都要合併，各語言的資料順序仍然要對齊

而 Trainings.upload_and_trigger_training，是呼叫這個新的入口

# Review TranslatorV2

關於 Phase 4 的 TranslationPipeline 和 TranslatorV2
我有以下問題討論調整

1.
一次會處理大量的 Strings，如果將所有既有翻譯一次查詢放在 Context 資料結構中。
可能會有效能上的問題。j
請考慮在開始處理時，先將 Strings 拆分成多個批次來進行。

2
在翻譯整合服務 Integrations 模組的部分。
- Kotoba 比較像是一種 provider 。指的是自架的 model translation 服務，提供模型像是 mt5, gemma 之類的
- 其他則是第三方雲端 LLM provider API 的服務，例如 Gemini, ChatGPT，有M T Phi。

所以目前的 LLM provider module 放在 Kotoba 下可能不太適合，這邊的 llm provider 指的就是用其他第三方的 api 服務
請適當地調整 Integrations 下的模組結構

3 
請考慮使用更符合 Elixir OTP / GenStage 架構的方式來實作各個 stage 的處理。
分析評估使用這個套件
https://hexdocs.pm/broadway/Broadway.html
先詳細了解這個套件的用途，並評估提出它是如何適用(或不適用) TranslatorV2 的架構中
再做決定

上述想法請幫我仔細評估、若要進行調整
請以不要影響到 TranslatorV2 與外界構通的介面為前提做修改



# 目標

對 Rosetta 的翻譯資料/功能系統增加以下變動
- 支援對原文進行分割處理後再翻譯
- 增加 pattern base 翻譯方式
- 增加 m5 翻譯方式
- 增加兩種 LLM Translation Corrector
- 重新設計翻譯流程進行整合
- 將現有資料 migrate 生成 segment strings 和 formatted strings
- 增加產生 mt5 訓練資料集的功能

目前有兩個工作 branch 
- 當下的 branch，已有部分的實作(詳細後續說明)
- 參考 branch (origin/feature/translation-batch-checkpoint)，有大部分的實作，但程式碼太過繁雜且部分不太符合預望的設計

我需要你：
- 先理解目前 app/rosetta 的整體功能
- 詳細了解以下的需求說明
- 理解當下 branch 目前已實作的部分 (git diff between develop branch )
- 理解參考 branch 已實作的部分 (git diff between develop and origin/feature/translation-batch-checkpoint)
- 與我討論確認後，重新規劃架構、並有計畫地，逐步在當下 branch 實作完成整個需求

以下先說明此次要新增的功能需求

# 功能需求
## 一、擴展對原文進行分割處理
首先會定義 TranslationSource 的 SegmentationRule(s)
來做為對原文進行分割的規則：
> TranslationSource has many segmentation rules

而每個 segmentation rules 就會衍生出對應的 (子) translation source
例如有一 TranslationSource，其 {namespace, property} 為 {passive_skill_sets, itemized_description}

它擁有兩條 segmentation rules，分別名為
- condition
- effect

代表它會衍生出兩個 TranslationSource:
- namespace: 與 parent 相同的
- property: 用 parent name + "." name

例如
- {passive_skill_sets, itemized_description.condition}
- {passive_skill_sets, itemized_description.effect}

在處理特定 TranslationSource 的文字時，若該 TranslationSource 擁有 segmentation rules，就會依 segmentation rule 裡的 regex 將原文 (parent)拆成多個子 string

例如：
rule1 從 parent1 的 string 取出了 r1_child1、r1_child2
rule2 從 parent1 的 string 取出了 r2_child1、r2_child2、r2_child2

rule1 取出的 segment string，一樣會存入 source_string 資料表，並 belongs to rule1 衍生的 TranslationSource

同理 r2_child1, r2_child2, r2_child3 等 segment string，也會存入 source_string，belongs to rule2 衍生的 TranslationSource

## 二、增加 Pattern Base 翻譯方式

pattern base 翻譯方式是指當一些詞句有類似的格式(pattern)，即 `句型是一樣，只有部分關鍵字不同`，這時我們當做它們是當一種 format pattern

會將該 format pattern 和其對應的翻譯存成 formatted_source_strings 與 formatted_translated_strings
不同之處用特定的「符號」來代替，翻譯時就只要替代對應的符號

例如原文 (ja)  的 content 為
> 3ターンの間、気絶する

轉換的 formatted_source_strings 是
> [n1]ターンの間、気絶する

對應的六個語言翻譯 (formatted_translated_string) 是

| lang  | content                                       |
| ----- | --------------------------------------------- |
| es    | El personaje quedará aturdido por [n1] turnos |
| fr    | Pendant [n1] tours, s'étourdit                |
| zh_tw | [n1]回合內將陷入暈眩狀態                                |
| en    | Character will be stunned for [n1] turns      |
| ko    | [n1]턴 동안 기절한다                                 |
| de    | wird für [n1] Runden betäubt                  |

之後若遇到另一筆原文(ja) 要翻譯，例如
> 6ターンの間、気絶する

就會經過 formatted 轉換取得 formatted string 的格式
然後在 formatted_source_strings 資料中找到存在的資料和 formatted_translated_strings 的翻譯，
再把 `[n1] = 6` 替換回去，就會得到翻譯：

| lang  | content                                    |
| ----- | ------------------------------------------ |
| es    | El personaje quedará aturdido por 6 turnos |
| fr    | Pendant 6 tours, s'étourdit                |
| zh_tw | 6回合內將陷入暈眩狀態                                |
| en    | Character will be stunned for 6 turns      |
| ko    | 6턴 동안 기절한다                                 |
| de    | wird für 6 Runden betäubt                  |

目前我們只實作以「數字」為關鍵字的處理方式，我們稱此方法為 n1，
未來可能會支援更複雜的 pattern

一個句子裡可以有多個被替換的數字符號

例如
> ATK +20%，氣力+3
> ATK+30%，氣力+5

都是屬於同一種 formatted_string：
> ATK+[n1]%，氣力+[n2]

這筆 formatted_source_strings 就會有對應的翻譯，內容一樣會用 `[n1]`, `[n2]` 來替代關鍵字

## 三、增加 m5 翻譯方式
原本的 Rosetta 僅支援外部呼叫 Vertex AI 來進行翻譯
這裡要再增加另一種外部服務，我們稱為 Kotoba
實際上是呼叫一個我們自建立的 jenkins job
對這個 job 傳入參數包括

- namespace: 即 TranslationSource 的 namespace
- property: 即 TranslationSource 的 property
- source_uri: 一個 Google Cloud Storage 的 path，來做為 input 需要被翻譯的文字資料檔案，檔案格式為 jsonl
- source_language: 翻譯的原始語言
- target_languages: 翻譯的目標語言，用 "," 間隔

系統這邊要做的事
- 產生一個 job id
- 將要翻譯的文字，生成 jsonl 格式的檔案
	- 傳到 cloud storage 上帶有 job_id 的路徑
- 呼叫 jenkins job
- 監控等候 jenkins job 完成
- 從 cloud storage 對應 job_id 的路徑下載檔案回來
- 將檔案中翻譯好的資料拿來處理或存入資料庫

## 四、增加兩種 LLM Translation Corrector
由於 n1 處理或 mt5 翻譯會有一些缺陷
因此我們會需要設計兩個工具模組，它是利用 LLM + Prompt 來幫助我們將翻好的資料再做校正，

一、N1 LLM Corrector
因為 n1 pattern 在處理過程中會把「換行符號」去除掉以方便比對
所以透過 n1 翻譯的結果是沒有換行符號的

N1 LLM Corrector 就是讓我們傳入原文與翻譯後的文字，
然後請 LLM 將原文中的「換行符號」適當地放回翻譯後的文字裡
這個會需要調校給 LLM 的 prompt 來實現
所以只要先規劃這個功能模組的雛型和介面即可

二、Glossary LLM Corrector
這是為了校正翻譯後的文字，是否有正確的套用系統給的 glosaary 用語
一樣會需要調校給 LLM 的 prompt 來實現
所以只要先規劃這個功能模組的雛型和介面即可

## 五、重新設計翻譯流程並整合

rosetta 已有實作翻譯的功能，位於 Rosetta.Translators，但目前功能並不完善，且僅有 model base (呼叫 LLM 模式) 的翻譯方式
要重新設計製作新的翻譯處理模組，並整合 pattern base 翻譯方法，以及擴充 model base 的 mt5(kotoba) 翻譯方法

###TranslationSource 增加 pattern strategies 與 segmentation rules
翻譯流程依然是以 TranslationSource 為主體(又稱 subject)，即一組 namespace + property 得到的 TranslationSource
而現在每個 subject 上會可以有兩種翻譯模式的設定值

- pattern_strategies: 是否啟用 pattern base 的翻譯方法
	- 目前設計成 bitmask，表示可以啟用多種 pattern base，但目前只會有 n1 這一種
	- 若空 list 或 nil 則代表不啟用
- (原本的) strategies: 是否啟用 model base 的翻譯方法
	- disabled: 不啟用
	- kotoba: 使用 mt5 模型
	- default: 使用 llm (簡單 prompt 的方法)
	- rag: 使用 llm + 搭配 glossary 做為 rag + prompt 的方法

此外 subject(translation source) 可以設定(has 0 to many) segmentation rules
經過 segment 又會衍生出下一層的 translation source，各自也會有自己的 pattern strategies or (original) strategies

### 新的翻譯流程

以下是新設計的翻譯流程

> TranslatorV2.process(translation_source, strings)

輸入：
- Subject (translation source) 要翻譯的文字是屬於哪種 TranslationSource
- 多筆要翻譯的文字 contents，每個 content 會帶有
	- content: 要翻的原文
	- identifier: 一個識別字串

首先要檢查該 subject 是否有 segmentations rules, 
#### 情境A: 沒有 segmentation rule
若 subject(TranslationSource) 沒有 segmentation rule，
直接將 subject + strings 傳入 TranslationPipeline 執行翻譯：

#### TranslationPipeline

基本上分成「Transform」、「Canonical Translation」、「Pattern-based Translation」「Model-based Translation」+「Correction」幾個動作：


1、先使用 Transformer 對每筆 string content 做 normalization，
- transformer 會定義一些依照各 subject 對原文做清洗資料的規則
2、先將每筆原文，直接比對 source_strings 資料表有沒有一樣的內容，
- 這個步驟(方法) 稱為 Canonical Translation
- 若有的話，直接取用這個符合的資料的翻譯( %SourceString{}.translations) ，剩下的(找不到翻譯)進到下一步
3、檢查 subject 是否有啟用 pattern base 
- 若有的話(目前只有 n1)，使用 n1 規則轉成 formatted string，用 formatted string 查找 formatted_source_strings 資料表是否有符合的資料，若有的話
	- 取得對應翻譯做 n1 資料替換
	- 並呼叫 N1 LLM Corrector 做校正
	- 校正後取用翻譯
- 剩下的(沒有啟用 pattern 或 pattern base 找不到可用的 formatted 資料/翻譯) 進到下一步
4、是否有啟用 model 翻譯 (strategy)，若沒有啟用就結果翻譯流程
- model 翻譯包括 mt5 或是 llm (一般 prompt 或 promp + rag) 都是呼叫外部 服務，所以會需要把前面3步無法完成翻譯的剩餘資料收集後，一起送去外部服務
- 視 strategy 的設定來決定要使用哪種外部服務
	- kotoba (mt5) 呼叫 mt5 翻譯服務，等候完成後，對每筆資料再使用 Glossary LLM Corrector 做校正，剛成後回存資料
	- default/tag (LLM) 呼叫 LLM 翻譯服務，等候完成回存資料
	  (llm 暫時不需要再呼叫 Glossary LLM Corrector，但未來可能視情況加上)
5、完成，回傳翻譯結果

上述流程，從 2 都會開始會持續 累積/更新「已翻完 vs 未翻完」的結果
每一步得到的翻譯都要紀錄是在哪一個步驟(經由哪個方法)完成翻譯的
每整 string 基本上都會有

- identifier: 一開始的識別符
- content: 翻譯原文，在 step1 經過 normalization
- translations: step 2 ~ step4 填入的翻譯
- metadata: step 2 ~ step 4 填入的元資料，包括
	- 在哪個階段/哪種方法進行翻譯
	- segmentation 資料(如果有的話)

#### 情境B: 有 Segmentation Rule
如果 Subject(TranslationSource)，有 segmentation rules

會先將每筆文字，用 segmentation rules 拆分出 segment strings 與每個 segment string 的 metadata
每個 segment strings 會帶有自己的 identifier，並可用這個 identifier，做為之後重組的資訊

拆出來的所有 segment strings，依 rules 對應的衍生 subject(translation source) 分組，各組再丟入翻譯流程 `Translator.process(translation_source, strings)`

各組的翻譯資料翻譯後，會依照分割時 metadata 將文字重組，成為真的的翻譯結果

若是經過重組的而得到的翻譯結果，也需要同時回傳各 string 重組前各分斷的翻譯資料資訊，該呼叫者可以進一步去做使用。

### Translator Batch and Checkpoint
一次 TranslatorV2 執行時有多個階段且耗費時間
會需要用 background process (Oban) 來處理
而且中間途有可能會中斷或失敗重跑

過程中應該要紀錄下已完成的部分，例如依每階段先將當下的結果與進度存入資料表，當作 checkpoint
一方面可方便追蹤進度、除錯
二方面若遇到狀況被中斷，下次重啟可以從失敗的階段繼續，不用全部重新跑一篇
## 六、將現有資料 migrate 生成 segment strings 和 formatted strings

因為系統有現有資料，需要建立

- 預設的 segmentation rules
- 經由 segmentation rules 衍生的 translation_sources
- 從即有資料依 segmentation rules 分割產生的 segment source_strings 和 translated_strings
- 對有啟用 n1 pattern 的 translation source，對其即有的 source strings/translations 產生 formatted source strings/translations

## 七、增加產生 mt5 訓練資料集的功能

另外，要實作一個將既有資料庫來產生 mt5 模型訓練資料機的功能
從 formatted_source_strings，搭配一些專案的規則
自動展開生成多條各種數字嵌入的 n1 字串
然後一樣生成 jsonl 檔案，傳送到 Google Cloud Storage 後
然後呼叫自建的 Jenkins Job

- namespace
- property
- cloud storage URI
- source language
- target languages

讓它啟動 job 進行訓練即可

# 工作規劃與進度

許多實作的細節，可以從參考 branch 中找到
但參考 branch 中的程式架構不太乾淨，所以我希望在當前這個 branch 重新實作或重構那邊的程式碼

並希望逐步完成功能方便理解與測試驗正
我規劃目前已完的進度包括：

一、資料表 migration
建立所需的資料表與 schema
(除了 translator batch 尚未建立)

二、Patterns 與 Segmentations 模組，既有資料產生
實作  Patterns 的純 function module，負責 n1 formation
實作 Segmentations 的純 function module，負責對 string content 做 segmentations
實作 DataSeed module，來完成上述的：六、將現有資料 migrate 生成 segment strings 和 formatted strings

註：參考 branch 裡有增加一些 seed .exs 程式或建立資料 mix task 可以先不要理


三、實作生成 training data
這段只做產生 training data，先不涉及格式 (csv or jsonl) 與上傳

以下則是尚未實作的部分，我目前是這樣規畫

四、實作整合 jenkins 進行 training
將三得到的資料，進行 cloud storage 上傳、呼叫 jenkins 執行訓練的動作
(可以先 mock jenkins 段，假設 jenkins 有正常運作)

五、實作整合 jenkins 進行 mt5 翻譯
將要翻譯的文字清單，上傳 cloud storage，呼叫 jenkins 執行翻譯
等候 jenkins job 完成，將對應資料下載回來
(可以先 mock jenkins 段，假設 jenkins 有正常運作)

六、實作 TranslationPipeline
處理 strings 的翻譯的 SOP：
- Transform, 
- Canonical Translation, 
- Pattern Translation + (N1 Correction)
- Model Translation + (Glossary Correction
要 {translated、untranslated} 兩者的資料在每一段 batch 收集進來

七、TranslatorV2 
整合 Segmentation Rule 做 Decompose, 
然後呼叫 TranslationPipeline
再做 Re-compose

以上是我目前的規劃，請參考，並且依整體的需求詳細評估
尤其有些是我沒有考量到的，或是參考 branch 裡有的關鍵內容也一併納入考慮
若有任何問題也請與我討論，擬定計畫確認後再開始進行





PR Issues
- 參數都是 opts，沒有序列或語意



Translations
DataSeed
Segmentations
- 處理將 string + translations 依 rules，再拆成更多 (segments) strings + translations
Patterns
- 將 strings 收斂成 formatted strings + translations

目前 Patterns 會同時做 segmentation 再做 formatted strings
- 耦合度太高
- 無法單獨執行
- 無法 batch 執行

Translations.Translator.Batch Process
Translations.Translator.Pipelines
- content strings + translation source
	- 有 segmentations
		- 拆 segements + seg translation source-> 再呼叫執行 pipeline 翻譯
	- 無 segmentations
		- -> 執行 sop 翻譯


第一層 domain module 會處理流程邏輯，子 module 只負責資料處理

Patterns -> 處理 batch 分批次
，每個批次會呼叫  Builder.build_formations
拿到 formations 處理存入 db
Builder 不負責存入 db



LQA/Planner 整理好 LQA Sheet 提出 request

- chara_sheet
	- Rosetta 依 subject 分組，篩選出需要採用 kotoba 翻譯的 subjects，對每個 subjects
		- 產生 temporate_sheet
		- 執行 generate request
			- 呼叫 generate_mt5_test_ishin
		- 完成 generate 後，將翻好的資料送回 rosetta
	- Rosetta 產生翻譯包
		- 對翻譯好的內容，使用 AI correction
		- 可對不滿意或一開始就標記的資料，採用 AI translation
		- 產生 AI 校閱、評分
		- 下載翻譯包




## Rosetta 支援 Kotoba

Subject

我把 Kotoba 定義為一種 pattern base 拆分 + n1 轉換 + 重組的機制
- 在生成翻譯端，會做 pattern base 拆分、查找翻譯、重組回原文
- 在訓練端，會做 pattern base 拆分、n1 轉換、產生 training set

然後 mt5 只是一種翻譯方法，跟目前 Rosetta 裡設定的其他 LLM(gemini) 一樣的一個外部服務
所以 jenkins + 4090 那段就是做成一個純做翻譯的服務

先說明目前 Rosetta 的產生 LQA 翻譯包功能，
對 Rosetta 來說一個詞條( term) 在處理時，都會經過以下動作

- T: 先做文字轉換 (Transform) 如果需要；例如原文有東西要被濾掉，或是 symbol 轉換
- C: 檢查概有翻譯 (我們稱 Canonical db )
- A: 若無既有翻譯 -> 嚐試做 Auto translation，視該詞條的 subject (table+field 或在 rosetta 叫 namespace+property) 決定要用哪種翻譯方法(模型)，例如用 gemini、 openai

加上 kotoba 機制，我覺得就是多一個 pattern base 做拆分 segment 的動作
其它流程應該可以繼續上面的 TCA

原本一段詞條 (Term) 進來，因為它的 subject(passive_skill_sets.itemized_description) 
設定有啟用 kotoba 機制

所以要對文字做 pattern base 拆分成更多個 segment
每個 segment 其實就又是另一個 term，

這裡故意叫 segment，是覺得這邊的概念跟我們在做 ai review 概念有點像，後面解釋可以透過 key(identifier) 的設計，可以還原重組的邏輯

拆分時會依照 pattern 分成 condition 與 effect，對應又是不同的 subject(translation source)

ex(TBD):
- passive_skill_sets.itemized_description#condition
- passive_skill_sets.itemized_description#effect

對每個 segment (term)，再跑一樣的流程

T: 無(TBD)，目前不需做事，反正跑原本的流程
C: 從 Canonical DB 尋找現有翻譯
A: 無既有翻譯，進行 Auto translation 

Auto translation 時，該 subject 設定用的方法是 mt5
所以會把這些 segment 收集後，批次送去外部 mt5 翻譯服務  (jenkins)
收集完成的翻譯後，把所有的 segment 依 kotobe strategy 反組回原文

和原本一樣，Auto Translation 設定用 gemini，也是收集詞條後發 batch request 給 Vertex AI 進行翻譯，等候結果 (只是不需要重組原文) 存回 term

另一個關鍵問題是 condition / effect segment
裡面的斷行，是否要再做拆分

若要的話就會要做兩層的拆分，反過來也是兩層的重組

但呼叫 mt5 那邊我覺得應該還是可以一次呼叫，


至於要不要對 condition / effect segment 做斷行拆分

若不拆就送去給 mt5 翻，現行 mt5 可能品質會下降
  現行做法則是把斷行去掉(當作單行)，翻譯結果也是單行，由 lqa 人工斷行
但這有可能經由 改用新的/更強的 local 模型或 Cloud LLM 得到解決

若拆了，就上面說的會要多做一次重組



但 segment 最後存進 rosetta 還是一樣會是 source_string 
這裡一個關鍵問題是 segment 存進 source_string 時，它的 identifier 會是什麼





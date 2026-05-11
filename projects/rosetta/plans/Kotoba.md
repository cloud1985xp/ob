# 從 TranslationSource 生成 TrainingData





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





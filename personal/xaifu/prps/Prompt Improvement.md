# 改良 Prompt 功能

對 GenerationInput 加上 position 欄位，作為在 Generation 下的 inputs 的排序

包括
- 建立 migration 新增欄位
- 在 Generation 的 form_component 中，要對每個 input 加上 position 欄位供編輯，在 save 時將值存入
- 在 Generation 取得其 inputs 時要依 position 排序，包括
	- 在 show 頁面顯示時
	- 在 Generations.process 時，使用 Generation.inputs 要照 position 排序

# Prompt width and height

在 prompt 上增加 width 對 height 欄位，允許為空值
並更新 prompt 的 form_component，在新增/編輯 prompt 時可以設定該 prompt 的 width 和 height 


## 在生成圖片時加上參照 Prompt Level 的尺寸設定

目前 Generations.process 與 Generations.preview 時，會呼叫 Processor.execute 或 Processor.generate_single 來產生圖片

這邊會透過傳入的參數或是 generation 自身的尺寸設定來決定生成圖片的 width 與 height
而當沒有指定的參數值時，會使用原本預設的尺寸

我想做出以下調整：
在當沒有指定時，會改從當下使用 prompts 上設定的 width 與 height
- 如果沒有任何 prompts 有設定尺寸，那就一樣用預設的
- 如果有多個 prompts 有設定，那會依照 position 排序(小到大)，使最後一個 prompt 的設定

所以運作方式大概是
- 所有 prompts，排除掉沒有設定尺寸的prompt
- 照 position 排序
- reverse
- 最第一個
- 若沒有資料就用預設(=不特定 opts)

注意，如果 generation 自身有指定，或 preview 時的參數已有指定尺寸，那就還是跟原本一樣，不需要使用 prompts level 的尺寸設定
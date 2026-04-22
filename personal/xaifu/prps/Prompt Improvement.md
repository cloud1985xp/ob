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



# Prompt 優化 4/18

一、調整 prompt 與 prompt category 的管理表單
點擊新增或編輯 prompt 時，改成完整獨立頁面，而非只在左側欄
為了讓編輯的 form 可以有較大的空間顯示 textarea 欄位

二、因為 prompt 一定會在 category 下，所以希望可以：
- 一定要先點選 category，才能新增 prompt
- 新增 prompt 是進入建立的表單，按下送出後才建立，而非先直接建立，再做編輯
- 新增/篇輯 prompt 成功之後，預設回到同一個 category 下瀏覽 prompt 清單
- 新增 prompt 時，有一個 checkbox 來代表「繼續新增」，如果有被勾選時，在新增成功後，會繼續停留在新增 prompt 的表單頁，讓使用者可以繼續新增 prompt 在同個 category 下
	- title、text_en、text_zh、width、height、remark 等欄位會清空
	- 繼續新增的 checkbox 會維持勾選
- 瀏覽 prompt 時，要有連結可以回到 category 瀏覽 prompt 清單

三、增加複製 prompt 的功能
在瀏覽 prompt 時，要有一個複製的按鈕
複製的動作等於是進入新增 prompt 的頁面，只是因為帶入要複製的來源，會把預設的資料填入，包括：category、title、text_en、text_zh、width、height、remark


## Prompt 增加 Likes 功能
- 在 prompt 上增加欄位來代表 likes 數量
- 在顯示 prompt 列表時依 likes 數從大到小排序
- 在 prompt 列表或瀏覽，加上對 prompt 進行 like or dislike 的按鈕，來增加或減少 likes 數
	- likes 數最小為 0 ，不會變成負的
	- 不用紀錄是誰按 like/dislike，可以重複的執行
- 在顯示 prompt 時 (列表或瀏覽)，加上顯示 likes 數


- [x] 複製 prompt 的功能，方便複製->微調 or 改尺寸
- [x] 加上 rank 或置頂、like？
- [x] 建立提示詞後，繼續在同分類下建立下一個
- [x] 瀏覽提示詞，在同一分類下 navi
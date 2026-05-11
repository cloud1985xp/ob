# Image2Text Batch Records to Prompt
一、增加將 image2text record 轉成 prompt 的功能
先實做一個模組函式，可以將輸入的文字，建立成指定 prompt category 下的 prompt
建立的過程中會依照所指的 prompt category 的 prompt category group 所設定的排除字
將輸入的文字進行排除，請將這個功能實作成一個 function 方便重複呼叫
先設計放在 Image2text Domain 下，與一般的 Prompts 分開

排除的邏輯：
將原輸入文字先用 `,` 拆開，並對每個 element 做 strip
將 CategoryGroup 設定的排除文字，用 `,` 拆開，並對每個 element 做 strip
將原輸入的文字拆開後的清單，扣掉排除文字的清單，
再用 `, ` (,後面有空格) join 回一個字串，得到結果

二、實作單一 image2text record 轉換成 prompt 的功能。
在 image2text record 的頁面中增加一個表單，可將當下 image2text 的文字建立成 prompt

表單中會有：
- 原文 textarea，帶入 image2text 的 text(content) 
- 選擇目標 prompt category
- 要建立的 prompt 的 text_en (textarea)，一開始內容同原文 textarea
- position 欄位，預設為 0

當選擇 prompt category 之後會先處理經由所選的  prompt category 所屬的 group，將排除字套用後的結果，更新在要建立的 prompt text_en 欄位，在表單上讓使用者可以預覽排除後的結果。
按下表單，送出建立 prompt

三、實作批次功能。
在 image2text 的批次 Show 畫面中，可以：
- 勾選多筆 records 後，
- 選擇目標 prompt category
然後按下批次建立 prompt 的按鈕
會進到 image2text records 批次建立 prompts 的畫面

此時批次建立的表單
會列出每一筆勾選的 records，每一筆都會出現跟 (二) 的單一建立一樣的表單欄位
並帶入已選擇的 prompt category，以及已套用排除後的結果文字

使用者仍可對每一筆修改 prompt Category，然後表單會套用文字排除後的結果。
按下送出後，就會將所有的 records 建立成 prompt

請盡量將 (二)、(三) 可共用的部分抽出成元件
並確保程式簡潔可維護，以及整體 ui 操作介面的友善
若有任何不確定的問題，請提出來討論，確認後開始進行

## 追加自訂排除文字

我想在 image2text batch 建立 prompt 的表單中，增加一個自訂全域的排除文字輸入方塊。
整個 batch 表單都會套用這一份輸入的排除文字。

等於排除文字時，會排除「所選的 category group 的排除文字 + 自訂的排除文字」

表單裡當這個自訂文字有變動時，
會送出 request、套用到每組 record 的進行文字排除，更新回畫面上各自的 record 。
若只對單一筆 record 選擇 prompt category 觸發更新時，也會連同這組自訂文字送出來套用排除

單筆建立的表單，則不需要這個行為。

請再加上一下功能修改
一、
在從 image2text record 建立 prompt 的共用元件中，加上prompt title 的欄位，允許空白
讓批次或單一建立的時候，都會存入 prompt 的 title。

二
在一般的 image2text (不透過 batch record)的表單也加上同樣的功能
即： /app/img2text 這個畫面
在生成 text 後，也可以用同個元件來實現建立 prompt
同樣包括行為：
- 選擇 prompt category、套用排除文字 (但不需要自訂排除文字的欄位)
- 可輸入 prompt title


# 優化 PromptIndex 的 Category 與  CategoryGroup 
將這兩項資料的 New/Edit 都做成獨立的頁面，不在只是在側邊欄嵌入 form，各自有獨立的
PromptCategory Edit/New, 共用 prompt_category_form
PromptCategoryGroup Edit/New, 共用 prompt_category_group_form

路徑可以是
/app/prompt_categories/new
/app/prompt_categories/:id/edit
與
/app/prompt_category_groups/new
/app/prompt_category_groups/:id/edit

在新增或編輯成功後，redirect 回 /app/prompts 畫面

# 優化 PromptCategoryGroup

在 prompt_category_groups 增加兩個欄位
- elimination_tags, text, default to nil
- decoration_flags, text, default to nil：用來定義該 group 下的 prompt 會擁有的 decorations

elimination_tags
用來定義該 group 下的 prompt 的 text 中要被自動排除的 tag，內容會用 ","  間格，
未來在新增、修改 prompt 時，會依該 prompt 所屬的 category_group (via category) 取得要排除的 tag list，然後自動將 prompt 的內容把出現的 tags 排除掉

decoration_flags
目前有定義一組 hard code 的 prompt decorations，未來會依該 prompt 所屬的 categroy_group(via category) 得可以使用的 decorations，代表不同的 category_group 可以定義不同的 decorations

請在 Prompt Index 的 PromptCategoryGroup 的 crud 的功能中，在表裡加入上述兩個欄位，都是多行的文字方塊



# 增加 PromptCategoryGroup

我想在  Category 再加上一層 group，可以再把 category 分組

group has many categories
category belongs to group (optional)

group 有以下欄位
- title
- code: 英文代碼，必填，不可重覆
- remark: 備註文字
- position: 排序

在 prompt index 左側的 category 列表
調整成依 group 來分組顯示：
覺依 group 排序
group 內的 categories 用 updated_at desc 排序
未分組(no group) 的 categories 放在最後一組

要增加 group_form 來做新增/編輯 group 的操作
在原本的 category form 裡也要有可以指定 group 的 select 欄位 

左側選單最下方有新增 group 的按鈕
左側選單每個 group 名稱旁邊有編輯與刪除的圖示

其他 prompt 列表、select 的部分都先不要變動

# 快速設定 Prompt 尺寸

在 prompt form 裡有 width / height 欄位來設定尺寸
我想加上除了原本的手動各別輸入尺寸之外
旁邊可以有按鈕直接快取設定尺寸，按鈕包括：
- L(M): 設成  1536 x 1024
- L(S): 設成 1536 x 768
- L(L): 設定 1536 x 1280
- P(M): 設定 1024 x 1536
- P(S) : 設定成 768 x 1536
- P(L): 設定成 1280 x 1536

請將這個組合設為常數，方便我日後可以調整

另外，
在 PromptIndex 的清單中，每個 prompt，
目前已有可以設定 decorations 與 likes 的的圖示功能
也請加上上述的 prompt 尺寸快速設定功能


# 整合 Prompt Decorations 與 Generation.Processor
請完成以下三項調整：

一
prompt 的 decorations 每一個 flag 會對應到一串系統預設的文字
先用常數定義這些 flag to text 的 mapping

二
在 Generations.Processor 的 set_prompt_inputs 方法裡
在組成 input_texts，對每一個 input(prompt)，原本僅是取出該 prompt 的 text_en
請改成 text_en 加上 (用「,」 串接) 該 prompt 的 decorations 對應的每組文字

三
在 Generations.Processor 的 set_prompt_inputs 方法裡
組成最後要使用的 prompt_text 時，加下以下處理
- 將整個內容先用「,」拆分，
- 對每個 string 做 strip，
- 去除空字串
- 取 uniq
- 最後再用「, 」join 回來




# Prompt Decorations

## 一、Promp Schema 變動
在 Prompt 加上新欄位 decorations, :integer, default: 0
在 Schema 設定中，使用 Bitmask 類型

ref:
https://hexdocs.pm/bitmask/Bitmask.html

並定義 PromptDecorations 來做為 decorations 欄位的 type ，依序定義：
- overture
- main
- oral
- hand
- ejac_1
- ejac_2
- ejac_3
- grouping_2
- grouping_3
- grouping_4
- grouping_n
- grabbing
- clothing

來做為複選的設計

## 二 、PromptLive 的更動
- 將 Prompt new/edit 畫面的 Prompt Form 加上用 check boxes 來做為 decorations 的複選操作
- 在 PromptLivew.Show 時，列出的 prompt list，每個 prompt 區塊右下方，有可以直接設定 decorations 的 icon，點擊 icon 就可以啟用/關閉該項 decoration


# Prompt Position Ordered in Category

請在 Prompt 加上 position 欄位，整數，預設是 0
為了代表 prompt 在 prompt category 下的順序
並做出以下修改：


- 請在 prompt_form 裡也加上對應的欄位供編輯修改
- 在 prompt category 下瀏覽 prompts 時，依照 position (小到大) 、inserted_at(小到大)排序
- 在Generations.Processor 的 expand_input_prompts，若 amount 是 0，取用的 category.prompts 要依照 position 排序

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
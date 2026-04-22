在 Subject 上增加 likes 功能
- 在 subjects 上增加欄位來代表 likes 數量
- 在顯示 Subject 列表時依 likes 數從大到小排序
- 在 Subject 列表或瀏覽，加上對 subject 進行 like or dislike 的按鈕，來增加或減少 likes 數
	- likes 數最小為 0 ，不會變成負的
	- 不用紀錄是誰按 like/dislike，可以重複的執行
- 在顯示 subject 時 (列表或瀏覽)，加上顯示 likes 數

請參考 prompt 也有相同的功能，請盡量將這個功能重構成可以重複使用，包括像是 ui component、javascript、共用的 macro module 等等，未來也有可能有其他地方也會想要套用它

調整 Subject Index 的列表
- 在 query subjects，同時 preload appearance generation
- 決定 represent image：
	- 如果有 appearance generation ，從 appearance generation 的 images 依 likes 排序後，隨機挑一張當作代表圖片
	- 如果無法從 appearance generation 得到，就從所有 generations 的 images，依 likes 排序後，隨機挑一張
- Index 頁改成支援兩種顯示模式 grid(預設) 和 list
- 調整 Subject Card 的 with_image component
	- 主圖片改以 represent image 來顯示
	- 此外另外增加顯示多張 images，直接從 generations 的 images 中依 likes 排序後隨機挑 n 張
		- n 預設為 10，可在呼叫 with_image 時用參數控制
	- 分成 grid 與 list 兩種顯示方法，建議的版型為：
		- grid 模式時：
			- 上方顯示 subject name 與 like 數
			- 下方分成兩欄，左側欄佔 50% 寬，顯示主圖片，右側欄顯示多張 images，每張圖為相同高度，會換行，若超過整個右側欄的空間都 overflow hidden，
		- list 模式時
			- 上方顯示 subject name 與 likes 數
			- 下方分成兩欄，左側主圖，右側依水平排列顯示 images，若超出範圍可以 scroll
	- grid 和 list 內容版型與整體介面請依你的專業設計來調整美化，確保 ui 友善且現代化的設計

請加上可以從 Subject Show 頁面，可以連到 new generation 頁面，並且傳入 subject_id，預先填入 new generation 的 subject_id
# Done

## 重整 Subject 的 new/edit 功能
- 將 new 與 edit 都改成獨立的畫面，但共用 subject_form
- 在 subject_form 裡，加上 appearance_generation_id 的欄位，並使用 generation_select 元件來輸入選擇



# 調整 Subject Index 與 Show 頁面

請將 Subject index 頁面中，每個 subject 設為可以點擊，點擊進入 Show 畫面瀏覽該 Subject

Subject Show 版型

分為三欄：
- 左側資訊區塊：顯示 subject 的基本資訊，包括 Subject name，Subject 封面圖片
- 中間主內容區塊，：subject images 清單，列出該 Subject 所有 Images
- 右側 image 資訊區塊：當點擊 subject images 清單的 image 時，會在右側欄顯示 image 資訊以及一些互動操作表單

### Subject 基本資訊
包括 Subject name，Subject 封面圖片
Subject 封面的定義方式目前未實作，可以先把方法建立，內容之後再補上
暫時先從 subject.images 取第一張就好

有編輯按鈕可以直接編輯 subject 的資料

### Subject Images 清單
中間區塊列出該 Subject 的所有 Images
每個 Image 用 ImageCard 呈現
支援兩種排列方式可作切換：
- Grid 格式
- 垂直清單，每列就一張圖片，往下排列，像是在瀏覽 instagram 

清單最上方有 filter，支援用 workflow 來做篩選
- 列出 Subject.workflows，水平排列出 workflow names 的選項
- 可複選，做為篩選 images 的條件
	- Subject.images -> generation -> workflow 的關聯
- 清單支援 infinite scroll

### Subject Image 資訊
右方預設是 empty state
當清單的 image 被點擊時，右方側欄會顯示該 image 的資訊
請將 image 的資料製作成 ImageCard.info component
內容先依照跟 GalleryLive.Show 的右側欄一樣的內容
之後再來做細部調整


# 優化 SubjectLive.Show  的 Image FullView
在 SubjectLive.Show 頁面
中間是 image 的清單，而每個 image 圖片被點擊後
會顯示 image 資訊在右側欄

右側欄有一個 Full View 的按鈕，點擊後會變成 full view 的模式
但目前沒有辦法關閉 full view

請進行以下調整
- 在 full view 的畫面加上一個 Close 的按鈕，可以切換回原本的清單模式
- 在 full view 的圖片顯示區，加上左右導覽的按鈕，點擊後會依清單順序切換正在瀏覽的圖片
	- 左：上一張
	- 右：下一張
	- 若已到清單的開頭或結尾，按鈕就不可再點擊



目前在 GenerationLive.Show 的頁面，
也有顯示 images 的列表

請參考 SubjectLive.Show 的圖片列表，
讓 GenerationLive.Show 也有以下同樣的行為：

- 右側欄為 image 資訊
- 圖片清單的圖片可以點擊，點擊後會在右側欄顯示該圖片的資訊
- 右側欄的圖片資訊有 full view 可以切換
- full view 有導覽按鈕可以切換上一張、下一張圖片

請將「圖片清單」、「圖片資訊」、「full view 模式」等製作成可以共用的元件
讓 SubjectLive.Show 與 GenerationLive.Show 裡共用這些元件

注意不可破壞 SubjectLive.Show 與 GenerationLive.Show 自身原本的功能

## 調整 SubjectLive.Show 與 GenerationLive

在 SubjectLive.Show 的畫面，圖片列表的上方，也加上與該 Subject 相關的 Generation List
跟 PromptLive.Show 一樣，請把這個 Generation List 也做成共用的 Component
GenerationList 裡的 Generation 用 GenerationCard 來實現

 並調整 GenerationList  與 GenerationCard，
 在 GenerationList 要 preload 每個 generation 的最近十筆 generated Images
 並在 GenerationCard 裡用水平排列呈現 image 的 thumb 縮圖
 如果 PromptLive.Index

# 調整 Gallery 畫面

將 Gallery Index 的圖片列表，調整成跟 Subject Show 裡的 圖片列表一樣
- 加上右側圖片資訊欄
- 點擊圖片會顯示圖片資訊
- 圖片資訊欄有 Full View 可展開圖片
- Full View 可以切換上一張、下一頁圖片

請將以下三處相同的行為整理為使用共同的組件來實現，包括
- Subject Show 裡的圖片列表
- Generation Show 裡的圖片列表
- 以及這次的 Gallery Index 裡的圖片列


# 調整 PromptLive 畫面

請將 PromptLive 改成 PromptLive.Index
並讓列表中的 prompt 可以被點擊瀏覽，進到 PromptLive.Show 頁面
並加上以下的調整：

## PromptLive.Show
- 將原本放在 PromptLive 裡的「編緝」 與 「刪除」 Prompt 的功能，移到 PromptLive.Show 
- 在 PromptLive.Show 頁面，顯示
	- 名稱
	- 所屬 category
	- 英文內容
	- 中文內容
	- 備註
- 並且列出
	- 全部相關的 generations
		- 請將 GenerationLive.Index 中的 generation 列表，每筆 generation 的呈現重構成 GenerationCard Component
		- 並在這裡 PrompLive.Show 裡列出相關 generations 使用同樣的 GenerationCard
			- 點擊後會進行 GenerationLive.Show
	- 相關的 generated images
		- 請重複利用 Gallery Component 來顯示 images
		- 一樣要有 infinite scroll 來載入更多 image
		- 一樣設計成點擊圖片，在側邊欄顯示 info
		- 一樣在側欄欄 info 可以切換成 full view
		- 請參考 SubjectLive.Show, GenerationLive.Show 的作法，使用相同的 component

# PromptLive.Index
PromptLive.Index 中，對讀入的 prompts 列表
- 每個 prompt 要 preload 最新的 10 筆 generated images
- 在列表中顯示每筆 prompt 顯示這些 images 的 thumb 
	- 用水平的方式排列顯示 image



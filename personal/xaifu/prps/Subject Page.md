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


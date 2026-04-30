## 加上 Lazy Load 功能
對所有 render image list 的地方，加上 lazy load 的功能

將 image 列表中 render image 的部分，(預期應該要有個 component 來統一 render image)，包括：
- gallery 的 image list 縮圖
- subject index 中每個 subject image 縮圖
- subject show 裡的 image list 縮圖
- generation index 中的每個 generation image 縮圖
- generation show 裡的 image list 縮圖
- prompt index 裡每個 prompt 的 image 縮圖
- prompt show 裡的每個 prompt image 縮圖
都套用 lazy load 的功能

lazy load：
請評估使用現有套件或簡單實作
需滿足以下使用需求：
- 在滑鼠 hover 時才真的載入圖片
- 在整個 layout 右上方選單，加一個 switch 按鈕，可以啟用或關閉  lazy load 的功能
	- 會在瀏覽器(session、cookie 或 local storage) 紀錄當下用戶是否有啟用 lazy load
	- 若關閉 lazy load 功能，在使用者開啟/重載畫面時，所有 image 就不會有 lazy load 的效果
- 在整個 layout 右上方選單，加一個 load all 的按鈕(用圖示即可)，按下後，就會把畫面所有 lazy load 效果的圖片進行載入
lazy load 效果僅限縮圖，例如以下的情況圖片就不需要 lazy load
- 點擊縮圖後顯示 detail 的時候
- 點擊 full view 顯示的原圖，以在在 full view 下切換上一張/下一張圖片的時候 

請先擬定計畫，確認後開始執行實作

# 重整 ImageList and Viewer 功能

在以下頁面
- Generation Show
- Subject Show
- Prompt Show
- Gallery Index

都會有 image list and viewer 的行為，即：
- 在頁面中會依傳入的 images，列出清單
- 支援 grid 或 list 模式(預設為 grid)
- 右側有側邊欄，當 image 被點擊時，在側邊欄會顯示 image detail
- 側邊欄的可按鈕可點擊進入 Full View 模式
	- Full View 模式下可以切換上一張、下一張圖片
- image 有可能會在 liveview 中動態增加
	- 例如 generation 生成新的 image
	- 或 infinite scroll 讀入更多的 image

請將這些頁面的 image list + viewer 功能
盡量重構成共用的 component 與 module
並依以下需求調整

- 在側邊欄顯示 image detail 時，加上顯示 likes 數以及執行 like/dislike 的按鈕
- 在 full view 時，也加上顯示 likes 數以及執行 like/dislike 的按鈕



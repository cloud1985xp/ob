重整 ImageList and Viewer 功能

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



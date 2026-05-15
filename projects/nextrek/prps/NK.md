- Report migrate to v3 + refactor, path, statement group
- Rubocop fix, by parts of domains
- Refactor Journal Related
- Refactor View by subject
- Refactor Entry Related
- Seed data with group builder using mini data sample
- Tidewave Integration

NK Engineer Workflow


## 優化 Lookbook
- 標題從 Nexalon 改為 Nextrek
- 各種基本 component 的樣式目前看起來都怪怪的，似乎沒有套到專案的 style，請檢查確認現狀
- 請美化整個 lookbook 的樣式風格，可以獨立用 ui-ux-max-pro 設計，搭配 tailwindcss + daisyui
- Subject Menu 裡並沒有選項的項目，請加上有選單資料
- 
	- 

## Migrate 前端專案
請詳細閱讀理解前端專案(/Users/aaron.kuo/projects/nextrek-tw)的內容，
規劃將其整合進現有專案

- 不要完全照搬程式碼，請維持現有專案的架構
- 但用前端專案的設計規則至 v3 對應的 component 中
	- 確保不影響舊版本
	- 確保不影響 v2 component
- 將前端專案的 design_system 規範整合至現有專案 ai_docs/design
	- 將 design_system/components 裡的樣式整合至 v3 components
	- 若有尚未建立的 component 則建立
- 確保 components 對應的樣式、guidelines 都有出現在 lookbooks 中
- 將 前端專案中的 .claude/skills 中的 skills，適度的整合至現有專案中
	- 調整成現有專案的架構
	- 若有功能與 lookbook 重疊，調整成與 lookbook 整合的方式

若有任何不確定有疑問的地方，請提出討論
若有覺得有更好的作法，請提出建議

## Migrate 專用 component
請把新版 import journal：
即 /accountings/draft_stages 的相關功能

裡用到的以下元件，包括像是

- contact select
- tag select
- subject menu
- selection bar
- date picker
- drop zone

另外還有現有的其他元件

- numeric_input

將它們設計規範對做對齊，或做出需要的更新調整
但務必確保不要影響其功能性

並且
- 更新到 lookbook 裡的元件庫 
- 更新到 ai_docs 裡的 design 與 ui 文件
	- 包括說明使用場合、範例與注意事項
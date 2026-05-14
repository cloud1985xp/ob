- Report migrate to v3 + refactor, path, statement group
- Rubocop fix, by parts of domains
- Refactor Journal Related
- Refactor View by subject
- Refactor Entry Related
- Seed data with group builder using mini data sample
- Tidewave Integration

NK Engineer Workflow


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

請把 新版 import journal：

draft_stages 的相關功能

裡用到的以下元件

- contact select
- tag select
- subject menu

與 ui 
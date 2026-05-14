- Report migrate to v3 + refactor, path, statement group
- Rubocop fix, by parts of domains
- Refactor Journal Related
- Refactor View by subject
- Refactor Entry Related
- Seed data with group builder using mini data sample
- Tidewave Integration

NK Engineer Workflow


請參前端專案(/Users/aaron.kuo/projects/nextrek-tw)的內容，整和進現有專案

- 不要完全照搬程式碼，請維持現有專案的架構
- 但用前端專案的設計規則至 v3 對應的 component 中
	- 確保不影響舊版本
	- 確保不影響 v2 component
- 將前端專案的 design_system 規範整合至現有專案 ai_docs/design
	- 將 design_system/components 裡的樣式整合至 v3 components
	- 若有尚未建立的 component 則建立
- 確保 components 對應的樣式、guidelines 都有出現在 lookbooks 中
- 將 前端專案中的
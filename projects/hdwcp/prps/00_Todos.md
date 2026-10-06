
  未來可進行的工作

  - Phase 11 REST API（若需要外部系統整合）
  - PDF 匯出 (ChromicPDF)
  - SMS (Twilio) — 目前 OTP 僅透過 Email
  - 審計追蹤 (PaperTrail 替代)
  - Dockerfile / Release / 正式部署配置
  - Favor rule handler 實際邏輯
  - 訂單批次匯入功能

Settings

- Users 選擇 dealer -> 整體要先制定 select ui
- Users 發送password 信件
- Users Login As 功能


這個專案是要將舊版 rails 版本專案(位於 /Users/aaron.kuo/projects/hdwcp) 完全移植成新的 elixir phoenix 版本專案，目前已有部分功能進行移植

請重新完全檢查舊版本的功能，確保移植到新版本的專案中

已知有一部分是完全沒移植的，例如
- 前台網站 CMS 功能
- 前台消費者端的保固登錄功能
- 後台部分功能
	- 公告管理
	- 社群
	- 權限機制
- 其他，請詳細檢查

另外後台有一些部分是在新版本開發的，需要維持新版本，像是
- 介面設計與優化後的操作行為
- 新版本的產品項目下單機制
	- 且支援詢價功能(不需建立訂單即可預覽產品與報價，這是舊版沒有的)

請詳細理解舊版與新版的功能，繼續規劃後續的工作進行開發

若有不確定或是可改善的部分，請提出問題與建議
並且可以考慮拆分成後續獨立進行的工作
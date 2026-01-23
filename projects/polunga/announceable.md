---
tags:
  - polunga
  - project
  - elixir
created: 2024-01-01
updated: 2025-01-23
status: active
---


定義 Announceable 範圍

Announcement
AnnouncementMaster
Banners


Announcement
AnnouncementMaster
Banners

Announcement/Master 下的 RandomLoginBonus
RandomLoginBonus -> RandomLoginBonusMovie

Announcement{Master}Body -> AnnouncementMissionCampaign
AnnouncementMissionCampaign -> AnnouncementMissionCampaignSection


Announcement-like
- 無法在 production 上直接編輯，一律從 staging sync
	- 未來會上鎖
- 可以在測試環境間 sync
## Apology

- 改從 polunga 編輯管理
- Q: 未來是否要改成從 staging 編輯，sync 到 production
## Announcement / AnnouncementMaster / Banner

- 從 polunga 編輯管理資料 (照舊)
- 改用 Polunga Synchronization

Q1: 公告翻譯編輯的 (old ver.) 還有在使用嗎？
Q2: 需不需要指定瀏覽語系
Q3: AnnouncementMaster 的 preview功能是壞的，是否要修復

## AnnouncementMissionCampaign
- 改從 polunga 編輯管理資料
	- 增加 banner 上傳功能
- 不須額外 sync
	- 會經由 announcement 執行 sync 時一起

## RandomLoginBonus
- 會同 announcement 一同 sync
- 複製功能


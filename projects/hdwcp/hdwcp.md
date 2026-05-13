---
tags:
  - project
  - hdwcp
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# HDWCP

[進銷存](HDWCP/%E9%80%B2%E9%8A%B7%E5%AD%98%20aac891a36278435c9d8ddd8bcf94913e.md)

[價目更新 2024-25](HDWCP/%E5%83%B9%E7%9B%AE%E6%9B%B4%E6%96%B0%202024-25%20765f08ee18194131825fb751179576d9.md)

[價目更新2025-26](HDWCP/%E5%83%B9%E7%9B%AE%E6%9B%B4%E6%96%B02025-26%2021e15131ca6d80649ee3f4b8ebddfa5b.md)

## Operation Troubleshooting

### MySQL

ONLY_FULL_GROUP_BY SQL Mode

[MySQL 5.7+ ONLY_FULL_GROUP_BY 問題 · MySQL 學習筆記](https://kejyuntw.gitbooks.io/mysql-learning-notes/content/question/mysql-only-full-group-by.html)

```sql
#set the complete "forgiving" mode
mysql> SET GLOBAL sql_mode='';

# alternatively you can set sql mode to the following
mysql> SET GLOBAL sql_mode='STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_ENGINE_SUBSTITUTION';

```

## 2024

- 升級 Rails6, 7. Ruby
- 容器化
- 測試
    - [ ]  下單
        - [x]  一般訂單
        - [x]  維修
        - [x]  修改
        - [x]  樣品
        - [x]  其他
    - [x]  出貨流程
        - [x]  批次出貨
        - [x]  海外出貨
    - [x]  用戶登入、
        - [x]  編輯、
        - [x]  忘記密碼
    - [x]  經銷身份操作
    - [x]  會計功能
    - [x]  coupon 管理
    - [x]  Check schedule.rb tasks
    - 後台
        - [x]  管理用戶
        - 管理經銷
        - 價目表管理、年度更新
        - 經銷合管理、年度更新
        - 更新前台文章
    - [x]  前台用戶保固註冊
- 開 staging 環境
    - 設定部署流程
    - 設定 ssl

### 產品瀏覽頁面

- 僅顯示當年度價目表，其他年份切到另外畫面瀏覽
- 系統/產品 升級項目是經由當年度價目表關聯到的，但當前瀏覽的產品，不見得真的可以升級該項目，應該給予標示 (或警告該升級項目是 unavailable 的)，可能原因包括：
    - 該產品沒有建立對應升級項目的極限規則
    - 該升級選項是關閉狀態
- 極限規格依操作、產品系統分組合併顯示，依 DU 最為明顯
- 佈局
    - 基本資訊、顏色資訊、當年度價目表 (現貨/期貨)
    - 產品組合
        - 極限規格
        - 加價
    - 其他升級選項
    - 直接試算產品價格

### 價目表

- 清楚標示是何種價目表，「一般產品」、「系統升級」「其他商品」

### 操作系統管理頁面

- 若設定是電動操作，要提示：
    - 若有提供供電選項，則預設價格會無效
    - 否則就會用預設價格
    - 所以要這個畫面可以直接設定供電選項 (當被設電動操作時)

## 下單優化

- 整理重複的其他附加選項
- 提交時必須必定要有產品項目

## 其他

- 合約管理確認列表功能

2024-25

- 底層更新 80 (16d)
    - 升級 ruby
    - 升級 rails6、7
- UI 全面翻新 102 (20d)
    - 引入 tailwinds
    - 優化 mobile view
- settings 模組更新 40 (8d)
- products 管理模組更新 80 (16d)
    - 增加 builds 模組
    - 優化新產品建立
- pricing 管理模組更新 50 (10d)
    - 優化價目表更新，上傳先預覽再更新
- sales 模組更新 60 (12d
    - 新增「待料」功能

2021-2022

[無標題](HDWCP/%E7%84%A1%E6%A8%99%E9%A1%8C%2061a256306027455bb074d8bcc88e06db.csv)

- [x]  主圖管理
- [x]  產品頁支援影片
- [x]  字體一致

公告加檔案下載 複數

- 優化
    - [x]  年度價格更新方式
    - [x]  經銷合約更新機制
    - [ ]  ?產品新增方式
    - [ ]  產品代碼對應機制
    - [x]  ?經銷商合約紀錄
    - [x]  維修單/修改單 可變更經銷商
        - [ ]  只有總公司使用者才能變更經銷商
        - [ ]  上下游瀏覽時要注意
    - [x]  修改「還原生產」的功能
        - [ ]  可將已在出貨批次作業中的狀態也一併還原
- 英文介面
    - 中英文分別系統建置
- 介面改版
- 優化二
    - [x]  內容搜尋：主要針對討論區 15
    - [x]  公告搜尋：  15
    - [x]  下單時切換產品說明 15
    - [ ]  布料與顏色統計 25
        - [ ]  資料太密集
        - [ ]  下載功能
    - [ ]  保固分析統計 25
        - 計算金額
        - 下載功能
    - [ ]  切換經銷身份，關閉系統回不來
    - [x]  下單時有圖形尺寸示意 15
- 首頁 35

w1 - before 11/15

[x] 修改 Revision 流程，允許從部分出貨、已出貨

- [x]  轉維修/修改單時，可以變更經銷商

[] dealer agreement history & notification

[] public website

[] search announcement

w2 - before 12/1

[] search forum

[] Display category info when switching category at order form

[] stat improvement and download

- color and fabrics

- warranty

[] Display graphical window size

w3 - 12/31

[] product code mapping rule

[] create new model with related components

w4

[] English ver.

[] UI Refine

### 問題一

訂單已被客服務還原回待處理

但同時已被加到 出貨 batch，然後進行出貨

→ 取消明細項的出貨狀態

→ 選擇：取消/作廢出貨單

### 問題二

已經出貨(列入統計)

這時才被手動調整金額
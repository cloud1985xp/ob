
# 更新 Coupon Application 功能
將 InfiniteScrollList 套用到 marketings 下的 Application (CouponApplication) 功能


套用 DateRangeFilter
- 可以用 date, published_on, expirefd_on 當作條件欄位
- 預設提到1天、3天、5天、15天丶1個月、3 個月、6個月的快速篩選條件

# 更新 Coupon 功能

將 InfiniteScrollList 套用到 marketings 下的 coupons 功能，並且

一、套用 DateRangeFilter
- 可以用 date_published, date_expiration, date_deducted 當作條件欄位
- 預設提到1天、3天、5天、15天丶1個月、3 個月、6個月的快速篩選條件

二、以 CouponApplication 當作分類篩○
layout 用 left
但只取最近 12 個月 (欄位 date 做條件) 的 CouponApplication 做為選項，選項以降冪排序

三、增加更多篩選條件
對 Coupons 的 query 還要支援以下的條件

可用 state 來篩選，篩選值有

- formal: 流通中
- deducted: 已拆抵
- 或不限狀態

可用 dealer 來篩選，請使用既有的 Dealer Filter component



# 更新 Deduction 功能

將 InfiniteScrollList 套用到 marketings 下的 deductions (DeductionBatch) 功能，並且

一、套用 DateRangeFilter
- 可以用 date_apply 當作條件欄位
- 預設提到1天、3天、5天、15天丶1個月、3 個月、6個月的快速篩選條件

二、可用 dealer 來篩選
請使用既有的 Dealer Filter component

三、增加更多篩選條件
對 Coupons 的 query 還要支援以下的條件

可用 state 來篩選，篩選值有
- draft: 草稿
- submitted: 等候審核
- processing: 系統處理中
- completed: 處理完成
- 或不限狀態



## 修正 Promotions 和 Market Channels

請將原本是在 marketings 下的 promotions 和 market_channels 這兩項資源的管理功能，移動到 settings 下，它們應該是屬於 admin 的功能

並在  admin layout 的主選單
增加一個「marketing」的第一層選單
裡面放 promotions 和 market_channel 的連結選項
---
tags:
  - business
  - planning
created: 2024-01-01
updated: 2025-01-23
status: active
---

# Server Cost Analysis

## 主要的 On Demand 費用
- RDS
- CloudFront
- Data Transfer
- EC2
- S3

---

## 2024-04 情況與可施對策

由易到難

### Simple Storage Service (S3)
- S3 Storage 費用也是逐月攀升，從 每月 (2021 March) $3662.21 到 (2024 March) $9,880.17

對策：盤查 S3 上的存放資料，刪除不需要的檔案

- 確認 history assets 的保留版本，正否正常刪除舊版，確認保留版本總計空間
- 確認各 bucket 的用途與資料是否可刪除
- 確認 bq sync 資料是否保留或正確刪除


### CloudFront

Cloudfront 的費用每月從不到 $20000 逐漸攀升到現在都大約 $30000上下
不過這幾年 DAU 是下降的
(當然還有版更次數與 new install 數也有影響)

對策：預計將於 v5.21 優化 assets 打包與下載程式
- [https://youtrack.ishin.aktsk.dev/issue/ISHINKI-25929](https://youtrack.ishin.aktsk.dev/issue/ISHINKI-25929)
- https://youtrack.ishin.aktsk.dev/issue/ISHINKI-25931/DL-cpk
- https://youtrack.ishin.aktsk.dev/issue/ISHINTW-13575/Asset-DL
- [https://github.aktsk.dev/ishin-jp/ishin-client/pull/18016](https://github.aktsk.dev/ishin-jp/ishin-client/pull/18016)
- [https://github.aktsk.dev/ishin-jp/ishin-design-lfs/pull/2227/files](https://github.aktsk.dev/ishin-jp/ishin-design-lfs/pull/2227/files)

### RDS
- RDS 主要成本來自
	- Aurora Storage and I/O
	- Backup Storage Exceeding free allocation
	- On-Demand instances
		- Weekly user snapshot sync

對策
- 刪除/減少 backup
- 設法加速/減小 user sync db size？
- 了解 Aurora Storage 有無減少可能

### EC2

- 主要成長來自 on-demand 的 cpu scale-out instances
- 舊的 ebs(ami) snapshot
- Loader Balancer

對策

- 優化 server application，增加 processes count，減少 min app instance 數量 24/30/32
- 增加購買部分 app reserved instance
- 刪除/減少 EBS Snapshot
- 刪除沒在使用的 ALB
	- on-demand 環境日後也是需要使用才建立 ALB

### DataTransfer

- EIP 直接對外流出：
	- bq loader
	- weekly bq sync
		- 減少 weekly sync 的 user table -> 沒在用的就不 sync
		- 改成用 incremental sync
			- 部分改
			- 定期做 fully re-sync + LOG 資料堆積來計算
		- 改成用 upsert sync
			- 在 sync 撈資料的 query  加 update_at 的條件，只過濾出過去七(n)天有變動的資料 (因為 user db 裡應該有很多 inactive user 的資料根本沒變動)
			- 然後 sync 的目的地，改成是傳到 `暫存` 的 dataset
			- 用 upsert (在 bq 叫 merge) 的方式把這些有變動的資料 merge 進主 dataset 裡的 tablequery 速度可能會慢一點 (但一樣有用 id range 可以 hit index，只是再加上 updated_at 條件)， 但傳資料到 cloud storage 跟 load 資料進 bq 的量應該會少很多

---

## 2025-03 年度結算 (FY23-24)

### 合計費用
未稅 ¥500,703,826 (含稅 ¥550,774,209)
原預估(未稅)= ¥560,753,514

差額 -¥60,049,688 (未稅)

### AWS Billing (prod+dev) 未稅
原預估 $2,292,709.12
實際    $2,285,358.22
差額 -$7,350.90

### RI 費用 (未稅)
原預估 ¥207,783,012
實際 ¥148,639,673
差額 -¥59,143,339

### 共同費用(未稅)
實際 ¥33,578,923
原預估 ¥40,328,349
差額 -¥6,749,426

### 純看 AWS OnDemand Billing

實際 $2,324,103.98
原預估 2,292,709.12
差額 -$31,394.86

---

## 2025-26 施策方向
1. 在導入 SimpleMaster 後 RDS 流量可望減少
2. 在啟用 Battle Anomaly/Device Check 後 ELB/EC2 用量可望減少

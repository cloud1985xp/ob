---
tags:
  - polunga
  - project
  - elixir
created: 2024-01-01
updated: 2025-01-23
status: active
---


https://docs.google.com/spreadsheets/d/1SbY_Soiqv3EbgKyNzwOEAqG0c0lxLq0B3JF8LObLJ7Y/edit?gid=1124200398#gid=1124200398

```
`神龍資料排序優化`  
現狀：紅框顯示部分,目前是按照資料輸入的排序  
優化需求：紅框顯示部分,希望可以按照`id小>>大`排序
```


```
原因： 由於要調查每個活動/gasha的master參數,所以基本上OM都會先做草稿版的『神龍清單』。 但完成後,為了給各成員查看,就還要再手動騰到神龍清單上,就有重工的感覺。
```

> 目前在使用sheet（但應該很難抓參數）

[https://docs.google.com/spreadsheets/d/1SbY_Soiqv3EbgKyNzwOEAqG0c0lxLq0B3JF8LObLJ7Y/edit?gid=158864850#gid=158864850](https://docs.google.com/spreadsheets/d/1SbY_Soiqv3EbgKyNzwOEAqG0c0lxLq0B3JF8LObLJ7Y/edit?gid=158864850#gid=158864850)

直接讀取應該也是可以


這樣是否有機會是：先在神龍輸入，預覽神龍結果，再輸出產生給成員檢查的版本


輸入的其他解決方方案
以純文字以類似 markdown 語法來輸入

Example

# MyData 重置
begins_at: 2024-07-10 
endes_at: 
D,Others

# Pick
## DotCharacterLvRewardSetRelation
New:10,11,12,13
Reopen:1031,1230

## DotCharacter
New:10



分成 issue summary 與 issue details

Issue Summary
issue 基本資料
以及追加 summary 和 category 欄位
summary -> Doris 從康佛那邊來的文字說明
category -> 用來對照這個 issue 會涉及哪些 masterdata tables，在產生 detail template 時使用


Issue Detail
實際包含的 masterdata resources

批次新增 issues 
從 google sheet 匯入大量 issues，包含 summary

批次更新 detail

更新 details 時會一次只更新一個 issue 的 detail 對嗎？


會議紀錄  
[https://docs.google.com/document/d/1It0S4-Z7X5KQkxdc_rRmuQmUbfsmq7gdwqSS3vFmrf0/edit?usp=sharing](https://docs.google.com/document/d/1It0S4-Z7X5KQkxdc_rRmuQmUbfsmq7gdwqSS3vFmrf0/edit?usp=sharing)

doris提案  
[https://docs.google.com/presentation/d/1REtlCzCfAD-igmDD3rEMwsomQQ7wuqNyHLLEoz-Mz6M/edit#slide=id.g2f26cb95bc3_0_3](https://docs.google.com/presentation/d/1REtlCzCfAD-igmDD3rEMwsomQQ7wuqNyHLLEoz-Mz6M/edit#slide=id.g2f26cb95bc3_0_3)

doris summary模板（test版本）  
[https://docs.google.com/spreadsheets/d/1JEsQRy3Ul0DWJ8HIQZ7nigmaEtoFm-zOJGBKRhvCVeA/edit?gid=981344865#gid=981344865](https://docs.google.com/spreadsheets/d/1JEsQRy3Ul0DWJ8HIQZ7nigmaEtoFm-zOJGBKRhvCVeA/edit?gid=981344865#gid=981344865)
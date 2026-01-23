---
tags:
  - business
  - planning
created: 2024-01-01
updated: 2025-01-23
status: active
---


## 2025-04-22 Discussion
Google Drive Scan -> Excel(Legacy Version) -> CSV -> DuckDB -> Import into Rosetta DB
Google Drive Scan -> Excel(Miles Version) -> CSV -> DuckDB -> Import into Rosetta DB with UUID referenced

### LQA Import Translation
v5.26 released
v5.27 仍在用 legacy version excel
v5.28 仍在用 legacy version excel
- 這些 term 還不存在於 source string table = 沒有 uuid 可以做比對
	- 給予特殊 prefix 的 identifier (temporary identifier)
	- 必須要求 lqa 在 identifier (string id) 確保 (temporary) identifier + term 是 unique 的
- 這些 excel 回來的檔案，都用 insert into source strings，
	- 在 metadata 紀錄來源資訊	- 
	- scanner 處理到 5.27 File A SourceString X 時，會 insert 一筆在 source string
	- 當處理到 5.27 File B 又遇到 SourceString X 時，
		- 若 identifier 不同，會再 insert 一筆在 source string 裡 
		- 若 identifier + 相同，就
	- 特別設計一個機能去合併/解除重複(且md_id 未知)的 source string
v5.29 改用 miles version excel 

v5.27, v5.28 期間的 CustomImporter

v5.29 以後的 NormalImporter
每個 source string 在 upsert 時
- 用 uuid 比對，若是相同 uuid 就增加 history 紀錄
- 若不同 uuid 就是 insert


### Planner Update Translation
用 namespace + property + master_id 比對，若有找到就套用翻譯
若沒有找到，就取消 master_id 並加上原文比對
- 若只有一筆找到，就套用翻譯
- 若有複數筆，就列入備選清單
- 若沒有找到，就列入修正查詢清單

在畫面上列出
- 套用清單
- 備選清單
- 待修正查詢清單

讓用戶自行解決2&3的情況
完成後送出，將套用更新 回 google sheet 上


## 研究使用 Umbrella App

## 2025-03-25 Discussion

- Rosetta Roadmap

WIP
- 可維護偵測 table 表頭時的關鍵字清單
	- 目前簡化成 ID 結尾的文字
- 在 Subject Mappings 時可以手動調整拆分成多個 subjects
- 輸入的 document，系統會定期自動截取內容，在發現有新資料時，發送提醒通知
- [x] Restart Package Preprocess 時要把 term 的狀態先還原成 :selected
- [ ] Background Process Error Handling, display failure state
	- [ ] Package preprocess

Issue
- Search Function, what's the scenario to use search

Next Goal
- Test parser
- How to increase efficiency of doing "resolution"
- Documentation
	- share the flow chart
- Next Step / Plan / Schedule
	- 

Text Transformed Example
https://wikib.aktsk.jp/pages/viewpage.action?pageId=397779534
https://stg.rosetta.aktsk.com.tw/documents/0195cb3b-e372-74ea-93a1-112e3433cacd

> namespace:missions property:name 超戦士列伝 ドラゴンボールGT編 を条件付きクリア



Add googlesheet to report test progress of wiki URL(document)


Rosetta
- Astel
- Canonical DB

Crawler / Scratcher
專門把指定網址的內容截取存進資料庫
問題？
- 如何判斷對象是否有變動
	- 單純整個內容做 digest，但有可能被不必要的資訊變動影響
	- 依網址對象截取特定內容，再做 digest
		- 使 crawer 變複雜，相當於多了 parse 的
- 結論：不用在 crawler 端做 change detection
	- 把所有 terms 不斷紀綠到 db 就好
	- 之後發現有「未翻譯」的 terms，再用 metadata 去比對來判斷是否有變動就好




- Crawler service：在設定日期區間內，每日對 url 下載內容
	- 負責處理 session、authentication 的問題
	- 負責 encoding 問題


source
- URL
- parser: wikib
- strategy: ishin:zbattle
- raw_html
- state
source_subjects
- scheme
source_subject_entries
- uuid
- property
- text
- metadata
	- masterdata_id
	- position
- state
- remark: filtered by system, ignored by user ... etc


- Parser service：對 url content 依照不同的 strategy 做資料 extraction
	- 依 strategy 將內容轉成 struct json，成為 page data with metadata (source info, context, masterdata domain)
	- Basic Strategy
		- 依 heading 層級變化拆分 section
			- 一個 section 有多個 headings，由小到大
				- 通常一個 table
			- iteration 過程中發生 heading 變小，表示是新的 section
		- 再將 sections 做過濾: 能否判斷 domain: ex table 是否有 header，header 是否有 `ID`
			- 對可判斷 domain 的 sections，逐一
				- 將 table 中的 header + rows 做資料轉換
					- rows -> records
```
headers = ["ID", "ミッション名称", "説明文", "条件", "報酬内容", "備考"]
=>
doc = {
  headings: ["ミッション", "146仕様詳細（クエスト）"]
  subject: "missions",
  schema:[
    { title: "ID", field: "id" },
    { title: "ミッション名称", field: "name" },
    { title: "説明文", field: "description" },
    { title: "条件", field: "unknown" },
    { title: "報酬内容", field: "unknown" },
    { title: "備考", field: "unknown" }
  ],
  records: [
    [
    . { 
    .   row: 1, 
        masterdata_id: "",
        field: "name", 
        text: "究極のレッドゾーン 孫悟空の軌跡編\nステージ1を1回クリア", 
        key: "ミッション名称"
      },
      { 
        row: 1, 
        masterdata_id: ""
        field: "description", 
        text: "チャレンジイベント\n「究極のレッドゾーン 孫悟空の軌跡編」の\nステージ1を1回クリアしましょう！", 
        key: "説明文" 
      },
    ],
    [
      {
        row: 2,
        masterdata_id: "26662",
        field: "name"
        text: "究極のレッドゾーン 孫悟空の軌跡編  
ステージ2を1回クリア"
      },
      {
        row: 2,
        masterdata_id: "26662",
        field: "description",
        text: "チャレンジイベント\n「究極のレッドゾーン 孫悟空の軌跡編」の\nステージ2を1回クリアしましょう！"
      }
    ]
  ]
]

```
- Filter service：對 subject entries 做資料的過濾，可設定過濾 rule
	- 存進 raw terms database
	- terms has state
以上都是對 subject_entries 操作 state

- Transformer：對 page data 做資料轉換
	- 補上 metadata
		- 已翻過
		- 在 history 出現過
	- 自動翻譯
		- Rosetta
		- ML
		- Grammar
		- LLM
subject_entry_translations

- Workspace
	- 從 subject entries 取出資料產生 web ui
	- 提供 UI 讓用戶從 terms 中過濾篩選要放入 staging area 的資料
		- 標記 raw_term state
	- 將 staging area 產生成送翻檔案 excel
		- 次選項：生成 googlesheet，讓用戶自行轉換成 excel 格式


## Canonical


ishin.area:name/12345
ishin.area:name/アルティメット孫悟飯


- Service to load files from google drive and combined translations into canonical db
- UI to browse translations
	- Query for translations
- API
	- fetch translations by given JP text (domain)
	- fetch translations by given masterdata_id
	- update masterdata_id by given JP text(domain)

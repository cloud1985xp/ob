
# Crawler Service

請從 ../venus 的舊專案中，將原本用 ruby/rails 開發的 Crawler 功能
migrate 至當下 elixir/phoenix 專案中

請參考舊專案中的 app/services 下的檔案

- app/services/t66y_crawler_service.rb

將它改寫在 elixir 版本

放在 lib/venux/crawlers.ex 這個 domain context 下
目前只需要 migrate 一種 site (t66y) 的 crawler，可放在 lib/venux/crawlers/t66y.ex

並請對程式碼內容改寫，以 elixir 的風格重新設計，並試著降低對 Entry struct 耦合度


# Crawler Update Function

請從 ../venus 的舊專案中，將原本用 ruby/rails 開發的 Crawler 功能
migrate 至當下 elixir/phoenix 專案中

請參考舊專案中的 lib/tasks/venus.rake 的 venus:update 指令

將對應的功能，migrate 成 elixir 版本，放在  Crawlers context 下
以 Crawlers.collect 做為啟動的程式

功能應包括
- 對目前啟動的 site，每一個 site 呼叫 crawler 功能去截取新內容(依不同的 site 可能用不同的截取策略，目前僅實作 t66y)
- 對目前截取的內容進行解析，依目前有啟用的 subject 清單，篩選出將符合主題的關聯內容，建立成對應 subject 下的 actress 的 actress work
- 對(自上次更新後)新增加的 actress work，進行更新取得詳細內容(cover)
- 對這次有取得內容的 actress work 及其 actress，整理成 slack 格式化的訊息內容，發送 slack 通知

網站頁面截取的相關的程式應都放在 Crawlers context 下
可以增加 submodule 來實作，以提高可維護性與程式碼品質
並可以建立對應需要的 Schema 在 Crawlers context 下

ActressWork 在解析取得詳細內容的部分，需要 migrate 舊版本的 `app/services/image_url_service.rb` 程式碼

Slack 通知的部分，則建立 Notifications context module 來實作 



# Refactoring

我要重構整個專案，以符合 elixir/phoenix best practice 的作法來改善

先完成目前我所知需要調整：

- Schema module 應放在對應的 context module 下，例如目前的 Venus.ActressFiture, Venus.ActressMedia，應該搬到 Venus.Actresses 模組下
	- Venus.Actress 則是多餘的程式，應已被 Venus.Actresses.Actress 取代，請檢查確認沒問題後刪除它
- 所有 Schema 為了相舊舊專案的 db table 欄位，都有 `timestamps(type: :utc_datetime, inserted_at: :created_at)` 的定義，應抽出成 Venux.Schema 來共同定義，並取代 `use Ecto.Schema`



# 調整 Uploaders
請將 /lib/venus_web/uploaders/ 下的模組，包括：
- cover
- avatar
- picture

進行改寫修正：
- 改移到 lib/venus/uploaders/
- 改成正確的模組命名，例如：Venus.Uploaders.Cover

並且將存放檔案的方式，改成用 aws s3 bucket，並且
- 檔案要經過 signed url
- 儲存的檔案路徑請維持原本的格式

對應所需的環境變數我已經設定，包括：
AWS_REGION
AWS_S3_BUCKET
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY

可以參考 ../xaifu/ 目錄下另一個專案的作法



# 實作 pending work 管理頁面

請實作新功能 Pending Work 管理
目的是管理目前狀態為 pending 的 ActressWork

這個功能主要包括

- 列出 pending ActressWork
	- 和 HomeIndex 相同的方式取出 pending works
	- 呈現上一樣先以 actress 來分組，再列出 works
- 可以對單筆 work
	- 單獨執行 collect 的動作
	- 單獨執行 ignore 的動作
- 也可以勾選多個 work
	- 整批執行 collect 的動作
	- 整批執行 ignore 的動作


### 需求：

### 介面與行為
列出 pending work 的畫面和 HomeIndex 不同
要列表呈現，以方便列出資訊和操作
- 一樣以 actress 分組，照組別依序呈現
- 每一組以兩欄呈現
	- 左側欄較小，顯示 actress 資訊，可用 actress card
	- 右則欄較大，顯示 work 列表，用 table，欄位包括
		- 勾選欄：做為批次執行動作的選取方塊
		- 資訊欄：顯示 actress work 的資訊，包括：
			- 封面
			- 一些 properties 資料
			- 關聯 site_entriess 的 url links
		- 動作欄：有單獨 collect 與 ignore 的操作
列表上方有「批次動作」的按鈕，表示「將勾選的 work」整批執行「collect」或「ignore」
注：所以勾選後執行批次的跨分組的，只要有勾選的就都會一起執行

當動作送出 (無論是單筆或整批)
會執行 collect 或 ignore，代表會將目標 work 的狀態進行修改，不再是 pending 狀態
因些送出完成更新後，會要將變動的 work 從目前的列表消失

### 後端處理
server 端若收到批次 collect 或 ignore 的請求時
實作方式我希望還是各別 work 呼叫 collect 或 ignore
可以包裝成 Works.batch_collect_work(), Works.batch_ignore_work
但裡面還是各別呼叫 .collect_work / .ignore_work

### 設計
請確保整個 ui 設計是友善直覺，而且風格與目前整個網站是一致的

若不清楚目前的風格，請先理解整個網站目前的設計，並整理成一份 design guidelines 放在  ai_docs/guidelines/ 下，並更新 CLAUDE.md


## Collect Work 時進行 Attach Pictures
請調整 Works.collect_work 的行為，加入以下動作

在 collect work 時，同時將該 work (ActressWork) 的 parsed_data 裡的 "images"
裡的 urls，建立成 ActressWork 的 pictures

- parsed_data 有可能是 json 或 yaml，請先用 Utils.decode_data 
- 從中取出 key 為 "images" 的資料，有可能是空值，就不處理
- 得到每筆的 url，有可能是各種 host 的網址，需要用 ImageResolver 處理得到目標圖片
- 將各個 url 的圖片，建立成該 work 的 pictures，但要排除既有重複的檔案，請做成像 upsert 的行為
- 未來有可能會需要過濾檔案不建成 picture，例如用圖片尺寸、url host 等，請先實作保留過濾機制，實際條件之後再建立&擴充

可以參考舊版(rails) 程式的作法，改寫成 elixir 實務上的設計
參考檔案 (../venus) 下：

- app/controllers/actress_work_images_controller.rb:7
- app/models/actress_work.rb:60
- app/services/image_url_service.rb

## 允許對 Pending Work 執行 attach images
請在 pending work 的功能中，增加對 pending work 執行 attach images 的動作
- 可以對單一 work 執行
- 也可以勾選多筆批次執行

attach image 只是對對象 works 執行 attach images 的動作，但不會改動 work 的 state

另外，因為 attach images 可能會花費較長時間
請將它 (PictureAttacher.attach_pictures) 改成 async tasks 的方式來運行

未來可能也會有類似的 async 的需要，但我不想用 oban
傾向在系統裡建立簡單的 task supervisor 就好



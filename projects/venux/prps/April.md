
# Todo

- [x] 加上 avatar 的 裁切設定
- [x] 對各種圖片加上點擊放大瀏覽的功能
- [x] 對各種 actress 圖片加點擊裁切功能
	- 裁切存為 avatar
	- 裁切存為新的 picture
- 增加設定 actress metadata
- 實作首頁看板
- 在 Actress List 增加篩選
	- by state
	- by rank

# Done
## 增加圖片放大瀏覽功能
可用現成的套件，例如 colorbox

需求：
- 只要在 dom 裡加上特定的 button 或 link，點擊就能跳出介面，顯示該 button 裡設定的圖片
- 跳出的介面可以是 modal 或任何現升套件來實作，但尺出不超過當前螢幕的 90%
- 若允許的話，可以切換上一張/下一張圖片，在當下頁面裡有同樣設定的圖片之間切換

先套用到 pending work 頁面裡，每個 work 的 cover 圖片
- 套用顯示的圖片，是 cover 的 original version


## 優化 Actress 新增/編輯表單
請優化 actress_controller 的 new 和 edit 頁面
- 裡面的編輯表單，請抽出成共用的 form_component
	- 將 description 調整成 textarea 欄位
	- 將 rank 調整成 scrollbar 的型式
	- 且確保表單送出後，create 和 update 的功能正確寫入資料庫
- 並重新設計表單的介面，附合整體的樣式風格，並且確保 ui friendly
	- 要有可以回到 actress 列表或 show 的連結

## 調整 PhotoCard component
對 PhotoCard component render 的 photo_card 進行以下調整
-  photo_card 裡的圖片可被點擊，點擊後會跳出 modal，顯示整張完整的原始圖片，
- 在 render photo_card 時，允許傳入 actress_id
	- 如果有傳入 actress_id，圖片被點擊時跳出的 modal，modal 下方會出現：
	- 左方有有兩按鈕，按下後可以對 modal 中的圖片範圍選擇
		- 第一個按鈕代表進行 avatar 裁切，此時選擇範圍會限制比例為 1:1
		- 第二個按鈕代表進行 figure 裁切，此時選擇範圍的比例不受限制，可任意框選
	- 右方有一個送出按，按下後表示送出裁切，參數用 crop 包裝命名，裡面包括
		- crop[type] :目前正在裁切的類型，avatar or figure
		- crop[actress_picture_id] : 當前的 picture_id(如果有需要的話，視實作方式而用)
		- 選擇的切裁位置資訊: 視實作方式將資料用 crop[...] 傳出
		- 
	- 送出會將資料送到 actresses/:id/crop 這個端點
		- 實作 actress_crop_controller
		- 用 :id 取得 actress
		- 用 crop[actress_picture_id] 取得圖片
		- 用 crop[type] 得到類型，依類型決定動作
			- 若是 avatar，則將該 actress 的 avatar 圖片換成傳進來的裁切圖片內容
			- 若是 figure，則新增一筆 actress_figure 給該 actress
				- 注意，actress_figure 在存入時，也要建立對應的 width, height, ratio 資料
	- 處理裁切圖片的實作方式，可以使用 js 套件
		- 可參考 cropper.js 這個套件
			- 可參考舊專案(rails 版本的作法)，位於 ../venus 路徑下的檔案:
				- app/assets/javascripts/application.js
				- app/controllers/actress_croppers_controller.rb
				- app/models/actress.rb
		- 或者有更適合 phoenix / elixir 的作法
	- 送出的 request 處理完成後，用 flash 訊息跳出結果，不用重新載入畫面
- 將 WorksLive.Show 頁面中 pictures 在 render PhotoCard.photo_card 時，傳入當前的 actress_id

請進行以下修正
- 點擊圖片跳出的 modal，將它固定在當前畫面螢幕的中央(而不是整個頁面)
- 當點擊 crop 的按鈕，cropperjs 並沒有作用，沒有出現裁切選取的效果，請檢查並修正

Cropper 仍然沒有運作，照理說它應該會在 new Cropper Init 時，在 image 的下方 dom 產生 cropper 相關的 html，但卻沒有

另外，請修改成，當按下 avatar or figure 的按鈕時，才建立 cropper

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


# 增加建立與編輯 Subject 的功能

在 /subjects 頁面，加上 建立和編輯 subject 的功能

請將建立/編輯 拆成一個共用的 form_component for subject

輸入的欄位會因是 新增 或 編輯 而不同

新增 subject 時，欄位包括
- belongs_to 的 site
- name
- term
- enabled: 開關，代表是否啟用
- trackable: reference to Actress，提供下拉選單
- sites: subject has many sites through subject_site，可設定相關的 sites，用 check boxes 做多選

另外，可以允許在建立 subject 時同時建立  trackable 的 Actress
也就是當沒有選擇 trackble 時，會同時建立(upsert) Actress

因此畫面，同時提供屬於 trackable (actress) 的欄位：
- description
- remark

在沒有選擇 trackable，要處理 actress 時
會用 subject.name 或 subject.term 作為 Actress 的 name，會先找尋有沒有 name 相符的 actress
若有就會用既存的 actress 當作此 subject 的 trackable
若沒有就會新增 actress，並寫入 description 和 remark

當有選擇 trackable，就不需理會 description 和 remark 欄位
當沒有選擇 trackable，就會在建立 subject 時同時新增 actress (name+description+remark)

但編輯 subject 時，只可以修改 
- name
- term 
- enabled
- sites

但不會變動任何有關 trackable(actress) 的資料



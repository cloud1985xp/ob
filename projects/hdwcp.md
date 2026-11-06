
# 期貨布料/葉片增加產地機制


一、新增生產商資料
新增一張表(manufacturers)來存生產商
- id
- name: 字串欄位，不可空白

並建立兩筆資料，
- HDCN
- HDNA

二、訂單商品，關聯至生產商
在 order_detail 訂單產品，要增加生產商 manufacturer_id (foreign key) 欄位來代表期貨的生產商
- 允許空值
	- 非期貨不需要設定生產商
	- 舊資料不需 migrate，維持空值
		- 空值及代表預設生產商，顯示為預設(中文) 或 default(英文) 就好
- 建立 order_detail (下單)時，若是期貨商品，也會先維持空值，待接單時才處理/設定生產商

三、定義布料/葉片(model) 的布料顏色生產商
有部分的葉片布料(model)，會要有「期貨產地(生產商)設定」
代表這個布料有多個生產商 model has many color_manufacturers

開一張資料表 (model_color_manufacturers)，關聯至 model 及 manufacturer，包含欄位
- model_id (fk),  not null
- manufacturer_id: (fk), not null
- color_code: 字串欄位, 不可空白

四、調整訂單接受功能
在訂單接受畫面(Acceptance)表單

在列出訂單明細項目時，多出一欄「產地」的設定，在 `顏色碼` 欄後面、`說明` 欄前面

如果該筆商品(order_detail) 
- 是期貨商品，且
- 任一葉片(model1 或 model2) 的顏色 (對應的 color1_code 或 color2_code) ，有命中對應的期貨多產地設定裡的 color_code
就會出現產地的選項(select)，可以選擇，預設為空值，且必填

若該筆商品(order_detail) 不是期貨，或沒有布料顏色有符合的 model_color_manufacturer 資料
那該欄就顯示為 default

註：就算符合的 manufacturer 資料只有一筆，一樣要顯示下拉選單選擇

表單送出後，會將 manufacturer_id 資料送出，寫入 order_detail
並會依該筆 order_detail 的 manufacturer_id 是否為必填來進行驗證

五、管理 Model Color Manufacturers
在葉片/布料瀏覽頁面，即 /products/models/:id
增加一個 panel，列出與這個 model 關聯的 model_color_manufactures 資料，中文標題為「期貨顏色生產地」，並有一個編輯按鈕，連到管理畫面：/products/models/:id/manufacturers

管理畫面列出表單讓使用者管理與該 model 相關的 model_color_manufacturers 資料

可以新增、修改、刪除資料列，每列需要設定
- 選擇 manufacturer: 下拉選單
- 輸入 color_code: 文字欄位
需檢查 model_id、manufacturer_id、color_code 的組合不可重複

六、批次上傳 model color manufacturers





在產品的布料/葉片，當期貨顏色時，會區分
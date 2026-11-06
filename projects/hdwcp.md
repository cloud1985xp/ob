
# 期貨布料/葉片增加產地機制


一、新增生產商資料
新增一張表(manufacturers)來存生產商
- id
- name

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


五、


在產品的布料/葉片，當期貨顏色時，會區分
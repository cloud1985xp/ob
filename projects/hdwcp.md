
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

有部分的葉片布料(model)，會要設定「期貨產地設定」

開一張資料表，關聯至 model table，包含欄位
- model_id (fk)
- manufacturer_id: (fk)
- color_code: 字串欄位

在訂單接受畫面(Acceptance)表單
在列出訂單明細項目時，如果該筆商品(order_detail) 
- 是期貨商品，且
- 任一葉片(model1 或 model2) 的顏色 (對應的 color1_code 或 color2_code) ，有命中對應的期貨多產地設定 裡的 color_strings 
若 model 沒有設定 


在產品的布料/葉片，當期貨顏色時，會區分

# 期貨布料/葉片增加產地機制


order_detail 下訂的產品，要增加產地的欄位

- 新增一張表來存生產地(manufacturers)
- 在 order_detail 加欄位 manufacturer_id (foreign key) 來代表期貨的產地
	- 允許空值
		- 非期貨不需要設定產地
		- 舊資料不需 migrate，維持空值
			- 空值及代表預設產地，顯示為預設(中文) 或 default(英文) 就好
- 建立(下單)時，若是期貨商品，也會先維持空值，待接單時才處理/設定生產商

定義布料/葉片(model) 的布料顏色(color)產地設定

有部分的葉片布料(model)，會要設定「期貨多產地設定提醒)

開一張資料表，關聯至 model table，包含欄位
- model_id (fk)
- color_strings: 字串欄位，用來填寫多組色碼，用「,」間格

在訂單接受畫面(Acceptance)表單
在列出訂單明細項目時，如果該筆商品(order_detail) 是期貨商品
且任一葉片(model1 或 model2) 
的顏色 (對應的 color1_code 或 color2_code) 
有命中 對應的期貨多產地設定 裡的 color_strings 
若 model 沒有設定 


在產品的布料/葉片，當期貨顏色時，會區分
# 實作 LoraBuild 功能

系統可以定義 LoraBuild
每一個 LoraBuild 會有：
- name: string, 標題名稱，必填
- description: text, 描述，可不填
- nodes: 可以有多個 LoraBuildNode

每一個 LoraBuildNode 上有：
- position: 數字，在同一個 lora build 下 position 不得重複，取出 nodes 會依此排序
- name: string, 紀錄 lora model name
- strength: float, 紀錄 model 強度

需在 app 端實作一個 LoraBuild + Nodes 的管理功能：
- 路徑: /app/loras/
- index 頁，列出所有 lora builds，列出名稱和描述
- new 與 edit 頁，共用 lora_form
- 在 lora_form 中：
	- 填寫 name 和 description
	- 可以增加/編輯 node
		- 增寫 name、strength
	- 可以刪除 node
	- 在送出 form 時 save (create or update) LoraBuild 和 LoraBuildNode
		- 會依當下的順序設定 node 的 position


# Generation 可以套用 LoraBuild
實作 Generation 套用 LoraBuild 的功能
即 Generation has one LoraBuild，可以是 optional

當 Generation 套用 LoraBuild 時，會依 LoraBuild 裡的每組 Node 指定對應的參數

需建立關聯資料表 GenerationLoraBuild，裡面有
- generation_id: foreign key to Generation
- lora_build_id: foreign key to LoraBuild
- parameters: map，用來存放設定 lora build 的參數

在 Generation form 裡，可以選擇 lora build
當 generation 有套用 lora build 時
會出現對該 lora build 的各 nodes 的參數覆寫設定
可以覆寫各 node 的:
- name -> 要使用的 lora model name
- strength -> 要使用的 strength 參數

將所有 nodes 的設定，存在 parameters 欄位(map) 裡，可用 node position 當作 key

實作 Processor 支援 Lora Parameters
實作 Processor 在處理 Generation 時，將 lora build 的資料製作成參數加到 json_content 中
當傳入的 generation 帶有 lora build 時
需進行 lora build 的 parameters 處理，包括

找出原 json中的 加載器節點
找出 type 為 `CheckpointLoaderSimple` 的節點，紀錄它的 id(key) 來作為第一個 lora node 的 input


將參考 generation 的 lora build parameters
取出對應的 lora build nodes，沒有被 disabled 的 nodes
依序組出 lora nodes
用加載器節點的 id 當作第一個 lora node 的 input id
下一個 lora node 用前一個 lora node 的 input id

- 每個 lora node 的 id 可用 9000 為 base 開始增加，避免碰撞
- 每個 node 會有 name、strength 的參數，從 generation lora build 的 parameter 中取得，若沒有則用 LoraBuildNode 裡的預設值
每個 node 的內容
```
id: {
	"_meta": {
	  "title": node_title
	},
	"class_type": "LoraLoaderModelOnly",
	"inputs": {
	  "lora_name": name,
	  "model": [
		"134",
		0
	  ],
	  "strength_model": 1
}
}
```


找出原 json 中的採樣器節點
找出 type 為 `KSampler` 的節點

將採樣器節點中的 inputs 裡的 model 設為 lora 最後一的 node 的 id
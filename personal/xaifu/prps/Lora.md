# LoraBuild

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
- 在 lora_form 中T


Generation 可以套用 LoraBuild
當 Generation 套用 LoraBuild 時，會依 LoraBuild 裡的每組 Lora 指定對應的參數
# LoraBuild

系統可以定義 LoraBuild
每一個 LoraBuild 可以有多個 LoraBuildNode
每一個 LoraBuildNode 上有：
- position: 數字，在同一個 lora build 下 position 不得重複，取出 nodes 會依此排序
- 

Generation 可以套用 LoraBuild
當 Generation 套用 LoraBuild 時，會依 LoraBuild 裡的每組 Lora 指定對應的參數
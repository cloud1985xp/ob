我想使用 Obsidian 做知識管理
目前將筆記放在 /Users/aaron.kuo/Documents/000_Note/Note/ob 路徑下

但內容有些雜亂，有包括個人、工作以及知識學習各方面的文件

我想將知識學習管理的部分做統整
已知相關的目錄有：
 - learning
 - tech

除此之外，還有一些其他資料收集的來源，包括

- 利用 Obsidian Clipper 的 Google Chrome extension，將收集的資料文件放在 00_inbox 目錄下
- 不定期在 facebook 上收藏的資料，放在我的珍藏裡的「程式技術」分類下
- 不定期在 youtube 上收集的「稍後觀看」影片

我想建立一個適當的管理流程與 wiki 知識庫
目前我的規劃是：

每天啟動一個 agent，進行：

1. 將 facebook 上收集的珍藏文章，轉存到 00_inbox 下
2. 將 youtube 上收集的稍後觀看影片，透過 notebookLLM mcp 解析內容，產生 markdown 文件存到 00_inbox 下
3. 將 00_inbox 內的所有文章做彙整，分析分類與內容，依分類，存在 00_wiki 目錄下

各分類下再依文章主題做統整，例如相關主題的文章合併，並對文章設定好 tag、日期 等 metadata，方便 obsidian 檢索

或是建置成 wiki 頁面
甚至搭配建立更完整的搜尋索引或是 rag 系統

請給我一些作法的建議

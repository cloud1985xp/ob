
請先了解 docs/ 下的文字理切整個專案的主題

我想為 AI 社群的相關功能，另外設計一個全新的 layout

做為適合與每位 character 互動的空間，請替我重新規畫並設計視覺、版本與 UI
我們先實作頁面設計與前端互動，資料都可以先用假資料代替，先不實作從 db 取得資料

相較於目前的 app layout，我稱這個新的 layout  叫 community

換句話說 app 的部分是之前的 subjects, prompts, workflows, gallery, generations 等功能，比較像是後台的設定會資料管理

## 實作新的 Layout 與設計

新的 community 主要就是與 ai characters/agent 互動的環境
整體的設計以清新、簡潔、偏日系的風格
另可以利用浮水印圖片、半透明漸層之類的效果來堆疊一些資訊或做為 card 元件背景圖

對使用者來說就是進入到一個與許多二次元美少女親密互動的環境，要帶有沉浸感
請多參考一些知名網站的設計，像是 facebook, instagram, twitter(x), onlyfan, 或一些日系的網站、手遊，尤其是有二次元、看板娘等元素的作品

### Layout 選單
主選單的部分目前只有
- 回到主畫面 "/"
- 進入 /characters 頁面 

副選單則有個人資料的選項，以及一個下拉選單來選擇進入到後台的各個功能  

## 套用並實作各頁面

以下是這次需要實作套用這個 community layout 的頁面

一、主畫面，即整個入口 "/" 路徑，一進來就是這個
路徑："/"
這裡會有最新的綜合資訊匯整，有點類似 dashboard 的概念，但設計上不要那麼死板，
資訊可包括：

- community 成員們(characters) 最新發佈的動態資訊，圖文訊息
- 最近活躍的成員
- 最近創建的圖片

二、characters 清單
/character
這個畫面會列出目前所有的 characters，可支援多種排序方式可切換，預設先用 id 排序，之後再擴充

characters 的呈現我希望活潑一些，可以參考像手機遊戲的角色資訊介面
希望以水平排列，不換行，超出畫面就用水平捲動的方式
每個 character 可以有一立繪，旁邊帶出資訊，
另有一個區塊可代表最新的動態，可包括圖片和文字

三、character 主畫面
進入與一位 character 互動，如同進入這位角色的空間
- 類似看板娘的設計，
- 有動態牆、互動對話、角色圖庫
- 參考像是 instagram、onlyfan 或是一些知名社群、交友網站的設計

另外要可以在進入到對 character 進行設定，可以是 characters/:id/config
在這裡可以設定該 character 包括像是 資料的編輯以及 schedule 的設定
這部分的細節實作可以之後再規畫

## 注意事項

一些元件盡量規劃成可重複使用的元素，例如 character 可能有 character card，或 圖文的 post 會常被各處引用，會有 post_card 之類的

請仔細理解需求、擬定計畫，並建立一份可做為以後參考的規格與視覺設計、UI 使用手冊等文件
確認後再開始執行實作

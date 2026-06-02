
# PromoCard Preview Dev
https://github.com/aktsk-pjt-ishin/tw-ishin-devops/blob/feature/ISHINTW-15490-promo-chara-info/web/promo-chara-info/BACKEND_API_SPEC.md

資料來源

- jp csv

支援多語言？
- 若遇到該語言沒有的資料，要可以輸入？


# 新增 Card PromoPage 功能

##需求
整體功能如下：
- 使用者輸入
	- card_id (text field)
	- 想要的 server stage 環境 (ex: chaozu、bulma、beerus...etc)，單選
	- 想要使用的 design assets branch(text field)
	- 想要瀏覽的語言(單選)
	- 想要顯示的技能資訊項目(複選)
	- 型態變化文字標題(單選，會依瀏覽語言變化)
- 畫面上會出現這張 card 的 promo pages

實作上想分成兩個階段

階段一只先實作前端 Card Promo Page 的顯示畫面功能
先假設收到所需的參數(card_id、server stage、design assets branch)
card promo page 會向 server 呼叫 api，會回傳一份 json 資料
這份 json 會顯示在 textarea 中，對前端 app 來說是讀這份 json 資料來顯示
使用者也可以手動調整 json 的內容，來影響 render 的結果，例如調整文案

所以階段一只先做前端的部分，可以讀 textarea 的 json 來 render pages
階段二才做串接呼叫 api 取得 json 資料的部分

## 階段一目標
- 讀取 textarea 裡的 json 成為資料
- 前端 render 出多個不同 page 的內容，目前包括：
	- 角色基本資料 (character profile)，
		- 一個頁面
	- 技能介紹 (skill information) 
		- 會依使用者勾選要顯示「哪些技能資訊」來決定要顯示哪些內容，分成兩頁
		- 技能資訊會有：
			- 隊長技
			- 被動技
			- 主動技
			- 預備技能
			- 預備完成效果
			- 必殺技
			- 超必殺技
			- EX 必殺技
			- 聯合必殺技
			- 領域技能資訊
			- 型態變化
		- 但不是每個 card 都會有全部的資訊，依 json 內容而定

## 技術實作

要注意
- layout page 的定義與管理，要方便未來修改或日後擴充
- layout page 裡用到的一些文字或圖示、圖片，會依瀏覽的語言而不同
	- 每種語言會對應一組圖片、文案

## 參考程式碼
目前已經有一種 prototype 的前端 mockup，程式碼位置：
/Users/aaron.kuo/aktsk/ishin-tw/ishin-devops/web/promo-chara-info

但只需要參照它的頁面樣式設計，設計的部分必須完全符合
而程式實作的部分
請用符合現下專案的架構風格與技術
包括
- 使用 tailwindcss 來實現
- 整合 js 進 esbuild with rails project
- 可選擇導入 vuejs 來實作，以達到方便管理 layout page 樣版
- 圖片檔名全用英文
- 整理成乾淨好維護的架構

json 檔的內容，請參考 mockup 專案下
> mock/card_1027220.json

若有需要，可以拜訪 http://localhost:3000/control.html 頁面看上述專案運行的結果

請先詳細了解 mock 專案的內容
重新規劃清楚完整、乾淨的新前端程式架構
建立實作規格和計畫，再開始實作
若有任何問題或建議，請提出來討論

## 幾項調整

一、
「輸出語言」的選項，請移到跟上方的 Card ID, Environment, Git Branch 等一起
放在該區塊的下方即可，
未來這區塊內的欄位，應該會做為呼叫 server api request 的參數，來取得 json 內容

二、
Canvas 所在的元素  (cp-page-frame, cp-canvas) 區塊，
可以隨著父元素變大嗎？最大到 1080 
讓 canvas 的內容也能按比例變大，但內容結構不變


## 階段二：實作 assets image url

一、實作產生 github design repo 圖像的 url

在 server 端實作一個端點
 /design/:branch/images/{:file_path}
路徑參數:
- branch，代表 design github repo 的 branch，可能會包含 '/'，
	- 例如：`merge/v6.4.5-gl`, `20260604_deployment_asset`
- file_path 會是一個圖檔的路徑字串
	- ex: `/resources/card/1027220/card_1027220_character.png`

Server 端實作功能，會用指定的 branch 和 file_path
配合 ENV["GITHUB_ACCESS_TOKEN"] 生成一個轉傳到 github  取得對應檔案內容的 request
也就是要讓 client 端 browser 可以直接用像 
`<img src="/design/:branch/images/:file_path" />`
這樣的網址，顯示 github 上的圖檔案

github repo 為：https://github.com/aktsk-pjt-ishin/tw-ishin-design-lfs.git

二、 調整前端讀取圖片的 url
在 CardPromo Client 端
調整在從 json 取得圖片路徑要使用時
將圖片路徑的 base url 設成：  /design/{當下選擇的branch}/images/{路徑}

請詳細評估上述需求與實作方法
若有疑問或建議請提出討論

## 階段三、實作 Promo Card Data API
接下來要實現 server 端的 data api
讓前端傳入參呼，server 從 db 取得對應的資料，回傳 api 結果

預計回傳格式
成功:
```
{
  status: "success", 
  data: {真實的 data }
}
```

前端從 data 得到的資料，當作 json 放進 textarea，然後觸發畫面更新

失敗
```
{
  status: "error",
  message: {ERROR_MESSAGE}
}
```
前端會顯示錯誤的 message 資訊

取資料時
- 要用對應的語言切換 i18n
- 要依傳入的 environment(stage name) 取得對應的 server_stage
- 用該 server_stage 切到對應的 shared，以及用對應的 app version，來取得對應的 model

ex:
```
I18n.with_locale(language) do 
	Globalization.with_stage(server_stage.name) do
		card = IshinServer.versioned(server_stage.app_version, "Card").find(card_id)
	end
end
```

取得 card promo data 的邏輯，
可以參考 mock 專案 (/Users/aaron.kuo/aktsk/ishin-tw/ishin-devops/web/promo-chara-info)
中的 ./BACKEND_API_SPEC.md 文件說明
來取得資料庫裡的資料，但請改寫成 rails / activerecord 風格與符合本專案的架構

對應的 model 和資料表欄位，
可以參考本專案的 config/ishin_server/schema 下的 yaml 定義檔案
本專案會用這些 yaml 來動態生成 IshinServer 下的各種 model class
例如 IshinServer::Card，或定特 app version 的 class: IshinServer::V6_0_0::Card
詳情可以參考  lib/model_builder.rb 裡的設計

關於 i18n 語言，設計上已經實作，只要有切換到對應的 i18n.locale
在呼叫物件的方法時，例如  IshinServer::Card#name，會自動使用對應的語言資料

請詳細閱讀以上參考資料來規劃設計
實作取得正確 card promo data 的 API

若有內容不確定如何取得，或上述 API SPEC 文件不正確之處，請提出來
任何疑問或建議也請提出跟我討論討論







PSD 字形大量轉換
https://github.com/psd-tools/psd-tools
https://psd-tools.readthedocs.io/en/latest/

Skills
- investigate issue of master check

To build ISHIN Code KM
https://github.com/safishamsi/graphify
# Env Tool 

https://github.com/jdx/mise

https://github.com/zhongwencool/observer_cli


Alternative of btop
https://github.com/nicolargo/glances

nvim with agentic workflow
https://github.com/olimorris/codecompanion.nvim

nvim with MCP
https://github.com/ravitemer/mcphub.nvim

AutoResearch
https://github.com/karpathy/autoresearch

Skills

Elixir 
https://github.com/oliver-kriska/claude-elixir-phoenix

SDD
https://github.com/garrytan/gstack/blob/main/docs/skills.md
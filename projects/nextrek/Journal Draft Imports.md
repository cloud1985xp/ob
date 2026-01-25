---
tags:
  - nextrek
  - project
---
# 調整前後端運作架構
在繼續補足 Journal Draft Import 的功能細節前
我想先針對目前已實作的部分先做調整
## 建置共用 API
我想先建置一些取得群組常用資源的 API，資源包括
	- tags
	- contacts
	- subject menu
- 這三項資源是許多情境都會用到，所以事先規劃成獨立未來也可重複使用的 API

我希望可以同時有分別三種 resources 的 api，例如
/api/tags - 取得標籤列表
/api/contacts - 取得聯絡人清單
/api/menu - 取得會計科目選單
/api/accounts - 取得資金帳戶清單

但也可以有一支 api 透過參數來一次取得多種資源，例如

/api/resources?tags=1&contacts=1&menu=1

## 重整前端組件
目前的前端 js 設計太過僵化，請適度做重構，提高使用彈性並降低耦合
這裡先列舉幾個未來可能也會需要考慮到的功能，希望在未來的開發上也能依循/複用現在開發出來的前端元件，功能包括：

- 單一筆 Journal 輸入與編輯表單，也會有 accounts 、subject menu、tags、contacts 等輸入元件
- 搜尋的篩選，也會有 subject、tag、contacts 等輸入條件
- 交易移轉的表單，同時有多組 (移出 與 移入) 的 account 選單、多組 subject 

而這次 import 功能裡有使用到幾個 input 元件，包括 
- tag 選單
- contacts 選單
- subject 選單，
目前是寫死的，需要做出以下調整
- 選單的選項是依當下使用者所在帳本(group)呼叫 api 取得的
- 這幾個 input 的 javascript/html/css 應被設計成可重複使用的組件
- 介面上的一些文字、屬性應可被參數化，例如 placeholder 的文字，可以動態透過像是 data-attribute 來定義(例如考慮 i18n，由 server render 屬性)

如果可以，把這些組件抽出用 Stimulus 來做綁定，但
- 保留目前設計的動態效果(例如繼續使用 toastr)
- 維持目前的設計樣式，放到獨立的對應 css 檔案來管理樣式

如果有其他建議或可評估的方案，也請提出來討論

# 改善&重構 Journal Draft 的前端程式
好，接下來請開始將新版的 journal drafts 功能，替換整合成使用新設計的 select 元件，可以重構 journal drafts 的前端程式

# 建立 mock api 測試

我想製作可以讓前端工程師方便工作/測試的 mock api，
可以用 swagger 之類的解決方案嗎？
等於同時建立 api 文件以及 mock api 供測試

請評估適合的解決方案
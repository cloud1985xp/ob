目前 Async Load 的設計，在用戶有大量資料(tag/contact)的情況下依然會很慢
原因應該是：
- api 回應資料過多
- render option內容過多

我想調整成
- async load 載入時，對 api 傳送 limit 值，即只 render 最多 limit 數個選項
- 這個 limit 值取自 select input 上的 data attribute，讓它可依視情境決定不同的 limit 值
- 並且，當使用者輸入時 ，會再呼叫 api 去用名稱做 like %term% 查詢，來動態更新選項
	- 要做 debounce 避免太頻繁呼叫
	- 可能要修改 api 的部分

並更新元件並重新檢查所有修改的地方，如果有不同行為可以設計成不同的元件
但請維持設計風格與使用方法的一致性，並盡量重複設計好的使用元件
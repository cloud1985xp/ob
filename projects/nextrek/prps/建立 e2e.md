我想讓專案調整成能讓 Claude Code Agent 確實完成 e2e 的測試
並把這整個機制建立成一套系統化的方法(例如 skills)

未來每個功能，都能透過讓 agent：

- 經由閱讀規格與程式碼，理解功能需求
- 擬定 e2e 測試
- 進行 e2e 測試，甚至寫成 rspec 自動化的 integration test、feature test
- 整合 tidewave
- 產生測試報告

請與我討論並協助我完成這項目標
若為了實現目標需要針對 development / test 環境進行程式碼調整，也請提出來

請用目前這個 branch 開始的新版 journal-import 功能
(compare with develop branch)
來當作第一個嚐試

用 brainstorming 詳細規劃確認後，再開始執行


我想加強 rb-e2e 這個 skill
能達到我需要的：請 ai 繼續增強測試的仔細度與完整度

希望可以想像是：

使用者給一段提示

一、先列出針對當下這個 feature 現在已經實現的測試
二、請 ai 依使用給的提示，進一步深度思考還需要的測試情境
三、列出情境與使用者討論，確認要增加的項目
四、開始實作增加的項目
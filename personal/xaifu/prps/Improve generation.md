## 實作複製功能
請在 GenerationLive.Show 增加一個複製的功能

實際行為即為「另存 Generation」的方式
可對當前的 Generation 點選複製
出現一個另存 Generation 的表單
使用者可以在表單中，調整要複製的 Generation 的設定
按下送出後，會存成一個新的 Generation

在複製 Generation 時
- 可設定的內容，與編輯 Generation 時相同，請試著 reuse 相同的表單、抽出成共用的表單元件

追加「設定 subject」
- 請調整編輯/複製 Generation 用的表單，加上可以修改 Generation 所屬 Subject 的選項


原本的編輯功能現在會發生錯誤：

```
key :cancel_event not found in: %{
  id: "generation-form",
  socket: #Phoenix.LiveView.Socket<
    id: "phx-GJWIS1mF_K1OZQjJ",
    endpoint: XaifuWeb.Endpoint,
    view: XaifuWeb.GenerationLive.Edit,
    parent_pid: nil,
    root_pid: nil,
    router: XaifuWeb.Router,
    assigns: #Phoenix.LiveView.Socket.AssignsNotInSocket<>,
    transport_pid: nil,
    sticky?: false,
    ...
  >,
```

請修正

# 強化 Generation Form 的功能

對現在的 Generation Form 做以下功能強化

## 可同時輸入 prompt text
對每一組 GenerationInputs 的 prompt，除了可用選擇的方式之外，
增加支援可以用輸入文字的方式，在輸入的同時建立新的 prompt
會同時提供 text_en 與 text_zh 的 textarea 欄位供輸入

若使用者選擇用輸入文字，在送出 save 時，會建立新的 prompt 並套用至 generation input

### Promp Text 欄位支援翻譯
text_en 與 text_zh 的欄位，和 XaifuWeb.PromptLive.Show 裡表單的欄位一樣會提供 translate 的功能，可以點擊翻譯圖示直接進行將 text_en 中的內容翻成中文，傳至 text_zh 的欄位，或反過來將 text_zh 內容翻成英文傳至 text_en
請將這段翻譯的功能儘量與 XaifuWeb.PromptLive.Show 裡做成共用的模組

## 可在送出 save 前先進行 preview 生成
在 save 之外提供一個 preview 的按鈕
按下後會先呼叫 Generations.preview 的函式執行圖片生成
生成後會將圖片內容先顯示在畫面上

- preview 時會接受 Generation Form 裡的 prompt 資料，可以是用選擇的 prompt_id，也可以是 text_en 裡的文字
- 進行 preview 不會真的產生 GeneratedImage 資料，請將從 comfyui 生成的圖片資料讀進入轉成 base64 編碼後顯示在畫面上

### 重構 Generations.Processor

為了實現 preview 功能，請在 Processor module 增加另一個 generate 函式
來接受 preview 情境下的參數生成一張圖片，回傳 url 給 Generations.preview 再做後續的圖片處理

先確認目前 Processor 模組的現有實作，並進行適當的重構來減少重複的程式碼與複雜度
但不可破壞現有的功能




在 Generation (copy) Duplicate 的加上一個 preview 功能
進入讓使用者可以


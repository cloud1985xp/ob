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

# 實作先預覽再儲存的功能

支援指定尺寸的功能


提示詞解析
輸入圖片網址、檔案或貼上
使用 grok API 取得提示詞
使用 comfyui 取得標籤
使用 grok API 過濾標籤：服裝顏色、場景
串預覽的功能流程

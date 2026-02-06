

# Prompt Translator

我想要製作一個功能
來進行對提示詞的文字翻譯，包括英翻中或中翻英

背後的翻譯 provider 有兩種

- 使用 LLM AI 模型來進行翻譯
- 呼叫 Google Translate API 來翻譯

預設是使用 LLM AI 模型，Google Translate 之後再實作
而 LLM AI 也可以自由選擇要用的模型；預設是  `llama3.1`

## 一、實作翻譯模組

實作 Xaifu.Translations 這個 Domain Module
提供方法 Translations.translate 來進行翻譯
可以用 opts 參數來設定

- target_language: 目標語言
	- 若目標語言為 en，即假設傳入的文字是中文(zh_tw)
	- 若目標語言為其他，即假設傳入的文字是英文
- provider: 要使用的翻譯服務，例如
	- :llm, 表示用 llm 且模型是用預設模型 (llama3.1)
		- 或可以是 tuple 例如：{:llm, "gemini-2.5-flash"}，表示指定 gemini-2.5-flash 模型
	- :google, 表示用 google translate API (以後再實作)

收到參數後，決定用哪種翻譯服務，呼叫該服務對應的模組
- Xaifu.Translations.Providers.LLM
- Xaifu.Translations.Providers.GoogleTranslate (之後再實作內容)

Xaifu.Translations.Providers.LLM
- 會依中翻英，或英翻中，選擇不同的提示詞
	- 請將兩種提示詞定義成常數，方便調整
	- 提示詞中會有一組實際要翻譯內容的文字的 placeholder，是真的翻譯內容要置入的區段
- 實際呼叫 LLM 模型用 ReqLLM 這個 library
	- 定義適當的結構傳入 reqllm，方便取得執行後回傳的翻譯結果

## 二、實作功能

目前為要完成以下的使用情境

在 Prompt 管理的頁面 (PromptLive) 中，當編輯 Prompt 時，表單裡會有 text_en 與 text_zh 的欄位
我希望：
- 在 text_en 的欄位旁加一個翻譯 icon 的按鈕，點下後，會把 texten 的 textarea 裡的英文內容，呼叫翻譯模組翻成中文，放進 text_zh 的欄位裡
- 同理，text_zh 欄位旁有翻譯 icon 的按鈕，點下後將 text_zh textarea 裡的中文內容，呼叫翻譯模組翻成英文，放進 text_en 的 textarea 欄位

處理過程中可以把 翻譯 icon 按鈕轉成無法再點擊的狀態，且顯示正在翻譯中(或直接替換 icon)，直到翻譯完成後再恢復，恢復後使用者可以再次點擊

注意不要影響 PromptLive 的其他功能



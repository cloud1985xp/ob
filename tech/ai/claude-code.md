---
tags:
  - ai
  - ml
created: 2025-01-01
updated: 2025-01-23
status: active
---

# 2026 Jan

善用 Claude Code Web
https://code.claude.com/docs/zh-TW/claude-code-on-the-web

### 待研究資源
OpenSpec, Spec Driven Development
https://github.com/Fission-AI/OpenSpec

Ralph Wuffin / Loop
```
/ralph-loop:ralph-loop "use command `mix credo --all` to get the coding suggestions excepted the parts of `Software Design`. Try to modify the codes to meet those suggestions. Output <promise>COMPLETE</promise> when
  done." --completion-promise "COMPLETE" --max-iterations 10

# Use chrome to debug and fix
/ralph-loop:ralph-loop "use chrome to visit http://nrosetta.test/translation_reviews/ to check the CRUD function of translation_views. When the page got errors, read the error message and fix the function. Output <promise>COMPLETE</promise> when
  done." --completion-promise "COMPLETE" --max-iterations 10

```


# superpower

# chrome-devtools-mcp
主要是給 ide 整合(cursor等)，但目前我是用 claude-code + Chrome extension 的解決方案
所以不確定用不用得到

可以考慮，讓 agent 去瀏覽 localhost 時，使用 dev-tool 去看有沒有優化前端效能的可能

根據 Claude 給的建議：

專注於**Web 開發調試和效能分析**，提供額外功能：

|功能|Chrome Extension|Chrome DevTools MCP|
|---|---|---|
|**效能追蹤 (Performance Trace)**|❌|✅ 可錄製並分析 LCP、FCP 等指標|
|**CPU 節流模擬**|❌|✅ 測試慢速裝置效能|
|**網路速度模擬**|❌|✅ 模擬 3G/4G 等環境|
|**Lighthouse 整合**|❌|✅|
|**連接現有 DevTools 會話**|❌|✅ (Chrome 144+) 可選取 Elements/Network panel 中的元素|
|**使用 Puppeteer**|❌|✅ 更可靠的自動化等待機制|

## 建議

**如果你主要做 Web 開發和效能優化**，chrome-devtools-mcp 會很有價值，因為它能：

- 分析頁面效能瓶頸
- 測試不同網路/裝置環境
- 與 DevTools 面板互動（例如選取 Elements panel 中的元素讓 AI 分析）

**如果你主要做一般瀏覽器任務自動化**（填表單、抓資料、測試流程），現有的 Chrome Extension 已經足夠。


# Agents
https://github.com/contains-studio/agents
已經複製到 .claude/agents 下，可以試試看
不確定會不會已經過時了


Zed(Editor) with Claude Code
https://zed.dev/blog/claude-code-via-acp
https://zed.dev/

Claude Code Anywhere
https://happy.engineering/
https://github.com/slopus/happy
但應該已經可以被 Claude Code Web 取代

Google Workspace AI Workflow
https://workspace.google.com/studio/



## Boris Cherny 在 X 上的分享
Claude Code 的作者 Boris Cherny 在 x 上提到了自己如何使用 Claude Code 的方式，使用方法非常簡潔，其實有長期追蹤 Claude Code release note，就會發現他的使用方式就是他開發流程上需要，所以設計出來的模式，一種非**傳統** IDE，一種新的使用 pattern。

他也在近期的訪談中提到 Anthropic 80%以上的開發都是直接使用 Claude 直接撰寫完成的。剩下的 20% ，我想就是 [Claude.md](https://l.facebook.com/l.php?u=http%3A%2F%2FClaude.md%2F%3Ffbclid%3DIwZXh0bgNhZW0CMTAAYnJpZBExUVZtalNQbVNMZGV2SElxbHNydGMGYXBwX2lkEDIyMjAzOTE3ODgyMDA4OTIAAR75X0xOCqiMhPZfjYaYYSPu7pEp3cy_9zz3KBJZ5gdRYUQXeyzA2Psqkg6iag_aem_kqdbuGbGpPdfkJiUwzafjw&h=AT3ezAa7fWbl7E_NBG75A94TW65IVC-HGshMavFfM8dqx-8C_vKc7EDhNKnVHBfqCiwEx3PoQjJsNuDlY3442oXTHVDkoZEtpDUxvaQINfnH80CZeEhh_TbPSeK5qiOKsF0lQSXuRkAkSNTqQKM&__tn__=-UK-R&c[0]=AT1MS7IWfjvIf8josJml3dGdmXeql8BTm_sfJbVVWC8j7Gbf7sWdMMQxx2m4vOI6IQgpF8IcRRA13ikB0UYwW06uMtSs6hNzP13VoXc6f5xz8M6MDujNpfnFrXSSRmcexmrQDa_r5pMRqMIyjiTXVKGNFrYrKj8-ELk0dI8CMC5H4h8Fq5ICP74f_PY) 管理專案記憶以及 subagent/skill 的設計。以下是他的方法：

**1/ 終端並行運行 5 個 Claude**

Boris 在終端開 5 個分頁，編號 1-5，用系統通知來提醒哪個 Claude 需要輸入。這是最基本的並行架構。

**2/ 網頁版再開 5-10 個 Claude**

除了本地終端，他同時在 [[claude.ai/code](http://claude.ai/code?fbclid=IwZXh0bgNhZW0CMTAAYnJpZBExUVZtalNQbVNMZGV2SElxbHNydGMGYXBwX2lkEDIyMjAzOTE3ODgyMDA4OTIAAR5_uYZ2-Ha1xMgJoeSlbJRLnhgu1E4hSJwzvnhF4CvWXNCcceKmzv_DCdqkCA_aem_ipQt_-Iii30AWd4DbJcF0g)]([http://claude.ai/code](https://l.facebook.com/l.php?u=http%3A%2F%2Fclaude.ai%2Fcode%3Ffbclid%3DIwZXh0bgNhZW0CMTAAYnJpZBExUVZtalNQbVNMZGV2SElxbHNydGMGYXBwX2lkEDIyMjAzOTE3ODgyMDA4OTIAAR5BoH66QdC53TRAexBCU4LzjCHVwkKMgIb1wovy3BSuVsO5xDNqU1Hjz5r30A_aem_X_m7xjTRrQ7o1u6_5BYZ0Q&h=AT34TLKbGBMP-6p2lgyzIzrXSRcA98ZyCl2JJUgsRH2LYQS8nOLnV0EtztIlczwXM0WmBc2nRfAitMQWW_75Gh0AGVq9NCCP8rKGxUPwPZyNilSBhqpdBGHOfH5sjCaKdMN9COutqq3ttRVBDjg&__tn__=-UK-R&c[0]=AT1MS7IWfjvIf8josJml3dGdmXeql8BTm_sfJbVVWC8j7Gbf7sWdMMQxx2m4vOI6IQgpF8IcRRA13ikB0UYwW06uMtSs6hNzP13VoXc6f5xz8M6MDujNpfnFrXSSRmcexmrQDa_r5pMRqMIyjiTXVKGNFrYrKj8-ELk0dI8CMC5H4h8Fq5ICP74f_PY)) 開 5-10 個會話。本地會話可以用 `&` 交給網頁繼續，也能用 `--teleport` 來回切換。他甚至每天早上從手機的 Claude iOS app 啟動幾個任務，稍後再檢視結果。

**3/ 全面使用 Opus 4.5 with thinking**

雖然 Opus 比 Sonnet 更大更慢，但 Boris 認為它是他用過最好的 coding model。原因是：需要的引導更少、工具使用更精準，**最終反而比小模型更快**。

**4/ 團隊共享單一 [CLAUDE.md](https://l.facebook.com/l.php?u=http%3A%2F%2FCLAUDE.md%2F%3Ffbclid%3DIwZXh0bgNhZW0CMTAAYnJpZBExUVZtalNQbVNMZGV2SElxbHNydGMGYXBwX2lkEDIyMjAzOTE3ODgyMDA4OTIAAR5P7iEDVN-nm2GyytRBr2na9G_lO6CHUkzfKJlZTwSkc_mSL6kSLbLsCJAN4g_aem__Za4ZKsUaRd4DwRB_O9wxw&h=AT1B6u3jxb5YPpFa2fRM8NhzbH10gX5j6W5sIK9TsQ2RCeEH9f6cRzNwRykwxCnbeAYdhmC0y0oIbplEyxP_ggYUsa8QM3aBFIyWAYp8IxXcfE_qa-9WPd9c3HaFxHfwySJq-jfVkMP63hXsRqU&__tn__=-UK-R&c[0]=AT1MS7IWfjvIf8josJml3dGdmXeql8BTm_sfJbVVWC8j7Gbf7sWdMMQxx2m4vOI6IQgpF8IcRRA13ikB0UYwW06uMtSs6hNzP13VoXc6f5xz8M6MDujNpfnFrXSSRmcexmrQDa_r5pMRqMIyjiTXVKGNFrYrKj8-ELk0dI8CMC5H4h8Fq5ICP74f_PY)**

整個 Claude Code 團隊共用一份 [CLAUDE.md](https://l.facebook.com/l.php?u=http%3A%2F%2FCLAUDE.md%2F%3Ffbclid%3DIwZXh0bgNhZW0CMTAAYnJpZBExUVZtalNQbVNMZGV2SElxbHNydGMGYXBwX2lkEDIyMjAzOTE3ODgyMDA4OTIAAR4Gm6f7Rk3UbQn5hjTLKWua3UnKoCudtZDa434r6xW--0yeBHq0ZCM8xzlIqQ_aem_sl1sEkmAnQ4QBYD1epg0RA&h=AT1B6u3jxb5YPpFa2fRM8NhzbH10gX5j6W5sIK9TsQ2RCeEH9f6cRzNwRykwxCnbeAYdhmC0y0oIbplEyxP_ggYUsa8QM3aBFIyWAYp8IxXcfE_qa-9WPd9c3HaFxHfwySJq-jfVkMP63hXsRqU&__tn__=-UK-R&c[0]=AT1MS7IWfjvIf8josJml3dGdmXeql8BTm_sfJbVVWC8j7Gbf7sWdMMQxx2m4vOI6IQgpF8IcRRA13ikB0UYwW06uMtSs6hNzP13VoXc6f5xz8M6MDujNpfnFrXSSRmcexmrQDa_r5pMRqMIyjiTXVKGNFrYrKj8-ELk0dI8CMC5H4h8Fq5ICP74f_PY)，檢入 git，每週多次更新。規則很簡單：每次看到 Claude 做錯什麼，就加進去，讓 Claude 下次不再犯。其他團隊各自維護自己的版本。

**5/ Code Review 時用 @.claude 更新 [CLAUDE.md](https://l.facebook.com/l.php?u=http%3A%2F%2FCLAUDE.md%2F%3Ffbclid%3DIwZXh0bgNhZW0CMTAAYnJpZBExUVZtalNQbVNMZGV2SElxbHNydGMGYXBwX2lkEDIyMjAzOTE3ODgyMDA4OTIAAR79bkCKafZiGKgS3t-aY6lJ4stgrNViewwBP3_159URAbgbi_p0rZWcOfUaog_aem_-oPdoB-R_UltnwrM3exngQ&h=AT1B6u3jxb5YPpFa2fRM8NhzbH10gX5j6W5sIK9TsQ2RCeEH9f6cRzNwRykwxCnbeAYdhmC0y0oIbplEyxP_ggYUsa8QM3aBFIyWAYp8IxXcfE_qa-9WPd9c3HaFxHfwySJq-jfVkMP63hXsRqU&__tn__=-UK-R&c[0]=AT1MS7IWfjvIf8josJml3dGdmXeql8BTm_sfJbVVWC8j7Gbf7sWdMMQxx2m4vOI6IQgpF8IcRRA13ikB0UYwW06uMtSs6hNzP13VoXc6f5xz8M6MDujNpfnFrXSSRmcexmrQDa_r5pMRqMIyjiTXVKGNFrYrKj8-ELk0dI8CMC5H4h8Fq5ICP74f_PY)**

Boris 會在同事的 PR 上標註 @.claude，讓 Claude 把新的規則加進 [CLAUDE.md](https://l.facebook.com/l.php?u=http%3A%2F%2FCLAUDE.md%2F%3Ffbclid%3DIwZXh0bgNhZW0CMTAAYnJpZBExUVZtalNQbVNMZGV2SElxbHNydGMGYXBwX2lkEDIyMjAzOTE3ODgyMDA4OTIAAR4Gm6f7Rk3UbQn5hjTLKWua3UnKoCudtZDa434r6xW--0yeBHq0ZCM8xzlIqQ_aem_sl1sEkmAnQ4QBYD1epg0RA&h=AT1B6u3jxb5YPpFa2fRM8NhzbH10gX5j6W5sIK9TsQ2RCeEH9f6cRzNwRykwxCnbeAYdhmC0y0oIbplEyxP_ggYUsa8QM3aBFIyWAYp8IxXcfE_qa-9WPd9c3HaFxHfwySJq-jfVkMP63hXsRqU&__tn__=-UK-R&c[0]=AT1MS7IWfjvIf8josJml3dGdmXeql8BTm_sfJbVVWC8j7Gbf7sWdMMQxx2m4vOI6IQgpF8IcRRA13ikB0UYwW06uMtSs6hNzP13VoXc6f5xz8M6MDujNpfnFrXSSRmcexmrQDa_r5pMRqMIyjiTXVKGNFrYrKj8-ELk0dI8CMC5H4h8Fq5ICP74f_PY)。這透過 GitHub Action 實現，是團隊版的「複利式工程」。

**6/ 多數會話從 Plan mode 開始**

按兩次 shift+tab 進入 Plan mode。如果目標是寫一個 PR，Boris 會在 Plan mode 來回討論直到計畫滿意，然後切換到 auto-accept edits mode，讓 Claude 一次完成。**好的計畫是關鍵**。

**7/ 用 slash commands 處理高頻工作流**

每天重複做的事都變成 slash command，存在 `.claude/commands/` 並檢入 git。比如 `/commit-push-pr` 他每天用幾十次，命令內用 inline bash 預先計算 git status 等資訊，減少與模型的來回。

**8/ 使用 subagents 自動化常見流程**

Boris 有幾個常用的 subagent：code-simplifier 在 Claude 完成後簡化程式碼，verify-app 有端到端測試的詳細指令。Subagents 本質上是把大多數 PR 都需要的工作流自動化。

**9/ PostToolUse hook 自動格式化**

Claude 生成的程式碼通常格式良好，hook 處理最後 10%，避免 CI 時出現格式錯誤。

**10/ 不用 –dangerously-skip-permissions**

Boris 用 `/permissions` 預先允許他知道安全的 bash 命令，避免不必要的權限提示。這些設定存在 `.claude/settings.json`，與團隊共享。

**11/ Claude Code 使用所有外部工具**

Claude 幫他搜尋和發送 Slack 訊息（透過 MCP server）、跑 BigQuery 查詢回答分析問題、從 Sentry 抓錯誤日誌。Slack MCP 配置檢入 `.mcp.json`，團隊共享。

**12/ 長時間任務的處理方式**

三種策略：(a) 提示 Claude 完成後用 background agent 驗證，(b) 用 agent Stop hook 更確定性地執行驗證，(c) 用 ralph-wiggum 插件。長任務時他會用 `--permission-mode=dontAsk` 或 `--dangerously-skip-permissions`（在沙盒環境），讓 Claude 不被權限提示卡住。

**13/ 最重要的一條：給 Claude 驗證自己工作的方法**

這能帶來 2-3 倍的品質提升。Claude 用 Chrome 擴充功能測試每一個要上線到 [[claude.ai/code](https://l.facebook.com/l.php?u=http%3A%2F%2Fclaude.ai%2Fcode%3Ffbclid%3DIwZXh0bgNhZW0CMTAAYnJpZBExUVZtalNQbVNMZGV2SElxbHNydGMGYXBwX2lkEDIyMjAzOTE3ODgyMDA4OTIAAR56l8afohCu7Ve238mCMZ4alp1qrNZiFUOm0w-acb6tYUzZAOsT1TFLGX7rgg_aem_sv9YsuXi79U4B01EPjDCkg&h=AT34TLKbGBMP-6p2lgyzIzrXSRcA98ZyCl2JJUgsRH2LYQS8nOLnV0EtztIlczwXM0WmBc2nRfAitMQWW_75Gh0AGVq9NCCP8rKGxUPwPZyNilSBhqpdBGHOfH5sjCaKdMN9COutqq3ttRVBDjg&__tn__=-UK-R&c[0]=AT1MS7IWfjvIf8josJml3dGdmXeql8BTm_sfJbVVWC8j7Gbf7sWdMMQxx2m4vOI6IQgpF8IcRRA13ikB0UYwW06uMtSs6hNzP13VoXc6f5xz8M6MDujNpfnFrXSSRmcexmrQDa_r5pMRqMIyjiTXVKGNFrYrKj8-ELk0dI8CMC5H4h8Fq5ICP74f_PY)]([http://claude.ai/code](https://l.facebook.com/l.php?u=http%3A%2F%2Fclaude.ai%2Fcode%3Ffbclid%3DIwZXh0bgNhZW0CMTAAYnJpZBExUVZtalNQbVNMZGV2SElxbHNydGMGYXBwX2lkEDIyMjAzOTE3ODgyMDA4OTIAAR5zD8yETSMxXBFiQrn68toh2u6KLGjVBm7V2Kb32bPMPPHux3xrETfzHBAmjg_aem_KF8s0nAvlFvbYhU-1xgG4Q&h=AT34TLKbGBMP-6p2lgyzIzrXSRcA98ZyCl2JJUgsRH2LYQS8nOLnV0EtztIlczwXM0WmBc2nRfAitMQWW_75Gh0AGVq9NCCP8rKGxUPwPZyNilSBhqpdBGHOfH5sjCaKdMN9COutqq3ttRVBDjg&__tn__=-UK-R&c[0]=AT1MS7IWfjvIf8josJml3dGdmXeql8BTm_sfJbVVWC8j7Gbf7sWdMMQxx2m4vOI6IQgpF8IcRRA13ikB0UYwW06uMtSs6hNzP13VoXc6f5xz8M6MDujNpfnFrXSSRmcexmrQDa_r5pMRqMIyjiTXVKGNFrYrKj8-ELk0dI8CMC5H4h8Fq5ICP74f_PY)) 的改動——開瀏覽器、測試 UI、反覆迭代直到程式碼能用且體驗良好。驗證方式因領域而異：可能是跑一個 bash 命令、跑測試套件、或在瀏覽器/手機模擬器中測試應用。**這個環節值得投資做到堅實可靠**。

-----

**總結**

Boris 的方法論揭示一個核心理念：**Claude Code 的威力不在於複雜的客製化，而在於建立正確的人機協作結構**。

第一層是「規模化並行」。同時運行 10-15 個 Claude 實例，把它們當成可委派任務的團隊成員，各自運作，有需要時再介入。選擇 Opus 而非更快的 Sonnet，因為大模型需要的人工介入更少，總體反而更快。

第二層是「知識累積的飛輪」。[CLAUDE.md](https://l.facebook.com/l.php?u=http%3A%2F%2FCLAUDE.md%2F%3Ffbclid%3DIwZXh0bgNhZW0CMTAAYnJpZBExUVZtalNQbVNMZGV2SElxbHNydGMGYXBwX2lkEDIyMjAzOTE3ODgyMDA4OTIAAR5zD8yETSMxXBFiQrn68toh2u6KLGjVBm7V2Kb32bPMPPHux3xrETfzHBAmjg_aem_KF8s0nAvlFvbYhU-1xgG4Q&h=AT1B6u3jxb5YPpFa2fRM8NhzbH10gX5j6W5sIK9TsQ2RCeEH9f6cRzNwRykwxCnbeAYdhmC0y0oIbplEyxP_ggYUsa8QM3aBFIyWAYp8IxXcfE_qa-9WPd9c3HaFxHfwySJq-jfVkMP63hXsRqU&__tn__=-UK-R&c[0]=AT1MS7IWfjvIf8josJml3dGdmXeql8BTm_sfJbVVWC8j7Gbf7sWdMMQxx2m4vOI6IQgpF8IcRRA13ikB0UYwW06uMtSs6hNzP13VoXc6f5xz8M6MDujNpfnFrXSSRmcexmrQDa_r5pMRqMIyjiTXVKGNFrYrKj8-ELk0dI8CMC5H4h8Fq5ICP74f_PY) 是團隊共享的「錯誤記憶庫」，配合 GitHub Action 讓 code review 的修正直接轉化為 Claude 的學習。今天的修正成為明天的預設行為。

第三層是「結構化的工作流」。Slash commands 和 subagents 把重複的「人類判斷」編碼成可重用指令，減少每次互動的認知負擔和不確定性。

但最關鍵的是最後一條：**給 Claude 驗證自己工作的方法**。沒有反饋迴路，Claude 就是在盲目生成；有了它，Claude 就能自我迭代直到達標。Plan mode 的工作流完美體現這點——計畫階段的投資，換來執行階段的確定性。

# 2025
# Context Engineering

使用 
.claude/commands/generate-prp.md
.claude/commands/execute-prp.md

會參照 PRPs/templates/prp_base.md 這個檔要依專案情況做調整

# MCP
- docfork: 各種語言的文件
- context7
- sequential-thinking (serena dependency)
- puppeteer
- serena

https://www.reddit.com/r/ClaudeAI/comments/1jf4hnt/setting_up_mcp_servers_in_claude_code_a_tech/


# Prompt

> Analysis the code quality and create a refactor plan, save the plan to memory

# 8 Tricks of Claude Code
https://www.youtube.com/watch?v=cjW6ofe7AY4&list=WL&index=6
I. Put principles:

1. First think through the problem, read the codebase for relevant files, and write a plan to tasks/todo.md. 
2. The plan should have a list of todo items that you can check off as you complete them 
3. Before you begin working, check in with me and I will verify the plan. 
4. Then, begin working on the todo items, marking them as complete as you go. 
5. Please every step of the way just give me a high level explanation of what changes you made 
6. Make every task and code change you do as simple as possible. We want to avoid making any massive or complex changes. Every change should impact as little code as possible. Everything is about simplicity. 
7. Finally, add a review section to the [todo.md]([http://todo.md/](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqa1BlZTBKTGJyVTl5Smc1Nmo3SFBUWURrTnkzQXxBQ3Jtc0tsWVBSNHlBdTNiQnM0N0d1WXhRcjNxLVN5bExCUUtZR21vZ0Rha1ZMUVR0eGdDMldwcnNnUFYyZjZIR1pTaUpGOHBkN01DNnZrQzQyaFlEQzI5ejRBVEhaejh0UnhVTTlEdFFpeVlvN2NtTEpVbFF0dw&q=http%3A%2F%2Ftodo.md%2F&v=cjW6ofe7AY4)) file with a summary of the changes you made and any other relevant information.

II. Use plan mode, and use right model
- plan mode: opus
- action: sonnet

III. Save checkpoints (Commit)

IV. Use image and when to use image, ex:
- UI inspiration
- Fix bugs

V. Use /clear command
and Use /compact command

VI. Security Checks
ex:
after building:
```
Please check through all the code you just wrote and make sure it follows security best practices. Make sure no sensitive information is i the front end and there are no vulnerabilities people can exploit.
```

VII. Learn from Claude
Learning from Claude prompt: 
```
Please explain the functionality and code you just built out in detail. Walk me through wehat you changed and how it works. Act like you’re a senior engineer teaching me code
```

VIII. Be Productive while Claude cooks
Doomscroll hack
```
When I am coding with AI there are long breaks into between me giving me commands to the AI. Typically I spend that time doom scrolling which distracts me and pu†s me in a bad mental state. I'd like to use that time now to chat with you and generate new ideas, and also reflect on my other ideas and businesses and content. I'm not sure how I'd like to use this chat or what role I'd like you to play, but I think it could be much more useful than me doom scrolling. What do you think? What could be the best way for us to use this chat?
```


# Serena
基於 LSP 分析程式碼語意
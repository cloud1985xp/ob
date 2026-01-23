---
tags:
  - ai
  - ml
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# LLM

Created: 2023年11月9日 下午10:18

AIGC > LLM > ChatGPT(Application Level)

OpenAI API

- Completions API
- Chat API
- Embeddings API
    - 很有用的中間產物，給一段文字，回傳一個 1536 維度的向量代表語意
    - 可以用來做語意比對，找出最近的兩個向量/兩段相似的內容

各種限制

LangChain

- Tools
- Base on Python
- Ruby pycall.rb

LLMMathChain

LLMRequestChain

RouterChain

APIChain

LLMBashChain

SQLDatabaseChain

### Prompt Engineering

Principle 1:  Write Clear and Specific Instruction

- Clean ≠ short
- 4 Tactics:
    - Use separator to split different part of inputs
    - Assign output format, ex: json
    - Assign condition
    - Give example (Few-shot vs zero-shot prompt/learning)

Principle 2: Give the model time to think

- Tactic:
    - Specify the steps to complete a task
        - Chain of Thought
        - 太困難的任務，要gpt一句話回應，它就會亂掰
    - Summarizing 告訴他要摘要給誰
    - Inferring 推理
    - Transforming 文本轉換
    - Proofread and correct
    - Expanding 擴寫，短文擴寫成長文，客服/brainstorming

## LLM API Limitation

- Conversational Memory 記住對話
- LangChain Memory 功能
    - ConversationBufferMemory
    - ConversationBufferWindowMemory
    - ConversationSummaryMemory
    - ConversationSummaryBufferMemory
    - Entity Memory
- 

![Untitled](LLM/Untitled.png)

### 摘要

如何去處理超過 token 限制的超長內容，例如一本書

Stuff, Map-reduce, Refine

[如何讓 ChatGPT 摘要大量內容：不同方法的優缺點](https://wylin.tw/pages/how-to-summarize-long-texts/)

只拿開頭摘要

切段之後，每段都看，摘要再摘要 map reduce

- 可平行執行
- 但有可能切段落破壞上下文

Refine，給前情提要，

- 一樣拆段落，第一段摘要後送給第二段一起摘要，循序漸進不斷去改善
- 但很慢，要循序處理

分群摘要

- 先把相似的內容分在一起再做摘要

## RAG: Retrieval-Augmented Generation

![Untitled](LLM/Untitled%201.png)

先搜尋再下 prompt → Q/A

對很長的文本(一本書) 做 QA

先搜尋相關的內容，再把相關的內容塞到 prompt 去問

像 Bing

- 先用自己的搜尋引擎找到你問題相關的內容
- 再把搜到的內容先做摘要當作背景知識
- 然後跟著你的問題塞一起到 prompt 裡去問 GPT

LangChain

- 讀入文本 Integration > Document 支援各種格式讀取
- 做 parse 拆段落 chunk
- 丟到 OpenAI Embedding API 算向量
- 存在向量 db (LangChain Integration 各種 DB store)
- 再將 query 丟進來算向
- 從 db 裡找類似的 doc
- 再用 doc 去 query answer

Embedding API 也可以用別家，不一定 OpenAI

huggingface 上有各家的 可以查 leaderboard 

Retriever 的方式不只有 vector，也可以用別方式 ex: Elasticsearch

LangChain > Retriver

## Agent

做代理人

告訴它有什麼工具，讓LLM自己選擇要用哪些

兩種策略

ReAct

![Untitled](LLM/Untitled%202.png)

Plan-and-Execute 又叫 Autonomous agents

- AutoGPT
- BabyGPT
- JARVIS

![Untitled](LLM/Untitled%203.png)

# Todo

來做個 AI 排旅遊行程

強化 UX 直接輸入文字來產生行
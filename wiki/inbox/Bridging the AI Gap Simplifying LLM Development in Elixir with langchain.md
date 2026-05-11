---
title: "Bridging the AI Gap: Simplifying LLM Development in Elixir with langchain"
source: "https://dev.to/darnahsan/bridging-the-ai-gap-simplifying-llm-development-in-elixir-with-langchainex-3o42"
author:
  - "[[Ahsan Nabi Dar]]"
published: 2025-05-02
created: 2026-05-05
description: "Artificial Intelligence is evolving at a breakneck pace. Every week, it feels like there’s a new... Tagged with elixir, ai, langchain, gemini."
tags:
  - "clippings"
---
Artificial Intelligence is evolving at a breakneck pace. Every week, it feels like there’s a new breakthrough, a novel architecture, or yet another company launching its own large language model (LLM). With so much innovation happening simultaneously, keeping up with the ever-changing landscape can feel overwhelming — even for seasoned developers.

One of the biggest challenges developers face today is inconsistency across platforms. While OpenAI pioneered the API format that many now follow, major inference service providers such as Anthropic, Hugging Face, and Google Cloud have each developed their own unique interfaces. This fragmentation makes it difficult to switch between models or providers without rewriting significant chunks of code.

This is where LangChain comes into play. Originally built for Python, LangChain quickly became the go-to framework for working with LLMs. It introduced a standardized way to interact with different models and APIs, abstracting away the complexity and letting developers focus on building powerful applications.

But what if you're an Elixir developer? That’s where the Elixir community steps in with langchain — an Elixir port of the LangChain framework.

Created by members of the Elixir ecosystem, langchain brings the same power and flexibility of LangChain to Elixir developers. It provides a consistent abstraction layer over various inference providers, allowing you to seamlessly integrate with OpenAI, Anthropic, Hugging Face, and more — all without worrying about the nuances of each provider’s API.

Recently, I was working with Google Gemini models via their Vertex AI API. Initially, I had hand-rolled the HTTP requests using Google’s API format. But after discovering langchain, I decided to refactor my implementation to use the framework instead.

However, I hit a roadblock — file URL support wasn’t implemented yet for Vertex AI.

Now here’s where things get interesting:

While Google AI Studio only allows file URLs uploaded through its own file service, Vertex AI supports processing media from any publicly crawlable third-party URL — a very useful feature for real-world applications where files might live outside of Google’s ecosystem.

At the time, the Elixir community had just added file URL support as recent as last month for Google AI Studio, likely because it’s the more commonly used platform among casual users. But Vertex AI, popular among enterprise developers and production setups, didn't yet have this capability in langchain.

So I decided to step in.

Building on top of the great work already done by the community, I submitted a [PR](https://github.com/brainlid/langchain/pull/296) to add full file URL support for Gemini models via Vertex AI in langchain — and good news: it got merged!

This enhancement now enables developers to:

Pass public URLs directly to Gemini models via Vertex AI  
Process images, PDFs, and other media formats hosted externally  
Leverage the full capabilities of Gemini within the Elixir ecosystem

Without langchain the implementation to Vertex AI looks as such where you need to maintain the request format and which will change for each provider.

```
defp inference(prompt, mime_type, content_url) do
    json = %{
      "contents" => %{
        "role" => "user",
        "parts" => [
          %{
            "fileData" => %{
              "mimeType" => mime_type,
              "fileUri" => content_url
            }
          },
          %{
            "text" => prompt
          }
        ]
      },
      "generationConfig" => %{
        "temperature" => 1.0,
        "topP" => 0.8,
        "topK" => 10
      }
    }

    Req.post!(vertex_endpoint(),
      auth: {:bearer, auth_token()},
      json: json,
      receive_timeout: @timeout
    ).body
    |> case do
      %{"candidates" => [%{"content" => %{"parts" => [%{"text" => response}]}}]} -> {:ok, response}

      %{"error" => %{
          "code" => code,
           "details" => _details,
          "message" => message,
          "status" => status
        }
      } ->
        msg = "GCloud Vertex API error {status} ({code}), {message}"
        Logger.info(msg)
        {:error, msg}
    end
  end
```

with Elixir langchain

```
defp inference(prompt, mime_type, content_url) do
    config = %{
      model: model_id(),
      endpoint: vertex_endpoint(),
      api_key: auth_token(),
      temperature: 1.0,
      top_p: 0.8,
      top_k: 10,
      receive_timeout: @timeout
    }

    content = %{
      mime_type: mime_type,
      url: content_url
    }

    vertex_ai(config, prompt, content)
    |> case do
      {:ok, response} ->
        {:ok, response}

      {:error, changeset} ->
        {:error, changeset}
    end
  end

def vertex_ai(config, prompt, content) do
    model = ChatVertexAI.new!(config)

    %{llm: model, verbose: false, stream: false}
    |> LLMChain.new!()
    |> LLMChain.add_message(
      Message.new_user!([
        ContentPart.new!(%{type: :text, content: prompt}),
        ContentPart.new!(%{
          type: :file_url,
          content: content.url,
          options: [media: content.mime_type]
        })
      ])
    )
    |> LLMChain.run()
    |> parse_content()
  end

  defp parse_content(
         {:ok,
          %LangChain.Chains.LLMChain{
            llm: _llm,
            last_message: %LangChain.Message{
              content: [
                %LangChain.Message.ContentPart{type: :text, content: content}
              ]
            }
          }}
       ) do

    {:ok, content}
  end

  defp parse_content({:error, changeset}) do
    Logger.info(changeset |> inspect())
    {:error, "Langchain encountered error !!!!"}
  end
```

As AI continues to evolve, tools like LangChain and its Elixir counterpart, langchain, are essential for developers looking to harness the power of LLMs without getting bogged down by platform-specific quirks.

Whether you're building chatbots, data analyzers, or intelligent agents, langchain empowers you to stay agile, innovative, and ahead of the curve — all while staying true to the elegance and performance of Elixir.

So if you’re an Elixir developer diving into the world of AI, don’t reinvent the wheel. Let langchain do the heavy lifting, while you focus on creating something amazing with the power of Elixir.[Sonar](https://dev.to/sonar)Promoted

[![Vibe check: Do developers trust AI?](https://media2.dev.to/dynamic/image/width=775%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fucarecdn.com%2F2f950e18-c9b4-450d-871d-d249bc86b321%2F)](https://www.sonarsource.com/sem/the-state-of-code/developer-survey-report/?utm_medium=paid&utm_source=dev&utm_campaign=ss-state-of-code-developer-survey26&utm_content=report-devsurvey-banner-x-1&utm_term=ww-all-x&s_category=Paid&s_source=Paid+Social&s_origin=dev&bb=259985)

## Vibe check: Do developers trust AI?

Based on a survey of over 1,100 developers, Sonar's newest State of Code report analyzes the impact of generative AI on software engineering workflows and how developers are adapting to address it.
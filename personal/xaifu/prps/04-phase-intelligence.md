# Phase 4: 智慧化與進階功能

## 預估工時：3-4 週（單人開發）

---

## 1. 階段目標

提升角色的智能程度與互動深度，讓 AI 角色更像「真實的數位生命」：
- 向量資料庫記憶系統（長期記憶）
- 動態日程決策（AI 自主決定活動）
- 情緒與狀態系統
- 角色間互動（留言、按讚）
- 進階對話管理

**里程碑**：角色能記住過去互動、根據心情自主決定行為、並與其他角色互動

---

## 2. 架構設計

### 2.1 記憶系統架構

```
┌──────────────────────────────────────────────────────────────────┐
│                     Memory System Architecture                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    Memory Manager                           │ │
│  │                                                             │ │
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │ │
│  │  │  Short-term │    │  Long-term  │    │  Episodic   │    │ │
│  │  │   Memory    │    │   Memory    │    │   Memory    │    │ │
│  │  │             │    │             │    │             │    │ │
│  │  │ - 最近對話  │    │ - 重要事實  │    │ - 特定事件  │    │ │
│  │  │ - 當前狀態  │    │ - 用戶偏好  │    │ - 情感回憶  │    │ │
│  │  │ - 即時情緒  │    │ - 關係定義  │    │ - 里程碑    │    │ │
│  │  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘    │ │
│  │         │                  │                  │            │ │
│  │         └──────────────────┼──────────────────┘            │ │
│  │                            ▼                               │ │
│  │              ┌─────────────────────────┐                  │ │
│  │              │    Retrieval Engine     │                  │ │
│  │              │                         │                  │ │
│  │              │  1. 語意搜尋 (pgvector) │                  │ │
│  │              │  2. 關鍵字匹配          │                  │ │
│  │              │  3. 時間相關性          │                  │ │
│  │              │  4. 重要性排序          │                  │ │
│  │              └───────────┬─────────────┘                  │ │
│  │                          │                                 │ │
│  │                          ▼                                 │ │
│  │              ┌─────────────────────────┐                  │ │
│  │              │    Context Builder      │                  │ │
│  │              │                         │                  │ │
│  │              │  組合相關記憶成為        │                  │ │
│  │              │  LLM 可用的上下文        │                  │ │
│  │              └─────────────────────────┘                  │ │
│  │                                                            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    pgvector (PostgreSQL)                    │ │
│  │                                                             │ │
│  │  memories table:                                            │ │
│  │  - id, character_id, user_id                                │ │
│  │  - content (text)                                          │ │
│  │  - embedding (vector(1536))  <- OpenAI embeddings          │ │
│  │  - memory_type (short/long/episodic)                       │ │
│  │  - importance (float)                                       │ │
│  │  - created_at, accessed_at                                  │ │
│  │                                                             │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### 2.2 情緒系統架構

```
┌──────────────────────────────────────────────────────────────────┐
│                      Emotion System                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  核心情緒維度:                                                    │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Valence (情感正負)  ──────────────────────  -100 to +100  │ │
│  │  ├ -100: 非常負面 (sad, angry)                              │ │
│  │  ├    0: 中性                                               │ │
│  │  └ +100: 非常正面 (happy, excited)                          │ │
│  │                                                             │ │
│  │  Arousal (激活程度)  ──────────────────────  0 to 100      │ │
│  │  ├    0: 低激活 (calm, tired)                              │ │
│  │  └  100: 高激活 (excited, anxious)                         │ │
│  │                                                             │ │
│  │  Energy (能量值)     ──────────────────────  0 to 100      │ │
│  │  ├    0: 精疲力竭                                          │ │
│  │  └  100: 精力充沛                                          │ │
│  │                                                             │ │
│  │  Social (社交需求)   ──────────────────────  0 to 100      │ │
│  │  ├    0: 需要獨處                                          │ │
│  │  └  100: 渴望社交                                          │ │
│  │                                                             │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  情緒影響因素:                                                    │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  時間因素:                                                   │ │
│  │  - 早晨: energy ↑, arousal 逐漸 ↑                          │ │
│  │  - 深夜: energy ↓, valence 略 ↓                            │ │
│  │                                                             │ │
│  │  活動影響:                                                   │ │
│  │  - 運動: energy ↓ 然後 ↑, valence ↑                        │ │
│  │  - 社交: social ↓, arousal ↑                               │ │
│  │  - 休息: energy ↑, arousal ↓                               │ │
│  │                                                             │ │
│  │  互動影響:                                                   │ │
│  │  - 收到讚/留言: valence ↑, social ↓                        │ │
│  │  - 愉快對話: valence ↑                                     │ │
│  │  - 長時間無互動: social ↑                                  │ │
│  │                                                             │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### 2.3 角色互動系統

```
┌──────────────────────────────────────────────────────────────────┐
│                   Character Interaction System                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  互動類型:                                                        │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  1. 貼文按讚                                                │ │
│  │     角色 A 看到角色 B 的貼文 → 根據關係/內容決定是否按讚    │ │
│  │                                                             │ │
│  │  2. 貼文留言                                                │ │
│  │     角色 A 對角色 B 的貼文發表評論                          │ │
│  │     需考慮: 角色個性、與 B 的關係、貼文內容                 │ │
│  │                                                             │ │
│  │  3. 私訊（未來）                                            │ │
│  │     角色間私下對話                                          │ │
│  │                                                             │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  關係系統:                                                        │ │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Relationship Schema:                                        │ │
│  │  - character_a_id                                           │ │
│  │  - character_b_id                                           │ │
│  │  - affinity: -100 to +100 (好感度)                         │ │
│  │  - familiarity: 0 to 100 (熟悉度)                          │ │
│  │  - interaction_count: integer                               │ │
│  │  - last_interaction_at: datetime                            │ │
│  │                                                             │ │
│  │  互動機率計算:                                               │ │
│  │  P(interact) = base_rate * affinity_modifier * mood_factor  │ │
│  │                                                             │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. 任務分解

### Week 1: 記憶系統

| 任務 | 預估時間 | 優先級 |
|------|----------|--------|
| 設定 pgvector 擴展 | 2h | P0 |
| 設計 Memory Schema | 4h | P0 |
| 實作 Embedding 模組 | 6h | P0 |
| 實作 Memory Context | 8h | P0 |
| 記憶檢索引擎 | 8h | P0 |
| 整合到聊天系統 | 6h | P0 |
| 單元測試 | 6h | P0 |

### Week 2: 情緒系統

| 任務 | 預估時間 | 優先級 |
|------|----------|--------|
| 設計情緒模型 | 4h | P0 |
| 實作 EmotionEngine | 8h | P0 |
| 時間衰減機制 | 4h | P0 |
| 活動影響計算 | 6h | P0 |
| 整合到 Agent | 6h | P0 |
| 動態日程決策 | 8h | P0 |
| 測試 | 4h | P0 |

### Week 3: 角色互動

| 任務 | 預估時間 | 優先級 |
|------|----------|--------|
| 設計 Relationship Schema | 4h | P0 |
| Comment Schema 與功能 | 6h | P0 |
| Like Schema 與功能 | 4h | P0 |
| 互動決策邏輯 | 8h | P0 |
| InteractionWorker | 6h | P0 |
| 前端留言顯示 | 6h | P0 |
| 測試 | 6h | P0 |

### Week 4: 整合與優化

| 任務 | 預估時間 | 優先級 |
|------|----------|--------|
| 系統整合測試 | 8h | P0 |
| 效能優化 | 6h | P1 |
| 記憶清理機制 | 4h | P1 |
| 管理介面增強 | 6h | P1 |
| 文件完善 | 4h | P1 |
| 端到端測試 | 8h | P0 |

---

## 4. 記憶系統實作

### 4.1 pgvector 設定

```elixir
# priv/repo/migrations/20260224000001_enable_pgvector.exs
defmodule Xaifu.Repo.Migrations.EnablePgvector do
  use Ecto.Migration

  def up do
    execute "CREATE EXTENSION IF NOT EXISTS vector"
  end

  def down do
    execute "DROP EXTENSION IF EXISTS vector"
  end
end
```

### 4.2 Memory Schema

```elixir
# lib/xaifu/ai/memory/memory.ex
defmodule Xaifu.AI.Memory.Memory do
  use Ecto.Schema
  import Ecto.Changeset

  @primary_key {:id, :binary_id, autogenerate: true}
  @foreign_key_type :binary_id

  schema "memories" do
    belongs_to :character, Xaifu.Characters.Character
    field :user_identifier, :string

    field :content, :string
    field :embedding, Pgvector.Ecto.Vector  # 1536 維度 (OpenAI)
    field :memory_type, Ecto.Enum,
      values: [:short_term, :long_term, :episodic],
      default: :short_term

    field :importance, :float, default: 0.5  # 0-1
    field :metadata, :map, default: %{}

    # 用於衰減計算
    field :accessed_at, :utc_datetime
    field :access_count, :integer, default: 0

    timestamps(type: :utc_datetime)
  end

  def changeset(memory, attrs) do
    memory
    |> cast(attrs, [
      :character_id, :user_identifier, :content, :embedding,
      :memory_type, :importance, :metadata, :accessed_at, :access_count
    ])
    |> validate_required([:character_id, :content])
    |> validate_number(:importance,
        greater_than_or_equal_to: 0,
        less_than_or_equal_to: 1)
  end
end
```

### 4.3 資料庫遷移

```elixir
# priv/repo/migrations/20260224000002_create_memories.exs
defmodule Xaifu.Repo.Migrations.CreateMemories do
  use Ecto.Migration

  def change do
    create table(:memories, primary_key: false) do
      add :id, :binary_id, primary_key: true
      add :character_id, references(:characters, type: :binary_id, on_delete: :delete_all),
        null: false
      add :user_identifier, :string

      add :content, :text, null: false
      add :embedding, :vector, size: 1536  # OpenAI embedding size
      add :memory_type, :string, default: "short_term"

      add :importance, :float, default: 0.5
      add :metadata, :map, default: %{}

      add :accessed_at, :utc_datetime
      add :access_count, :integer, default: 0

      timestamps(type: :utc_datetime)
    end

    create index(:memories, [:character_id])
    create index(:memories, [:character_id, :user_identifier])
    create index(:memories, [:memory_type])
    create index(:memories, [:importance])

    # 向量相似度搜尋索引
    execute """
    CREATE INDEX memories_embedding_idx ON memories
    USING ivfflat (embedding vector_cosine_ops)
    WITH (lists = 100)
    """
  end
end
```

### 4.4 Embedding 模組

```elixir
# lib/xaifu/ai/memory/embedding.ex
defmodule Xaifu.AI.Memory.Embedding do
  @moduledoc """
  文字向量化模組
  使用 OpenAI Embeddings API
  """

  require Logger

  @embedding_model "text-embedding-3-small"
  @embedding_dimensions 1536

  @doc """
  將文字轉換為向量
  """
  def create(text) when is_binary(text) do
    body = %{
      model: @embedding_model,
      input: text
    }

    case Req.post("https://api.openai.com/v1/embeddings",
           json: body,
           headers: auth_headers(),
           receive_timeout: 30_000) do
      {:ok, %{status: 200, body: %{"data" => [%{"embedding" => embedding} | _]}}} ->
        {:ok, embedding}

      {:ok, %{status: status, body: body}} ->
        Logger.error("Embedding API error: #{status} - #{inspect(body)}")
        {:error, {:api_error, status}}

      {:error, reason} ->
        {:error, reason}
    end
  end

  @doc """
  批次向量化
  """
  def create_batch(texts) when is_list(texts) do
    body = %{
      model: @embedding_model,
      input: texts
    }

    case Req.post("https://api.openai.com/v1/embeddings",
           json: body,
           headers: auth_headers(),
           receive_timeout: 60_000) do
      {:ok, %{status: 200, body: %{"data" => data}}} ->
        embeddings = Enum.sort_by(data, & &1["index"])
        |> Enum.map(& &1["embedding"])
        {:ok, embeddings}

      {:ok, %{status: status, body: body}} ->
        {:error, {:api_error, status, body}}

      {:error, reason} ->
        {:error, reason}
    end
  end

  @doc """
  計算兩個向量的餘弦相似度
  """
  def cosine_similarity(vec1, vec2) when length(vec1) == length(vec2) do
    dot_product = Enum.zip(vec1, vec2)
    |> Enum.map(fn {a, b} -> a * b end)
    |> Enum.sum()

    magnitude1 = :math.sqrt(Enum.map(vec1, &(&1 * &1)) |> Enum.sum())
    magnitude2 = :math.sqrt(Enum.map(vec2, &(&1 * &1)) |> Enum.sum())

    if magnitude1 * magnitude2 == 0 do
      0.0
    else
      dot_product / (magnitude1 * magnitude2)
    end
  end

  defp auth_headers do
    api_key = Application.get_env(:xaifu, Xaifu.AI)[:llm_api_key]
    [
      {"Authorization", "Bearer #{api_key}"},
      {"Content-Type", "application/json"}
    ]
  end
end
```

### 4.5 Memory Context

```elixir
# lib/xaifu/ai/memory.ex
defmodule Xaifu.AI.Memory do
  @moduledoc """
  記憶系統 Context
  管理角色的長期記憶
  """

  import Ecto.Query, warn: false
  alias Xaifu.Repo
  alias Xaifu.AI.Memory.{Memory, Embedding}

  @doc """
  儲存新記憶
  """
  def store(character_id, content, opts \\ []) do
    user_identifier = opts[:user_identifier]
    memory_type = opts[:memory_type] || :short_term
    importance = opts[:importance] || calculate_importance(content)

    with {:ok, embedding} <- Embedding.create(content) do
      %Memory{}
      |> Memory.changeset(%{
        character_id: character_id,
        user_identifier: user_identifier,
        content: content,
        embedding: embedding,
        memory_type: memory_type,
        importance: importance,
        accessed_at: DateTime.utc_now()
      })
      |> Repo.insert()
    end
  end

  @doc """
  檢索相關記憶
  """
  def retrieve(character_id, query, opts \\ []) do
    user_identifier = opts[:user_identifier]
    limit = opts[:limit] || 5
    min_similarity = opts[:min_similarity] || 0.7

    with {:ok, query_embedding} <- Embedding.create(query) do
      # 使用 pgvector 的餘弦距離搜尋
      memories = Memory
      |> where([m], m.character_id == ^character_id)
      |> maybe_filter_user(user_identifier)
      |> select([m], %{
        memory: m,
        similarity: fragment(
          "1 - (? <=> ?)",
          m.embedding,
          ^Pgvector.new(query_embedding)
        )
      })
      |> order_by([m], fragment(
        "? <=> ?",
        m.embedding,
        ^Pgvector.new(query_embedding)
      ))
      |> limit(^(limit * 2))  # 取多一些用於後續過濾
      |> Repo.all()

      # 過濾低相似度並更新存取記錄
      relevant_memories = memories
      |> Enum.filter(fn %{similarity: sim} -> sim >= min_similarity end)
      |> Enum.take(limit)
      |> Enum.map(fn %{memory: m} ->
        update_access(m)
        m
      end)

      {:ok, relevant_memories}
    end
  end

  @doc """
  根據查詢建構上下文
  """
  def build_context(character_id, query, opts \\ []) do
    case retrieve(character_id, query, opts) do
      {:ok, memories} when memories != [] ->
        context = memories
        |> Enum.map(&format_memory/1)
        |> Enum.join("\n\n")

        {:ok, context}

      {:ok, []} ->
        {:ok, nil}

      {:error, reason} ->
        {:error, reason}
    end
  end

  @doc """
  從對話中提取重要記憶
  """
  def extract_from_conversation(character_id, user_identifier, messages) do
    # 使用 LLM 提取重要資訊
    prompt = """
    分析以下對話，提取值得記住的重要資訊。
    請以條列式列出（每條一行），只列出重要的事實、偏好或事件。
    如果沒有重要資訊，回覆「無」。

    對話內容：
    #{format_messages(messages)}
    """

    case Xaifu.AI.LLM.chat_completion([
      %{role: "system", content: "你是一個資訊提取助手，請精確地提取重要資訊。"},
      %{role: "user", content: prompt}
    ], temperature: 0.3) do
      {:ok, response} ->
        if response != "無" do
          # 解析並儲存每條記憶
          response
          |> String.split("\n")
          |> Enum.map(&String.trim/1)
          |> Enum.reject(&(&1 == "" or &1 == "無"))
          |> Enum.each(fn content ->
            store(character_id, content,
              user_identifier: user_identifier,
              memory_type: :long_term
            )
          end)
        end

        :ok

      {:error, reason} ->
        {:error, reason}
    end
  end

  @doc """
  升級短期記憶為長期記憶
  """
  def promote_memory(memory_id) do
    Memory
    |> Repo.get!(memory_id)
    |> Memory.changeset(%{memory_type: :long_term})
    |> Repo.update()
  end

  @doc """
  清理過期的短期記憶
  """
  def cleanup_old_memories(days_threshold \\ 7) do
    cutoff = DateTime.utc_now() |> DateTime.add(-days_threshold, :day)

    Memory
    |> where([m], m.memory_type == :short_term)
    |> where([m], m.accessed_at < ^cutoff)
    |> where([m], m.importance < 0.7)  # 保留重要的
    |> Repo.delete_all()
  end

  # ============================================
  # Private Functions
  # ============================================

  defp maybe_filter_user(query, nil), do: query
  defp maybe_filter_user(query, user_identifier) do
    where(query, [m], is_nil(m.user_identifier) or m.user_identifier == ^user_identifier)
  end

  defp update_access(memory) do
    Memory
    |> where([m], m.id == ^memory.id)
    |> Repo.update_all(
      set: [accessed_at: DateTime.utc_now()],
      inc: [access_count: 1]
    )
  end

  defp format_memory(memory) do
    time_ago = time_ago_text(memory.inserted_at)
    "[#{time_ago}] #{memory.content}"
  end

  defp time_ago_text(datetime) do
    diff = DateTime.diff(DateTime.utc_now(), datetime, :day)

    cond do
      diff == 0 -> "今天"
      diff == 1 -> "昨天"
      diff < 7 -> "#{diff} 天前"
      diff < 30 -> "#{div(diff, 7)} 週前"
      true -> "#{div(diff, 30)} 個月前"
    end
  end

  defp format_messages(messages) do
    messages
    |> Enum.map(fn msg ->
      role = if msg.role in [:user, "user"], do: "用戶", else: "角色"
      "#{role}: #{msg.content}"
    end)
    |> Enum.join("\n")
  end

  defp calculate_importance(content) do
    # 簡單的重要性計算（可以用 LLM 改進）
    cond do
      String.contains?(content, ["喜歡", "討厭", "最愛", "重要"]) -> 0.8
      String.contains?(content, ["名字", "生日", "工作", "住"]) -> 0.7
      String.length(content) > 100 -> 0.6
      true -> 0.5
    end
  end
end
```

---

## 5. 情緒系統實作

### 5.1 Emotion Engine

```elixir
# lib/xaifu/agents/emotion_engine.ex
defmodule Xaifu.Agents.EmotionEngine do
  @moduledoc """
  情緒引擎
  計算和管理角色的情緒狀態
  """

  defstruct [
    :valence,    # 正負情感 -100 to +100
    :arousal,    # 激活程度 0 to 100
    :energy,     # 能量值 0 to 100
    :social      # 社交需求 0 to 100
  ]

  @type t :: %__MODULE__{
    valence: integer(),
    arousal: integer(),
    energy: integer(),
    social: integer()
  }

  @doc """
  建立初始情緒狀態
  """
  def new(opts \\ []) do
    %__MODULE__{
      valence: opts[:valence] || Enum.random(20..60),
      arousal: opts[:arousal] || Enum.random(30..60),
      energy: opts[:energy] || Enum.random(60..90),
      social: opts[:social] || Enum.random(40..70)
    }
  end

  @doc """
  應用時間衰減
  """
  def apply_time_decay(emotion, hours_passed) do
    # 能量自然衰減
    energy_decay = hours_passed * 2

    # 情緒趨向中性
    valence_decay = if emotion.valence > 0, do: -hours_passed, else: hours_passed

    # 激活程度下降
    arousal_decay = hours_passed * 1.5

    # 社交需求上升
    social_increase = hours_passed * 3

    %{emotion |
      energy: clamp(emotion.energy - energy_decay, 0, 100),
      valence: clamp(emotion.valence + valence_decay, -100, 100),
      arousal: clamp(emotion.arousal - arousal_decay, 0, 100),
      social: clamp(emotion.social + social_increase, 0, 100)
    }
  end

  @doc """
  應用活動影響
  """
  def apply_activity(emotion, activity_type) do
    effects = activity_effects()[activity_type] || %{}

    %{emotion |
      valence: clamp(emotion.valence + (effects[:valence] || 0), -100, 100),
      arousal: clamp(emotion.arousal + (effects[:arousal] || 0), 0, 100),
      energy: clamp(emotion.energy + (effects[:energy] || 0), 0, 100),
      social: clamp(emotion.social + (effects[:social] || 0), 0, 100)
    }
  end

  @doc """
  應用互動影響
  """
  def apply_interaction(emotion, interaction_type, sentiment \\ :neutral) do
    base_effect = interaction_effects()[interaction_type] || %{}

    # 根據情感調整影響
    sentiment_modifier = case sentiment do
      :positive -> 1.5
      :negative -> -0.5
      :neutral -> 1.0
    end

    valence_change = (base_effect[:valence] || 0) * sentiment_modifier

    %{emotion |
      valence: clamp(emotion.valence + valence_change, -100, 100),
      arousal: clamp(emotion.arousal + (base_effect[:arousal] || 0), 0, 100),
      social: clamp(emotion.social + (base_effect[:social] || 0), 0, 100)
    }
  end

  @doc """
  取得當前心情標籤
  """
  def current_mood(emotion) do
    cond do
      emotion.valence > 50 and emotion.arousal > 50 -> :excited
      emotion.valence > 50 and emotion.arousal <= 50 -> :happy
      emotion.valence > 20 and emotion.arousal <= 30 -> :calm
      emotion.valence < -50 and emotion.arousal > 50 -> :angry
      emotion.valence < -50 and emotion.arousal <= 50 -> :sad
      emotion.valence < -20 and emotion.arousal <= 30 -> :tired
      emotion.energy < 30 -> :exhausted
      emotion.social > 80 -> :lonely
      true -> :neutral
    end
  end

  @doc """
  取得情緒描述（用於 prompt）
  """
  def describe(emotion) do
    mood = current_mood(emotion)

    mood_descriptions = %{
      excited: "非常興奮和開心",
      happy: "心情愉快",
      calm: "平靜放鬆",
      angry: "有點煩躁",
      sad: "心情低落",
      tired: "感到疲憊",
      exhausted: "精疲力竭",
      lonely: "有點想找人聊天",
      neutral: "心情平常"
    }

    energy_desc = cond do
      emotion.energy > 80 -> "精力充沛"
      emotion.energy > 50 -> "精神還不錯"
      emotion.energy > 30 -> "有點累"
      true -> "很疲憊"
    end

    "#{mood_descriptions[mood]}，#{energy_desc}"
  end

  @doc """
  決定是否想要進行某活動
  """
  def wants_activity?(emotion, activity_type) do
    base_desire = activity_desire_base()[activity_type] || 0.5

    # 根據當前狀態調整
    modifier = case activity_type do
      "rest" -> if emotion.energy < 40, do: 0.4, else: -0.2
      "exercise" -> if emotion.energy > 60, do: 0.2, else: -0.3
      "social" -> (emotion.social - 50) / 100
      "work" -> if emotion.arousal > 40, do: 0.1, else: -0.1
      _ -> 0
    end

    probability = clamp(base_desire + modifier, 0, 1)
    :rand.uniform() < probability
  end

  # ============================================
  # Private Functions
  # ============================================

  defp activity_effects do
    %{
      "cafe" => %{valence: 10, arousal: 5, energy: -5, social: -5},
      "work" => %{valence: -5, arousal: 10, energy: -15, social: 5},
      "exercise" => %{valence: 15, arousal: 20, energy: -20, social: -5},
      "shopping" => %{valence: 10, arousal: 10, energy: -10, social: -10},
      "social" => %{valence: 15, arousal: 15, energy: -10, social: -30},
      "hobby" => %{valence: 20, arousal: 10, energy: -5, social: 0},
      "travel" => %{valence: 25, arousal: 20, energy: -15, social: -10},
      "rest" => %{valence: 5, arousal: -20, energy: 30, social: 10}
    }
  end

  defp interaction_effects do
    %{
      :received_like => %{valence: 5, arousal: 5, social: -5},
      :received_comment => %{valence: 10, arousal: 10, social: -10},
      :chat_message => %{valence: 5, arousal: 5, social: -15},
      :positive_chat => %{valence: 15, arousal: 5, social: -10},
      :negative_chat => %{valence: -10, arousal: 10, social: 5}
    }
  end

  defp activity_desire_base do
    %{
      "cafe" => 0.6,
      "work" => 0.4,
      "exercise" => 0.4,
      "shopping" => 0.5,
      "social" => 0.5,
      "hobby" => 0.6,
      "travel" => 0.3,
      "rest" => 0.5
    }
  end

  defp clamp(value, min, max) do
    value
    |> max(min)
    |> min(max)
    |> round()
  end
end
```

### 5.2 動態日程決策

```elixir
# lib/xaifu/agents/dynamic_scheduler.ex
defmodule Xaifu.Agents.DynamicScheduler do
  @moduledoc """
  動態日程決策
  根據情緒狀態決定下一個活動
  """

  alias Xaifu.Agents.EmotionEngine
  alias Xaifu.Characters

  @activity_types ["cafe", "work", "exercise", "shopping", "social", "hobby", "rest"]

  @doc """
  決定下一個活動
  """
  def decide_next_activity(character, emotion, opts \\ []) do
    # 取得角色的預設排程（如果有）
    scheduled = opts[:scheduled_activity]

    # 計算各活動的欲望值
    desires = calculate_desires(character, emotion)

    # 決定是否按照排程
    if scheduled && should_follow_schedule?(emotion, scheduled) do
      scheduled
    else
      # 根據欲望值隨機選擇
      select_by_desire(desires)
    end
  end

  @doc """
  使用 LLM 做更智能的決策
  """
  def decide_with_llm(character, emotion, context \\ %{}) do
    mood_desc = EmotionEngine.describe(emotion)
    time_desc = get_time_description()

    prompt = """
    你是 #{character.name}，性格是：#{character.personality}

    現在的狀態：
    - 心情：#{mood_desc}
    - 時間：#{time_desc}
    #{if context[:weather], do: "- 天氣：#{context[:weather]}", else: ""}
    #{if context[:last_activity], do: "- 剛剛做了：#{context[:last_activity]}", else: ""}

    請決定現在想做什麼活動。只回覆活動名稱，不要解釋：
    可選：咖啡店、工作、運動、逛街、社交、興趣活動、休息
    """

    case Xaifu.AI.LLM.chat_completion([
      %{role: "user", content: prompt}
    ], temperature: 0.9) do
      {:ok, response} ->
        parse_activity(response)

      {:error, _} ->
        # 降級到基於規則的決策
        decide_next_activity(character, emotion)
    end
  end

  # ============================================
  # Private Functions
  # ============================================

  defp calculate_desires(character, emotion) do
    interests = character.interests || []

    @activity_types
    |> Enum.map(fn activity ->
      base_desire = base_desire(activity)
      emotion_modifier = emotion_modifier(activity, emotion)
      interest_bonus = interest_bonus(activity, interests)

      {activity, base_desire + emotion_modifier + interest_bonus}
    end)
    |> Map.new()
  end

  defp base_desire("rest"), do: 0.3
  defp base_desire("cafe"), do: 0.5
  defp base_desire("hobby"), do: 0.5
  defp base_desire(_), do: 0.4

  defp emotion_modifier(activity, emotion) do
    case activity do
      "rest" when emotion.energy < 40 -> 0.4
      "exercise" when emotion.energy > 70 -> 0.2
      "social" when emotion.social > 70 -> 0.3
      "hobby" when emotion.valence < 0 -> 0.2  # 心情不好做喜歡的事
      _ -> 0
    end
  end

  defp interest_bonus(activity, interests) do
    mappings = %{
      "cafe" => ["咖啡", "閱讀", "工作"],
      "exercise" => ["運動", "健身", "跑步", "瑜珈"],
      "shopping" => ["購物", "時尚", "逛街"],
      "hobby" => ["攝影", "音樂", "繪畫", "遊戲", "手作"]
    }

    related = Map.get(mappings, activity, [])
    matches = Enum.count(interests, fn i -> i in related end)

    matches * 0.15
  end

  defp should_follow_schedule?(emotion, activity) do
    # 如果情緒極端，可能不想按照排程
    if emotion.valence < -50 or emotion.energy < 20 do
      :rand.uniform() < 0.3  # 30% 機率按排程
    else
      :rand.uniform() < 0.7  # 70% 機率按排程
    end
  end

  defp select_by_desire(desires) do
    # 加權隨機選擇
    total = desires |> Map.values() |> Enum.sum()
    roll = :rand.uniform() * total

    desires
    |> Enum.reduce_while(0, fn {activity, desire}, acc ->
      new_acc = acc + desire
      if new_acc >= roll do
        {:halt, activity}
      else
        {:cont, new_acc}
      end
    end)
  end

  defp parse_activity(response) do
    mappings = %{
      "咖啡店" => "cafe",
      "咖啡" => "cafe",
      "工作" => "work",
      "運動" => "exercise",
      "健身" => "exercise",
      "逛街" => "shopping",
      "購物" => "shopping",
      "社交" => "social",
      "興趣活動" => "hobby",
      "興趣" => "hobby",
      "休息" => "rest"
    }

    response = String.trim(response)

    Enum.find_value(mappings, "rest", fn {key, value} ->
      if String.contains?(response, key), do: value
    end)
  end

  defp get_time_description do
    hour = DateTime.utc_now().hour + 8  # 轉換為台北時間

    cond do
      hour >= 6 and hour < 9 -> "早晨"
      hour >= 9 and hour < 12 -> "上午"
      hour >= 12 and hour < 14 -> "中午"
      hour >= 14 and hour < 18 -> "下午"
      hour >= 18 and hour < 21 -> "傍晚"
      hour >= 21 or hour < 6 -> "深夜"
    end
  end
end
```

---

## 6. 角色互動系統

### 6.1 Relationship Schema

```elixir
# lib/xaifu/social/relationship.ex
defmodule Xaifu.Social.Relationship do
  use Ecto.Schema
  import Ecto.Changeset

  @primary_key {:id, :binary_id, autogenerate: true}
  @foreign_key_type :binary_id

  schema "relationships" do
    belongs_to :character_a, Xaifu.Characters.Character
    belongs_to :character_b, Xaifu.Characters.Character

    field :affinity, :integer, default: 50      # -100 到 +100
    field :familiarity, :integer, default: 0    # 0 到 100
    field :interaction_count, :integer, default: 0
    field :last_interaction_at, :utc_datetime

    timestamps(type: :utc_datetime)
  end

  def changeset(relationship, attrs) do
    relationship
    |> cast(attrs, [
      :character_a_id, :character_b_id, :affinity,
      :familiarity, :interaction_count, :last_interaction_at
    ])
    |> validate_required([:character_a_id, :character_b_id])
    |> validate_number(:affinity,
        greater_than_or_equal_to: -100,
        less_than_or_equal_to: 100)
    |> validate_number(:familiarity,
        greater_than_or_equal_to: 0,
        less_than_or_equal_to: 100)
    |> unique_constraint([:character_a_id, :character_b_id])
  end
end
```

### 6.2 Comment Schema

```elixir
# lib/xaifu/social/comment.ex
defmodule Xaifu.Social.Comment do
  use Ecto.Schema
  import Ecto.Changeset

  @primary_key {:id, :binary_id, autogenerate: true}
  @foreign_key_type :binary_id

  schema "comments" do
    belongs_to :post, Xaifu.Social.Post
    belongs_to :character, Xaifu.Characters.Character  # 留言者
    field :user_identifier, :string  # 如果是用戶留言

    field :content, :string
    field :is_ai_generated, :boolean, default: false

    timestamps(type: :utc_datetime)
  end

  def changeset(comment, attrs) do
    comment
    |> cast(attrs, [:post_id, :character_id, :user_identifier, :content, :is_ai_generated])
    |> validate_required([:post_id, :content])
    |> validate_length(:content, min: 1, max: 500)
    |> validate_commenter()
  end

  defp validate_commenter(changeset) do
    character_id = get_field(changeset, :character_id)
    user_identifier = get_field(changeset, :user_identifier)

    if is_nil(character_id) and is_nil(user_identifier) do
      add_error(changeset, :base, "必須指定留言者")
    else
      changeset
    end
  end
end
```

### 6.3 Interaction Worker

```elixir
# lib/xaifu/workers/character_interaction_worker.ex
defmodule Xaifu.Workers.CharacterInteractionWorker do
  @moduledoc """
  處理角色間自動互動
  """

  use Oban.Worker,
    queue: :default,
    max_attempts: 2

  require Logger

  alias Xaifu.Characters
  alias Xaifu.Social
  alias Xaifu.AI.LLM

  @impl Oban.Worker
  def perform(%Oban.Job{args: %{"action" => "check_interactions"}}) do
    # 取得所有活躍角色
    active_characters = Characters.list_active_characters()

    # 取得最近的貼文
    recent_posts = Social.list_feed(limit: 20)

    # 為每個角色檢查是否要互動
    Enum.each(active_characters, fn character ->
      check_and_interact(character, recent_posts)
    end)

    :ok
  end

  def perform(%Oban.Job{args: %{
    "action" => "generate_comment",
    "character_id" => character_id,
    "post_id" => post_id
  }}) do
    character = Characters.get_character!(character_id)
    post = Social.get_post!(post_id)

    case generate_comment(character, post) do
      {:ok, content} ->
        Social.create_comment(%{
          post_id: post.id,
          character_id: character.id,
          content: content,
          is_ai_generated: true
        })

        # 更新關係
        update_relationship(character.id, post.character_id)

        :ok

      {:error, reason} ->
        Logger.error("Failed to generate comment: #{inspect(reason)}")
        {:error, reason}
    end
  end

  # ============================================
  # Private Functions
  # ============================================

  defp check_and_interact(character, posts) do
    # 過濾掉自己的貼文
    other_posts = Enum.filter(posts, fn p -> p.character_id != character.id end)

    Enum.each(other_posts, fn post ->
      if should_interact?(character, post) do
        action = decide_action(character, post)
        perform_action(character, post, action)
      end
    end)
  end

  defp should_interact?(character, post) do
    # 取得關係
    relationship = get_relationship(character.id, post.character_id)
    affinity = relationship.affinity

    # 基礎機率 + 好感度調整
    base_probability = 0.15
    affinity_modifier = affinity / 500  # -0.2 到 +0.2

    probability = base_probability + affinity_modifier
    :rand.uniform() < probability
  end

  defp decide_action(character, post) do
    # 決定是按讚還是留言
    if :rand.uniform() < 0.7 do
      :like
    else
      :comment
    end
  end

  defp perform_action(character, post, :like) do
    # 檢查是否已按讚
    unless Social.has_liked?(character.id, post.id) do
      Social.create_like(%{
        post_id: post.id,
        character_id: character.id
      })
    end
  end

  defp perform_action(character, post, :comment) do
    # 排入留言生成任務
    %{
      action: "generate_comment",
      character_id: character.id,
      post_id: post.id
    }
    |> __MODULE__.new(schedule_in: Enum.random(60..300))  # 1-5 分鐘後
    |> Oban.insert()
  end

  defp generate_comment(character, post) do
    post_author = Characters.get_character!(post.character_id)
    relationship = get_relationship(character.id, post.character_id)

    prompt = """
    你是 #{character.name}，性格：#{character.personality}

    你看到了 #{post_author.name} 的貼文：
    「#{post.content}」

    你和 #{post_author.name} 的關係：#{relationship_description(relationship)}

    請以 #{character.name} 的身份，寫一則簡短的留言回覆（20-50字）。
    保持角色一致性，自然地回應。
    """

    LLM.chat_completion([
      %{role: "user", content: prompt}
    ], temperature: 0.9, max_tokens: 100)
  end

  defp get_relationship(char_a_id, char_b_id) do
    Social.get_or_create_relationship(char_a_id, char_b_id)
  end

  defp update_relationship(from_id, to_id) do
    relationship = get_relationship(from_id, to_id)

    Social.update_relationship(relationship, %{
      interaction_count: relationship.interaction_count + 1,
      familiarity: min(relationship.familiarity + 1, 100),
      last_interaction_at: DateTime.utc_now()
    })
  end

  defp relationship_description(relationship) do
    cond do
      relationship.familiarity < 20 -> "幾乎不認識"
      relationship.familiarity < 50 -> "有點熟悉"
      relationship.familiarity < 80 -> "熟識的朋友"
      true -> "非常親密的朋友"
    end
  end
end
```

---

## 7. 整合更新

### 7.1 更新 Agent 加入情緒系統

```elixir
# lib/xaifu/agents/agent.ex - 更新版本

defmodule Xaifu.Agents.Agent do
  use GenServer
  require Logger

  alias Xaifu.Agents.{State, EmotionEngine, DynamicScheduler}
  alias Xaifu.Characters
  alias Xaifu.AI.Memory

  # ... 其他程式碼保持不變 ...

  # 更新初始化，加入情緒
  @impl true
  def init(character_id) do
    case Characters.get_character(character_id) do
      nil ->
        {:stop, :character_not_found}

      character ->
        emotion = EmotionEngine.new()
        state = State.new(character)
        |> Map.put(:emotion, emotion)
        |> Map.put(:last_emotion_update, DateTime.utc_now())

        schedule_next_check()
        schedule_emotion_tick()

        {:ok, state}
    end
  end

  # 情緒定時更新
  @impl true
  def handle_info(:emotion_tick, state) do
    hours_passed = DateTime.diff(
      DateTime.utc_now(),
      state.last_emotion_update,
      :hour
    )

    new_emotion = EmotionEngine.apply_time_decay(state.emotion, max(hours_passed, 1))

    schedule_emotion_tick()

    {:noreply, %{state |
      emotion: new_emotion,
      last_emotion_update: DateTime.utc_now()
    }}
  end

  # 動態活動決策
  defp do_check_activity(state) do
    # 使用動態排程決策
    activity = DynamicScheduler.decide_next_activity(
      state.character,
      state.emotion,
      scheduled_activity: find_scheduled_activity(state)
    )

    # 更新情緒
    new_emotion = EmotionEngine.apply_activity(state.emotion, activity)

    # 執行活動
    execute_activity(state, activity)
    |> Map.put(:emotion, new_emotion)
  end

  defp schedule_emotion_tick do
    # 每小時更新情緒
    Process.send_after(self(), :emotion_tick, :timer.hours(1))
  end

  # ... 其他程式碼 ...
end
```

### 7.2 更新聊天系統加入記憶

```elixir
# lib/xaifu_web/channels/chat_channel.ex - 更新版本

defmodule XaifuWeb.ChatChannel do
  use XaifuWeb, :channel

  alias Xaifu.{Chat, Characters}
  alias Xaifu.AI.{LLM, Memory}
  alias Xaifu.Agents.{Agent, EmotionEngine}

  # ... 其他程式碼保持不變 ...

  defp generate_and_send_response(socket, character, conversation) do
    user_id = socket.assigns.user_id

    # 取得對話歷史
    history = Chat.get_recent_messages(conversation.id, 10)
    formatted_history = Chat.format_messages_for_llm(history)

    # 取得最後一則用戶訊息
    user_message = List.last(history).content

    # 取得相關記憶
    memory_context = case Memory.build_context(
      character.id,
      user_message,
      user_identifier: user_id
    ) do
      {:ok, ctx} -> ctx
      _ -> nil
    end

    # 取得當前情緒（如果 Agent 在運行）
    emotion_context = if Agent.exists?(character.id) do
      state = Agent.get_state(character.id)
      EmotionEngine.describe(state.emotion)
    else
      nil
    end

    # 建構增強的系統提示
    enhanced_system = build_enhanced_prompt(character, memory_context, emotion_context)

    # 生成回應
    case generate_response(enhanced_system, formatted_history, user_message) do
      {:ok, response_content} ->
        # 儲存 AI 回應
        {:ok, ai_message} = Chat.create_message(conversation.id, %{
          role: :assistant,
          content: response_content
        })

        # 廣播回應
        Phoenix.Channel.broadcast!(socket, "new_message", %{
          message: serialize_message(ai_message)
        })

        # 非同步提取記憶
        Task.start(fn ->
          Memory.extract_from_conversation(
            character.id,
            user_id,
            [List.last(history), ai_message]
          )
        end)

        # 更新統計和情緒
        Characters.increment_stat(character, :total_chats)
        maybe_update_emotion(character.id, :chat_message, response_content)

      {:error, reason} ->
        Phoenix.Channel.push(socket, "error", %{
          message: "生成回應時發生錯誤"
        })
    end
  end

  defp build_enhanced_prompt(character, memory_context, emotion_context) do
    base = character.system_prompt || """
    你是 #{character.name}。#{character.personality}
    """

    parts = [base]

    if memory_context do
      parts = parts ++ ["\n\n過去的對話記憶：\n#{memory_context}"]
    end

    if emotion_context do
      parts = parts ++ ["\n\n目前的心情：#{emotion_context}"]
    end

    Enum.join(parts, "\n")
  end

  defp generate_response(system_prompt, history, user_message) do
    messages = [%{role: "system", content: system_prompt}] ++
      history ++
      [%{role: "user", content: user_message}]

    LLM.chat_completion(messages)
  end

  defp maybe_update_emotion(character_id, interaction_type, content) do
    if Agent.exists?(character_id) do
      sentiment = analyze_sentiment(content)
      Agent.apply_interaction(character_id, interaction_type, sentiment)
    end
  end

  defp analyze_sentiment(content) do
    cond do
      String.contains?(content, ["開心", "謝謝", "太好了", "喜歡"]) -> :positive
      String.contains?(content, ["討厭", "煩", "難過", "不好"]) -> :negative
      true -> :neutral
    end
  end

  # ... 其他程式碼 ...
end
```

---

## 8. 測試案例

### 8.1 Memory 測試

```elixir
# test/xaifu/ai/memory_test.exs
defmodule Xaifu.AI.MemoryTest do
  use Xaifu.DataCase, async: true

  alias Xaifu.AI.Memory
  alias Xaifu.Characters

  setup do
    {:ok, character} = Characters.create_character(%{
      name: "Memory Test",
      personality: "測試角色"
    })

    %{character: character}
  end

  describe "store/3" do
    test "stores memory with embedding", %{character: character} do
      # 注意：此測試需要有效的 OpenAI API key
      # 在 CI 環境可以 mock

      assert {:ok, memory} = Memory.store(
        character.id,
        "用戶喜歡喝咖啡",
        user_identifier: "test_user",
        memory_type: :long_term
      )

      assert memory.content == "用戶喜歡喝咖啡"
      assert memory.memory_type == :long_term
      assert memory.embedding != nil
    end
  end

  describe "retrieve/3" do
    @tag :external
    test "retrieves relevant memories", %{character: character} do
      # 儲存一些記憶
      Memory.store(character.id, "用戶喜歡喝咖啡", user_identifier: "test_user")
      Memory.store(character.id, "用戶養了一隻貓", user_identifier: "test_user")
      Memory.store(character.id, "用戶在科技公司工作", user_identifier: "test_user")

      # 搜尋相關記憶
      {:ok, memories} = Memory.retrieve(
        character.id,
        "你記得我喜歡喝什麼嗎？",
        user_identifier: "test_user",
        limit: 2
      )

      # 應該要找到咖啡相關的記憶
      contents = Enum.map(memories, & &1.content)
      assert Enum.any?(contents, &String.contains?(&1, "咖啡"))
    end
  end
end
```

### 8.2 EmotionEngine 測試

```elixir
# test/xaifu/agents/emotion_engine_test.exs
defmodule Xaifu.Agents.EmotionEngineTest do
  use ExUnit.Case, async: true

  alias Xaifu.Agents.EmotionEngine

  describe "new/1" do
    test "creates emotion with default values" do
      emotion = EmotionEngine.new()

      assert emotion.valence >= -100 and emotion.valence <= 100
      assert emotion.arousal >= 0 and emotion.arousal <= 100
      assert emotion.energy >= 0 and emotion.energy <= 100
      assert emotion.social >= 0 and emotion.social <= 100
    end

    test "accepts custom values" do
      emotion = EmotionEngine.new(valence: 80, energy: 90)

      assert emotion.valence == 80
      assert emotion.energy == 90
    end
  end

  describe "apply_time_decay/2" do
    test "decreases energy over time" do
      emotion = EmotionEngine.new(energy: 80)
      decayed = EmotionEngine.apply_time_decay(emotion, 2)

      assert decayed.energy < emotion.energy
    end

    test "increases social need over time" do
      emotion = EmotionEngine.new(social: 50)
      decayed = EmotionEngine.apply_time_decay(emotion, 3)

      assert decayed.social > emotion.social
    end
  end

  describe "apply_activity/2" do
    test "exercise increases valence but decreases energy" do
      emotion = EmotionEngine.new(valence: 50, energy: 80)
      after_exercise = EmotionEngine.apply_activity(emotion, "exercise")

      assert after_exercise.valence > emotion.valence
      assert after_exercise.energy < emotion.energy
    end

    test "rest increases energy" do
      emotion = EmotionEngine.new(energy: 40)
      after_rest = EmotionEngine.apply_activity(emotion, "rest")

      assert after_rest.energy > emotion.energy
    end
  end

  describe "current_mood/1" do
    test "high valence and arousal returns excited" do
      emotion = %EmotionEngine{valence: 70, arousal: 70, energy: 50, social: 50}
      assert EmotionEngine.current_mood(emotion) == :excited
    end

    test "low energy returns exhausted" do
      emotion = %EmotionEngine{valence: 30, arousal: 30, energy: 20, social: 50}
      assert EmotionEngine.current_mood(emotion) == :exhausted
    end

    test "high social need returns lonely" do
      emotion = %EmotionEngine{valence: 30, arousal: 30, energy: 50, social: 90}
      assert EmotionEngine.current_mood(emotion) == :lonely
    end
  end

  describe "describe/1" do
    test "returns readable description" do
      emotion = EmotionEngine.new(valence: 70, energy: 80)
      description = EmotionEngine.describe(emotion)

      assert is_binary(description)
      assert description != ""
    end
  end
end
```

---

## 9. 配置更新

```elixir
# config/config.exs

# pgvector 設定
config :xaifu, Xaifu.Repo,
  types: Xaifu.PostgresTypes

# 記憶系統設定
config :xaifu, Xaifu.AI.Memory,
  max_memories_per_character: 1000,
  cleanup_interval_hours: 24,
  short_term_retention_days: 7

# 互動排程
config :xaifu, Oban,
  plugins: [
    Oban.Plugins.Pruner,
    {Oban.Plugins.Cron, crontab: [
      # 每小時檢查角色互動
      {"0 * * * *", Xaifu.Workers.CharacterInteractionWorker, args: %{action: "check_interactions"}},
      # 每天清理舊記憶
      {"0 3 * * *", Xaifu.Workers.MemoryCleanupWorker}
    ]}
  ]
```

---

## 10. 驗收標準

### 10.1 功能驗收

| 項目 | 驗收條件 | 狀態 |
|------|----------|------|
| 記憶儲存 | 對話重要內容自動存入記憶 | ☐ |
| 記憶檢索 | 對話時能檢索相關記憶 | ☐ |
| 情緒系統 | 情緒隨時間和活動變化 | ☐ |
| 動態日程 | 角色根據心情決定活動 | ☐ |
| 角色按讚 | 角色會對其他角色貼文按讚 | ☐ |
| 角色留言 | 角色會對其他角色貼文留言 | ☐ |
| 關係系統 | 互動影響角色間關係 | ☐ |

### 10.2 品質驗收

| 指標 | 目標值 |
|------|--------|
| 記憶檢索準確率 | > 80% |
| 記憶檢索時間 | < 500ms |
| 情緒計算時間 | < 50ms |
| 留言生成時間 | < 5s |

---

## 11. 未來擴展

完成 Phase 4 後，可考慮的進階功能：

1. **多模態記憶**：記住圖片、影片等內容
2. **群組互動**：角色組成小團體進行群聊
3. **事件系統**：特殊節日、生日等事件
4. **角色成長**：隨時間學習新技能、改變性格
5. **用戶關係**：追蹤系統、親密度系統

---

*文件版本: 1.0*
*對應主文件: [00-overview.md](./00-overview.md)*

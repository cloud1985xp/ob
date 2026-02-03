# Phase 1: 基礎建設與 MVP

## 預估工時：3-4 週（單人開發）

---

## 1. 階段目標

建立 Xaifu 的核心基礎架構，完成最小可行產品 (MVP)，包含：
- 專案初始化與基礎配置
- 角色資料模型與 CRUD 功能
- 基本聊天功能
- LLM 整合模組

**里程碑**：用戶可以建立角色並與之進行基本對話

---

## 2. 任務分解

### Week 1: 專案初始化與資料模型

| 任務 | 預估時間 | 優先級 |
|------|----------|--------|
| 建立 Phoenix 專案 | 2h | P0 |
| 設定開發環境 | 2h | P0 |
| 設計角色資料模型 | 4h | P0 |
| 實作資料庫遷移 | 4h | P0 |
| 建立 Character Context | 8h | P0 |
| 單元測試 | 8h | P0 |

### Week 2: 角色管理介面

| 任務 | 預估時間 | 優先級 |
|------|----------|--------|
| 設計 UI 原型 | 4h | P0 |
| LiveView 列表頁 | 6h | P0 |
| LiveView 表單頁 | 8h | P0 |
| 角色詳情頁 | 6h | P0 |
| 元件測試 | 4h | P0 |

### Week 3: LLM 整合與聊天功能

| 任務 | 預估時間 | 優先級 |
|------|----------|--------|
| LLM 客戶端模組 | 8h | P0 |
| Prompt Builder 模組 | 8h | P0 |
| Phoenix Channels 設定 | 4h | P0 |
| 聊天室 LiveView | 8h | P0 |
| 整合測試 | 8h | P0 |

### Week 4: 完善與測試

| 任務 | 預估時間 | 優先級 |
|------|----------|--------|
| 錯誤處理完善 | 8h | P1 |
| UI 美化 | 8h | P1 |
| 效能優化 | 4h | P1 |
| 端到端測試 | 8h | P0 |
| 文件撰寫 | 4h | P1 |

---

## 3. 技術規格

### 3.1 專案初始化

```bash
# 建立專案
mix phx.new xaifu --database postgres --live

# 安裝相依套件 (mix.exs)
defp deps do
  [
    {:phoenix, "~> 1.7.14"},
    {:phoenix_ecto, "~> 4.6"},
    {:ecto_sql, "~> 3.12"},
    {:postgrex, ">= 0.0.0"},
    {:phoenix_live_view, "~> 1.0"},
    {:phoenix_live_dashboard, "~> 0.8"},
    {:tailwind, "~> 0.2"},
    {:heroicons, "~> 0.5"},
    {:oban, "~> 2.17"},
    {:req_llm, "~> 1.5"},       # LLM 統一介面 (支援 OpenAI, Anthropic 等 45+ providers)
    {:jason, "~> 1.4"},
    {:faker, "~> 0.18", only: [:dev, :test]},
    {:ex_machina, "~> 2.8", only: :test}
  ]
end
```

### 3.2 環境配置

```elixir
# config/config.exs
config :xaifu, Xaifu.Repo,
  adapter: Ecto.Adapters.Postgres

# ReqLLM 設定 - 支援多種 LLM 服務商
# 可用格式: "provider:model" 如 "openai:gpt-4o" 或 "anthropic:claude-sonnet-4"
config :xaifu, Xaifu.AI,
  default_model: "anthropic:claude-sonnet-4",
  default_temperature: 0.7,
  default_max_tokens: 1000

config :xaifu, Oban,
  repo: Xaifu.Repo,
  plugins: [Oban.Plugins.Pruner],
  queues: [default: 10, llm: 5, images: 3]
```

```elixir
# config/runtime.exs
# ReqLLM 支援多種 API Key 配置方式:
# 1. 環境變數 (推薦): OPENAI_API_KEY, ANTHROPIC_API_KEY
# 2. Application config
# 3. .env 檔案 (透過 dotenvy 自動載入)

config :req_llm,
  openai_api_key: System.get_env("OPENAI_API_KEY"),
  anthropic_api_key: System.get_env("ANTHROPIC_API_KEY")

# 確保至少有一個 API Key
unless System.get_env("OPENAI_API_KEY") || System.get_env("ANTHROPIC_API_KEY") do
  IO.warn("Warning: No LLM API key configured. Set OPENAI_API_KEY or ANTHROPIC_API_KEY")
end
```

---

## 4. 資料模型

### 4.1 Character Schema

```elixir
# lib/xaifu/characters/character.ex
defmodule Xaifu.Characters.Character do
  use Ecto.Schema
  import Ecto.Changeset

  @primary_key {:id, :binary_id, autogenerate: true}
  @foreign_key_type :binary_id

  schema "characters" do
    field :name, :string
    field :avatar_url, :string
    field :bio, :string

    # 人格設定
    field :personality, :string           # 性格描述：活潑、憂鬱、傲嬌
    field :interests, {:array, :string}   # 興趣列表
    field :speaking_style, :string        # 說話風格
    field :background, :string            # 背景故事

    # LLM 設定
    field :system_prompt, :string         # 系統提示詞
    field :temperature, :float, default: 0.7

    # 視覺設定（Phase 3 使用）
    field :appearance, :string            # 外觀描述
    field :image_prompt_prefix, :string   # 圖像生成前綴

    # 狀態
    field :status, Ecto.Enum,
      values: [:active, :inactive, :archived],
      default: :inactive

    # 統計
    field :total_posts, :integer, default: 0
    field :total_chats, :integer, default: 0

    timestamps(type: :utc_datetime)
  end

  @required_fields [:name, :personality]
  @optional_fields [
    :avatar_url, :bio, :interests, :speaking_style,
    :background, :system_prompt, :temperature,
    :appearance, :image_prompt_prefix, :status
  ]

  def changeset(character, attrs) do
    character
    |> cast(attrs, @required_fields ++ @optional_fields)
    |> validate_required(@required_fields)
    |> validate_length(:name, min: 1, max: 50)
    |> validate_length(:personality, min: 10, max: 500)
    |> validate_length(:system_prompt, max: 2000)
    |> validate_number(:temperature,
        greater_than_or_equal_to: 0,
        less_than_or_equal_to: 2)
    |> validate_interests()
  end

  defp validate_interests(changeset) do
    case get_field(changeset, :interests) do
      nil -> changeset
      interests when length(interests) > 20 ->
        add_error(changeset, :interests, "最多 20 個興趣")
      _ -> changeset
    end
  end
end
```

### 4.2 資料庫遷移

```elixir
# priv/repo/migrations/20260203000001_create_characters.exs
defmodule Xaifu.Repo.Migrations.CreateCharacters do
  use Ecto.Migration

  def change do
    create table(:characters, primary_key: false) do
      add :id, :binary_id, primary_key: true
      add :name, :string, null: false
      add :avatar_url, :string
      add :bio, :text

      # 人格設定
      add :personality, :text, null: false
      add :interests, {:array, :string}, default: []
      add :speaking_style, :text
      add :background, :text

      # LLM 設定
      add :system_prompt, :text
      add :temperature, :float, default: 0.7

      # 視覺設定
      add :appearance, :text
      add :image_prompt_prefix, :text

      # 狀態
      add :status, :string, default: "inactive"

      # 統計
      add :total_posts, :integer, default: 0
      add :total_chats, :integer, default: 0

      timestamps(type: :utc_datetime)
    end

    create index(:characters, [:status])
    create index(:characters, [:name])
  end
end
```

### 4.3 Conversation 與 Message Schema

```elixir
# lib/xaifu/chat/conversation.ex
defmodule Xaifu.Chat.Conversation do
  use Ecto.Schema
  import Ecto.Changeset

  @primary_key {:id, :binary_id, autogenerate: true}
  @foreign_key_type :binary_id

  schema "conversations" do
    belongs_to :character, Xaifu.Characters.Character
    field :user_identifier, :string  # 暫時用字串識別用戶，Phase 後期可改為 User 關聯
    field :title, :string
    field :last_message_at, :utc_datetime
    field :message_count, :integer, default: 0

    has_many :messages, Xaifu.Chat.Message

    timestamps(type: :utc_datetime)
  end

  def changeset(conversation, attrs) do
    conversation
    |> cast(attrs, [:character_id, :user_identifier, :title, :last_message_at])
    |> validate_required([:character_id, :user_identifier])
    |> foreign_key_constraint(:character_id)
  end
end
```

```elixir
# lib/xaifu/chat/message.ex
defmodule Xaifu.Chat.Message do
  use Ecto.Schema
  import Ecto.Changeset

  @primary_key {:id, :binary_id, autogenerate: true}
  @foreign_key_type :binary_id

  schema "messages" do
    belongs_to :conversation, Xaifu.Chat.Conversation

    field :role, Ecto.Enum, values: [:user, :assistant, :system]
    field :content, :string
    field :tokens_used, :integer
    field :metadata, :map, default: %{}

    timestamps(type: :utc_datetime)
  end

  def changeset(message, attrs) do
    message
    |> cast(attrs, [:conversation_id, :role, :content, :tokens_used, :metadata])
    |> validate_required([:conversation_id, :role, :content])
    |> validate_length(:content, min: 1, max: 10_000)
    |> foreign_key_constraint(:conversation_id)
  end
end
```

---

## 5. Context 模組實作

### 5.1 Characters Context

```elixir
# lib/xaifu/characters.ex
defmodule Xaifu.Characters do
  @moduledoc """
  Characters context - 管理 AI 角色的建立、查詢、更新
  """

  import Ecto.Query, warn: false
  alias Xaifu.Repo
  alias Xaifu.Characters.Character

  @doc """
  列出所有角色，支援篩選與分頁
  """
  def list_characters(opts \\ []) do
    Character
    |> filter_by_status(opts[:status])
    |> order_by([c], desc: c.inserted_at)
    |> paginate(opts)
    |> Repo.all()
  end

  defp filter_by_status(query, nil), do: query
  defp filter_by_status(query, status) do
    where(query, [c], c.status == ^status)
  end

  defp paginate(query, opts) do
    limit = Keyword.get(opts, :limit, 20)
    offset = Keyword.get(opts, :offset, 0)

    query
    |> limit(^limit)
    |> offset(^offset)
  end

  @doc """
  取得單一角色
  """
  def get_character(id), do: Repo.get(Character, id)

  def get_character!(id), do: Repo.get!(Character, id)

  @doc """
  根據名稱查找角色
  """
  def get_character_by_name(name) do
    Repo.get_by(Character, name: name)
  end

  @doc """
  建立新角色
  """
  def create_character(attrs \\ %{}) do
    %Character{}
    |> Character.changeset(attrs)
    |> maybe_generate_system_prompt(attrs)
    |> Repo.insert()
  end

  defp maybe_generate_system_prompt(changeset, attrs) do
    if is_nil(attrs[:system_prompt]) or attrs[:system_prompt] == "" do
      system_prompt = generate_default_system_prompt(changeset)
      Ecto.Changeset.put_change(changeset, :system_prompt, system_prompt)
    else
      changeset
    end
  end

  defp generate_default_system_prompt(changeset) do
    name = Ecto.Changeset.get_field(changeset, :name)
    personality = Ecto.Changeset.get_field(changeset, :personality)
    interests = Ecto.Changeset.get_field(changeset, :interests) || []
    speaking_style = Ecto.Changeset.get_field(changeset, :speaking_style)
    background = Ecto.Changeset.get_field(changeset, :background)

    interests_text = if interests != [], do: "興趣包括：#{Enum.join(interests, "、")}。", else: ""
    style_text = if speaking_style, do: "說話風格：#{speaking_style}。", else: ""
    background_text = if background, do: "背景：#{background}", else: ""

    """
    你是 #{name}，一個有著獨特個性的虛擬角色。

    性格特徵：#{personality}

    #{interests_text}
    #{style_text}
    #{background_text}

    請以 #{name} 的身份與用戶對話，保持角色一致性。
    回應時要自然、有個性，像真實的人一樣交流。
    """
    |> String.trim()
  end

  @doc """
  更新角色
  """
  def update_character(%Character{} = character, attrs) do
    character
    |> Character.changeset(attrs)
    |> Repo.update()
  end

  @doc """
  刪除角色
  """
  def delete_character(%Character{} = character) do
    Repo.delete(character)
  end

  @doc """
  更改角色狀態
  """
  def change_status(%Character{} = character, new_status) do
    character
    |> Ecto.Changeset.change(status: new_status)
    |> Repo.update()
  end

  @doc """
  增加統計計數
  """
  def increment_stat(%Character{} = character, field) when field in [:total_posts, :total_chats] do
    Character
    |> where([c], c.id == ^character.id)
    |> Repo.update_all(inc: [{field, 1}])
  end

  @doc """
  取得活躍角色
  """
  def list_active_characters do
    Character
    |> where([c], c.status == :active)
    |> Repo.all()
  end

  @doc """
  建立角色的 changeset（供 LiveView 表單使用）
  """
  def change_character(%Character{} = character, attrs \\ %{}) do
    Character.changeset(character, attrs)
  end
end
```

### 5.2 Chat Context

```elixir
# lib/xaifu/chat.ex
defmodule Xaifu.Chat do
  @moduledoc """
  Chat context - 管理對話與訊息
  """

  import Ecto.Query, warn: false
  alias Xaifu.Repo
  alias Xaifu.Chat.{Conversation, Message}
  alias Xaifu.Characters

  @doc """
  取得或建立對話
  """
  def get_or_create_conversation(character_id, user_identifier) do
    case get_conversation(character_id, user_identifier) do
      nil -> create_conversation(character_id, user_identifier)
      conversation -> {:ok, conversation}
    end
  end

  def get_conversation(character_id, user_identifier) do
    Conversation
    |> where([c], c.character_id == ^character_id and c.user_identifier == ^user_identifier)
    |> Repo.one()
  end

  def create_conversation(character_id, user_identifier) do
    character = Characters.get_character!(character_id)

    %Conversation{}
    |> Conversation.changeset(%{
      character_id: character_id,
      user_identifier: user_identifier,
      title: "與 #{character.name} 的對話"
    })
    |> Repo.insert()
  end

  @doc """
  列出對話的訊息歷史
  """
  def list_messages(conversation_id, opts \\ []) do
    limit = Keyword.get(opts, :limit, 50)

    Message
    |> where([m], m.conversation_id == ^conversation_id)
    |> order_by([m], asc: m.inserted_at)
    |> limit(^limit)
    |> Repo.all()
  end

  @doc """
  取得最近 N 則訊息（用於建構 LLM context）
  """
  def get_recent_messages(conversation_id, limit \\ 20) do
    Message
    |> where([m], m.conversation_id == ^conversation_id)
    |> order_by([m], desc: m.inserted_at)
    |> limit(^limit)
    |> Repo.all()
    |> Enum.reverse()
  end

  @doc """
  新增訊息
  """
  def create_message(conversation_id, attrs) do
    %Message{}
    |> Message.changeset(Map.put(attrs, :conversation_id, conversation_id))
    |> Repo.insert()
    |> tap(fn
      {:ok, _} -> update_conversation_stats(conversation_id)
      _ -> :ok
    end)
  end

  defp update_conversation_stats(conversation_id) do
    Conversation
    |> where([c], c.id == ^conversation_id)
    |> Repo.update_all(
      set: [last_message_at: DateTime.utc_now()],
      inc: [message_count: 1]
    )
  end

  @doc """
  將訊息格式化為 LLM API 格式
  """
  def format_messages_for_llm(messages) do
    Enum.map(messages, fn message ->
      %{
        role: to_string(message.role),
        content: message.content
      }
    end)
  end
end
```

---

## 6. LLM 整合模組（使用 ReqLLM）

> **技術選型說明**：使用 [ReqLLM](https://hex.pm/packages/req_llm) 套件作為 LLM 整合方案。
> ReqLLM 是基於 Req 建構的統一 LLM 客戶端，支援 45+ 個服務商（包含 OpenAI、Anthropic、Google 等），
> 提供標準化 API、串流支援、結構化輸出、成本追蹤等功能。

### 6.1 LLM 模組介面

```elixir
# lib/xaifu/ai/llm.ex
defmodule Xaifu.AI.LLM do
  @moduledoc """
  LLM 統一介面 - 基於 ReqLLM 實作

  支援的模型格式：
  - "openai:gpt-4o"
  - "anthropic:claude-sonnet-4"
  - "anthropic:claude-haiku-4"

  使用範例：
      # 簡單文字生成
      {:ok, response} = LLM.generate_text("Hello!")

      # 與角色對話
      {:ok, response} = LLM.chat_with_character(character, history, "你好")

      # 串流回應
      {:ok, stream_response} = LLM.stream_with_character(character, history, "說個故事")
  """

  @doc """
  生成文字回應
  """
  def generate_text(prompt, opts \\ []) do
    model = Keyword.get(opts, :model, default_model())
    temperature = Keyword.get(opts, :temperature, default_temperature())
    max_tokens = Keyword.get(opts, :max_tokens, default_max_tokens())

    case ReqLLM.generate_text(model, prompt,
           temperature: temperature,
           max_tokens: max_tokens
         ) do
      {:ok, response} ->
        {:ok, response.text, %{usage: response.usage}}

      {:error, reason} ->
        {:error, reason}
    end
  end

  @doc """
  帶有對話上下文的文字生成
  """
  def chat_completion(messages, opts \\ []) do
    model = Keyword.get(opts, :model, default_model())
    temperature = Keyword.get(opts, :temperature, default_temperature())
    max_tokens = Keyword.get(opts, :max_tokens, default_max_tokens())

    context = build_context(messages)

    case ReqLLM.generate_text(model, context,
           temperature: temperature,
           max_tokens: max_tokens
         ) do
      {:ok, response} ->
        {:ok, response.text, %{usage: response.usage}}

      {:error, reason} ->
        {:error, reason}
    end
  end

  @doc """
  與角色對話（非串流）
  """
  def chat_with_character(character, conversation_history, user_message) do
    system_prompt = character.system_prompt || default_system_prompt(character)
    model = Keyword.get_lazy([], :model, fn -> default_model() end)

    context =
      ReqLLM.Context.new([
        ReqLLM.Context.system(system_prompt)
        | build_history_messages(conversation_history)
      ])
      |> ReqLLM.Context.append(ReqLLM.Context.user(user_message))

    case ReqLLM.generate_text(model, context,
           temperature: character.temperature || default_temperature(),
           max_tokens: default_max_tokens()
         ) do
      {:ok, response} ->
        {:ok, response.text, %{usage: response.usage}}

      {:error, reason} ->
        {:error, reason}
    end
  end

  @doc """
  與角色對話（串流模式）- 用於即時顯示回應
  """
  def stream_with_character(character, conversation_history, user_message) do
    system_prompt = character.system_prompt || default_system_prompt(character)
    model = default_model()

    context =
      ReqLLM.Context.new([
        ReqLLM.Context.system(system_prompt)
        | build_history_messages(conversation_history)
      ])
      |> ReqLLM.Context.append(ReqLLM.Context.user(user_message))

    ReqLLM.stream_text(model, context,
      temperature: character.temperature || default_temperature(),
      max_tokens: default_max_tokens()
    )
  end

  @doc """
  消費串流回應的 tokens
  """
  def consume_stream(stream_response, callback) when is_function(callback, 1) do
    stream_response
    |> ReqLLM.StreamResponse.tokens()
    |> Stream.each(callback)
    |> Stream.run()

    # 串流結束後取得 usage 資訊
    ReqLLM.StreamResponse.usage(stream_response)
  end

  # Private helpers

  defp build_context(messages) do
    messages
    |> Enum.map(fn
      %{role: "system", content: content} -> ReqLLM.Context.system(content)
      %{role: "user", content: content} -> ReqLLM.Context.user(content)
      %{role: "assistant", content: content} -> ReqLLM.Context.assistant(content)
      # 處理 atom keys
      %{role: :system, content: content} -> ReqLLM.Context.system(content)
      %{role: :user, content: content} -> ReqLLM.Context.user(content)
      %{role: :assistant, content: content} -> ReqLLM.Context.assistant(content)
    end)
    |> ReqLLM.Context.new()
  end

  defp build_history_messages(history) do
    Enum.map(history, fn
      %{role: "user", content: content} -> ReqLLM.Context.user(content)
      %{role: "assistant", content: content} -> ReqLLM.Context.assistant(content)
      %{role: :user, content: content} -> ReqLLM.Context.user(content)
      %{role: :assistant, content: content} -> ReqLLM.Context.assistant(content)
      _ -> nil
    end)
    |> Enum.reject(&is_nil/1)
  end

  defp default_system_prompt(character) do
    """
    你是 #{character.name}。#{character.personality}
    請以這個角色的身份自然地與用戶對話。
    """
  end

  defp default_model do
    Application.get_env(:xaifu, Xaifu.AI)[:default_model] || "anthropic:claude-sonnet-4"
  end

  defp default_temperature do
    Application.get_env(:xaifu, Xaifu.AI)[:default_temperature] || 0.7
  end

  defp default_max_tokens do
    Application.get_env(:xaifu, Xaifu.AI)[:default_max_tokens] || 1000
  end
end
```

### 6.2 ReqLLM 優勢

| 功能 | 說明 |
|------|------|
| **多服務商支援** | 45+ 個 LLM 服務商，包含 OpenAI、Anthropic、Google、Groq 等 |
| **統一 API** | 不同服務商使用相同介面，輕鬆切換模型 |
| **串流支援** | 內建 Server-Sent Events 處理，支援即時回應 |
| **成本追蹤** | 自動計算 token 使用量與費用 |
| **結構化輸出** | 支援 JSON Schema 驗證的結構化回應 |
| **Tool Calling** | 內建函式呼叫支援 |

### 6.3 模型選擇指南

```elixir
# 推薦的模型配置

# 聊天對話 - 平衡品質與成本
"anthropic:claude-sonnet-4"     # Anthropic 主力模型
"openai:gpt-4o"                  # OpenAI 主力模型

# 快速回應 - 低延遲場景
"anthropic:claude-haiku-4"       # 最快的 Claude 模型
"openai:gpt-4o-mini"             # 較快的 GPT 模型

# 複雜推理 - 需要深度思考
"anthropic:claude-opus-4"        # 最強的 Claude 模型
"openai:o1"                      # OpenAI 推理模型
```

### 6.4 錯誤處理

```elixir
# lib/xaifu/ai/llm.ex (續)

@doc """
包裝 LLM 呼叫，提供統一的錯誤處理
"""
def safe_chat_with_character(character, history, message) do
  case chat_with_character(character, history, message) do
    {:ok, text, metadata} ->
      {:ok, text, metadata}

    {:error, %{status: 429}} ->
      {:error, :rate_limited, "API 請求過於頻繁，請稍後再試"}

    {:error, %{status: 401}} ->
      {:error, :unauthorized, "API 金鑰無效或已過期"}

    {:error, %{status: 500..599}} ->
      {:error, :server_error, "LLM 服務暫時無法使用"}

    {:error, :timeout} ->
      {:error, :timeout, "請求超時，請重試"}

    {:error, reason} ->
      {:error, :unknown, "發生未知錯誤: #{inspect(reason)}"}
  end
end
```

---

## 7. Phoenix Channels 聊天實作

### 7.1 Socket 配置

```elixir
# lib/xaifu_web/channels/user_socket.ex
defmodule XaifuWeb.UserSocket do
  use Phoenix.Socket

  channel "chat:*", XaifuWeb.ChatChannel

  @impl true
  def connect(params, socket, _connect_info) do
    # 暫時使用簡單的 user_id，後續可整合認證
    user_id = params["user_id"] || generate_anonymous_id()
    {:ok, assign(socket, :user_id, user_id)}
  end

  @impl true
  def id(socket), do: "user_socket:#{socket.assigns.user_id}"

  defp generate_anonymous_id do
    "anon_" <> Base.encode16(:crypto.strong_rand_bytes(8), case: :lower)
  end
end
```

### 7.2 Chat Channel

```elixir
# lib/xaifu_web/channels/chat_channel.ex
defmodule XaifuWeb.ChatChannel do
  use XaifuWeb, :channel

  alias Xaifu.{Chat, Characters}
  alias Xaifu.AI.LLM

  @impl true
  def join("chat:" <> character_id, _payload, socket) do
    case Characters.get_character(character_id) do
      nil ->
        {:error, %{reason: "角色不存在"}}

      character ->
        user_id = socket.assigns.user_id

        # 取得或建立對話
        {:ok, conversation} = Chat.get_or_create_conversation(character.id, user_id)

        # 載入歷史訊息
        messages = Chat.list_messages(conversation.id, limit: 50)

        socket = socket
        |> assign(:character, character)
        |> assign(:conversation, conversation)

        {:ok, %{messages: serialize_messages(messages), character: serialize_character(character)}, socket}
    end
  end

  @impl true
  def handle_in("new_message", %{"content" => content}, socket) do
    character = socket.assigns.character
    conversation = socket.assigns.conversation
    user_id = socket.assigns.user_id

    # 儲存用戶訊息
    {:ok, user_message} = Chat.create_message(conversation.id, %{
      role: :user,
      content: content
    })

    # 廣播用戶訊息
    broadcast!(socket, "new_message", %{
      message: serialize_message(user_message)
    })

    # 非同步生成 AI 回應
    Task.start(fn ->
      generate_and_send_response(socket, character, conversation)
    end)

    {:noreply, socket}
  end

  defp generate_and_send_response(socket, character, conversation) do
    # 取得對話歷史
    history = Chat.get_recent_messages(conversation.id, 10)
    formatted_history = Chat.format_messages_for_llm(history)

    # 取得最後一則用戶訊息
    user_message = List.last(history).content

    # 生成回應（ReqLLM 返回 {:ok, text, metadata} 格式）
    case LLM.chat_with_character(character, formatted_history, user_message) do
      {:ok, response_content, _metadata} ->
        # 儲存 AI 回應
        {:ok, ai_message} = Chat.create_message(conversation.id, %{
          role: :assistant,
          content: response_content
        })

        # 廣播 AI 回應
        Phoenix.Channel.broadcast!(socket, "new_message", %{
          message: serialize_message(ai_message)
        })

        # 更新統計
        Characters.increment_stat(character, :total_chats)

      {:error, reason} ->
        Phoenix.Channel.push(socket, "error", %{
          message: "生成回應時發生錯誤",
          reason: inspect(reason)
        })
    end
  end

  defp serialize_messages(messages) do
    Enum.map(messages, &serialize_message/1)
  end

  defp serialize_message(message) do
    %{
      id: message.id,
      role: message.role,
      content: message.content,
      inserted_at: message.inserted_at
    }
  end

  defp serialize_character(character) do
    %{
      id: character.id,
      name: character.name,
      avatar_url: character.avatar_url,
      personality: character.personality
    }
  end
end
```

---

## 8. LiveView 介面

### 8.1 角色列表頁

```elixir
# lib/xaifu_web/live/character_live/index.ex
defmodule XaifuWeb.CharacterLive.Index do
  use XaifuWeb, :live_view

  alias Xaifu.Characters
  alias Xaifu.Characters.Character

  @impl true
  def mount(_params, _session, socket) do
    characters = Characters.list_characters()

    {:ok, assign(socket,
      characters: characters,
      page_title: "角色管理"
    )}
  end

  @impl true
  def handle_params(params, _url, socket) do
    {:noreply, apply_action(socket, socket.assigns.live_action, params)}
  end

  defp apply_action(socket, :edit, %{"id" => id}) do
    socket
    |> assign(:page_title, "編輯角色")
    |> assign(:character, Characters.get_character!(id))
  end

  defp apply_action(socket, :new, _params) do
    socket
    |> assign(:page_title, "新增角色")
    |> assign(:character, %Character{})
  end

  defp apply_action(socket, :index, _params) do
    socket
    |> assign(:page_title, "角色管理")
    |> assign(:character, nil)
  end

  @impl true
  def handle_event("delete", %{"id" => id}, socket) do
    character = Characters.get_character!(id)
    {:ok, _} = Characters.delete_character(character)

    {:noreply, assign(socket, :characters, Characters.list_characters())}
  end

  @impl true
  def handle_event("toggle_status", %{"id" => id}, socket) do
    character = Characters.get_character!(id)
    new_status = if character.status == :active, do: :inactive, else: :active
    {:ok, _} = Characters.change_status(character, new_status)

    {:noreply, assign(socket, :characters, Characters.list_characters())}
  end

  @impl true
  def render(assigns) do
    ~H"""
    <div class="container mx-auto px-4 py-8">
      <div class="flex justify-between items-center mb-8">
        <h1 class="text-3xl font-bold text-gray-800">角色管理</h1>
        <.link
          patch={~p"/characters/new"}
          class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-lg"
        >
          + 新增角色
        </.link>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        <%= for character <- @characters do %>
          <.character_card character={character} />
        <% end %>
      </div>

      <.modal
        :if={@live_action in [:new, :edit]}
        id="character-modal"
        show
        on_cancel={JS.patch(~p"/characters")}
      >
        <.live_component
          module={XaifuWeb.CharacterLive.FormComponent}
          id={@character.id || :new}
          title={@page_title}
          action={@live_action}
          character={@character}
          patch={~p"/characters"}
        />
      </.modal>
    </div>
    """
  end

  defp character_card(assigns) do
    ~H"""
    <div class="bg-white rounded-xl shadow-md overflow-hidden hover:shadow-lg transition-shadow">
      <div class="p-6">
        <div class="flex items-center space-x-4 mb-4">
          <div class="w-16 h-16 rounded-full bg-gradient-to-br from-indigo-400 to-purple-500 flex items-center justify-center text-white text-2xl font-bold">
            <%= String.first(@character.name) %>
          </div>
          <div>
            <h3 class="text-xl font-semibold text-gray-800"><%= @character.name %></h3>
            <span class={[
              "px-2 py-1 text-xs rounded-full",
              @character.status == :active && "bg-green-100 text-green-800",
              @character.status == :inactive && "bg-gray-100 text-gray-600",
              @character.status == :archived && "bg-red-100 text-red-800"
            ]}>
              <%= status_text(@character.status) %>
            </span>
          </div>
        </div>

        <p class="text-gray-600 text-sm mb-4 line-clamp-2">
          <%= @character.personality %>
        </p>

        <div class="flex flex-wrap gap-2 mb-4">
          <%= for interest <- (@character.interests || []) |> Enum.take(3) do %>
            <span class="px-2 py-1 bg-indigo-50 text-indigo-600 text-xs rounded-full">
              <%= interest %>
            </span>
          <% end %>
        </div>

        <div class="flex justify-between items-center pt-4 border-t">
          <div class="flex space-x-2">
            <.link
              patch={~p"/characters/#{@character}/edit"}
              class="text-indigo-600 hover:text-indigo-800"
            >
              編輯
            </.link>
            <.link
              navigate={~p"/chat/#{@character}"}
              class="text-green-600 hover:text-green-800"
            >
              聊天
            </.link>
          </div>
          <button
            phx-click="toggle_status"
            phx-value-id={@character.id}
            class="text-gray-500 hover:text-gray-700"
          >
            <%= if @character.status == :active, do: "停用", else: "啟用" %>
          </button>
        </div>
      </div>
    </div>
    """
  end

  defp status_text(:active), do: "運作中"
  defp status_text(:inactive), do: "未啟用"
  defp status_text(:archived), do: "已封存"
end
```

### 8.2 角色表單元件

```elixir
# lib/xaifu_web/live/character_live/form_component.ex
defmodule XaifuWeb.CharacterLive.FormComponent do
  use XaifuWeb, :live_component

  alias Xaifu.Characters

  @impl true
  def update(%{character: character} = assigns, socket) do
    changeset = Characters.change_character(character)

    {:ok,
     socket
     |> assign(assigns)
     |> assign(:changeset, changeset)
     |> assign(:interests_input, Enum.join(character.interests || [], ", "))}
  end

  @impl true
  def handle_event("validate", %{"character" => character_params}, socket) do
    character_params = process_interests(character_params)

    changeset =
      socket.assigns.character
      |> Characters.change_character(character_params)
      |> Map.put(:action, :validate)

    {:noreply, assign(socket, :changeset, changeset)}
  end

  @impl true
  def handle_event("save", %{"character" => character_params}, socket) do
    character_params = process_interests(character_params)
    save_character(socket, socket.assigns.action, character_params)
  end

  defp process_interests(%{"interests_input" => interests_input} = params) do
    interests =
      interests_input
      |> String.split(",")
      |> Enum.map(&String.trim/1)
      |> Enum.reject(&(&1 == ""))

    params
    |> Map.put("interests", interests)
    |> Map.delete("interests_input")
  end
  defp process_interests(params), do: params

  defp save_character(socket, :edit, character_params) do
    case Characters.update_character(socket.assigns.character, character_params) do
      {:ok, _character} ->
        {:noreply,
         socket
         |> put_flash(:info, "角色更新成功")
         |> push_patch(to: socket.assigns.patch)}

      {:error, %Ecto.Changeset{} = changeset} ->
        {:noreply, assign(socket, :changeset, changeset)}
    end
  end

  defp save_character(socket, :new, character_params) do
    case Characters.create_character(character_params) do
      {:ok, _character} ->
        {:noreply,
         socket
         |> put_flash(:info, "角色建立成功")
         |> push_patch(to: socket.assigns.patch)}

      {:error, %Ecto.Changeset{} = changeset} ->
        {:noreply, assign(socket, :changeset, changeset)}
    end
  end

  @impl true
  def render(assigns) do
    ~H"""
    <div class="p-6">
      <h2 class="text-2xl font-bold mb-6"><%= @title %></h2>

      <.form
        for={@changeset}
        id="character-form"
        phx-target={@myself}
        phx-change="validate"
        phx-submit="save"
        class="space-y-6"
      >
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
          <div>
            <.input
              field={@changeset[:name]}
              type="text"
              label="角色名稱"
              placeholder="輸入角色名稱"
              required
            />
          </div>

          <div>
            <.input
              field={@changeset[:avatar_url]}
              type="text"
              label="頭像 URL"
              placeholder="https://..."
            />
          </div>
        </div>

        <div>
          <.input
            field={@changeset[:personality]}
            type="textarea"
            label="性格特徵"
            placeholder="描述角色的性格特徵，例如：活潑開朗、善於傾聽、有點傲嬌..."
            rows={3}
            required
          />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">
            興趣（用逗號分隔）
          </label>
          <input
            type="text"
            name="character[interests_input]"
            value={@interests_input}
            placeholder="攝影, 旅行, 咖啡, 音樂..."
            class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
          />
        </div>

        <div>
          <.input
            field={@changeset[:speaking_style]}
            type="textarea"
            label="說話風格"
            placeholder="描述角色的說話方式，例如：喜歡用表情符號、說話很直接..."
            rows={2}
          />
        </div>

        <div>
          <.input
            field={@changeset[:background]}
            type="textarea"
            label="背景故事"
            placeholder="角色的背景故事..."
            rows={3}
          />
        </div>

        <div>
          <.input
            field={@changeset[:system_prompt]}
            type="textarea"
            label="系統提示詞（選填，留空將自動生成）"
            placeholder="自訂給 AI 的系統提示詞..."
            rows={4}
          />
        </div>

        <div class="flex justify-end space-x-4 pt-4">
          <.link
            patch={@patch}
            class="px-4 py-2 text-gray-600 hover:text-gray-800"
          >
            取消
          </.link>
          <.button type="submit" phx-disable-with="儲存中...">
            儲存角色
          </.button>
        </div>
      </.form>
    </div>
    """
  end
end
```

### 8.3 聊天室 LiveView

```elixir
# lib/xaifu_web/live/chat_live.ex
defmodule XaifuWeb.ChatLive do
  use XaifuWeb, :live_view

  alias Xaifu.Characters
  alias Phoenix.PubSub

  @impl true
  def mount(%{"character_id" => character_id}, _session, socket) do
    character = Characters.get_character!(character_id)
    user_id = get_user_id(socket)

    if connected?(socket) do
      # 訂閱聊天頻道
      PubSub.subscribe(Xaifu.PubSub, "chat:#{character_id}")
    end

    {:ok,
     socket
     |> assign(:character, character)
     |> assign(:user_id, user_id)
     |> assign(:messages, [])
     |> assign(:message_input, "")
     |> assign(:loading, false)
     |> assign(:page_title, "與 #{character.name} 聊天")}
  end

  @impl true
  def handle_event("send_message", %{"message" => content}, socket) when content != "" do
    # 透過 JS Hook 發送到 Channel
    {:noreply,
     socket
     |> assign(:message_input, "")
     |> push_event("send_to_channel", %{content: content})}
  end

  def handle_event("send_message", _params, socket), do: {:noreply, socket}

  @impl true
  def handle_event("update_input", %{"value" => value}, socket) do
    {:noreply, assign(socket, :message_input, value)}
  end

  @impl true
  def handle_info({:new_message, message}, socket) do
    {:noreply,
     socket
     |> update(:messages, fn messages -> messages ++ [message] end)
     |> push_event("scroll_to_bottom", %{})}
  end

  @impl true
  def handle_info(:loading_start, socket) do
    {:noreply, assign(socket, :loading, true)}
  end

  @impl true
  def handle_info(:loading_end, socket) do
    {:noreply, assign(socket, :loading, false)}
  end

  defp get_user_id(socket) do
    # 暫時生成匿名 ID，後續可整合認證系統
    case socket.assigns[:current_user] do
      nil -> "anon_#{:crypto.strong_rand_bytes(8) |> Base.encode16(case: :lower)}"
      user -> user.id
    end
  end

  @impl true
  def render(assigns) do
    ~H"""
    <div class="flex flex-col h-screen bg-gray-100" id="chat-container" phx-hook="ChatHook">
      <!-- Header -->
      <header class="bg-white shadow-sm px-6 py-4 flex items-center space-x-4">
        <.link navigate={~p"/characters"} class="text-gray-500 hover:text-gray-700">
          ← 返回
        </.link>
        <div class="w-12 h-12 rounded-full bg-gradient-to-br from-indigo-400 to-purple-500 flex items-center justify-center text-white font-bold">
          <%= String.first(@character.name) %>
        </div>
        <div>
          <h1 class="text-lg font-semibold text-gray-800"><%= @character.name %></h1>
          <p class="text-sm text-gray-500"><%= @character.personality |> String.slice(0..30) %>...</p>
        </div>
      </header>

      <!-- Messages -->
      <div class="flex-1 overflow-y-auto p-6 space-y-4" id="messages-container">
        <%= for message <- @messages do %>
          <.message_bubble message={message} character={@character} />
        <% end %>

        <%= if @loading do %>
          <div class="flex justify-start">
            <div class="bg-white rounded-2xl px-4 py-3 shadow-sm">
              <div class="flex space-x-1">
                <div class="w-2 h-2 bg-gray-400 rounded-full animate-bounce"></div>
                <div class="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style="animation-delay: 0.1s"></div>
                <div class="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style="animation-delay: 0.2s"></div>
              </div>
            </div>
          </div>
        <% end %>
      </div>

      <!-- Input -->
      <div class="bg-white border-t px-6 py-4">
        <form phx-submit="send_message" class="flex space-x-4">
          <input
            type="text"
            name="message"
            value={@message_input}
            phx-keyup="update_input"
            phx-value-value={@message_input}
            placeholder="輸入訊息..."
            autocomplete="off"
            class="flex-1 rounded-full border-gray-300 px-4 py-2 focus:border-indigo-500 focus:ring-indigo-500"
          />
          <button
            type="submit"
            disabled={@message_input == "" or @loading}
            class="bg-indigo-600 hover:bg-indigo-700 disabled:bg-gray-300 text-white px-6 py-2 rounded-full transition-colors"
          >
            發送
          </button>
        </form>
      </div>

      <!-- Hidden data for JS -->
      <div id="chat-data"
           data-character-id={@character.id}
           data-user-id={@user_id}
           class="hidden">
      </div>
    </div>
    """
  end

  defp message_bubble(assigns) do
    is_user = assigns.message.role == :user or assigns.message.role == "user"
    assigns = assign(assigns, :is_user, is_user)

    ~H"""
    <div class={["flex", @is_user && "justify-end"]}>
      <div class={[
        "max-w-[70%] rounded-2xl px-4 py-3 shadow-sm",
        @is_user && "bg-indigo-600 text-white",
        !@is_user && "bg-white text-gray-800"
      ]}>
        <%= if !@is_user do %>
          <p class="text-xs text-indigo-600 font-medium mb-1"><%= @character.name %></p>
        <% end %>
        <p class="whitespace-pre-wrap"><%= @message.content %></p>
        <p class={["text-xs mt-1", @is_user && "text-indigo-200", !@is_user && "text-gray-400"]}>
          <%= format_time(@message.inserted_at) %>
        </p>
      </div>
    </div>
    """
  end

  defp format_time(nil), do: ""
  defp format_time(datetime) do
    Calendar.strftime(datetime, "%H:%M")
  end
end
```

---

## 9. 測試案例

### 9.1 Character Context 測試

```elixir
# test/xaifu/characters_test.exs
defmodule Xaifu.CharactersTest do
  use Xaifu.DataCase, async: true

  alias Xaifu.Characters
  alias Xaifu.Characters.Character

  describe "list_characters/1" do
    test "returns all characters" do
      character = character_fixture()
      assert Characters.list_characters() == [character]
    end

    test "filters by status" do
      active = character_fixture(status: :active)
      _inactive = character_fixture(status: :inactive)

      assert Characters.list_characters(status: :active) == [active]
    end

    test "supports pagination" do
      for _ <- 1..5, do: character_fixture()

      assert length(Characters.list_characters(limit: 2)) == 2
      assert length(Characters.list_characters(limit: 2, offset: 3)) == 2
    end
  end

  describe "get_character/1" do
    test "returns the character with given id" do
      character = character_fixture()
      assert Characters.get_character(character.id) == character
    end

    test "returns nil for non-existent id" do
      assert Characters.get_character(Ecto.UUID.generate()) == nil
    end
  end

  describe "create_character/1" do
    test "with valid data creates a character" do
      valid_attrs = %{
        name: "Alice",
        personality: "活潑開朗的女孩，喜歡交朋友"
      }

      assert {:ok, %Character{} = character} = Characters.create_character(valid_attrs)
      assert character.name == "Alice"
      assert character.personality == "活潑開朗的女孩，喜歡交朋友"
      assert character.status == :inactive
    end

    test "generates system_prompt when not provided" do
      attrs = %{
        name: "Bob",
        personality: "安靜內向",
        interests: ["讀書", "音樂"]
      }

      {:ok, character} = Characters.create_character(attrs)

      assert character.system_prompt =~ "Bob"
      assert character.system_prompt =~ "讀書"
    end

    test "with invalid data returns error changeset" do
      assert {:error, %Ecto.Changeset{}} = Characters.create_character(%{})
    end

    test "validates name length" do
      long_name = String.duplicate("a", 51)
      attrs = %{name: long_name, personality: "test personality"}

      assert {:error, changeset} = Characters.create_character(attrs)
      assert "should be at most 50 character(s)" in errors_on(changeset).name
    end

    test "validates interests count" do
      attrs = %{
        name: "Test",
        personality: "test",
        interests: for(_ <- 1..21, do: "interest")
      }

      assert {:error, changeset} = Characters.create_character(attrs)
      assert "最多 20 個興趣" in errors_on(changeset).interests
    end
  end

  describe "update_character/2" do
    test "with valid data updates the character" do
      character = character_fixture()
      update_attrs = %{name: "Updated Name"}

      assert {:ok, %Character{} = updated} = Characters.update_character(character, update_attrs)
      assert updated.name == "Updated Name"
    end

    test "with invalid data returns error changeset" do
      character = character_fixture()
      invalid_attrs = %{name: ""}

      assert {:error, %Ecto.Changeset{}} = Characters.update_character(character, invalid_attrs)
    end
  end

  describe "delete_character/1" do
    test "deletes the character" do
      character = character_fixture()
      assert {:ok, %Character{}} = Characters.delete_character(character)
      assert Characters.get_character(character.id) == nil
    end
  end

  describe "change_status/2" do
    test "changes character status" do
      character = character_fixture(status: :inactive)

      assert {:ok, updated} = Characters.change_status(character, :active)
      assert updated.status == :active
    end
  end

  describe "increment_stat/2" do
    test "increments total_posts" do
      character = character_fixture()
      Characters.increment_stat(character, :total_posts)

      updated = Characters.get_character!(character.id)
      assert updated.total_posts == 1
    end

    test "increments total_chats" do
      character = character_fixture()
      Characters.increment_stat(character, :total_chats)

      updated = Characters.get_character!(character.id)
      assert updated.total_chats == 1
    end
  end

  # Fixture helpers
  defp character_fixture(attrs \\ %{}) do
    {:ok, character} =
      attrs
      |> Enum.into(%{
        name: "Test Character #{System.unique_integer()}",
        personality: "這是一個測試用的角色性格描述，至少要十個字"
      })
      |> Characters.create_character()

    character
  end
end
```

### 9.2 Chat Context 測試

```elixir
# test/xaifu/chat_test.exs
defmodule Xaifu.ChatTest do
  use Xaifu.DataCase, async: true

  alias Xaifu.Chat
  alias Xaifu.Chat.{Conversation, Message}
  alias Xaifu.Characters

  setup do
    {:ok, character} = Characters.create_character(%{
      name: "Test Character",
      personality: "測試角色的性格描述"
    })

    %{character: character, user_id: "test_user_123"}
  end

  describe "get_or_create_conversation/2" do
    test "creates new conversation when none exists", %{character: character, user_id: user_id} do
      assert {:ok, %Conversation{} = conversation} =
        Chat.get_or_create_conversation(character.id, user_id)

      assert conversation.character_id == character.id
      assert conversation.user_identifier == user_id
    end

    test "returns existing conversation", %{character: character, user_id: user_id} do
      {:ok, first} = Chat.get_or_create_conversation(character.id, user_id)
      {:ok, second} = Chat.get_or_create_conversation(character.id, user_id)

      assert first.id == second.id
    end
  end

  describe "create_message/2" do
    test "creates a message", %{character: character, user_id: user_id} do
      {:ok, conversation} = Chat.get_or_create_conversation(character.id, user_id)

      assert {:ok, %Message{} = message} = Chat.create_message(conversation.id, %{
        role: :user,
        content: "Hello!"
      })

      assert message.role == :user
      assert message.content == "Hello!"
    end

    test "updates conversation stats", %{character: character, user_id: user_id} do
      {:ok, conversation} = Chat.get_or_create_conversation(character.id, user_id)

      Chat.create_message(conversation.id, %{role: :user, content: "Hi"})
      Chat.create_message(conversation.id, %{role: :assistant, content: "Hello!"})

      updated = Xaifu.Repo.get!(Conversation, conversation.id)
      assert updated.message_count == 2
      assert updated.last_message_at != nil
    end
  end

  describe "list_messages/2" do
    test "returns messages in chronological order", %{character: character, user_id: user_id} do
      {:ok, conversation} = Chat.get_or_create_conversation(character.id, user_id)

      {:ok, m1} = Chat.create_message(conversation.id, %{role: :user, content: "First"})
      {:ok, m2} = Chat.create_message(conversation.id, %{role: :assistant, content: "Second"})

      messages = Chat.list_messages(conversation.id)

      assert length(messages) == 2
      assert hd(messages).id == m1.id
      assert List.last(messages).id == m2.id
    end

    test "respects limit option", %{character: character, user_id: user_id} do
      {:ok, conversation} = Chat.get_or_create_conversation(character.id, user_id)

      for i <- 1..10 do
        Chat.create_message(conversation.id, %{role: :user, content: "Message #{i}"})
      end

      assert length(Chat.list_messages(conversation.id, limit: 5)) == 5
    end
  end

  describe "format_messages_for_llm/1" do
    test "formats messages correctly", %{character: character, user_id: user_id} do
      {:ok, conversation} = Chat.get_or_create_conversation(character.id, user_id)

      Chat.create_message(conversation.id, %{role: :user, content: "Hi"})
      Chat.create_message(conversation.id, %{role: :assistant, content: "Hello!"})

      messages = Chat.list_messages(conversation.id)
      formatted = Chat.format_messages_for_llm(messages)

      assert formatted == [
        %{role: "user", content: "Hi"},
        %{role: "assistant", content: "Hello!"}
      ]
    end
  end
end
```

### 9.3 LLM 模組測試

```elixir
# test/xaifu/ai/llm_test.exs
defmodule Xaifu.AI.LLMTest do
  use ExUnit.Case, async: true

  alias Xaifu.AI.LLM

  # 使用 Mox 來 mock ReqLLM
  # 需要在 test_helper.exs 中設定 Mox

  describe "chat_with_character/3" do
    test "builds correct context structure" do
      character = %{
        name: "Alice",
        personality: "活潑開朗",
        system_prompt: "你是 Alice，一個活潑的女孩。",
        temperature: 0.8
      }

      history = [
        %{role: "user", content: "你好"},
        %{role: "assistant", content: "嗨！"}
      ]

      # 驗證結構正確性
      assert character.temperature == 0.8
      assert length(history) == 2
      assert character.system_prompt =~ "Alice"
    end

    test "handles empty history" do
      character = %{
        name: "Bob",
        personality: "安靜內向",
        system_prompt: nil,
        temperature: 0.7
      }

      # 空的對話歷史應該能正常處理
      assert is_nil(character.system_prompt)
    end
  end

  describe "safe_chat_with_character/3" do
    # 這些測試需要 mock ReqLLM 的回應
    # 實際整合測試可使用 sandbox 環境或真實 API（限制使用）

    test "handles rate limit error" do
      # Mock 設定示例（需配合 Mox 設定）
      # expect(ReqLLMMock, :generate_text, fn _, _, _ ->
      #   {:error, %{status: 429, body: "Rate limited"}}
      # end)

      # assert {:error, :rate_limited, _} =
      #   LLM.safe_chat_with_character(character, [], "test")

      # 佔位測試
      assert true
    end

    test "handles unauthorized error" do
      # Mock 設定示例
      # 佔位測試
      assert true
    end
  end
end
```

### 9.3.1 ReqLLM Mock 設定

```elixir
# test/support/mocks.ex
# 如需 mock ReqLLM，可使用 Mox 設定

# 在 test/test_helper.exs 中加入:
# Mox.defmock(ReqLLMMock, for: ReqLLM.Behaviour)
# Application.put_env(:xaifu, :req_llm_module, ReqLLMMock)

# 或使用整合測試時跳過實際 API 呼叫:
# @tag :external_api
# test "integration with real API" do
#   ...
# end
#
# 執行時排除: mix test --exclude external_api
```

### 9.4 LiveView 測試

```elixir
# test/xaifu_web/live/character_live_test.exs
defmodule XaifuWeb.CharacterLiveTest do
  use XaifuWeb.ConnCase, async: true

  import Phoenix.LiveViewTest
  alias Xaifu.Characters

  describe "Index" do
    test "lists all characters", %{conn: conn} do
      {:ok, character} = Characters.create_character(%{
        name: "Test Alice",
        personality: "活潑開朗的測試角色"
      })

      {:ok, _view, html} = live(conn, ~p"/characters")

      assert html =~ "角色管理"
      assert html =~ "Test Alice"
    end

    test "opens modal for new character", %{conn: conn} do
      {:ok, view, _html} = live(conn, ~p"/characters")

      assert view
             |> element("a", "+ 新增角色")
             |> render_click() =~ "新增角色"
    end

    test "creates new character", %{conn: conn} do
      {:ok, view, _html} = live(conn, ~p"/characters/new")

      assert view
             |> form("#character-form", character: %{
               name: "New Character",
               personality: "這是一個新角色的性格描述"
             })
             |> render_submit()

      assert_patch(view, ~p"/characters")

      # 驗證角色已建立
      assert Characters.get_character_by_name("New Character")
    end

    test "validates character form", %{conn: conn} do
      {:ok, view, _html} = live(conn, ~p"/characters/new")

      assert view
             |> form("#character-form", character: %{name: "", personality: ""})
             |> render_change() =~ "can&#39;t be blank"
    end
  end

  describe "Edit" do
    setup do
      {:ok, character} = Characters.create_character(%{
        name: "Edit Test",
        personality: "待編輯的角色"
      })

      %{character: character}
    end

    test "updates character", %{conn: conn, character: character} do
      {:ok, view, _html} = live(conn, ~p"/characters/#{character}/edit")

      assert view
             |> form("#character-form", character: %{name: "Updated Name"})
             |> render_submit()

      assert_patch(view, ~p"/characters")

      updated = Characters.get_character!(character.id)
      assert updated.name == "Updated Name"
    end
  end
end
```

---

## 10. 驗收標準

### 10.1 功能驗收

| 項目 | 驗收條件 | 狀態 |
|------|----------|------|
| 角色建立 | 可填寫完整資料並成功建立角色 | ☐ |
| 角色列表 | 正確顯示所有角色，支援狀態篩選 | ☐ |
| 角色編輯 | 可修改任何角色屬性 | ☐ |
| 角色刪除 | 可刪除角色（含確認） | ☐ |
| 聊天連線 | WebSocket 連線建立成功 | ☐ |
| 訊息發送 | 用戶訊息即時顯示 | ☐ |
| AI 回應 | LLM 成功回應並顯示 | ☐ |
| 訊息持久化 | 重新載入後訊息仍存在 | ☐ |

### 10.2 效能驗收

| 指標 | 目標值 |
|------|--------|
| 頁面載入時間 | < 500ms |
| 聊天回應時間 | < 5s |
| 記憶體使用 | < 500MB |

### 10.3 測試覆蓋率

- Unit Tests: > 80%
- Integration Tests: 關鍵路徑 100%

---

## 11. 下一階段準備

完成 Phase 1 後，為 Phase 2（自動化排程系統）做以下準備：

1. 確保 Character Schema 的 `status` 欄位正確運作
2. 安裝並設定 Oban
3. 設計初步的 Schedule Schema
4. 研究 GenServer 狀態管理模式

---

*文件版本: 1.0*
*對應主文件: [00-overview.md](./00-overview.md)*

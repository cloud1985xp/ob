# 附錄 A：資料庫設計

## 完整 ER 圖

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Xaifu Database Schema                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐         ┌─────────────────┐                           │
│  │    characters   │         │     users       │  (未來擴展)               │
│  ├─────────────────┤         ├─────────────────┤                           │
│  │ id (PK)         │         │ id (PK)         │                           │
│  │ name            │         │ email           │                           │
│  │ avatar_url      │         │ username        │                           │
│  │ bio             │         │ password_hash   │                           │
│  │ personality     │         │ ...             │                           │
│  │ interests[]     │         └────────┬────────┘                           │
│  │ speaking_style  │                  │                                    │
│  │ background      │                  │                                    │
│  │ system_prompt   │                  │                                    │
│  │ temperature     │                  │                                    │
│  │ appearance      │                  │                                    │
│  │ image_prompt_   │                  │                                    │
│  │   prefix        │                  │                                    │
│  │ status          │                  │                                    │
│  │ total_posts     │                  │                                    │
│  │ total_chats     │                  │                                    │
│  │ timestamps      │                  │                                    │
│  └────────┬────────┘                  │                                    │
│           │                           │                                    │
│           │ 1                         │                                    │
│           │                           │                                    │
│    ┌──────┴───────────────────────────┴──────┐                            │
│    │                                          │                            │
│    ▼ N                                        ▼ N                          │
│  ┌─────────────────┐                 ┌─────────────────┐                   │
│  │    schedules    │                 │  conversations  │                   │
│  ├─────────────────┤                 ├─────────────────┤                   │
│  │ id (PK)         │                 │ id (PK)         │                   │
│  │ character_id    │◄────────────────│ character_id    │                   │
│  │   (FK)          │                 │   (FK)          │                   │
│  │ day_of_week     │                 │ user_identifier │                   │
│  │ time_slot       │                 │ title           │                   │
│  │ hour            │                 │ last_message_at │                   │
│  │ activity_type   │                 │ message_count   │                   │
│  │ location        │                 │ timestamps      │                   │
│  │ description     │                 └────────┬────────┘                   │
│  │ post_probability│                          │                            │
│  │ is_active       │                          │ 1                          │
│  │ timestamps      │                          │                            │
│  └────────┬────────┘                          ▼ N                          │
│           │                          ┌─────────────────┐                   │
│           │ 1                        │    messages     │                   │
│           │                          ├─────────────────┤                   │
│           ▼ N                        │ id (PK)         │                   │
│  ┌─────────────────┐                 │ conversation_id │                   │
│  │   activities    │                 │   (FK)          │                   │
│  ├─────────────────┤                 │ role            │                   │
│  │ id (PK)         │                 │ content         │                   │
│  │ character_id    │                 │ tokens_used     │                   │
│  │   (FK)          │                 │ metadata{}      │                   │
│  │ schedule_id     │                 │ timestamps      │                   │
│  │   (FK)          │                 └─────────────────┘                   │
│  │ activity_type   │                                                       │
│  │ location        │                                                       │
│  │ description     │         ┌─────────────────┐                           │
│  │ started_at      │         │     posts       │                           │
│  │ ended_at        │         ├─────────────────┤                           │
│  │ inner_thought   │         │ id (PK)         │                           │
│  │ posted          │         │ character_id    │◄─────────────────┐       │
│  │ timestamps      │         │   (FK)          │                  │       │
│  └────────┬────────┘         │ activity_id     │                  │       │
│           │                  │   (FK)          │                  │       │
│           │                  │ content         │                  │       │
│           └──────────────────│ image_url       │                  │       │
│                              │ image_prompt    │                  │       │
│                              │ mood            │                  │       │
│                              │ location        │                  │       │
│                              │ likes_count     │                  │       │
│                              │ comments_count  │                  │       │
│                              │ visibility      │                  │       │
│                              │ timestamps      │                  │       │
│                              └────────┬────────┘                  │       │
│                                       │                           │       │
│                         ┌─────────────┼─────────────┐            │       │
│                         │ 1           │ 1           │            │       │
│                         │             │             │            │       │
│                         ▼ N           ▼ N           │            │       │
│                ┌─────────────────┐   ┌─────────────────┐         │       │
│                │    comments     │   │     likes       │         │       │
│                ├─────────────────┤   ├─────────────────┤         │       │
│                │ id (PK)         │   │ id (PK)         │         │       │
│                │ post_id (FK)    │   │ post_id (FK)    │         │       │
│                │ character_id    │   │ character_id    │─────────┘       │
│                │   (FK, nullable)│   │   (FK, nullable)│                 │
│                │ user_identifier │   │ user_identifier │                 │
│                │ content         │   │ timestamps      │                 │
│                │ is_ai_generated │   └─────────────────┘                 │
│                │ timestamps      │                                       │
│                └─────────────────┘                                       │
│                                                                          │
│  ┌─────────────────┐                 ┌─────────────────┐                 │
│  │   memories      │                 │  relationships  │                 │
│  ├─────────────────┤                 ├─────────────────┤                 │
│  │ id (PK)         │                 │ id (PK)         │                 │
│  │ character_id    │◄────────────────│ character_a_id  │◄────────────────┤
│  │   (FK)          │                 │   (FK)          │                 │
│  │ user_identifier │                 │ character_b_id  │◄────────────────┘
│  │ content         │                 │   (FK)          │
│  │ embedding       │                 │ affinity        │
│  │   (vector)      │                 │ familiarity     │
│  │ memory_type     │                 │ interaction_    │
│  │ importance      │                 │   count         │
│  │ metadata{}      │                 │ last_inter-     │
│  │ accessed_at     │                 │   action_at     │
│  │ access_count    │                 │ timestamps      │
│  │ timestamps      │                 └─────────────────┘
│  └─────────────────┘
│
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 完整遷移腳本

### 1. Characters Table

```elixir
# priv/repo/migrations/20260203000001_create_characters.exs
defmodule Xaifu.Repo.Migrations.CreateCharacters do
  use Ecto.Migration

  def change do
    create table(:characters, primary_key: false) do
      add :id, :binary_id, primary_key: true

      # 基本資料
      add :name, :string, null: false, size: 50
      add :avatar_url, :string, size: 500
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
      add :status, :string, default: "inactive", null: false

      # 統計
      add :total_posts, :integer, default: 0
      add :total_chats, :integer, default: 0

      timestamps(type: :utc_datetime)
    end

    create index(:characters, [:status])
    create index(:characters, [:name])
    create index(:characters, [:inserted_at])
  end
end
```

### 2. Conversations & Messages Tables

```elixir
# priv/repo/migrations/20260203000002_create_conversations_and_messages.exs
defmodule Xaifu.Repo.Migrations.CreateConversationsAndMessages do
  use Ecto.Migration

  def change do
    # Conversations
    create table(:conversations, primary_key: false) do
      add :id, :binary_id, primary_key: true
      add :character_id, references(:characters, type: :binary_id, on_delete: :delete_all),
        null: false
      add :user_identifier, :string, null: false, size: 100
      add :title, :string, size: 200
      add :last_message_at, :utc_datetime
      add :message_count, :integer, default: 0

      timestamps(type: :utc_datetime)
    end

    create index(:conversations, [:character_id])
    create index(:conversations, [:user_identifier])
    create unique_index(:conversations, [:character_id, :user_identifier])

    # Messages
    create table(:messages, primary_key: false) do
      add :id, :binary_id, primary_key: true
      add :conversation_id, references(:conversations, type: :binary_id, on_delete: :delete_all),
        null: false
      add :role, :string, null: false  # user, assistant, system
      add :content, :text, null: false
      add :tokens_used, :integer
      add :metadata, :map, default: %{}

      timestamps(type: :utc_datetime)
    end

    create index(:messages, [:conversation_id])
    create index(:messages, [:conversation_id, :inserted_at])
  end
end
```

### 3. Schedules & Activities Tables

```elixir
# priv/repo/migrations/20260210000001_create_schedules_and_activities.exs
defmodule Xaifu.Repo.Migrations.CreateSchedulesAndActivities do
  use Ecto.Migration

  def change do
    # Schedules
    create table(:schedules, primary_key: false) do
      add :id, :binary_id, primary_key: true
      add :character_id, references(:characters, type: :binary_id, on_delete: :delete_all),
        null: false

      add :day_of_week, :integer  # 0-6, nil = 每天
      add :time_slot, :string     # morning, afternoon, evening, night
      add :hour, :integer         # 0-23

      add :activity_type, :string, null: false
      add :location, :string, size: 100
      add :description, :text

      add :post_probability, :float, default: 0.5
      add :is_active, :boolean, default: true

      timestamps(type: :utc_datetime)
    end

    create index(:schedules, [:character_id])
    create index(:schedules, [:character_id, :day_of_week, :hour])
    create index(:schedules, [:character_id, :time_slot])
    create index(:schedules, [:is_active])

    # Activities
    create table(:activities, primary_key: false) do
      add :id, :binary_id, primary_key: true
      add :character_id, references(:characters, type: :binary_id, on_delete: :delete_all),
        null: false
      add :schedule_id, references(:schedules, type: :binary_id, on_delete: :nilify_all)

      add :activity_type, :string, null: false
      add :location, :string, size: 100
      add :description, :text
      add :started_at, :utc_datetime, null: false
      add :ended_at, :utc_datetime
      add :inner_thought, :text
      add :posted, :boolean, default: false

      timestamps(type: :utc_datetime)
    end

    create index(:activities, [:character_id])
    create index(:activities, [:character_id, :started_at])
    create index(:activities, [:posted])
  end
end
```

### 4. Posts, Comments & Likes Tables

```elixir
# priv/repo/migrations/20260210000002_create_social_tables.exs
defmodule Xaifu.Repo.Migrations.CreateSocialTables do
  use Ecto.Migration

  def change do
    # Posts
    create table(:posts, primary_key: false) do
      add :id, :binary_id, primary_key: true
      add :character_id, references(:characters, type: :binary_id, on_delete: :delete_all),
        null: false
      add :activity_id, references(:activities, type: :binary_id, on_delete: :nilify_all)

      add :content, :text, null: false
      add :image_url, :string, size: 500
      add :image_prompt, :text

      add :mood, :string, size: 50
      add :location, :string, size: 100

      add :likes_count, :integer, default: 0
      add :comments_count, :integer, default: 0
      add :visibility, :string, default: "public"

      timestamps(type: :utc_datetime)
    end

    create index(:posts, [:character_id])
    create index(:posts, [:inserted_at])
    create index(:posts, [:character_id, :inserted_at])
    create index(:posts, [:visibility])

    # Comments
    create table(:comments, primary_key: false) do
      add :id, :binary_id, primary_key: true
      add :post_id, references(:posts, type: :binary_id, on_delete: :delete_all),
        null: false
      add :character_id, references(:characters, type: :binary_id, on_delete: :delete_all)
      add :user_identifier, :string, size: 100

      add :content, :text, null: false
      add :is_ai_generated, :boolean, default: false

      timestamps(type: :utc_datetime)
    end

    create index(:comments, [:post_id])
    create index(:comments, [:character_id])
    create index(:comments, [:post_id, :inserted_at])

    # Likes
    create table(:likes, primary_key: false) do
      add :id, :binary_id, primary_key: true
      add :post_id, references(:posts, type: :binary_id, on_delete: :delete_all),
        null: false
      add :character_id, references(:characters, type: :binary_id, on_delete: :delete_all)
      add :user_identifier, :string, size: 100

      timestamps(type: :utc_datetime)
    end

    create index(:likes, [:post_id])
    create index(:likes, [:character_id])
    create unique_index(:likes, [:post_id, :character_id],
      where: "character_id IS NOT NULL",
      name: :likes_post_character_unique)
    create unique_index(:likes, [:post_id, :user_identifier],
      where: "user_identifier IS NOT NULL",
      name: :likes_post_user_unique)
  end
end
```

### 5. Memories Table (with pgvector)

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

# priv/repo/migrations/20260224000002_create_memories.exs
defmodule Xaifu.Repo.Migrations.CreateMemories do
  use Ecto.Migration

  def change do
    create table(:memories, primary_key: false) do
      add :id, :binary_id, primary_key: true
      add :character_id, references(:characters, type: :binary_id, on_delete: :delete_all),
        null: false
      add :user_identifier, :string, size: 100

      add :content, :text, null: false
      add :embedding, :vector, size: 1536  # OpenAI embedding dimension
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
    create index(:memories, [:accessed_at])

    # 向量相似度搜尋索引 (IVFFlat)
    execute """
    CREATE INDEX memories_embedding_idx ON memories
    USING ivfflat (embedding vector_cosine_ops)
    WITH (lists = 100)
    """
  end
end
```

### 6. Relationships Table

```elixir
# priv/repo/migrations/20260224000003_create_relationships.exs
defmodule Xaifu.Repo.Migrations.CreateRelationships do
  use Ecto.Migration

  def change do
    create table(:relationships, primary_key: false) do
      add :id, :binary_id, primary_key: true
      add :character_a_id, references(:characters, type: :binary_id, on_delete: :delete_all),
        null: false
      add :character_b_id, references(:characters, type: :binary_id, on_delete: :delete_all),
        null: false

      add :affinity, :integer, default: 50      # -100 到 +100
      add :familiarity, :integer, default: 0    # 0 到 100
      add :interaction_count, :integer, default: 0
      add :last_interaction_at, :utc_datetime

      timestamps(type: :utc_datetime)
    end

    create index(:relationships, [:character_a_id])
    create index(:relationships, [:character_b_id])
    create unique_index(:relationships, [:character_a_id, :character_b_id])
  end
end
```

---

## 索引策略

### 讀取優化索引

| 表格 | 索引 | 用途 |
|------|------|------|
| characters | status | 查詢活躍角色 |
| posts | inserted_at DESC | 動態牆排序 |
| posts | (character_id, inserted_at) | 角色貼文列表 |
| messages | (conversation_id, inserted_at) | 對話歷史 |
| memories | embedding (ivfflat) | 向量相似搜尋 |

### 寫入考量

- 使用 `binary_id` (UUID) 作為主鍵，支援分散式
- 外鍵使用 `on_delete` 策略自動清理
- 統計欄位使用 `Repo.update_all` 原子更新

---

## 資料保留策略

| 資料類型 | 保留期限 | 清理策略 |
|----------|----------|----------|
| 短期記憶 | 7 天 | 定時清理低重要性 |
| 長期記憶 | 永久 | 達上限時刪除最舊 |
| 活動記錄 | 90 天 | 定時清理 |
| 對話訊息 | 180 天 | 定時清理 |
| 貼文 | 永久 | 手動封存 |

---

*文件版本: 1.0*
*對應主文件: [00-overview.md](./00-overview.md)*

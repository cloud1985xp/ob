# Phase 2: 自動化排程系統

## 預估工時：2-3 週（單人開發）

---

## 1. 階段目標

建立角色的自主運作能力，讓每個角色成為獨立運作的「數位生命」：
- GenServer 角色代理架構
- 日程表系統
- Oban 任務排程
- 社群動態牆
- 純文字動態發佈

**里程碑**：角色按照預定日程自動發佈文字動態

---

## 2. 架構設計

### 2.1 角色代理系統架構

```
┌─────────────────────────────────────────────────────────────┐
│                   Agent System Architecture                  │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Xaifu.Agents.Supervisor                 │   │
│  │              (DynamicSupervisor)                     │   │
│  └─────────────────────┬───────────────────────────────┘   │
│                        │                                    │
│         ┌──────────────┼──────────────┐                    │
│         ▼              ▼              ▼                    │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐             │
│  │   Agent    │ │   Agent    │ │   Agent    │             │
│  │  (Alice)   │ │   (Bob)    │ │  (Carol)   │             │
│  │            │ │            │ │            │             │
│  │ State:     │ │ State:     │ │ State:     │             │
│  │ - mood     │ │ - mood     │ │ - mood     │             │
│  │ - energy   │ │ - energy   │ │ - energy   │             │
│  │ - activity │ │ - activity │ │ - activity │             │
│  │ - location │ │ - location │ │ - location │             │
│  └─────┬──────┘ └─────┬──────┘ └─────┬──────┘             │
│        │              │              │                     │
│        └──────────────┼──────────────┘                     │
│                       ▼                                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Xaifu.Agents.Registry                   │   │
│  │         (via Registry + :via tuple)                  │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Xaifu.Agents.Scheduler                  │   │
│  │         (定時檢查活動觸發)                           │   │
│  │                                                      │   │
│  │  每分鐘檢查:                                         │   │
│  │  1. 取得所有活躍角色的當前排程                       │   │
│  │  2. 對符合條件的角色發送 :perform_activity          │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 活動處理流程

```
┌──────────────────────────────────────────────────────────────┐
│                   Activity Processing Flow                    │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────┐     ┌─────────────┐     ┌──────────────────┐   │
│  │Scheduler│────▶│ Agent       │────▶│ Decide Activity  │   │
│  │(定時)   │     │ (GenServer) │     │ (根據日程/AI)    │   │
│  └─────────┘     └─────────────┘     └────────┬─────────┘   │
│                                                │              │
│                                                ▼              │
│                                   ┌────────────────────────┐ │
│                                   │  Should Post?          │ │
│                                   │  (判斷是否發文)        │ │
│                                   └───────────┬────────────┘ │
│                                               │              │
│                          ┌────────────────────┼──────────┐   │
│                          ▼                    ▼          │   │
│                   ┌────────────┐       ┌────────────┐    │   │
│                   │  Yes       │       │  No        │    │   │
│                   │  建立 Oban │       │  僅更新    │    │   │
│                   │  Job       │       │  狀態      │    │   │
│                   └─────┬──────┘       └────────────┘    │   │
│                         │                                │   │
│                         ▼                                │   │
│               ┌──────────────────┐                       │   │
│               │ GeneratePostWorker│                       │   │
│               │  1. 呼叫 LLM     │                       │   │
│               │  2. 生成貼文內容 │                       │   │
│               │  3. 儲存到資料庫 │                       │   │
│               │  4. 廣播到前端   │                       │   │
│               └──────────────────┘                       │   │
│                                                          │   │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. 任務分解

### Week 1: GenServer 與 Registry

| 任務 | 預估時間 | 優先級 |
|------|----------|--------|
| 設計 Agent 狀態結構 | 4h | P0 |
| 實作 Agent GenServer | 8h | P0 |
| 實作 DynamicSupervisor | 4h | P0 |
| 實作 Registry 整合 | 4h | P0 |
| Agent 生命週期管理 | 8h | P0 |
| 單元測試 | 8h | P0 |

### Week 2: 排程與動態發佈

| 任務 | 預估時間 | 優先級 |
|------|----------|--------|
| 設計 Schedule Schema | 4h | P0 |
| 設計 Activity Schema | 4h | P0 |
| 實作 Scheduler 模組 | 8h | P0 |
| 實作 GeneratePostWorker | 8h | P0 |
| Post Schema 與 Context | 8h | P0 |
| 整合測試 | 8h | P0 |

### Week 3: 社群牆與完善

| 任務 | 預估時間 | 優先級 |
|------|----------|--------|
| Feed LiveView | 8h | P0 |
| 即時更新 (PubSub) | 4h | P0 |
| 管理介面（排程編輯）| 8h | P1 |
| 效能優化 | 4h | P1 |
| 端到端測試 | 8h | P0 |

---

## 4. 資料模型

### 4.1 Schedule Schema（日程表）

```elixir
# lib/xaifu/characters/schedule.ex
defmodule Xaifu.Characters.Schedule do
  use Ecto.Schema
  import Ecto.Changeset

  @primary_key {:id, :binary_id, autogenerate: true}
  @foreign_key_type :binary_id

  schema "schedules" do
    belongs_to :character, Xaifu.Characters.Character

    field :day_of_week, :integer  # 0-6 (週日-週六), nil = 每天
    field :time_slot, :string     # "morning", "afternoon", "evening", "night"
    field :hour, :integer         # 0-23, 精確小時

    field :activity_type, :string # "cafe", "work", "exercise", "shopping", "rest"
    field :location, :string      # 地點描述
    field :description, :string   # 活動描述

    field :post_probability, :float, default: 0.5  # 發文機率 0-1
    field :is_active, :boolean, default: true

    timestamps(type: :utc_datetime)
  end

  @time_slots ["morning", "afternoon", "evening", "night"]
  @activity_types ["cafe", "work", "exercise", "shopping", "rest", "social", "hobby", "travel"]

  def changeset(schedule, attrs) do
    schedule
    |> cast(attrs, [
      :character_id, :day_of_week, :time_slot, :hour,
      :activity_type, :location, :description,
      :post_probability, :is_active
    ])
    |> validate_required([:character_id, :activity_type])
    |> validate_inclusion(:day_of_week, 0..6, message: "必須是 0-6")
    |> validate_inclusion(:time_slot, @time_slots)
    |> validate_inclusion(:hour, 0..23)
    |> validate_inclusion(:activity_type, @activity_types)
    |> validate_number(:post_probability,
        greater_than_or_equal_to: 0,
        less_than_or_equal_to: 1)
    |> foreign_key_constraint(:character_id)
  end

  def time_slots, do: @time_slots
  def activity_types, do: @activity_types
end
```

### 4.2 Activity Schema（活動記錄）

```elixir
# lib/xaifu/characters/activity.ex
defmodule Xaifu.Characters.Activity do
  use Ecto.Schema
  import Ecto.Changeset

  @primary_key {:id, :binary_id, autogenerate: true}
  @foreign_key_type :binary_id

  schema "activities" do
    belongs_to :character, Xaifu.Characters.Character
    belongs_to :schedule, Xaifu.Characters.Schedule

    field :activity_type, :string
    field :location, :string
    field :description, :string
    field :started_at, :utc_datetime
    field :ended_at, :utc_datetime

    # 活動產生的內心想法（用於生成貼文）
    field :inner_thought, :string

    # 是否已發文
    field :posted, :boolean, default: false

    timestamps(type: :utc_datetime)
  end

  def changeset(activity, attrs) do
    activity
    |> cast(attrs, [
      :character_id, :schedule_id, :activity_type, :location,
      :description, :started_at, :ended_at, :inner_thought, :posted
    ])
    |> validate_required([:character_id, :activity_type, :started_at])
    |> foreign_key_constraint(:character_id)
    |> foreign_key_constraint(:schedule_id)
  end
end
```

### 4.3 Post Schema（社群貼文）

```elixir
# lib/xaifu/social/post.ex
defmodule Xaifu.Social.Post do
  use Ecto.Schema
  import Ecto.Changeset

  @primary_key {:id, :binary_id, autogenerate: true}
  @foreign_key_type :binary_id

  schema "posts" do
    belongs_to :character, Xaifu.Characters.Character
    belongs_to :activity, Xaifu.Characters.Activity

    field :content, :string
    field :image_url, :string      # Phase 3 使用
    field :image_prompt, :string   # Phase 3 使用

    field :mood, :string           # 發文時的心情
    field :location, :string       # 打卡地點

    # 互動統計
    field :likes_count, :integer, default: 0
    field :comments_count, :integer, default: 0

    # 可見性
    field :visibility, Ecto.Enum,
      values: [:public, :followers, :private],
      default: :public

    has_many :comments, Xaifu.Social.Comment
    has_many :likes, Xaifu.Social.Like

    timestamps(type: :utc_datetime)
  end

  def changeset(post, attrs) do
    post
    |> cast(attrs, [
      :character_id, :activity_id, :content, :image_url,
      :image_prompt, :mood, :location, :visibility
    ])
    |> validate_required([:character_id, :content])
    |> validate_length(:content, min: 1, max: 1000)
    |> foreign_key_constraint(:character_id)
    |> foreign_key_constraint(:activity_id)
  end
end
```

### 4.4 資料庫遷移

```elixir
# priv/repo/migrations/20260210000001_create_schedules.exs
defmodule Xaifu.Repo.Migrations.CreateSchedules do
  use Ecto.Migration

  def change do
    create table(:schedules, primary_key: false) do
      add :id, :binary_id, primary_key: true
      add :character_id, references(:characters, type: :binary_id, on_delete: :delete_all),
        null: false

      add :day_of_week, :integer
      add :time_slot, :string
      add :hour, :integer

      add :activity_type, :string, null: false
      add :location, :string
      add :description, :text

      add :post_probability, :float, default: 0.5
      add :is_active, :boolean, default: true

      timestamps(type: :utc_datetime)
    end

    create index(:schedules, [:character_id])
    create index(:schedules, [:character_id, :day_of_week, :hour])
    create index(:schedules, [:character_id, :time_slot])
  end
end
```

```elixir
# priv/repo/migrations/20260210000002_create_activities.exs
defmodule Xaifu.Repo.Migrations.CreateActivities do
  use Ecto.Migration

  def change do
    create table(:activities, primary_key: false) do
      add :id, :binary_id, primary_key: true
      add :character_id, references(:characters, type: :binary_id, on_delete: :delete_all),
        null: false
      add :schedule_id, references(:schedules, type: :binary_id, on_delete: :nilify_all)

      add :activity_type, :string, null: false
      add :location, :string
      add :description, :text
      add :started_at, :utc_datetime, null: false
      add :ended_at, :utc_datetime
      add :inner_thought, :text
      add :posted, :boolean, default: false

      timestamps(type: :utc_datetime)
    end

    create index(:activities, [:character_id])
    create index(:activities, [:character_id, :started_at])
  end
end
```

```elixir
# priv/repo/migrations/20260210000003_create_posts.exs
defmodule Xaifu.Repo.Migrations.CreatePosts do
  use Ecto.Migration

  def change do
    create table(:posts, primary_key: false) do
      add :id, :binary_id, primary_key: true
      add :character_id, references(:characters, type: :binary_id, on_delete: :delete_all),
        null: false
      add :activity_id, references(:activities, type: :binary_id, on_delete: :nilify_all)

      add :content, :text, null: false
      add :image_url, :string
      add :image_prompt, :text

      add :mood, :string
      add :location, :string

      add :likes_count, :integer, default: 0
      add :comments_count, :integer, default: 0
      add :visibility, :string, default: "public"

      timestamps(type: :utc_datetime)
    end

    create index(:posts, [:character_id])
    create index(:posts, [:inserted_at])
    create index(:posts, [:character_id, :inserted_at])
  end
end
```

---

## 5. Agent 系統實作

### 5.1 Agent State 結構

```elixir
# lib/xaifu/agents/state.ex
defmodule Xaifu.Agents.State do
  @moduledoc """
  Agent 狀態結構定義
  """

  defstruct [
    :character_id,
    :character,
    :current_activity,
    :current_location,
    :mood,           # 1-100
    :energy,         # 1-100
    :last_post_at,
    :started_at
  ]

  @type t :: %__MODULE__{
    character_id: String.t(),
    character: map(),
    current_activity: String.t() | nil,
    current_location: String.t() | nil,
    mood: integer(),
    energy: integer(),
    last_post_at: DateTime.t() | nil,
    started_at: DateTime.t()
  }

  def new(character) do
    %__MODULE__{
      character_id: character.id,
      character: character,
      current_activity: nil,
      current_location: "home",
      mood: random_initial_mood(),
      energy: random_initial_energy(),
      last_post_at: nil,
      started_at: DateTime.utc_now()
    }
  end

  defp random_initial_mood, do: Enum.random(60..90)
  defp random_initial_energy, do: Enum.random(70..100)
end
```

### 5.2 Agent GenServer

```elixir
# lib/xaifu/agents/agent.ex
defmodule Xaifu.Agents.Agent do
  @moduledoc """
  角色代理 GenServer
  每個活躍的角色都是一個獨立運作的進程
  """

  use GenServer
  require Logger

  alias Xaifu.Agents.State
  alias Xaifu.Characters
  alias Xaifu.Characters.{Schedule, Activity}
  alias Xaifu.Workers.GeneratePostWorker

  # ============================================
  # Client API
  # ============================================

  def start_link(character_id) do
    GenServer.start_link(__MODULE__, character_id, name: via_tuple(character_id))
  end

  def via_tuple(character_id) do
    {:via, Registry, {Xaifu.Agents.Registry, character_id}}
  end

  @doc "取得 Agent 狀態"
  def get_state(character_id) do
    GenServer.call(via_tuple(character_id), :get_state)
  end

  @doc "觸發活動檢查"
  def check_activity(character_id) do
    GenServer.cast(via_tuple(character_id), :check_activity)
  end

  @doc "強制執行活動"
  def perform_activity(character_id, activity_type) do
    GenServer.cast(via_tuple(character_id), {:perform_activity, activity_type})
  end

  @doc "更新心情"
  def update_mood(character_id, delta) do
    GenServer.cast(via_tuple(character_id), {:update_mood, delta})
  end

  @doc "檢查 Agent 是否存在"
  def exists?(character_id) do
    case Registry.lookup(Xaifu.Agents.Registry, character_id) do
      [{_pid, _}] -> true
      [] -> false
    end
  end

  # ============================================
  # Server Callbacks
  # ============================================

  @impl true
  def init(character_id) do
    Logger.info("Starting agent for character: #{character_id}")

    case Characters.get_character(character_id) do
      nil ->
        {:stop, :character_not_found}

      character ->
        state = State.new(character)

        # 設定初始活動檢查
        schedule_next_check()

        {:ok, state}
    end
  end

  @impl true
  def handle_call(:get_state, _from, state) do
    {:reply, state, state}
  end

  @impl true
  def handle_cast(:check_activity, state) do
    new_state = do_check_activity(state)
    schedule_next_check()
    {:noreply, new_state}
  end

  @impl true
  def handle_cast({:perform_activity, activity_type}, state) do
    new_state = do_perform_activity(state, activity_type)
    {:noreply, new_state}
  end

  @impl true
  def handle_cast({:update_mood, delta}, state) do
    new_mood = clamp(state.mood + delta, 1, 100)
    {:noreply, %{state | mood: new_mood}}
  end

  @impl true
  def handle_info(:scheduled_check, state) do
    new_state = do_check_activity(state)
    schedule_next_check()
    {:noreply, new_state}
  end

  @impl true
  def handle_info({:activity_completed, activity_id}, state) do
    # 活動完成後的處理
    Logger.info("Activity completed: #{activity_id}")
    {:noreply, %{state | current_activity: nil}}
  end

  # ============================================
  # Private Functions
  # ============================================

  defp schedule_next_check do
    # 每 5 分鐘檢查一次
    Process.send_after(self(), :scheduled_check, :timer.minutes(5))
  end

  defp do_check_activity(state) do
    now = DateTime.utc_now()
    current_hour = now.hour
    current_day = Date.day_of_week(DateTime.to_date(now), :sunday) - 1  # 0-6

    # 找到當前時段的排程
    case find_matching_schedule(state.character_id, current_day, current_hour) do
      nil ->
        # 沒有排程，維持休息狀態
        %{state | current_activity: "rest", current_location: "home"}

      schedule ->
        # 根據排程執行活動
        execute_schedule(state, schedule)
    end
  end

  defp find_matching_schedule(character_id, day_of_week, hour) do
    import Ecto.Query

    Schedule
    |> where([s], s.character_id == ^character_id)
    |> where([s], s.is_active == true)
    |> where([s], is_nil(s.day_of_week) or s.day_of_week == ^day_of_week)
    |> where([s], is_nil(s.hour) or s.hour == ^hour)
    |> where([s], is_nil(s.time_slot) or s.time_slot == ^get_time_slot(hour))
    |> order_by([s], [desc: s.hour, desc: s.day_of_week])
    |> limit(1)
    |> Xaifu.Repo.one()
  end

  defp get_time_slot(hour) when hour >= 6 and hour < 12, do: "morning"
  defp get_time_slot(hour) when hour >= 12 and hour < 18, do: "afternoon"
  defp get_time_slot(hour) when hour >= 18 and hour < 22, do: "evening"
  defp get_time_slot(_hour), do: "night"

  defp execute_schedule(state, schedule) do
    # 建立活動記錄
    {:ok, activity} = create_activity(state.character_id, schedule)

    # 更新狀態
    new_state = %{state |
      current_activity: schedule.activity_type,
      current_location: schedule.location || "unknown"
    }

    # 決定是否發文
    if should_post?(state, schedule) do
      # 排入發文任務
      enqueue_post_generation(activity, state.character)
    end

    # 模擬能量消耗
    energy_cost = get_energy_cost(schedule.activity_type)
    new_state = %{new_state | energy: clamp(new_state.energy - energy_cost, 1, 100)}

    new_state
  end

  defp create_activity(character_id, schedule) do
    %Activity{}
    |> Activity.changeset(%{
      character_id: character_id,
      schedule_id: schedule.id,
      activity_type: schedule.activity_type,
      location: schedule.location,
      description: schedule.description,
      started_at: DateTime.utc_now()
    })
    |> Xaifu.Repo.insert()
  end

  defp should_post?(state, schedule) do
    # 檢查發文間隔（至少 2 小時）
    recent_post? = case state.last_post_at do
      nil -> false
      last_at -> DateTime.diff(DateTime.utc_now(), last_at, :hour) < 2
    end

    # 根據機率決定
    if recent_post? do
      false
    else
      :rand.uniform() < schedule.post_probability
    end
  end

  defp enqueue_post_generation(activity, character) do
    %{
      activity_id: activity.id,
      character_id: character.id,
      activity_type: activity.activity_type,
      location: activity.location
    }
    |> GeneratePostWorker.new()
    |> Oban.insert()
  end

  defp get_energy_cost("work"), do: 20
  defp get_energy_cost("exercise"), do: 25
  defp get_energy_cost("social"), do: 15
  defp get_energy_cost("shopping"), do: 10
  defp get_energy_cost("rest"), do: -30  # 休息恢復能量
  defp get_energy_cost(_), do: 5

  defp do_perform_activity(state, activity_type) do
    # 建立臨時活動（非排程觸發）
    {:ok, activity} = %Activity{}
    |> Activity.changeset(%{
      character_id: state.character_id,
      activity_type: activity_type,
      started_at: DateTime.utc_now()
    })
    |> Xaifu.Repo.insert()

    # 直接觸發發文
    enqueue_post_generation(activity, state.character)

    %{state |
      current_activity: activity_type,
      last_post_at: DateTime.utc_now()
    }
  end

  defp clamp(value, min, max) do
    value
    |> max(min)
    |> min(max)
  end
end
```

### 5.3 Agent Supervisor

```elixir
# lib/xaifu/agents/supervisor.ex
defmodule Xaifu.Agents.Supervisor do
  @moduledoc """
  管理所有角色 Agent 的 DynamicSupervisor
  """

  use DynamicSupervisor
  require Logger

  alias Xaifu.Agents.Agent
  alias Xaifu.Characters

  def start_link(init_arg) do
    DynamicSupervisor.start_link(__MODULE__, init_arg, name: __MODULE__)
  end

  @impl true
  def init(_init_arg) do
    DynamicSupervisor.init(strategy: :one_for_one, max_restarts: 10, max_seconds: 60)
  end

  @doc "啟動角色 Agent"
  def start_agent(character_id) do
    child_spec = %{
      id: Agent,
      start: {Agent, :start_link, [character_id]},
      restart: :transient
    }

    case DynamicSupervisor.start_child(__MODULE__, child_spec) do
      {:ok, pid} ->
        Logger.info("Agent started for character #{character_id}, pid: #{inspect(pid)}")
        {:ok, pid}

      {:error, {:already_started, pid}} ->
        Logger.debug("Agent already running for character #{character_id}")
        {:ok, pid}

      error ->
        Logger.error("Failed to start agent: #{inspect(error)}")
        error
    end
  end

  @doc "停止角色 Agent"
  def stop_agent(character_id) do
    case Registry.lookup(Xaifu.Agents.Registry, character_id) do
      [{pid, _}] ->
        DynamicSupervisor.terminate_child(__MODULE__, pid)

      [] ->
        {:error, :not_found}
    end
  end

  @doc "啟動所有活躍角色的 Agent"
  def start_all_active_agents do
    Characters.list_active_characters()
    |> Enum.each(fn character ->
      start_agent(character.id)
    end)
  end

  @doc "取得所有運行中的 Agent"
  def list_agents do
    DynamicSupervisor.which_children(__MODULE__)
  end

  @doc "計算運行中的 Agent 數量"
  def count_agents do
    DynamicSupervisor.count_children(__MODULE__).active
  end
end
```

### 5.4 Application 配置

```elixir
# lib/xaifu/application.ex
defmodule Xaifu.Application do
  use Application

  @impl true
  def start(_type, _args) do
    children = [
      # 資料庫
      Xaifu.Repo,

      # PubSub
      {Phoenix.PubSub, name: Xaifu.PubSub},

      # Agent Registry
      {Registry, keys: :unique, name: Xaifu.Agents.Registry},

      # Agent Supervisor
      Xaifu.Agents.Supervisor,

      # Oban
      {Oban, Application.fetch_env!(:xaifu, Oban)},

      # 全域排程器（啟動活躍角色）
      Xaifu.Agents.Scheduler,

      # Web endpoint
      XaifuWeb.Endpoint
    ]

    opts = [strategy: :one_for_one, name: Xaifu.Supervisor]
    Supervisor.start_link(children, opts)
  end

  @impl true
  def config_change(changed, _new, removed) do
    XaifuWeb.Endpoint.config_change(changed, removed)
    :ok
  end
end
```

### 5.5 全域排程器

```elixir
# lib/xaifu/agents/scheduler.ex
defmodule Xaifu.Agents.Scheduler do
  @moduledoc """
  全域排程器
  - 啟動時載入所有活躍角色
  - 定時觸發活動檢查
  """

  use GenServer
  require Logger

  alias Xaifu.Agents.Supervisor, as: AgentSupervisor
  alias Xaifu.Characters

  def start_link(opts) do
    GenServer.start_link(__MODULE__, opts, name: __MODULE__)
  end

  @impl true
  def init(_opts) do
    # 延遲啟動，等待其他服務就緒
    Process.send_after(self(), :init_agents, :timer.seconds(5))

    {:ok, %{started: false}}
  end

  @impl true
  def handle_info(:init_agents, state) do
    Logger.info("Initializing all active agents...")

    count = start_all_active()
    Logger.info("Started #{count} agents")

    # 設定定時檢查
    schedule_periodic_check()

    {:noreply, %{state | started: true}}
  end

  @impl true
  def handle_info(:periodic_check, state) do
    # 檢查是否有新的活躍角色需要啟動
    sync_active_agents()

    # 排程下一次檢查
    schedule_periodic_check()

    {:noreply, state}
  end

  defp start_all_active do
    Characters.list_active_characters()
    |> Enum.map(fn character ->
      AgentSupervisor.start_agent(character.id)
    end)
    |> Enum.count(fn
      {:ok, _} -> true
      _ -> false
    end)
  end

  defp sync_active_agents do
    active_ids = Characters.list_active_characters()
    |> Enum.map(& &1.id)
    |> MapSet.new()

    # 啟動新的活躍角色
    Enum.each(active_ids, fn id ->
      unless Xaifu.Agents.Agent.exists?(id) do
        AgentSupervisor.start_agent(id)
      end
    end)

    # 停止已不活躍的角色
    AgentSupervisor.list_agents()
    |> Enum.each(fn {_, pid, _, _} ->
      case Registry.keys(Xaifu.Agents.Registry, pid) do
        [character_id] ->
          unless MapSet.member?(active_ids, character_id) do
            AgentSupervisor.stop_agent(character_id)
          end
        _ -> :ok
      end
    end)
  end

  defp schedule_periodic_check do
    # 每 10 分鐘同步一次
    Process.send_after(self(), :periodic_check, :timer.minutes(10))
  end
end
```

---

## 6. Oban Worker 實作

### 6.1 貼文生成 Worker

```elixir
# lib/xaifu/workers/generate_post_worker.ex
defmodule Xaifu.Workers.GeneratePostWorker do
  @moduledoc """
  生成並發佈貼文的 Oban Worker
  """

  use Oban.Worker,
    queue: :llm,
    max_attempts: 3,
    priority: 1

  require Logger

  alias Xaifu.AI.LLM
  alias Xaifu.Characters
  alias Xaifu.Characters.Activity
  alias Xaifu.Social
  alias Xaifu.Repo

  @impl Oban.Worker
  def perform(%Oban.Job{args: args}) do
    %{
      "activity_id" => activity_id,
      "character_id" => character_id,
      "activity_type" => activity_type,
      "location" => location
    } = args

    character = Characters.get_character!(character_id)
    activity = Repo.get!(Activity, activity_id)

    Logger.info("Generating post for #{character.name}, activity: #{activity_type}")

    with {:ok, content} <- generate_post_content(character, activity_type, location),
         {:ok, post} <- create_post(character, activity, content, location) do

      # 更新活動狀態
      activity
      |> Ecto.Changeset.change(posted: true)
      |> Repo.update()

      # 更新角色統計
      Characters.increment_stat(character, :total_posts)

      # 廣播新貼文
      broadcast_new_post(post)

      Logger.info("Post created: #{post.id}")
      :ok
    else
      {:error, reason} ->
        Logger.error("Failed to generate post: #{inspect(reason)}")
        {:error, reason}
    end
  end

  defp generate_post_content(character, activity_type, location) do
    prompt = build_post_prompt(character, activity_type, location)

    messages = [
      %{role: "system", content: post_generation_system_prompt(character)},
      %{role: "user", content: prompt}
    ]

    LLM.chat_completion(messages, temperature: 0.9, max_tokens: 300)
  end

  defp post_generation_system_prompt(character) do
    """
    你是 #{character.name}，正在社群媒體上發文。

    性格特徵：#{character.personality}
    說話風格：#{character.speaking_style || "自然隨性"}

    請生成一則社群貼文（類似 IG 或 Twitter）：
    - 保持角色一致性
    - 內容要自然、生活化
    - 可以使用適當的表情符號
    - 長度約 50-150 字
    - 不要使用標題或標籤格式
    """
  end

  defp build_post_prompt(character, activity_type, location) do
    location_text = if location, do: "地點：#{location}", else: ""

    """
    #{character.name} 現在正在進行「#{activity_description(activity_type)}」活動。
    #{location_text}

    請以 #{character.name} 的身份，發一則分享此刻心情或活動的貼文。
    """
  end

  defp activity_description("cafe"), do: "在咖啡店喝咖啡、放鬆"
  defp activity_description("work"), do: "工作、處理事務"
  defp activity_description("exercise"), do: "運動、健身"
  defp activity_description("shopping"), do: "逛街購物"
  defp activity_description("social"), do: "與朋友社交、聚會"
  defp activity_description("hobby"), do: "從事興趣活動"
  defp activity_description("travel"), do: "旅行、探索新地方"
  defp activity_description("rest"), do: "在家休息、放鬆"
  defp activity_description(other), do: other

  defp create_post(character, activity, content, location) do
    Social.create_post(%{
      character_id: character.id,
      activity_id: activity.id,
      content: content,
      location: location,
      mood: infer_mood(content)
    })
  end

  defp infer_mood(content) do
    # 簡單的情緒推斷（Phase 4 可以用 LLM 更精確分析）
    cond do
      String.contains?(content, ["開心", "快樂", "棒", "讚", "😊", "😄", "🎉"]) -> "happy"
      String.contains?(content, ["累", "疲", "睏", "😴", "😩"]) -> "tired"
      String.contains?(content, ["無聊", "煩", "😐", "😑"]) -> "bored"
      true -> "neutral"
    end
  end

  defp broadcast_new_post(post) do
    post = Repo.preload(post, :character)

    Phoenix.PubSub.broadcast(
      Xaifu.PubSub,
      "social:feed",
      {:new_post, post}
    )
  end
end
```

---

## 7. Social Context

```elixir
# lib/xaifu/social.ex
defmodule Xaifu.Social do
  @moduledoc """
  Social context - 管理社群功能
  """

  import Ecto.Query, warn: false
  alias Xaifu.Repo
  alias Xaifu.Social.Post

  @doc """
  取得動態牆貼文
  """
  def list_feed(opts \\ []) do
    limit = Keyword.get(opts, :limit, 20)
    offset = Keyword.get(opts, :offset, 0)
    character_id = Keyword.get(opts, :character_id)

    Post
    |> filter_by_character(character_id)
    |> where([p], p.visibility == :public)
    |> order_by([p], desc: p.inserted_at)
    |> limit(^limit)
    |> offset(^offset)
    |> preload(:character)
    |> Repo.all()
  end

  defp filter_by_character(query, nil), do: query
  defp filter_by_character(query, character_id) do
    where(query, [p], p.character_id == ^character_id)
  end

  @doc """
  取得單一貼文
  """
  def get_post(id) do
    Post
    |> Repo.get(id)
    |> Repo.preload([:character, :comments])
  end

  def get_post!(id) do
    Post
    |> Repo.get!(id)
    |> Repo.preload([:character, :comments])
  end

  @doc """
  建立貼文
  """
  def create_post(attrs) do
    %Post{}
    |> Post.changeset(attrs)
    |> Repo.insert()
  end

  @doc """
  更新貼文
  """
  def update_post(%Post{} = post, attrs) do
    post
    |> Post.changeset(attrs)
    |> Repo.update()
  end

  @doc """
  刪除貼文
  """
  def delete_post(%Post{} = post) do
    Repo.delete(post)
  end

  @doc """
  增加按讚數
  """
  def increment_likes(%Post{} = post) do
    Post
    |> where([p], p.id == ^post.id)
    |> Repo.update_all(inc: [likes_count: 1])
  end

  @doc """
  取得角色的貼文
  """
  def list_character_posts(character_id, opts \\ []) do
    limit = Keyword.get(opts, :limit, 20)

    Post
    |> where([p], p.character_id == ^character_id)
    |> order_by([p], desc: p.inserted_at)
    |> limit(^limit)
    |> Repo.all()
  end

  @doc """
  計算角色今日發文數
  """
  def count_today_posts(character_id) do
    today_start = Date.utc_today() |> DateTime.new!(~T[00:00:00], "Etc/UTC")

    Post
    |> where([p], p.character_id == ^character_id)
    |> where([p], p.inserted_at >= ^today_start)
    |> Repo.aggregate(:count)
  end
end
```

---

## 8. Feed LiveView

```elixir
# lib/xaifu_web/live/feed_live.ex
defmodule XaifuWeb.FeedLive do
  use XaifuWeb, :live_view

  alias Xaifu.Social
  alias Phoenix.PubSub

  @impl true
  def mount(_params, _session, socket) do
    if connected?(socket) do
      PubSub.subscribe(Xaifu.PubSub, "social:feed")
    end

    posts = Social.list_feed(limit: 20)

    {:ok,
     socket
     |> assign(:posts, posts)
     |> assign(:page, 1)
     |> assign(:loading, false)
     |> assign(:page_title, "動態牆")}
  end

  @impl true
  def handle_event("load_more", _params, socket) do
    if socket.assigns.loading do
      {:noreply, socket}
    else
      page = socket.assigns.page + 1
      offset = (page - 1) * 20

      new_posts = Social.list_feed(limit: 20, offset: offset)

      {:noreply,
       socket
       |> update(:posts, fn posts -> posts ++ new_posts end)
       |> assign(:page, page)
       |> assign(:loading, false)}
    end
  end

  @impl true
  def handle_event("like_post", %{"id" => post_id}, socket) do
    post = Social.get_post!(post_id)
    Social.increment_likes(post)

    # 更新本地狀態
    {:noreply,
     update(socket, :posts, fn posts ->
       Enum.map(posts, fn p ->
         if p.id == post_id do
           %{p | likes_count: p.likes_count + 1}
         else
           p
         end
       end)
     end)}
  end

  @impl true
  def handle_info({:new_post, post}, socket) do
    # 新貼文加到最前面
    {:noreply, update(socket, :posts, fn posts -> [post | posts] end)}
  end

  @impl true
  def render(assigns) do
    ~H"""
    <div class="max-w-2xl mx-auto px-4 py-8">
      <header class="mb-8">
        <h1 class="text-3xl font-bold text-gray-800">動態牆</h1>
        <p class="text-gray-500">查看 AI 角色們的最新動態</p>
      </header>

      <div class="space-y-6" id="posts-feed" phx-update="append">
        <%= for post <- @posts do %>
          <.post_card post={post} />
        <% end %>
      </div>

      <div class="mt-8 text-center">
        <button
          phx-click="load_more"
          disabled={@loading}
          class="px-6 py-2 bg-gray-100 hover:bg-gray-200 rounded-full text-gray-700 transition-colors disabled:opacity-50"
        >
          <%= if @loading, do: "載入中...", else: "載入更多" %>
        </button>
      </div>
    </div>
    """
  end

  defp post_card(assigns) do
    ~H"""
    <article
      class="bg-white rounded-xl shadow-md overflow-hidden"
      id={"post-#{@post.id}"}
    >
      <div class="p-6">
        <!-- Header -->
        <div class="flex items-center space-x-3 mb-4">
          <div class="w-12 h-12 rounded-full bg-gradient-to-br from-indigo-400 to-purple-500 flex items-center justify-center text-white font-bold">
            <%= String.first(@post.character.name) %>
          </div>
          <div>
            <h3 class="font-semibold text-gray-800">
              <.link navigate={~p"/characters/#{@post.character}"} class="hover:text-indigo-600">
                <%= @post.character.name %>
              </.link>
            </h3>
            <p class="text-sm text-gray-500">
              <%= format_time(@post.inserted_at) %>
              <%= if @post.location do %>
                · 📍 <%= @post.location %>
              <% end %>
            </p>
          </div>
        </div>

        <!-- Content -->
        <div class="mb-4">
          <p class="text-gray-800 whitespace-pre-wrap"><%= @post.content %></p>
        </div>

        <!-- Image (Phase 3) -->
        <%= if @post.image_url do %>
          <div class="mb-4 rounded-lg overflow-hidden">
            <img src={@post.image_url} alt="Post image" class="w-full" />
          </div>
        <% end %>

        <!-- Actions -->
        <div class="flex items-center space-x-6 pt-4 border-t">
          <button
            phx-click="like_post"
            phx-value-id={@post.id}
            class="flex items-center space-x-2 text-gray-500 hover:text-red-500 transition-colors"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z" />
            </svg>
            <span><%= @post.likes_count %></span>
          </button>

          <button class="flex items-center space-x-2 text-gray-500 hover:text-indigo-500 transition-colors">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" />
            </svg>
            <span><%= @post.comments_count %></span>
          </button>

          <.link
            navigate={~p"/chat/#{@post.character}"}
            class="flex items-center space-x-2 text-gray-500 hover:text-green-500 transition-colors"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 8h2a2 2 0 012 2v6a2 2 0 01-2 2h-2v4l-4-4H9a1.994 1.994 0 01-1.414-.586m0 0L11 14h4a2 2 0 002-2V6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2v4l.586-.586z" />
            </svg>
            <span>聊天</span>
          </.link>
        </div>
      </div>
    </article>
    """
  end

  defp format_time(datetime) do
    now = DateTime.utc_now()
    diff = DateTime.diff(now, datetime, :minute)

    cond do
      diff < 1 -> "剛剛"
      diff < 60 -> "#{diff} 分鐘前"
      diff < 1440 -> "#{div(diff, 60)} 小時前"
      diff < 10080 -> "#{div(diff, 1440)} 天前"
      true -> Calendar.strftime(datetime, "%Y/%m/%d")
    end
  end
end
```

---

## 9. 測試案例

### 9.1 Agent 測試

```elixir
# test/xaifu/agents/agent_test.exs
defmodule Xaifu.Agents.AgentTest do
  use Xaifu.DataCase, async: false  # GenServer 測試不能 async

  alias Xaifu.Agents.{Agent, Supervisor}
  alias Xaifu.Characters

  setup do
    # 確保 Registry 已啟動
    start_supervised!({Registry, keys: :unique, name: Xaifu.Agents.Registry})
    start_supervised!(Supervisor)

    {:ok, character} = Characters.create_character(%{
      name: "Test Agent",
      personality: "測試用角色性格"
    })
    |> then(fn {:ok, c} -> Characters.change_status(c, :active) end)

    %{character: character}
  end

  describe "start_link/1" do
    test "starts agent for character", %{character: character} do
      assert {:ok, pid} = Supervisor.start_agent(character.id)
      assert Process.alive?(pid)
    end

    test "returns existing pid if already started", %{character: character} do
      {:ok, pid1} = Supervisor.start_agent(character.id)
      {:ok, pid2} = Supervisor.start_agent(character.id)

      assert pid1 == pid2
    end
  end

  describe "get_state/1" do
    test "returns current agent state", %{character: character} do
      {:ok, _pid} = Supervisor.start_agent(character.id)

      state = Agent.get_state(character.id)

      assert state.character_id == character.id
      assert state.mood >= 1 and state.mood <= 100
      assert state.energy >= 1 and state.energy <= 100
    end
  end

  describe "exists?/1" do
    test "returns true when agent is running", %{character: character} do
      {:ok, _pid} = Supervisor.start_agent(character.id)
      assert Agent.exists?(character.id)
    end

    test "returns false when agent is not running", %{character: character} do
      refute Agent.exists?(character.id)
    end
  end

  describe "update_mood/2" do
    test "updates agent mood", %{character: character} do
      {:ok, _pid} = Supervisor.start_agent(character.id)
      initial_state = Agent.get_state(character.id)

      Agent.update_mood(character.id, 10)
      Process.sleep(50)  # Give time for cast to process

      new_state = Agent.get_state(character.id)
      assert new_state.mood == min(initial_state.mood + 10, 100)
    end

    test "clamps mood to valid range", %{character: character} do
      {:ok, _pid} = Supervisor.start_agent(character.id)

      Agent.update_mood(character.id, 200)
      Process.sleep(50)

      state = Agent.get_state(character.id)
      assert state.mood == 100
    end
  end
end
```

### 9.2 Scheduler 測試

```elixir
# test/xaifu/characters/schedule_test.exs
defmodule Xaifu.Characters.ScheduleTest do
  use Xaifu.DataCase, async: true

  alias Xaifu.Characters
  alias Xaifu.Characters.Schedule

  setup do
    {:ok, character} = Characters.create_character(%{
      name: "Schedule Test",
      personality: "測試角色"
    })

    %{character: character}
  end

  describe "schedule changeset" do
    test "valid schedule", %{character: character} do
      attrs = %{
        character_id: character.id,
        activity_type: "cafe",
        hour: 14,
        location: "星巴克",
        post_probability: 0.7
      }

      changeset = Schedule.changeset(%Schedule{}, attrs)
      assert changeset.valid?
    end

    test "validates day_of_week range" do
      changeset = Schedule.changeset(%Schedule{}, %{
        activity_type: "cafe",
        day_of_week: 7
      })

      refute changeset.valid?
      assert "必須是 0-6" in errors_on(changeset).day_of_week
    end

    test "validates activity_type" do
      changeset = Schedule.changeset(%Schedule{}, %{
        activity_type: "invalid_type"
      })

      refute changeset.valid?
      assert "is invalid" in errors_on(changeset).activity_type
    end

    test "validates post_probability range" do
      changeset = Schedule.changeset(%Schedule{}, %{
        activity_type: "cafe",
        post_probability: 1.5
      })

      refute changeset.valid?
    end
  end
end
```

### 9.3 Social Context 測試

```elixir
# test/xaifu/social_test.exs
defmodule Xaifu.SocialTest do
  use Xaifu.DataCase, async: true

  alias Xaifu.Social
  alias Xaifu.Characters

  setup do
    {:ok, character} = Characters.create_character(%{
      name: "Social Test",
      personality: "測試角色"
    })

    %{character: character}
  end

  describe "list_feed/1" do
    test "returns posts ordered by date", %{character: character} do
      {:ok, post1} = Social.create_post(%{
        character_id: character.id,
        content: "First post"
      })

      Process.sleep(10)

      {:ok, post2} = Social.create_post(%{
        character_id: character.id,
        content: "Second post"
      })

      feed = Social.list_feed()

      assert length(feed) == 2
      assert hd(feed).id == post2.id
    end

    test "respects limit option", %{character: character} do
      for i <- 1..5 do
        Social.create_post(%{
          character_id: character.id,
          content: "Post #{i}"
        })
      end

      assert length(Social.list_feed(limit: 3)) == 3
    end

    test "filters by character_id", %{character: character} do
      {:ok, other} = Characters.create_character(%{
        name: "Other",
        personality: "其他角色"
      })

      Social.create_post(%{character_id: character.id, content: "Mine"})
      Social.create_post(%{character_id: other.id, content: "Others"})

      feed = Social.list_feed(character_id: character.id)

      assert length(feed) == 1
      assert hd(feed).content == "Mine"
    end
  end

  describe "create_post/1" do
    test "creates a post", %{character: character} do
      attrs = %{
        character_id: character.id,
        content: "Hello world!",
        location: "台北"
      }

      assert {:ok, post} = Social.create_post(attrs)
      assert post.content == "Hello world!"
      assert post.location == "台北"
      assert post.likes_count == 0
    end

    test "validates content length", %{character: character} do
      attrs = %{
        character_id: character.id,
        content: String.duplicate("a", 1001)
      }

      assert {:error, changeset} = Social.create_post(attrs)
      assert "should be at most 1000 character(s)" in errors_on(changeset).content
    end
  end

  describe "increment_likes/1" do
    test "increments likes count", %{character: character} do
      {:ok, post} = Social.create_post(%{
        character_id: character.id,
        content: "Like me!"
      })

      assert post.likes_count == 0

      Social.increment_likes(post)
      updated = Social.get_post!(post.id)

      assert updated.likes_count == 1
    end
  end
end
```

### 9.4 Worker 測試

```elixir
# test/xaifu/workers/generate_post_worker_test.exs
defmodule Xaifu.Workers.GeneratePostWorkerTest do
  use Xaifu.DataCase, async: true
  use Oban.Testing, repo: Xaifu.Repo

  alias Xaifu.Workers.GeneratePostWorker
  alias Xaifu.Characters
  alias Xaifu.Characters.Activity

  setup do
    {:ok, character} = Characters.create_character(%{
      name: "Worker Test",
      personality: "測試角色"
    })

    {:ok, activity} = %Activity{}
    |> Activity.changeset(%{
      character_id: character.id,
      activity_type: "cafe",
      location: "星巴克",
      started_at: DateTime.utc_now()
    })
    |> Xaifu.Repo.insert()

    %{character: character, activity: activity}
  end

  test "enqueues job correctly", %{character: character, activity: activity} do
    assert {:ok, _job} =
      %{
        activity_id: activity.id,
        character_id: character.id,
        activity_type: "cafe",
        location: "星巴克"
      }
      |> GeneratePostWorker.new()
      |> Oban.insert()

    assert_enqueued worker: GeneratePostWorker
  end

  # 注意：實際執行測試需要 mock LLM API
  # 這裡僅測試 job 結構
end
```

---

## 10. 驗收標準

### 10.1 功能驗收

| 項目 | 驗收條件 | 狀態 |
|------|----------|------|
| Agent 啟動 | 角色設為 active 後自動啟動 Agent | ☐ |
| Agent 狀態 | 可查詢 Agent 當前狀態 | ☐ |
| 排程系統 | 可為角色設定日程表 | ☐ |
| 活動觸發 | 到達排程時間自動觸發活動 | ☐ |
| 貼文生成 | 活動觸發後自動生成貼文 | ☐ |
| 動態牆 | 貼文即時顯示在 Feed | ☐ |
| 發文機率 | 依照設定機率決定是否發文 | ☐ |

### 10.2 效能驗收

| 指標 | 目標值 |
|------|--------|
| Agent 啟動時間 | < 100ms |
| 同時運行 Agent 數 | ≥ 100 |
| 貼文生成時間 | < 10s |
| Feed 載入時間 | < 500ms |

---

## 11. 下一階段準備

完成 Phase 2 後，為 Phase 3（多媒體內容生成）做以下準備：

1. 研究圖像生成 API（Replicate, ComfyUI）
2. 設計 Prompt 工程模組
3. 評估圖片儲存方案（Local vs S3）
4. 規劃角色視覺一致性方案

---

*文件版本: 1.0*
*對應主文件: [00-overview.md](./00-overview.md)*

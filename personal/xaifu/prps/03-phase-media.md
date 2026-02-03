# Phase 3: 多媒體內容生成

## 預估工時：2-3 週（單人開發）

---

## 1. 階段目標

整合 AI 圖像生成能力，讓角色的動態具有視覺內容：
- Prompt 工程模組（活動轉圖像提示詞）
- 圖像生成服務整合
- 角色視覺一致性方案
- 圖片儲存與最佳化
- 帶圖動態發佈

**里程碑**：角色發文自動附帶一致風格的 AI 生成圖片

---

## 2. 架構設計

### 2.1 圖像生成流程

```
┌──────────────────────────────────────────────────────────────────┐
│                   Image Generation Pipeline                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────────────┐ │
│  │  Activity   │────▶│   Prompt    │────▶│   Image Prompt      │ │
│  │  Context    │     │   Builder   │     │   (結構化)          │ │
│  └─────────────┘     └─────────────┘     └──────────┬──────────┘ │
│                                                      │            │
│  包含:                  處理:                        │            │
│  - activity_type        - 活動轉場景               ▼            │
│  - location             - 角色外觀注入     ┌─────────────────┐  │
│  - mood                 - 一致性參數       │ Consistency     │  │
│  - character.appearance - 品質標籤         │ Module          │  │
│                                            │                 │  │
│                                            │ - LoRA weights  │  │
│                                            │ - Style tokens  │  │
│                                            │ - Seed control  │  │
│                                            └────────┬────────┘  │
│                                                     │            │
│                                                     ▼            │
│                                          ┌─────────────────────┐ │
│                                          │   Image Generator   │ │
│                                          │   (Provider)        │ │
│                                          │                     │ │
│                                          │ - Replicate        │ │
│                                          │ - ComfyUI          │ │
│                                          │ - Stable Diffusion │ │
│                                          └────────┬────────────┘ │
│                                                   │              │
│                              ┌────────────────────┼───────────┐  │
│                              ▼                    ▼           │  │
│                      ┌────────────┐      ┌────────────┐      │  │
│                      │  Success   │      │  Failure   │      │  │
│                      │            │      │            │      │  │
│                      │ 1. 下載    │      │ 1. 重試   │      │  │
│                      │ 2. 最佳化  │      │ 2. 降級   │      │  │
│                      │ 3. 儲存    │      │ 3. 通知   │      │  │
│                      │ 4. 更新DB  │      │            │      │  │
│                      └────────────┘      └────────────┘      │  │
│                                                              │  │
└──────────────────────────────────────────────────────────────────┘
```

### 2.2 一致性解決方案

```
┌──────────────────────────────────────────────────────────────────┐
│              Character Visual Consistency Strategy                │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  方案 A: LoRA 微調（高品質，高成本）                              │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  1. 為每個角色訓練專屬 LoRA                              │   │
│  │  2. 上傳到 Replicate/HuggingFace                         │   │
│  │  3. 生成時指定 lora_weights                              │   │
│  │                                                          │   │
│  │  優點: 高度一致性、精確控制                              │   │
│  │  缺點: 需要訓練資料、訓練時間、額外成本                  │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  方案 B: IP-Adapter（推薦，平衡方案）                            │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  1. 為每個角色準備 1 張參考圖                            │   │
│  │  2. 使用 IP-Adapter 進行風格遷移                         │   │
│  │  3. 搭配固定 seed 增加穩定性                             │   │
│  │                                                          │   │
│  │  優點: 無需訓練、快速部署、成本低                        │   │
│  │  缺點: 一致性略低於 LoRA                                │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  方案 C: Prompt 工程（最簡單，作為基礎）                         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  1. 建立詳細的外觀描述模板                               │   │
│  │  2. 每次生成都包含完整外觀描述                           │   │
│  │  3. 使用固定的負面提示詞                                 │   │
│  │                                                          │   │
│  │  優點: 零額外成本、實作簡單                              │   │
│  │  缺點: 一致性最低                                        │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  本專案採用: B + C 混合方案                                      │
│  - 預設使用 Prompt 工程                                          │
│  - 重要角色可啟用 IP-Adapter                                     │
│  - 未來可升級到 LoRA                                             │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. 任務分解

### Week 1: Prompt 工程與圖像服務

| 任務 | 預估時間 | 優先級 |
|------|----------|--------|
| 設計 Prompt Builder 模組 | 8h | P0 |
| 活動場景提示詞庫 | 6h | P0 |
| 圖像服務抽象層 | 4h | P0 |
| Replicate API 整合 | 8h | P0 |
| 本地 ComfyUI 整合（備選） | 6h | P1 |
| 單元測試 | 4h | P0 |

### Week 2: 一致性與儲存

| 任務 | 預估時間 | 優先級 |
|------|----------|--------|
| IP-Adapter 整合 | 8h | P1 |
| 角色參考圖管理 | 4h | P1 |
| 圖片下載與處理 | 6h | P0 |
| 圖片儲存模組 | 6h | P0 |
| 圖片最佳化（壓縮、縮圖）| 4h | P0 |
| S3 上傳整合（可選）| 4h | P2 |

### Week 3: Worker 整合與完善

| 任務 | 預估時間 | 優先級 |
|------|----------|--------|
| GenerateImageWorker | 8h | P0 |
| 修改 GeneratePostWorker | 4h | P0 |
| 錯誤處理與重試 | 4h | P0 |
| 前端圖片顯示 | 4h | P0 |
| 後台圖片管理 | 4h | P1 |
| 端到端測試 | 8h | P0 |

---

## 4. Prompt 工程模組

### 4.1 Prompt Builder

```elixir
# lib/xaifu/ai/prompt_builder.ex
defmodule Xaifu.AI.PromptBuilder do
  @moduledoc """
  圖像生成 Prompt 建構器
  將活動上下文轉換為高品質的圖像提示詞
  """

  alias Xaifu.Characters.Character

  @type prompt_context :: %{
    character: Character.t(),
    activity_type: String.t(),
    location: String.t() | nil,
    mood: String.t() | nil,
    time_of_day: String.t() | nil
  }

  @type image_prompt :: %{
    positive: String.t(),
    negative: String.t(),
    style: String.t(),
    aspect_ratio: String.t()
  }

  @doc """
  建構完整的圖像生成提示詞
  """
  @spec build(prompt_context()) :: image_prompt()
  def build(context) do
    %{
      positive: build_positive_prompt(context),
      negative: build_negative_prompt(),
      style: determine_style(context),
      aspect_ratio: "1:1"
    }
  end

  @doc """
  建構正面提示詞
  """
  def build_positive_prompt(context) do
    [
      # 品質標籤
      quality_tags(),
      # 角色外觀
      character_appearance(context.character),
      # 場景描述
      scene_description(context),
      # 氛圍/光線
      atmosphere(context),
      # 風格標籤
      style_tags(context)
    ]
    |> List.flatten()
    |> Enum.reject(&is_nil/1)
    |> Enum.join(", ")
  end

  @doc """
  建構負面提示詞
  """
  def build_negative_prompt do
    [
      "nsfw", "nude", "naked",
      "bad anatomy", "bad hands", "bad proportions",
      "blurry", "cropped", "deformed",
      "disfigured", "duplicate", "error",
      "extra fingers", "extra limbs",
      "low quality", "lowres", "malformed",
      "missing fingers", "missing limbs",
      "mutated", "mutation", "out of frame",
      "poorly drawn", "signature", "text",
      "ugly", "watermark", "worst quality"
    ]
    |> Enum.join(", ")
  end

  # ============================================
  # Private Functions
  # ============================================

  defp quality_tags do
    [
      "masterpiece", "best quality", "highly detailed",
      "professional photography", "8k uhd", "sharp focus"
    ]
  end

  defp character_appearance(%Character{} = character) do
    # 從角色設定取得外觀描述
    base_appearance = character.appearance || default_appearance()

    # 加上角色特定的前綴（如果有設定）
    if character.image_prompt_prefix do
      [character.image_prompt_prefix, base_appearance]
    else
      base_appearance
    end
  end

  defp default_appearance do
    "young adult, attractive, natural look, casual outfit"
  end

  defp scene_description(context) do
    activity_scene = activity_to_scene(context.activity_type)
    location_detail = location_to_detail(context.location)

    [activity_scene, location_detail]
  end

  defp activity_to_scene(activity_type) do
    scenes = %{
      "cafe" => "sitting at a cozy cafe, holding a coffee cup, warm ambient lighting",
      "work" => "at a modern workspace, focused expression, professional setting",
      "exercise" => "at the gym, athletic pose, energetic, workout clothes",
      "shopping" => "walking through a shopping district, carrying shopping bags, stylish",
      "social" => "at a social gathering, smiling, friendly atmosphere",
      "hobby" => "engaged in creative activity, passionate expression",
      "travel" => "scenic travel location, exploring, adventurous mood",
      "rest" => "relaxing at home, comfortable setting, peaceful"
    }

    Map.get(scenes, activity_type, "casual daily life scene")
  end

  defp location_to_detail(nil), do: nil
  defp location_to_detail(location) do
    # 將中文地點轉換為英文描述
    location_mappings = %{
      "星巴克" => "Starbucks cafe interior",
      "公園" => "beautiful park with trees and sunshine",
      "健身房" => "modern gym interior",
      "百貨公司" => "upscale department store",
      "海邊" => "scenic beach with ocean view",
      "山上" => "mountain landscape with fresh air"
    }

    Map.get(location_mappings, location, "#{location} background")
  end

  defp atmosphere(context) do
    time_atmosphere = case context[:time_of_day] do
      "morning" -> "soft morning light, golden hour"
      "afternoon" -> "bright daylight, clear sky"
      "evening" -> "warm sunset glow, golden hour"
      "night" -> "night scene, ambient lighting, city lights"
      _ -> "natural lighting"
    end

    mood_atmosphere = case context[:mood] do
      "happy" -> "cheerful atmosphere, vibrant colors"
      "tired" -> "soft tones, relaxed mood"
      "bored" -> "muted colors, casual mood"
      _ -> "pleasant atmosphere"
    end

    [time_atmosphere, mood_atmosphere]
  end

  defp style_tags(context) do
    base_style = ["realistic photo", "instagram style", "lifestyle photography"]

    # 可以根據角色或場景調整風格
    case context.activity_type do
      "travel" -> base_style ++ ["travel photography", "scenic view"]
      "cafe" -> base_style ++ ["cafe aesthetic", "cozy vibe"]
      _ -> base_style
    end
  end

  defp determine_style(_context) do
    # 預設使用寫實風格，未來可擴展為動漫風等
    "realistic"
  end
end
```

### 4.2 場景提示詞庫

```elixir
# lib/xaifu/ai/scene_library.ex
defmodule Xaifu.AI.SceneLibrary do
  @moduledoc """
  場景提示詞庫
  提供豐富的場景描述模板
  """

  @doc """
  取得場景變體（增加多樣性）
  """
  def get_scene_variant(activity_type) do
    variants = scene_variants()[activity_type] || generic_variants()
    Enum.random(variants)
  end

  @doc """
  取得動作描述
  """
  def get_action(activity_type) do
    actions = activity_actions()[activity_type] || generic_actions()
    Enum.random(actions)
  end

  # ============================================
  # Scene Variants
  # ============================================

  defp scene_variants do
    %{
      "cafe" => [
        "cozy corner table at a trendy cafe, warm wooden interior",
        "outdoor cafe terrace, afternoon sunlight, urban backdrop",
        "minimalist Japanese style cafe, zen atmosphere",
        "vintage European cafe, classic decor, ambient lighting"
      ],
      "work" => [
        "modern open office space, multiple monitors",
        "home office setup, cozy workspace, plants nearby",
        "co-working space, creative atmosphere",
        "quiet library corner, studious mood"
      ],
      "exercise" => [
        "state-of-the-art gym, modern equipment",
        "outdoor jogging trail, morning sunshine",
        "yoga studio, peaceful environment, natural light",
        "swimming pool, clear blue water, athletic"
      ],
      "shopping" => [
        "luxury brand boutique, elegant interior",
        "trendy street shopping area, urban style",
        "local market, vibrant colors, bustling atmosphere",
        "modern shopping mall, bright and spacious"
      ],
      "travel" => [
        "scenic mountain viewpoint, breathtaking landscape",
        "historic european street, charming architecture",
        "tropical beach paradise, crystal clear water",
        "bustling asian night market, neon lights"
      ],
      "social" => [
        "rooftop party, city skyline background",
        "cozy dinner gathering, warm candlelight",
        "lively bar scene, social atmosphere",
        "outdoor picnic, sunny day, friends"
      ],
      "rest" => [
        "cozy bedroom, soft bedding, morning light",
        "comfortable sofa, reading corner, warm lighting",
        "balcony relaxation, city view, sunset",
        "spa-like bathroom, self-care moment"
      ]
    }
  end

  defp activity_actions do
    %{
      "cafe" => [
        "sipping coffee thoughtfully",
        "reading a book with coffee",
        "typing on laptop, focused",
        "taking a photo of latte art"
      ],
      "work" => [
        "typing on keyboard, concentrated",
        "video call meeting, professional",
        "writing notes, deep in thought",
        "stretching at desk, taking a break"
      ],
      "exercise" => [
        "running with determination",
        "lifting weights, strong pose",
        "yoga pose, balanced and calm",
        "post-workout selfie, satisfied"
      ],
      "shopping" => [
        "trying on clothes, mirror selfie",
        "browsing store displays, interested",
        "carrying shopping bags, happy",
        "examining a product closely"
      ],
      "travel" => [
        "taking photos of scenery",
        "exploring local streets",
        "enjoying local cuisine",
        "posing at landmark"
      ]
    }
  end

  defp generic_variants do
    [
      "pleasant everyday scene",
      "casual lifestyle moment",
      "natural daily activity"
    ]
  end

  defp generic_actions do
    [
      "casual pose, natural expression",
      "candid moment, authentic feel",
      "relaxed stance, everyday vibe"
    ]
  end
end
```

---

## 5. 圖像生成服務

### 5.1 服務行為定義

```elixir
# lib/xaifu/ai/image_generator_behaviour.ex
defmodule Xaifu.AI.ImageGeneratorBehaviour do
  @moduledoc """
  圖像生成服務行為定義
  """

  @type generation_opts :: %{
    positive_prompt: String.t(),
    negative_prompt: String.t(),
    width: integer(),
    height: integer(),
    seed: integer() | nil,
    reference_image: String.t() | nil,
    lora_weights: String.t() | nil
  }

  @type generation_result :: {:ok, String.t()} | {:error, term()}

  @callback generate(opts :: generation_opts()) :: generation_result()
  @callback health_check() :: :ok | {:error, term()}
end
```

### 5.2 Replicate Provider

```elixir
# lib/xaifu/ai/providers/replicate.ex
defmodule Xaifu.AI.Providers.Replicate do
  @moduledoc """
  Replicate API 整合
  支援 SDXL, Flux 等模型
  """

  @behaviour Xaifu.AI.ImageGeneratorBehaviour

  require Logger

  @base_url "https://api.replicate.com/v1"

  # 預設使用 SDXL
  @default_model "stability-ai/sdxl:39ed52f2a78e934b3ba6e2a89f5b1c712de7dfea535525255b1aa35c5565e08b"

  @impl true
  def generate(opts) do
    model = opts[:model] || @default_model

    input = build_input(opts)

    case create_prediction(model, input) do
      {:ok, prediction_id} ->
        wait_for_result(prediction_id)

      {:error, reason} ->
        {:error, reason}
    end
  end

  @impl true
  def health_check do
    # 簡單檢查 API 是否可用
    case Req.get("#{@base_url}/models/stability-ai/sdxl",
           headers: auth_headers()) do
      {:ok, %{status: 200}} -> :ok
      {:ok, %{status: status}} -> {:error, {:api_error, status}}
      {:error, reason} -> {:error, reason}
    end
  end

  # ============================================
  # Private Functions
  # ============================================

  defp build_input(opts) do
    base_input = %{
      prompt: opts.positive_prompt,
      negative_prompt: opts[:negative_prompt] || "",
      width: opts[:width] || 1024,
      height: opts[:height] || 1024,
      num_outputs: 1,
      guidance_scale: 7.5,
      num_inference_steps: 30
    }

    # 加入可選參數
    base_input
    |> maybe_add_seed(opts[:seed])
    |> maybe_add_lora(opts[:lora_weights])
  end

  defp maybe_add_seed(input, nil), do: input
  defp maybe_add_seed(input, seed), do: Map.put(input, :seed, seed)

  defp maybe_add_lora(input, nil), do: input
  defp maybe_add_lora(input, weights) do
    Map.put(input, :lora_weights, weights)
  end

  defp create_prediction(model, input) do
    body = %{
      version: extract_version(model),
      input: input
    }

    case Req.post("#{@base_url}/predictions",
           json: body,
           headers: auth_headers()) do
      {:ok, %{status: 201, body: %{"id" => id}}} ->
        {:ok, id}

      {:ok, %{status: status, body: body}} ->
        Logger.error("Replicate API error: #{status} - #{inspect(body)}")
        {:error, {:api_error, status, body}}

      {:error, reason} ->
        {:error, {:request_failed, reason}}
    end
  end

  defp wait_for_result(prediction_id, attempts \\ 0) do
    max_attempts = 60  # 最多等待 60 次（約 2 分鐘）

    if attempts >= max_attempts do
      {:error, :timeout}
    else
      case get_prediction(prediction_id) do
        {:ok, %{"status" => "succeeded", "output" => [url | _]}} ->
          {:ok, url}

        {:ok, %{"status" => "failed", "error" => error}} ->
          {:error, {:generation_failed, error}}

        {:ok, %{"status" => status}} when status in ["starting", "processing"] ->
          Process.sleep(2000)
          wait_for_result(prediction_id, attempts + 1)

        {:error, reason} ->
          {:error, reason}
      end
    end
  end

  defp get_prediction(id) do
    case Req.get("#{@base_url}/predictions/#{id}",
           headers: auth_headers()) do
      {:ok, %{status: 200, body: body}} ->
        {:ok, body}

      {:ok, %{status: status}} ->
        {:error, {:api_error, status}}

      {:error, reason} ->
        {:error, reason}
    end
  end

  defp auth_headers do
    [
      {"Authorization", "Bearer #{api_key()}"},
      {"Content-Type", "application/json"}
    ]
  end

  defp api_key do
    Application.get_env(:xaifu, __MODULE__)[:api_key] ||
      raise "Replicate API key not configured"
  end

  defp extract_version(model) do
    # 格式: "owner/model:version" -> "version"
    case String.split(model, ":") do
      [_, version] -> version
      _ -> model
    end
  end
end
```

### 5.3 ComfyUI Provider（本地方案）

```elixir
# lib/xaifu/ai/providers/comfyui.ex
defmodule Xaifu.AI.Providers.ComfyUI do
  @moduledoc """
  ComfyUI 本地 API 整合
  適用於自建伺服器方案
  """

  @behaviour Xaifu.AI.ImageGeneratorBehaviour

  require Logger

  @impl true
  def generate(opts) do
    workflow = build_workflow(opts)

    case queue_prompt(workflow) do
      {:ok, prompt_id} ->
        wait_for_result(prompt_id)

      {:error, reason} ->
        {:error, reason}
    end
  end

  @impl true
  def health_check do
    case Req.get("#{base_url()}/system_stats") do
      {:ok, %{status: 200}} -> :ok
      _ -> {:error, :unavailable}
    end
  end

  # ============================================
  # Private Functions
  # ============================================

  defp build_workflow(opts) do
    # 建立 ComfyUI workflow JSON
    # 這裡是簡化版，實際需要根據你的 workflow 調整
    %{
      "3" => %{
        "class_type" => "KSampler",
        "inputs" => %{
          "cfg" => 7.5,
          "denoise" => 1,
          "model" => ["4", 0],
          "negative" => ["7", 0],
          "positive" => ["6", 0],
          "sampler_name" => "euler",
          "scheduler" => "normal",
          "seed" => opts[:seed] || :rand.uniform(999_999_999),
          "steps" => 30
        }
      },
      "4" => %{
        "class_type" => "CheckpointLoaderSimple",
        "inputs" => %{
          "ckpt_name" => "sd_xl_base_1.0.safetensors"
        }
      },
      "5" => %{
        "class_type" => "EmptyLatentImage",
        "inputs" => %{
          "batch_size" => 1,
          "height" => opts[:height] || 1024,
          "width" => opts[:width] || 1024
        }
      },
      "6" => %{
        "class_type" => "CLIPTextEncode",
        "inputs" => %{
          "clip" => ["4", 1],
          "text" => opts.positive_prompt
        }
      },
      "7" => %{
        "class_type" => "CLIPTextEncode",
        "inputs" => %{
          "clip" => ["4", 1],
          "text" => opts[:negative_prompt] || ""
        }
      },
      "8" => %{
        "class_type" => "VAEDecode",
        "inputs" => %{
          "samples" => ["3", 0],
          "vae" => ["4", 2]
        }
      },
      "9" => %{
        "class_type" => "SaveImage",
        "inputs" => %{
          "filename_prefix" => "xaifu",
          "images" => ["8", 0]
        }
      }
    }
  end

  defp queue_prompt(workflow) do
    body = %{prompt: workflow}

    case Req.post("#{base_url()}/prompt", json: body) do
      {:ok, %{status: 200, body: %{"prompt_id" => id}}} ->
        {:ok, id}

      {:ok, %{body: body}} ->
        {:error, {:api_error, body}}

      {:error, reason} ->
        {:error, reason}
    end
  end

  defp wait_for_result(prompt_id, attempts \\ 0) do
    max_attempts = 120

    if attempts >= max_attempts do
      {:error, :timeout}
    else
      case check_history(prompt_id) do
        {:ok, image_url} ->
          {:ok, image_url}

        :pending ->
          Process.sleep(1000)
          wait_for_result(prompt_id, attempts + 1)

        {:error, reason} ->
          {:error, reason}
      end
    end
  end

  defp check_history(prompt_id) do
    case Req.get("#{base_url()}/history/#{prompt_id}") do
      {:ok, %{status: 200, body: body}} ->
        case body[prompt_id] do
          nil ->
            :pending

          %{"outputs" => outputs} ->
            extract_image_url(outputs)

          _ ->
            :pending
        end

      {:error, reason} ->
        {:error, reason}
    end
  end

  defp extract_image_url(outputs) do
    # 從 outputs 中提取圖片 URL
    case outputs do
      %{"9" => %{"images" => [%{"filename" => filename} | _]}} ->
        {:ok, "#{base_url()}/view?filename=#{filename}"}

      _ ->
        {:error, :no_output}
    end
  end

  defp base_url do
    Application.get_env(:xaifu, __MODULE__)[:base_url] || "http://localhost:8188"
  end
end
```

### 5.4 統一圖像生成介面

```elixir
# lib/xaifu/ai/image_generator.ex
defmodule Xaifu.AI.ImageGenerator do
  @moduledoc """
  圖像生成統一介面
  """

  alias Xaifu.AI.PromptBuilder
  alias Xaifu.AI.Providers.{Replicate, ComfyUI}

  require Logger

  @doc """
  為角色活動生成圖片
  """
  def generate_for_activity(character, activity_type, opts \\ []) do
    context = %{
      character: character,
      activity_type: activity_type,
      location: opts[:location],
      mood: opts[:mood],
      time_of_day: get_time_of_day()
    }

    prompt = PromptBuilder.build(context)

    generation_opts = %{
      positive_prompt: prompt.positive,
      negative_prompt: prompt.negative,
      width: 1024,
      height: 1024,
      seed: opts[:seed],
      reference_image: character.avatar_url,
      lora_weights: opts[:lora_weights]
    }

    Logger.info("Generating image with prompt: #{String.slice(prompt.positive, 0..100)}...")

    get_provider().generate(generation_opts)
  end

  @doc """
  檢查服務健康狀態
  """
  def health_check do
    get_provider().health_check()
  end

  defp get_provider do
    case Application.get_env(:xaifu, :image_provider, :replicate) do
      :replicate -> Replicate
      :comfyui -> ComfyUI
      provider -> provider
    end
  end

  defp get_time_of_day do
    hour = DateTime.utc_now().hour

    cond do
      hour >= 6 and hour < 12 -> "morning"
      hour >= 12 and hour < 18 -> "afternoon"
      hour >= 18 and hour < 22 -> "evening"
      true -> "night"
    end
  end
end
```

---

## 6. 圖片儲存模組

### 6.1 Storage Behaviour

```elixir
# lib/xaifu/storage/behaviour.ex
defmodule Xaifu.Storage.Behaviour do
  @moduledoc """
  儲存服務行為定義
  """

  @callback upload(binary(), String.t(), keyword()) :: {:ok, String.t()} | {:error, term()}
  @callback delete(String.t()) :: :ok | {:error, term()}
  @callback exists?(String.t()) :: boolean()
end
```

### 6.2 本地儲存

```elixir
# lib/xaifu/storage/local.ex
defmodule Xaifu.Storage.Local do
  @moduledoc """
  本地檔案儲存
  """

  @behaviour Xaifu.Storage.Behaviour

  @upload_dir "priv/static/uploads"

  @impl true
  def upload(binary, filename, _opts \\ []) do
    path = Path.join([@upload_dir, generate_path(filename)])

    # 確保目錄存在
    path |> Path.dirname() |> File.mkdir_p!()

    case File.write(path, binary) do
      :ok ->
        # 回傳可存取的 URL
        url = "/uploads/#{Path.relative_to(path, @upload_dir)}"
        {:ok, url}

      {:error, reason} ->
        {:error, reason}
    end
  end

  @impl true
  def delete(url) do
    path = url_to_path(url)

    case File.rm(path) do
      :ok -> :ok
      {:error, :enoent} -> :ok  # 檔案不存在視為成功
      {:error, reason} -> {:error, reason}
    end
  end

  @impl true
  def exists?(url) do
    path = url_to_path(url)
    File.exists?(path)
  end

  defp generate_path(filename) do
    date = Date.utc_today()
    random = :crypto.strong_rand_bytes(8) |> Base.encode16(case: :lower)
    ext = Path.extname(filename)

    "#{date.year}/#{date.month}/#{random}#{ext}"
  end

  defp url_to_path(url) do
    relative = String.replace_prefix(url, "/uploads/", "")
    Path.join(@upload_dir, relative)
  end
end
```

### 6.3 S3 儲存（可選）

```elixir
# lib/xaifu/storage/s3.ex
defmodule Xaifu.Storage.S3 do
  @moduledoc """
  S3 相容儲存（AWS S3, MinIO, R2 等）
  """

  @behaviour Xaifu.Storage.Behaviour

  @impl true
  def upload(binary, filename, opts \\ []) do
    bucket = opts[:bucket] || default_bucket()
    key = generate_key(filename)
    content_type = opts[:content_type] || "image/png"

    case ExAws.S3.put_object(bucket, key, binary,
           content_type: content_type,
           acl: :public_read)
         |> ExAws.request() do
      {:ok, _} ->
        url = build_url(bucket, key)
        {:ok, url}

      {:error, reason} ->
        {:error, reason}
    end
  end

  @impl true
  def delete(url) do
    {bucket, key} = parse_url(url)

    case ExAws.S3.delete_object(bucket, key) |> ExAws.request() do
      {:ok, _} -> :ok
      {:error, reason} -> {:error, reason}
    end
  end

  @impl true
  def exists?(url) do
    {bucket, key} = parse_url(url)

    case ExAws.S3.head_object(bucket, key) |> ExAws.request() do
      {:ok, _} -> true
      {:error, _} -> false
    end
  end

  defp generate_key(filename) do
    date = Date.utc_today()
    random = :crypto.strong_rand_bytes(8) |> Base.encode16(case: :lower)
    ext = Path.extname(filename)

    "xaifu/#{date.year}/#{date.month}/#{random}#{ext}"
  end

  defp build_url(bucket, key) do
    endpoint = Application.get_env(:xaifu, __MODULE__)[:endpoint]
    "#{endpoint}/#{bucket}/#{key}"
  end

  defp parse_url(url) do
    # 從 URL 解析 bucket 和 key
    uri = URI.parse(url)
    [_, bucket | key_parts] = String.split(uri.path, "/")
    key = Enum.join(key_parts, "/")
    {bucket, key}
  end

  defp default_bucket do
    Application.get_env(:xaifu, __MODULE__)[:bucket] || "xaifu-images"
  end
end
```

### 6.4 圖片處理模組

```elixir
# lib/xaifu/storage/image_processor.ex
defmodule Xaifu.Storage.ImageProcessor do
  @moduledoc """
  圖片處理：下載、壓縮、縮圖
  """

  require Logger

  @doc """
  從 URL 下載圖片
  """
  def download(url) do
    case Req.get(url, receive_timeout: 60_000) do
      {:ok, %{status: 200, body: body}} ->
        {:ok, body}

      {:ok, %{status: status}} ->
        {:error, {:download_failed, status}}

      {:error, reason} ->
        {:error, reason}
    end
  end

  @doc """
  壓縮圖片（需要 ImageMagick）
  """
  def compress(binary, opts \\ []) do
    quality = opts[:quality] || 85

    # 使用臨時檔案
    input_path = Temp.path!(suffix: ".png")
    output_path = Temp.path!(suffix: ".jpg")

    try do
      File.write!(input_path, binary)

      case System.cmd("convert", [
        input_path,
        "-quality", "#{quality}",
        "-strip",  # 移除 metadata
        output_path
      ]) do
        {_, 0} ->
          {:ok, File.read!(output_path)}

        {error, code} ->
          Logger.error("ImageMagick error: #{error}, code: #{code}")
          {:error, :compression_failed}
      end
    after
      File.rm(input_path)
      File.rm(output_path)
    end
  end

  @doc """
  生成縮圖
  """
  def thumbnail(binary, size \\ 256) do
    input_path = Temp.path!(suffix: ".png")
    output_path = Temp.path!(suffix: ".jpg")

    try do
      File.write!(input_path, binary)

      case System.cmd("convert", [
        input_path,
        "-thumbnail", "#{size}x#{size}^",
        "-gravity", "center",
        "-extent", "#{size}x#{size}",
        "-quality", "80",
        output_path
      ]) do
        {_, 0} ->
          {:ok, File.read!(output_path)}

        {_, _code} ->
          {:error, :thumbnail_failed}
      end
    after
      File.rm(input_path)
      File.rm(output_path)
    end
  end

  @doc """
  驗證圖片格式
  """
  def validate(binary) do
    case binary do
      <<0x89, 0x50, 0x4E, 0x47, _::binary>> -> {:ok, :png}
      <<0xFF, 0xD8, 0xFF, _::binary>> -> {:ok, :jpeg}
      <<0x47, 0x49, 0x46, _::binary>> -> {:ok, :gif}
      <<0x52, 0x49, 0x46, 0x46, _::binary-size(4), 0x57, 0x45, 0x42, 0x50, _::binary>> -> {:ok, :webp}
      _ -> {:error, :invalid_format}
    end
  end
end
```

---

## 7. Worker 整合

### 7.1 GenerateImageWorker

```elixir
# lib/xaifu/workers/generate_image_worker.ex
defmodule Xaifu.Workers.GenerateImageWorker do
  @moduledoc """
  圖像生成 Oban Worker
  """

  use Oban.Worker,
    queue: :images,
    max_attempts: 3,
    priority: 2

  require Logger

  alias Xaifu.AI.ImageGenerator
  alias Xaifu.Storage.{Local, ImageProcessor}
  alias Xaifu.Characters
  alias Xaifu.Social

  @impl Oban.Worker
  def perform(%Oban.Job{args: args}) do
    %{
      "post_id" => post_id,
      "character_id" => character_id,
      "activity_type" => activity_type,
      "location" => location,
      "mood" => mood
    } = args

    character = Characters.get_character!(character_id)

    Logger.info("Generating image for post #{post_id}, character: #{character.name}")

    with {:ok, image_url} <- generate_image(character, activity_type, location, mood),
         {:ok, image_binary} <- ImageProcessor.download(image_url),
         {:ok, compressed} <- ImageProcessor.compress(image_binary),
         {:ok, stored_url} <- storage().upload(compressed, "post_#{post_id}.jpg") do

      # 更新貼文的圖片 URL
      post = Social.get_post!(post_id)
      Social.update_post(post, %{image_url: stored_url})

      # 廣播更新
      broadcast_image_ready(post_id, stored_url)

      Logger.info("Image generated and stored: #{stored_url}")
      :ok
    else
      {:error, reason} ->
        Logger.error("Image generation failed: #{inspect(reason)}")
        {:error, reason}
    end
  end

  defp generate_image(character, activity_type, location, mood) do
    ImageGenerator.generate_for_activity(character, activity_type,
      location: location,
      mood: mood
    )
  end

  defp storage do
    Application.get_env(:xaifu, :storage, Local)
  end

  defp broadcast_image_ready(post_id, image_url) do
    Phoenix.PubSub.broadcast(
      Xaifu.PubSub,
      "social:feed",
      {:post_image_ready, post_id, image_url}
    )
  end
end
```

### 7.2 修改 GeneratePostWorker

```elixir
# lib/xaifu/workers/generate_post_worker.ex
# 在 Phase 2 的基礎上增加圖片生成

defmodule Xaifu.Workers.GeneratePostWorker do
  use Oban.Worker,
    queue: :llm,
    max_attempts: 3,
    priority: 1

  require Logger

  alias Xaifu.AI.LLM
  alias Xaifu.Characters
  alias Xaifu.Characters.Activity
  alias Xaifu.Social
  alias Xaifu.Workers.GenerateImageWorker
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

      # 排入圖片生成任務
      enqueue_image_generation(post, character, activity_type, location)

      # 廣播新貼文（先不含圖片）
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
    cond do
      String.contains?(content, ["開心", "快樂", "棒", "讚", "😊", "😄", "🎉"]) -> "happy"
      String.contains?(content, ["累", "疲", "睏", "😴", "😩"]) -> "tired"
      String.contains?(content, ["無聊", "煩", "😐", "😑"]) -> "bored"
      true -> "neutral"
    end
  end

  defp enqueue_image_generation(post, character, activity_type, location) do
    # 只有在啟用圖片生成時才排入
    if Application.get_env(:xaifu, :enable_image_generation, false) do
      %{
        post_id: post.id,
        character_id: character.id,
        activity_type: activity_type,
        location: location,
        mood: post.mood
      }
      |> GenerateImageWorker.new()
      |> Oban.insert()
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

## 8. 前端更新

### 8.1 更新 Feed LiveView

```elixir
# 在 lib/xaifu_web/live/feed_live.ex 中加入圖片更新處理

@impl true
def handle_info({:post_image_ready, post_id, image_url}, socket) do
  # 更新對應貼文的圖片
  {:noreply,
   update(socket, :posts, fn posts ->
     Enum.map(posts, fn p ->
       if p.id == post_id do
         %{p | image_url: image_url}
       else
         p
       end
     end)
   end)}
end
```

### 8.2 角色外觀設定表單

```elixir
# 在 FormComponent 中加入外觀設定欄位

<div class="mt-6 border-t pt-6">
  <h3 class="text-lg font-medium text-gray-900 mb-4">視覺設定</h3>

  <div>
    <.input
      field={@changeset[:appearance]}
      type="textarea"
      label="外觀描述"
      placeholder="描述角色的外觀特徵，如：長髮、棕色眼睛、年約25歲..."
      rows={3}
    />
    <p class="text-sm text-gray-500 mt-1">
      用於生成一致的角色圖片，建議包含：髮型、髮色、眼睛顏色、年齡、穿著風格
    </p>
  </div>

  <div class="mt-4">
    <.input
      field={@changeset[:image_prompt_prefix]}
      type="textarea"
      label="圖像提示詞前綴（進階）"
      placeholder="korean girl, long black hair, brown eyes..."
      rows={2}
    />
    <p class="text-sm text-gray-500 mt-1">
      直接用於圖像生成的英文描述，會加在所有提示詞最前面
    </p>
  </div>
</div>
```

---

## 9. 測試案例

### 9.1 Prompt Builder 測試

```elixir
# test/xaifu/ai/prompt_builder_test.exs
defmodule Xaifu.AI.PromptBuilderTest do
  use ExUnit.Case, async: true

  alias Xaifu.AI.PromptBuilder

  describe "build/1" do
    test "generates complete prompt structure" do
      character = %{
        name: "Alice",
        appearance: "young woman with long brown hair",
        image_prompt_prefix: nil
      }

      context = %{
        character: character,
        activity_type: "cafe",
        location: "星巴克",
        mood: "happy",
        time_of_day: "afternoon"
      }

      result = PromptBuilder.build(context)

      assert is_binary(result.positive)
      assert is_binary(result.negative)
      assert result.aspect_ratio == "1:1"

      # 驗證正面提示詞包含必要元素
      assert String.contains?(result.positive, "masterpiece")
      assert String.contains?(result.positive, "cafe")

      # 驗證負面提示詞包含品質控制
      assert String.contains?(result.negative, "bad anatomy")
    end

    test "includes character appearance" do
      character = %{
        name: "Bob",
        appearance: "tall man with short black hair",
        image_prompt_prefix: "asian male, handsome"
      }

      context = %{
        character: character,
        activity_type: "work",
        location: nil,
        mood: nil,
        time_of_day: nil
      }

      result = PromptBuilder.build(context)

      assert String.contains?(result.positive, "asian male")
      assert String.contains?(result.positive, "tall man")
    end
  end

  describe "build_negative_prompt/0" do
    test "includes essential negative tags" do
      negative = PromptBuilder.build_negative_prompt()

      assert String.contains?(negative, "nsfw")
      assert String.contains?(negative, "bad anatomy")
      assert String.contains?(negative, "low quality")
    end
  end
end
```

### 9.2 Image Processor 測試

```elixir
# test/xaifu/storage/image_processor_test.exs
defmodule Xaifu.Storage.ImageProcessorTest do
  use ExUnit.Case, async: true

  alias Xaifu.Storage.ImageProcessor

  # 建立測試用的小圖片
  @test_png <<0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A>>
  @test_jpeg <<0xFF, 0xD8, 0xFF, 0xE0>>

  describe "validate/1" do
    test "recognizes PNG" do
      assert {:ok, :png} = ImageProcessor.validate(@test_png <> "rest of file")
    end

    test "recognizes JPEG" do
      assert {:ok, :jpeg} = ImageProcessor.validate(@test_jpeg <> "rest of file")
    end

    test "rejects invalid format" do
      assert {:error, :invalid_format} = ImageProcessor.validate(<<1, 2, 3, 4>>)
    end
  end

  describe "download/1" do
    @tag :external
    test "downloads image from URL" do
      # 使用公開測試圖片
      url = "https://httpbin.org/image/png"

      assert {:ok, binary} = ImageProcessor.download(url)
      assert {:ok, :png} = ImageProcessor.validate(binary)
    end
  end
end
```

### 9.3 GenerateImageWorker 測試

```elixir
# test/xaifu/workers/generate_image_worker_test.exs
defmodule Xaifu.Workers.GenerateImageWorkerTest do
  use Xaifu.DataCase, async: true
  use Oban.Testing, repo: Xaifu.Repo

  alias Xaifu.Workers.GenerateImageWorker
  alias Xaifu.Characters
  alias Xaifu.Social

  setup do
    {:ok, character} = Characters.create_character(%{
      name: "Image Test",
      personality: "測試角色",
      appearance: "young woman"
    })

    {:ok, post} = Social.create_post(%{
      character_id: character.id,
      content: "Test post"
    })

    %{character: character, post: post}
  end

  test "enqueues job correctly", %{character: character, post: post} do
    assert {:ok, _job} =
      %{
        post_id: post.id,
        character_id: character.id,
        activity_type: "cafe",
        location: "星巴克",
        mood: "happy"
      }
      |> GenerateImageWorker.new()
      |> Oban.insert()

    assert_enqueued worker: GenerateImageWorker
  end

  # 注意：實際執行需要 mock 圖像生成 API
end
```

---

## 10. 配置與設定

### 10.1 環境配置

```elixir
# config/config.exs
config :xaifu,
  enable_image_generation: true,
  image_provider: :replicate,
  storage: Xaifu.Storage.Local

config :xaifu, Xaifu.AI.Providers.Replicate,
  api_key: System.get_env("REPLICATE_API_KEY")

config :xaifu, Xaifu.AI.Providers.ComfyUI,
  base_url: System.get_env("COMFYUI_URL") || "http://localhost:8188"

# 開發環境可停用圖片生成
# config/dev.exs
config :xaifu,
  enable_image_generation: false
```

### 10.2 Oban 佇列配置

```elixir
# config/config.exs
config :xaifu, Oban,
  repo: Xaifu.Repo,
  plugins: [
    Oban.Plugins.Pruner,
    {Oban.Plugins.Cron, crontab: []}
  ],
  queues: [
    default: 10,
    llm: 5,
    images: 3  # 圖片生成佇列，限制並發數以控制成本
  ]
```

---

## 11. 驗收標準

### 11.1 功能驗收

| 項目 | 驗收條件 | 狀態 |
|------|----------|------|
| Prompt 生成 | 能根據活動生成完整提示詞 | ☐ |
| 圖像生成 | 成功呼叫 API 生成圖片 | ☐ |
| 圖片下載 | 能從 URL 下載圖片 | ☐ |
| 圖片儲存 | 圖片成功儲存並可存取 | ☐ |
| 貼文更新 | 貼文正確顯示圖片 | ☐ |
| 即時更新 | 圖片生成後即時更新 Feed | ☐ |

### 11.2 品質驗收

| 指標 | 目標值 |
|------|--------|
| 圖片生成成功率 | > 95% |
| 圖片生成時間 | < 60s |
| 角色一致性評分 | > 70%（主觀評估）|
| 圖片載入時間 | < 2s |

---

## 12. 下一階段準備

完成 Phase 3 後，為 Phase 4（智慧化）做以下準備：

1. 研究 pgvector 設定與使用
2. 設計記憶存取策略
3. 規劃情緒系統
4. 設計角色間互動機制

---

*文件版本: 1.0*
*對應主文件: [00-overview.md](./00-overview.md)*

---
title: "超越預訓練：如何為特定業務需求微調 (Fine-Tune) LLM"
source: "https://dtes8617.github.io/2023/12/18/%E8%B6%85%E8%B6%8A%E9%A0%90%E8%A8%93%E7%B7%B4%EF%BC%9A%E5%A6%82%E4%BD%95%E7%82%BA%E7%89%B9%E5%AE%9A%E6%A5%AD%E5%8B%99%E9%9C%80%E6%B1%82%E5%BE%AE%E8%AA%BF%20(Fine-Tune)%20LLM/"
author:
  - "[[Jude Su]]"
published: 2023-12-18
created: 2026-05-05
description: "隨著 Meta 的 Llama 2 開放商用， 很多公司也摩拳擦掌想把 Llama 2 應用在自己的產品上。 然而 Pre-train 的模型不見的符合公司或使用者的需求， 為了求更好的表現，fine-tune LLM 就成了其中一個不錯的方法。本篇文章會介紹 Fine tune LLM 的基本概念，以及訓練的程式碼還有一些注意的事項。"
tags:
  - "clippings"
---
隨著 Meta 的 Llama 2 開放商用， 很多公司也摩拳擦掌想把 Llama 2 應用在自己的產品上。 然而 Pre-train 的模型不見的符合公司或使用者的需求， 為了求更好的表現，fine-tune LLM 就成了其中一個不錯的方法。本篇文章會介紹 Fine tune LLM 的基本概念，以及訓練的程式碼還有一些注意的事項。

## 前言

現在越來越多開源的大型語言模型釋出，除了早前僅供學術使用的模型，現在還多了像是 Llama 2, Mistral 7B 的模型可以應用在商業上。雖然有些模型透過 RLHF 的訓練方式讓模型在一開始就能流暢的對話，且表現也相當亮眼。但對於特定的需求，像是回答公司內部的資訊，或是解決不常見的任務需求，可能就不盡理想。

舉例來說，我們公司在內容審查上面，常會需要審核客戶上傳的內容。由於 Llama 2 特地加強在安全性上的表現，在我們的政策上它都會以非常嚴格的標準審視。譬如我們有個政策是不能誇大，模型在看到「最佳」、「最好」等字詞就會拒絕這個客戶的內容；甚至在遇到非常母湯的內容時，模型會直接說這個內容不安全而拒絕執行接下來的任務。我們也試過 prompt engineering 或是 in-context learning，但是結果就是不如預期。

Fine-tune 確實能讓模型更好的學習我們的問題，但是想要 Fine-tune LLM 就會有幾個痛點需要克服：

1. 硬體資源
	Llama 2 在 Hugging Face 的 [文件](https://huggingface.co/blog/llama2) 上說明要 deploy 70B 的資源至少也要兩個 A100, 儘管是 7B 也需要一個 A10G. 這還只是推論所需要的資源，更不要說訓練可能需要兩倍以上的顯卡記憶體需求。
2. 過度訓練導致破壞模型原有的能力
	一般我們想要 Fine-tune 模型都會採用監督式的訓練方式，由於模型會根據我們提供的 label 計算 loss 並更新權重，我們很難確保在訓練過程中，模型不會為了在我們的任務上獲得更準確的結果，而喪失了模型原有的能力，像是溝通能力等。

我們會用 QLoRA 跟 Instruction tuning 來解決上述的兩個問題， 並附上程式說明如何實現。

## QLoRA

QLoRA 其實就是將 LoRA adapter 應用在 Quantized 的 Pre-trained model 上。當中不管是 LoRA 或是 Quantization, 目的都是在降低訓練對記憶體的依賴，讓我們可以在更少的資源下訓練模型，且很多實驗證明這樣的效果不見得輸給直接 fine-tune 整個模型，甚至有可能更好。

### LoRA (Low-Rank Adaptation)

這裡舉個簡單的例子, 假設我們有兩層神經網路, 第一層有 3 個節點, 第二層有 5 個節點, 在不計算 bias 的情況下, 要訓練的參數共有 3x5=15 個.

LoRA 是透過在輸入及輸出層中多加一層節點數量較少的層 (秩), 來降低訓練參數. 譬如我們這裡加了一層 1 個節點的 layer, 這樣輸入/輸出的緯度維持不變, 總訓練參數在不計算 bias 的情況下, 降低至 3x1 + 1x5 = 8 個.

在舉一個實際一點的案例, 假設我們有個神經網路, 第一層有 100 個 nodes, 第二層有 100 個 nodes. 則總參數量為 100 x 100 (weights) + 100 (biases) = 10,100 個.

```python
class SimpleNetwork(nn.Module):
    def __init__(self):
        super(SimpleNetwork, self).__init__()
100
        
    def forward(self, x):
        x = torch.relu(self.fc1(x))
        return x

model = SimpleNetwork()

total_params = 0
trainable_params = 0

for param in model.parameters():
    if param.requires_grad:
        trainable_params += param.numel()
    total_params += param.numel()

print(f"Total parameters: {total_params}; Trainable parameters: {trainable_params}")
# Total parameters: 10100; Trainable parameters: 10100
```

如果我們改用 LoRA 進行訓練, 中間的秩 r 選擇 10, 則訓練參數變為 100 x 10 + 10 x 100 (weights) + 100 (biases) = 2,100 個.

```python
import loralib as lora

class LoraNetwork(nn.Module):
    def __init__(self):
        super(LoraNetwork, self).__init__()
        self.fc1 = lora.Linear(100, 100, r=10)
        
    def forward(self, x):
        x = torch.relu(self.fc1(x))
        return x
lora_model = LoraNetwork()

total_params = 0
trainable_params = 0

for param in lora_model.parameters():
    if param.requires_grad:
        trainable_params += param.numel()
    total_params += param.numel()

print(f"Total parameters: {total_params}; Trainable parameters: {trainable_params}")
# Total parameters: 12100; Trainable parameters: 2100
```

降低的訓練參數幅度高達 79.2%!

事實上在訓練時並不是用 LoRA 的層直接取代舊有的層, 而是將 LoRA 掛載在特定的層上. 基本上會凍結 Pretrained model 的 weights, 在訓練時僅對 LoRA 的權重做梯度更新, forward 時再把 LoRA 的參數加回去原本的 weights 中.### QLoRA

透過 LoRA adapter, 我們在訓練時可以很大程度的降低訓練的成本. 但現在大型語言模型動輒數十甚至數百 gigabytes, 以 Llama 2 70B chatt 的模型, 光是載下來就要 129G, 更不要說讀進顯卡記憶體需要 200G 以上. 由於 Llama 2 保存及訓練權重的 weights 精度採用 FP16, 當參數量越大 (7B, 13B, 70B), 所需要的內存就會劇烈上升. 也因此便有了降低一點精度 (FP16 → NF4) 的 Quantization 方法. 訓練時再將它轉成 16 bits 進行計算.

這樣的好處是大幅減少了讀進模型所需要的記憶體. 以 Llama 2 70B chat 來說, 顯卡記憶體只要 30 G 左右就可以把模型讀進來, 並進行預測. QLoRA 的 [paper](https://arxiv.org/abs/2305.14314) 證明了透過這樣去訓練模型, 模型的預測表現跟直接 fine-tuned model 的表現比不會比較差.

QLoRA 主要有幾個創新:

- 4-bit NormalFloat: 新的 4 bits 量化數據類型, 實驗結果優於4-bit Integers 及 4-bit Floats.
- Double Quantization: 對 quantized 的數據再做一次 quantization. 65B 的模型可以節省約 3GB
- Paged Optimizers: 使用 NVIDIA 統一內存來避免在處理長序列長度的小批量資料時, 突發性的大量內存需求。## Instruction Tuning

Meta 在訓練 Llama 2 chat 模型時, 採用的 Reinforcement Learning from Human Feedback (RLHF). 這樣的訓練方式讓模型可以學到大量知識外, 還能保有自然語言的溝通能力. 然而要採用這個方法訓練, 必須先建出一個 Reward model. 這個模型關係到 LLM 的品質, 他的參數甚至比 LLM 還要大非常多. 像我們這種市井小民實在是很難透過這種方式來 fine-tune 模型.

好在 Google 在 2021 年提出了 [Instruction fine-tuning](https://blog.research.google/2021/10/introducing-flan-more-generalizable.html) 的方法. 透過訓練模型在不同類型的問題上, 模型在沒見過任務的表現也可以顯著提升. 這樣當我們將自己想要訓練的資料合併在其他任務上一起訓練時, 模型不僅可以在特定任務上表現很好, 同時也還是能在其他任務上保有一定的表現, 不會因為訓練我們的任務而把模型原本的能力給破壞掉.### Instruction dataset

具體來說要訓練的資料集應該長什麼樣子, 其實並不限定於特定格式, 在 HuggingFace 的 [SFTTrainer](https://huggingface.co/docs/trl/sft_trainer) 也有一些示例. 這裡以 [Stanford Alpaca](https://github.com/tatsu-lab/stanford_alpaca#stanford-alpaca-an-instruction-following-llama-model) 做說明.

這裡先展示資料應該長什麼樣子:

```
Below is an instruction that describes a task, paired with an input that provides further context. Write a response that appropriately completes the request.

### Instruction:
{instruction}

### Input:
{input}

### Response:
{response}
```

主要我們需要填入的地方是大括號的部分:

- instruction: 我們要 LLM 為我們解決的問題
- input: LLM 要解決這個問題需要知道的事情, 這個部分可以留白
- response: 我們期望 LLM 給出的回答

這裡要注意的是資料的品質要求越高越好, 就像 QLoRA 的 paper 說的:

> Data quality is far more important than dataset size, e.g., a 9k sample dataset (OASST1) outperformed a 450k sample dataset (FLAN v2, subsampled) on chatbot performance, even when both are meant to support instruction following generalization.

我們看下面幾個範例會更好理解, 這些都是從 Databricks 提供的高品質開源 [instruction dataset](https://www.databricks.com/blog/2023/04/12/dolly-first-open-commercially-viable-instruction-tuned-llm) 抽出來的

**************Open QA**************

```python
Below is an instruction that describes a task. Write a response that appropriately completes the request.

### Instruction:
How do I build a campfire?

### Response:
Safety should always come first when starting a campfire. Ensure you have water or another way to extinguish your fire readily available. Use a campfire ring, pit, or clear a large area around your campfire location. Also, make sure campfires are permitted in your area.

With safety out of the way, all a fire needs are heat, fuel, and oxygen. Any combustible material will start a fire, but good campfire fuel sources include tinder, kindling, and dry logs.

Use stuff like dry pine needles, dry leaves, grasses, and wood shavings for tinder. You could also use newspaper (or any paper), dryer lint, or cardboard. For kindling, use smaller pieces of wood, such as twigs and sticks no thicker than 1 inch in diameter (about the thickness of your thumb).

Dry or well-seasoned logs will ensure a good burn. Stack the wood in a way that guarantees oxygen can easily reach your flame. Many prefer a teepee or log cabin stacking design. Now, use a heat source to create a flame or spark and apply this to your tinder. Once the tinder combusts, it should light the kindling, and the kindling should eventually ignite your logs. If all goes well, you should now be enjoying your campfire.
```

**Brainstorming**

```python
Below is an instruction that describes a task. Write a response that appropriately completes the request.

### Instruction:
Instead of making a peanut butter and jelly sandwich, what else could I combine peanut butter with in a sandwich? Give five ideas.

### Response:
Instead of jelly, try one of the following with peanut butter in a sandwich:

1. Honey

2. Raisins

3. Fruit preserves

4. Bananas

5. Sliced apples
```

**********Closed QA**********

```python
Below is an instruction that describes a task, paired with an input that provides further context. Write a response that appropriately completes the request.

### Instruction:
According to this paragraph about bull sharks, how long are male adult sharks in meters?

### Input:
Bull sharks are large and stout, with females being larger than males. The bull shark can be up to 81 cm (2 ft 8 in) in length at birth. Adult female bull sharks average 2.4 m (8 ft) long and typically weigh 130 kg (290 lb), whereas the slightly smaller adult male averages 2.25 m (7 ft) and 95 kg (209 lb). While a maximum size of 3.5 m (11 ft) is commonly reported, a single record exists of a female specimen of exactly 4.0 m (13 ft). A 3.25 m (10.7 ft) long pregnant individual reached 450 kg (990 lb). Bull sharks are wider and heavier than other requiem sharks of comparable length, and are grey on top and white below. The second dorsal fin is smaller than the first. The bull shark's caudal fin is longer and lower than that of the larger sharks, and it has a small snout, and lacks an interdorsal ridge.

### Response:
Male adult bull sharks average 2.25 meters in length.
```

這些任務會全部整理成上述的形式, 放在同一個資料集一起訓練. 由於我們自己準備的資料任務可能相對單一, 我們可以透過合併開源的資料集來增加高品質的資料. 像上述 Databricks 提供的這個資料集 ****databricks-dolly-15k**** 就是很好的資源.

## 程式實作

實作的部分主要有幾個重點, 將依序說明.

### 建構資料

這裡以開源的資料 ****databricks-dolly-15k**** 做示範, 自己的資料也可以整理成同樣的格式加進來.

```python
from datasets import load_dataset
from random import randrange

instruction_dataset_name = "databricks/databricks-dolly-15k"
dataset = load_dataset(instruction_dataset_name, split = "train")
print(dataset[randrange(len(dataset))])
```
```json
{
    "instruction": "Why are most plants green?",
    "context": "",
    "response": "Plants and algae are green due to a vital energy producing pigment in their leaves called chlorophyll. Chlorophyll molecules absorb light energy across the electromagnetic spectrum of sunshine but reflect light in the green part of the spectrum. As a result, plants appear to be green to human eyes. As an interesting side note, many animals and insects can see beyond the visible light spectrum that human's can see so that they may not view plants as being what we call 'green'.",
    "category": "general_qa"
}
```

將資料轉成 pandas 的 df, 並濾除 tokens 超過模型 context length 的資料. 而後轉換成 csv 保存起來

```python
import pandas as pd

instruct_df = pd.DataFrame(dataset)

# 讀取 LLM 的 tokenizer
model_name = "meta-llama/Llama-2-70b-chat-hf"
token = "HuggingFace 的 auth token"
tokenizer = AutoTokenizer.from_pretrained(model_name, use_auth_token = token)

# 計算每個資料多少個 tokens, 並濾除超過模型上限的資料 (Llama 2 的 context length 是 4096)
instruct_df["token_length"] = instruct_df.apply(lambda x: len(tokenizer.encode(x['instruction'] + x['context'] + x['response'])), axis=1)
instruct_df = instruct_df.loc[instruct_df['token_length'] <= 4096]

# 保存資料
instruct_df.to_csv("instruct_df.csv", index = False)
```

之後也可以透過 `datasets` 套件把資料讀進來

```python
from datasets import load_dataset

dataset = load_dataset("csv", data_files = "instruct_df.csv", split = "train")
```

### 讀取模型

要將模型透過 quantization 的方式讀進來, 我們可以靠 `bitsandbytes` 這個套件來做到這一點. 首先做參數設定

```python
import bitsandbytes as bnb
import torch
from transformers import BitsAndBytesConfig

# bitsandbytes 支援 QLoRA 的 nf4 及 double quantization
# 採用 google 的 bfloat (brain floating point) 效果更好
bnb_config = BitsAndBytesConfig(
    load_in_4bit = True,
    bnb_4bit_use_double_quant = True,
    bnb_4bit_quant_type = "nf4",
    bnb_4bit_compute_dtype = torch.bfloat16,
)
```

接下來就可以將模型讀進來了

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model = AutoModelForCausalLM.from_pretrained(
    model_name,
    quantization_config = bnb_config,
    device_map = "auto"
)

tokenizer = AutoTokenizer.from_pretrained(model_name, use_auth_token = True)
# 用 end of sentence token 作為 pad token
tokenizer.pad_token = tokenizer.eos_token
```

### 設定 LoRA Configuration

首先我們必須找出要替換掉的神經網路, 根據 QLoRA 的那篇 paper 說, 選擇全部的 linear layers 效果比只選擇 adaptation 的效果來的好 (ref. [https://arc.net/l/quote/ediwtoyl](https://arc.net/l/quote/ediwtoyl)). 因此我們這裡就選擇所有的 linear layers.

```python
from peft import LoraConfig

target_module_type = bnb.nn.Linear4bit
target_modules = set()
for name, module in model.named_modules():
    if isinstance(module, target_module_type):
        names = name.split('.')
        target_modules.add(names[0] if len(names) == 1 else names[-1])
target_modules = list(target_modules)

# config the LoRA adapater
lora_config = LoraConfig(
        r = 32,
        lora_alpha = 64,
        target_modules = target_modules,
        lora_dropout = 0.2,
        bias = "none",
        task_type = "CAUSAL_LM",
    )
```

### 建立數據轉換函數

為了將 dataset 轉成 instruction 的形式, 我們可以在訓練時添加轉換的函數讓模型在訓練前自動轉換資料, 以下是轉換的函數.

```python
# Stanford Alpaca 的形式
INTRO_BLURB = "Below is an instruction that describes a task, paired with an input that provides further context. Write a response that appropriately completes the request."
INTRO_BLURB_WITHOUT_INPUT = "Below is an instruction that describes a task. Write a response that appropriately completes the request."
INSTRUCTION_KEY = "### Instruction:"
INPUT_KEY = "### Input:"
RESPONSE_KEY = "### Response:"
    
def formatting_prompts_func(example):
    output_texts = []
    for i in range(len(example['instruction'])):
        blurb = f"{INTRO_BLURB}" if example["input"][i] else f"{INTRO_BLURB_WITHOUT_INPUT}"
        instruction = f"{INSTRUCTION_KEY}\n{example['instruction'][i]}"
        input_context = f"{INPUT_KEY}\n{example['input'][i]}" if example["input"][i] else None
        response = f"{RESPONSE_KEY}\n{example['output'][i]}"

        parts = [part for part in [blurb, instruction, input_context, response] if part]
        formatted_prompt = "\n\n".join(parts)
        output_texts.append(formatted_prompt)

    return output_texts
```

### 訓練模型

所有程序準備就緒後, 我們就可以開始來訓練模型了.

```python
from transformers import EarlyStoppingCallback
from peft import get_peft_model, prepare_model_for_kbit_training, PeftModel, AutoPeftModelForCausalLM
from trl import SFTTrainer, DataCollatorForCompletionOnlyLM

# 切割 train/validation datasets
train_dataset = dataset.train_test_split(test_size=0.2, seed=42)['train']
val_dataset = dataset.train_test_split(test_size=0.2, seed=42)['test']

# 設定 DataCollatorForCompletionOnlyLM 讓模型只拿 Response 後面的文字來計算 loss
# 如果沒有設定的話, 模型會將 prompt 也計入 loss function 的計算中, 請依照任務需求去做選擇
response_template = "\n### Response:"
response_template_ids = tokenizer.encode(response_template, add_special_tokens=False)[2:]
collator = DataCollatorForCompletionOnlyLM(response_template_ids, tokenizer=tokenizer, mlm = False)

# 轉換模型成 PEFT 支援的模型
model.gradient_checkpointing_enable()
model.config.pretraining_tp = 1 
model.config.use_cache = False
model = prepare_model_for_kbit_training(model)
model = get_peft_model(model, lora_config)

# 設定 training args
training_args = TrainingArguments(
        per_device_train_batch_size = 2,
        gradient_accumulation_steps = 8,
        warmup_steps = 2,
        max_steps = 1056,
        learning_rate = 2e-4,
        fp16 = True,
        logging_steps = 1,
        output_dir = "./results",
        optim = "paged_adamw_32bit",
        load_best_model_at_end = True,
        evaluation_strategy = IntervalStrategy.STEPS,
        eval_steps = 10
    )

# 設定 trainer
trainer = SFTTrainer(
    model = model,
    train_dataset = train_dataset,
    eval_dataset=val_dataset,
    args = training_args,
    data_collator = collator,
    callbacks = [EarlyStoppingCallback(early_stopping_patience=3)],
    max_seq_length = 4096,
    formatting_func=formatting_prompts_func,
)

train_result = trainer.train()
metrics = train_result.metrics
trainer.log_metrics("train", metrics)
trainer.save_metrics("train", metrics)
trainer.save_state()

# 儲存模型
print("Saving last checkpoint of the model...")
os.makedirs("./results", exist_ok = True)
trainer.model.save_pretrained(output_dir)
```

### 合併 Adapter

建議在合併之前將 kernel 清空, 因為合併需要將原始模型讀進來, 並將 adapter 的權重合上去, 所需要的記憶體會相當大. 可能有人會問我直接用掛載的就好, 為什麼一定要合併回去原始的模型, 主要是因為現在如果採用掛載的, LLM 推論的時間會非常長 (ref. [https://arc.net/l/quote/jyetbfcv](https://arc.net/l/quote/jyetbfcv)), 基本上根本沒辦法當作服務使用, 目前不確定是 bugs 還是什麼, 建議合併比較保險.

```python
# 將模型讀入, max_memory 可以選擇 gpu 或是 cpu 可以使用多少 memory, 確保不會因為 memory 不足爆掉
model = AutoPeftModelForCausalLM.from_pretrained(output_dir, 
                                                 device_map = "auto", 
                                                 torch_dtype = torch.bfloat16,
                                                 max_memory={0: "80GIB", 1: "80GIB","cpu": "75GIB"},
                                                 cache_dir="/cache/")
# 合併 LoRA adapter
model = model.merge_and_unload()
model.save_pretrained("./results_merged", safe_serialization = True)
tokenizer.save_pretrained("./results_merged")
```

## 結論

我們公司確實在 fine-tuned LLM 上取得很亮眼的成績, 而且 LLM 與以往的語言模型不同, 透過 instruction-tuning 的訓練方式, 也可以把多種不同的任務需求都放在同一個資料集讓 LLM 學習, 也就是不同任務只要一個模型就可以處理. 唯一需要注意的是訓練的資料集品質, 只要能確保是高品質的資料集, 相信訓練出來的 LLM 也可以讓你眼睛為之一亮的.
我想調整 Workflow 與 Generation 的設計

原本是將 Workflow 做為如同 Template 的格式來規範 Generation 的 input 結構
想改成：

Generation 不再依 Workflow 來限制有以有哪些 input

在新增/編輯/複製 Generation時，可以自由增減 Input (GenerationInput) 的數量與內容

Workflow Schema 的修改與作用
- 定義 json content：已有，用 json_content 欄位
- 定義 base prompt：已有，用 prompt 欄位
- 定義 prompt identifier：新增 prompt_identifier 欄位，string 型態，預設值為 "171"
- 定義 filename prefix identifier：新增 filename_identifier_prefix，string 型態，預設值為 "16"
- 定義 laten_identifier，新增 latent_identifier，string 型態，預設值為 "14"

Workflow 的 has many WorkflowInputs 基本上已沒有作用

Generation Schema 的修改：
- 增加 width 欄位：integer 型態，預設為 1024
- 增加 height 欄位：integer 型態，預設為 1536

GenerationInput Schema 的修改
- 刪除 identifier 的欄位


調整 Generation 的新增/編輯/複製 功能的調整

- 新增/編輯 Generation 時，它的 GenerationInput 可以自由增加/刪除，不受 Workflow 的限制
- Generation Form 裡加上 width 與 height 的輸入欄位

調整 Generation 在產生 Processor 生成 generation_instance 的邏輯
- 原本每個 generation_input 的 identifier 已刪除，改成使用 GenerationInput 的 Generation 的 Workflow 的 prompt_identifier




Generation

```
{

  "9": {

    "_meta": {

      "title": "K采样器"

    },

    "inputs": {

      "cfg": 5,

      "seed": 1116811957111570,

      "model": [

        "132",

        0

      ],

      "steps": 25,

      "denoise": 1,

      "negative": [

        "42",

        0

      ],

      "positive": [

        "11",

        0

      ],

      "scheduler": "karras",

      "latent_image": [

        "14",

        0

      ],

      "sampler_name": "dpmpp_2m_sde"

    },

    "class_type": "KSampler"

  },

  "11": {

    "_meta": {

      "title": "CLIP文本编码"

    },

    "inputs": {

      "clip": [

        "37",

        0

      ],

      "text": [

        "171",

        0

      ]

    },

    "class_type": "CLIPTextEncode"

  },

  "14": {

    "_meta": {

      "title": "空Latent图像（SD3）"

    },

    "inputs": {

      "width": 1024,

      "height": 1536,

      "batch_size": 1

    },

    "class_type": "EmptySD3LatentImage"

  },

  "16": {

    "_meta": {

      "title": "保存图像"

    },

    "inputs": {

      "images": [

        "35",

        0

      ],

      "filename_prefix": "2026-02-04_091944_"

    },

    "class_type": "SaveImage"

  },

  "34": {

    "_meta": {

      "title": "Checkpoint加载器（简易）"

    },

    "inputs": {

      "ckpt_name": "illustrious\\beautifulAnimeBP_v10.safetensors"

    },

    "class_type": "CheckpointLoaderSimple"

  },

  "35": {

    "_meta": {

      "title": "VAE解码"

    },

    "inputs": {

      "vae": [

        "51",

        0

      ],

      "samples": [

        "9",

        0

      ]

    },

    "class_type": "VAEDecode"

  },

  "37": {

    "_meta": {

      "title": "设置CLIP最后一层"

    },

    "inputs": {

      "clip": [

        "34",

        1

      ],

      "stop_at_clip_layer": -2

    },

    "class_type": "CLIPSetLastLayer"

  },

  "42": {

    "_meta": {

      "title": "CLIP文本编码"

    },

    "inputs": {

      "clip": [

        "37",

        0

      ],

      "text": "bad quality, simple background, sweat, shiny hair, shiny skin, 3d, lowres, bad quality, worst quality, sketch, jpeg artifacts, ugly, poorly drawn, blurry, transparent background, censored, artist name, signature, watermark, negativeXL_D, unaestheticXL_bp5, More than five fingers in one hand, More than 5 toes on one foot, hand with more than 5 fingers, hand with less than 4 fingers, lowres, bad quality, worst quality, sketch, jpeg artifacts, ugly, poorly drawn, blurry, transparent background, censored, (simple background:1.6), artist name, signature, anatomical nonsense, bad anatomy, interlocked fingers, extra fingers,\nlowres, bad quality, worst quality,bad anatomy, sketch, jpeg artifacts, ugly, poorly drawn,blurry, transparent background, tears, censored, (simple background:1.6), artist name, signature, watermark, pipes, tail, parking brake, multiple views, tattoon, bad head, twin heads, multiple_girls"

    },

    "class_type": "CLIPTextEncode"

  },

  "51": {

    "_meta": {

      "title": "加载VAE"

    },

    "inputs": {

      "vae_name": "sdxlVAE_sdxlVAE.safetensors"

    },

    "class_type": "VAELoader"

  },

  "132": {

    "_meta": {

      "title": "動漫化"

    },

    "inputs": {

      "model": [

        "134",

        0

      ],

      "lora_name": "illustrious\\style\\8be1e5d2cc7cd938037337603fb51565.safetensors",

      "strength_model": 1

    },

    "class_type": "LoraLoaderModelOnly"

  },

  "134": {

    "_meta": {

      "title": "皮膚光亮"

    },

    "inputs": {

      "model": [

        "135",

        0

      ],

      "lora_name": "illustrious\\style\\ShinyOiledSkin_v20-LyCORIS.safetensors",

      "strength_model": 1

    },

    "class_type": "LoraLoaderModelOnly"

  },

  "135": {

    "_meta": {

      "title": "衣飾細節、對比"

    },

    "inputs": {

      "model": [

        "138",

        0

      ],

      "lora_name": "illustrious\\style\\add-detail-xl.safetensors",

      "strength_model": 0.8

    },

    "class_type": "LoraLoaderModelOnly"

  },

  "138": {

    "_meta": {

      "title": "增強細節，髮絲"

    },

    "inputs": {

      "model": [

        "139",

        0

      ],

      "lora_name": "illustrious\\style\\Detail_Tweaker_Illustrious_BSY_V3.safetensors",

      "strength_model": 1

    },

    "class_type": "LoraLoaderModelOnly"

  },

  "139": {

    "_meta": {

      "title": "Hiten Artist"

    },

    "inputs": {

      "model": [

        "34",

        0

      ],

      "lora_name": "illustrious\\style\\hitenstyle.safetensors",

      "strength_model": 1

    },

    "class_type": "LoraLoaderModelOnly"

  },

  "171": {

    "_meta": {

      "title": "Custom Action"

    },

    "inputs": {

      "value": "source anime, masterpiece best quality, 8K, UHD, masterpiece, ultra-HD, very aesthetic, 8K, high detail, depth of field, best quality, unholy-aesthetic,masterpiece,best quality,amazing quality,very aesthetic,absurdres,ultra detailed face,ultra detailed eyes, 1girl,, short_hair, pink-brown hair, curled hair ends, straight even bangs slightly side-parted, black cross hair clips on both sides of forehead, black beret on top of head, pink satin ribbon bow on beret brim, pink round gem and gold small stars on beret top, large breast, White blouse, cropped blouse, unbutton blouse, black satin ribbon bow at collar with blue gem pendant, blouse front buttons fully open revealing black lace bra, pink jacket, black cross straps and small bows on jacket cuffs, midriff exposed, white low-waist panties, jacket draped as shawl, no stockings, barefoot, Arms crossed over chest, right leg extended with toe touching ground, left leg bearing weight, head tilted right, slight upturned mouth corners"

    },

    "class_type": "PrimitiveStringMultiline"

  }

}
```

請實作 Generations.store_generated_images 的內容

這個方法會傳入一個 generation 和多個 image urls
要將 image urls 的圖片內容下載下來
建立成  generated_image 並

- 與 generation 建立關聯
- generated_image 的 project 也等於 generation.project
- prompts 則是從 generation.inputs 裡取得每個對應的 prompt_id

下載的圖片先存在本地端，之後會實現可切換成用 aws s3 storage
請參考 Xaifu.Uploaders.GeneratedImage



請對原圖及各種 versions 做檔案大小的優化，可以移除不必要的檔案資訊
- 原圖，不做任何裁切，
- thumb: 維持原比例，但縮至(寬或高)最大 360
- cover: 縮放至寬度為 360


GeneratedImage 的 width 和 height 在 create_generated_images 時沒有存到
另外請在 GeneratedImage 增加 subject_id 來將 GeneratedImage 關聯至 subject，
即在 create_generated_images 時，把 GenerateImage 的 subject 設為同 Generation 的 subject

- Subject has many images (GeneratedImage)
- GeneratedImage belongs_to Subject (optional)



新增 SubjectImage 來關聯 Subject 與 GeneratedImage

Subject has many images(GeneratedImage)

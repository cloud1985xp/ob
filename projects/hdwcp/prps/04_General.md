
我要對 app layout 下的功能也開始更新套用 InfiniteScrollList 的功能
參考 ai_docs/ui/crud-pattern.md，但只先套用 index 的部分

我打算一步一步來，不要一次套到全部的功能，以便過程中再檢查這個 macro 是否有改善之步
請先套用到以下的功能

- categories
- models, 要加上 filter，用 category
- systems, 要加上 filter，用 category
- operations, 要加上 filter，用 category
- colors, 要加上 filter，用 category
- fabrics要加上 filter，用 category
- accessories
- category_options, 要加上 filter，用 category
- serials, 要加上 filter，用 category

請確保 infinite scroll, term search，category filter (如果有的話)，以及排序的功能都要正常運作。

接下來將 InfiniteScrollList 套用到 contacts 的 dealers 功能，包括 infinite scroll, term search, category filter 和排序的功能

目前 dealers 已有用 scope 做篩選的功能
我希望保有 scope 的篩選，同時加上 InfiniteScrollList 用 DealerType 做為 filter


# (WIP) 顯示 Dealer
- 要可以自訂顯示區域、地區、編號、名稱






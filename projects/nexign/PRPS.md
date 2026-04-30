請先理解專案結構與目前的功能規格
目前 admin 後台缺少 products 的管理(CRUD)功能
請參照後台其他 resources 的管理
為 admin 增加 admin/products 的 CRUD 功能

然後評估以下的設計變動是否合理：

Product 與 Plans 改為一對多的關係
表示每個 product 會各自定其自己的 plans，不與其他 product 共用

以下所有資
OAuth Application 必須 belongs to product

以下所有資源，應該都

實
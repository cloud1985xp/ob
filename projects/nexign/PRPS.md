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



# 優化 Admin 功能

請處理 admin 端的以下需求
- plans 的瀏覽畫面，會出現以下錯誤，請修正
```
undefined method 'active?' for an instance of Products::Price

/Users/aaron.kuo/projects/nexign/app/views/admin/plans/show.html.erb:83
```

- groups 下要可以
	- 管理 group 的 subscriptions
		- 新增或編輯 subscription，要觸發 Webhook dispatch
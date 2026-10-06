- 1passwordk 取得 wp-admin 登入資訊
	- aktsk -> TW-Akatsuki -> 官網WP Admin
- 本地安裝好 docker
- 本地安裝好 mysql GUI tool
	- Sequel Ace or Sequel Pro 或任何你習慣的 dba 工具
- 下載 production/staging db 資料


Docker 指令

mysql
```
docker run --name mysql8 -p 3306:3306 -e MYSQL_ROOT_PASSWORD={your_password} -v /Users/{your_data_path}:/var/lib/mysql -d mysql:8.4 --mysql-native-password=ON
```


wordpress
```
docker run -it --rm -v /Users/{your_project_path}/wordpress:/var/www/html -p 8081:80 wordpress:latest
```

建立 database
- create MySQL database: `aktsk_com_tw_wpdb`
- 將下載的 production/staging db 資料匯入 aktsk_com_tw_wpdb

修改 wordpress config

- copy wp-config.staging.php to wp-config.php
- 修改 wp-config.php 裡的
	- DB_HOST -> `host.docker.internal`
	- DB_PASSWORD -> `{your_password}`


Nancy 在 

加上並註冊了 enqueue_business_page_assets 
來實現

# Migrate to use Astro

這個專案現下是用 wordpress 實現的一個品牌官網，位於 ./wordpress 路徑下
網站的預覽可以至 http://localhost:8081/ 或 https://www.aktsk.com.tw/ 瀏覽

我想要改成用 astro (https://astro.build/) 這個解決方案，將整個網站 migrate 成產生靜態網頁的方式

其中原本的 NEWS (/new) 是用 wordpress 的後台管理讓用戶線上編輯的內容
我打算改成用 Astro 內建的 Content Collections（內容集合） 與 Dynamic Routes（動態路由）來實現，請幫我規劃在指定的目錄下讓使用者編寫內容(markdown)，然後 build 成對應路徑下的文件

請確保整體的視覺設計風格依照舊版的網頁
但可以優化或修正 html 語法和語意，來符合 SEO 的規範

請考慮以下工作內容，或提出建議與我討論：

1. 理解舊專案與需求
2. 評估套用 Astro 並建置新專案結構
3. 依專案需求和內容，建立並撰寫 CLAUDE.md 
4. 將舊專案 migrate 成新版本
5. 建立日後管理維護與操作的說明文件

除此之外，後續還有 CI/CD (用 Github Action) 的建置，可以之後再討論
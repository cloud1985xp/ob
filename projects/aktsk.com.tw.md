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
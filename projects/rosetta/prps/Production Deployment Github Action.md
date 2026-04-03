
請調整 .github/workflows/deploy_production.yml

調整部署 (deploy job) 時的行為
在執行 deploy step 前，加上額外的動作
會對指定的 (image_tag) image 
另外加上兩個 tag：

- production
- yyyyMMdd-hhmmss

然後在 deploy step 中，kustomize 設定 image 的動作
請修改成用剛才的 yyyyMMdd-hhmmss 的 image 版本

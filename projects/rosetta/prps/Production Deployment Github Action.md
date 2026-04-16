
請調整 .github/workflows/deploy_production.yml

調整部署 (deploy job) 時的行為
在執行 deploy step 前，加上額外的動作
會對指定的 (image_tag) image 
另外加上兩個 tag：

- production
- yyyyMMdd-hhmmss

然後在 deploy step 中，kustomize 設定 image 的動作
請修改成用剛才的 yyyyMMdd-hhmmss 的 image 版本

# 調整 Github Workflow

包括：
- 將 .github/workflows/build.yml 拆成兩個 workflow
	- build_image.yml
	- deploy_staging.yml
- 更新 deploy_production.yml

目標：
build_image.yaml 將只進行 build image
deploy_staging 與 deploy_production 則是會呼叫 build_image 後，再執行 deployment

## Build Image
build image 需要傳入參數包括：
- branch: 必填，會使用該 branch 的 code 來 build image
- tag: 選填，會使用這個 tag 當作 image tag，若沒有設定，就用 branch 對應的 commit sha1 當作 tag

執行 build step 
build image 完成後會 push image & tag 到 image registry，
並且輸出回傳 push image 的 tag


## Deploy to Staging
可接受 input:
- 要使用的 branch，若沒有指定就用當下執行 workflow 的 branch
- 是否要 skip build image

若沒有 skip build image
- 先呼叫 build image workflow，傳入 branch 參數
- 得到 build 完的 image tag
- 將剛才 build 的 image push 成 latest 的 tag

用 latest tag 的 image 部署到 staging

## Deploy to Production
可接受 input:
- 要使用的 branch，若沒有指定就用當下執行 workflow 的 branch
- 是否要 skip build image

預設用 latest 做為要部署的 image tag
如果沒有 skip build image
- 呼叫 build image workflow，傳入 branch 參數
- 將 build 完得到的 image tag 當作要部署的 image tag

決定 deploy_tag = yyyymmdd--hhMMSS
對要部署的 image tag，再打上 deploy_tag
將要部署的 image 再打上 production 的 tag

用 deploy_tag 的 image 部署到 production


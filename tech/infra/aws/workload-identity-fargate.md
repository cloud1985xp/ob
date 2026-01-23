---
tags:
  - aws
  - infrastructure
created: 2024-01-01
updated: 2025-01-23
status: active
---


# Workload Identity Federation

Ref:
https://medium.com/google-cloud/hey-google-cloud-api-trust-my-aws-application-f306972f10db
https://medium.com/@kellenjohn175/how-to-guides-multicloud-identity-federation-azure-to-gcp-186bbbbb8abf

流程
1. App (Instance Profile) 向 identity provider(AWS) 進行身份驗証 authentication，
2. 取得憑證 (AWS credentials)
3. App 以此 credentials 向 GCP 的 security token service (sts.googleapis) 將 AWS 給的 Credentials 換發取得 Access token (代表對應的 service account)
	1. 中間可以用 identity provider 的 credentials 中的屬性(attribute) 做 attribute mapping，將
	2. sts service 提取需要的屬性，映射到 google cloud 的身份和資源模型
		1. 其中 idp 的 資源叫 assertion，用 assertion.xxx 來代表取得 xxx 屬性
		2. 提取出 mapping 成 gcp attribute 的目標屬性有：
			1. google.subject
			2. google.groups
			3. google.user_id
			4. 其他自訂義屬性，可用於特定 iam 條件或策略
		3. 例如我們就是用 idp credentials 中的 aws:role 這項屬於，提取映射到 gcp 的自訂屬性，然後再 iam binding 中定義條件性政策，基於這個自訂屬性來授權可以存取的資源權限
4. App 用取得的 access token (代表對應的 service account) 向 google 存取資源


principalSet://iam.googleapis.com/${google_iam_workload_identity_pool.aws.name}/attirbute.aws_role/common-jenkins-JenkinsECSRole-YX0I8H2D8B6

建立 pool
建立 identity provider，ex: AWS, Azure, OIDC, kubernete clusters ... etc
- https://cloud.google.com/iam/docs/workload-identity-federation
- 依 provider 來決定要用哪種 type，不同的 type 的 provider 會有不同的設定
	- AWS
	- OpenID Connect(OIDC), ex Azure 要用這種
	- SAML
- 設定 attribute mapping
-


## ISHIN Works

可利用 gcloud create-cred-config 指令來產生 credential config

```
gcloud iam workload-identity-pools create-cred-config \
  projects/61735687619/locations/global/workloadIdentityPools/aws-prod/providers/aws-ishin-tw-prod  \
  --service-account=redash@ishin-168508.iam.gserviceaccount.com \
  --output-file=production-redash.config.json \
  --aws \
  --enable-imdsv2
```

https://cloud.google.com/sdk/gcloud/reference/iam/workload-identity-pools/create-cred-config#--aws

注意是否需要啟用 imdsv2
例如，目前的 redash 是不需要啟用的

```
{
  "type": "external_account",
  "audience": "//iam.googleapis.com/projects/61735687619/locations/global/workloadIdentityPools/aws-prod/providers/aws-ishin-tw-prod",
  "subject_token_type": "urn:ietf:params:aws:token-type:aws4_request",
  "token_url": "https://sts.googleapis.com/v1/token",
  "credential_source": {
    "environment_id": "aws1",
    "region_url": "http://169.254.169.254/latest/meta-data/placement/availability-zone",
    "url": "http://169.254.169.254/latest/meta-data/iam/security-credentials",
    "regional_cred_verification_url": "https://sts.{region}.amazonaws.com?Action=GetCallerIdentity&Version=2011-06-15",
    "imdsv2_session_token_url": "http://169.254.169.254/latest/api/token"
  },
  "service_account_impersonation_url": "https://iamcredentials.googleapis.com/v1/projects/-/serviceAccounts/bq-loader@ishin-168508.iam.gserviceaccount.com:generateAccessToken"
}

```


credential_source 裡定義了如何取得 identity provider 的身份憑証
會搭配 environment_id
例如這裡表明了用 aws + url
所以會用 restful api 的方式向 url 去取得 aws 憑証

另一個例子是用 "file" 參數，指向某一個身份憑証檔案

regional_cred_verification_url 提供地區專用的 url

在 Fargate 上使用會遇到 error

> HTTPConnectionPool(host='169.254.169.254', port=80): Max retries exceeded with url: /latest/meta-data/local-ipv4 (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x7f086aa8d438>: Failed to establish a new connection: [Errno 22] Invalid argument',))


別人也遇到
https://stackoverflow.com/questions/57065458/cannot-access-instance-metadata-from-within-a-fargate-task
https://stackoverflow.com/questions/70194948/connection-error-from-aws-fargete-to-gcp-bigquery-by-using-workload-identity

許多人在發 Feature Request
https://github.com/googleapis/google-auth-library-nodejs/issues/1594


追 code 發現無法
https://github.com/googleapis/google-auth-library-python/blob/main/google/auth/aws.py#L432

在 fargate 上取得 metadata 的 endpoint 是：

"http://169.254.170.2${AWS_CONTAINER_CREDENTIALS_RELATIVE_URI}"

傳統 ec2
http://169.254.169.254/latest/meta-data/iam/security-credentials

會取得session token

```
{
  "RoleArn": "arn:aws:iam::912510024466:role/staging-bq-loader-BqLoaderTaskRole-57Yyjee4Okpj",
  "AccessKeyId": "xxxx",
  "SecretAccessKey": "xxxx",
  "Token": "oooooo",
  "Expiration": "2024-12-04T08:50:39Z"
}
```


目前看來大家的作法都是改取得 container credentials
拿到改用 access key / secret 來做 authentication

ex:
https://stackoverflow.com/questions/57065458/cannot-access-instance-metadata-from-within-a-fargate-task

library team 叫你用 custom provider (然後去做上面說的事)
但那是 java 程式裡用的，從 cli 無法
https://github.com/googleapis/google-auth-library-java?tab=readme-ov-file#using-a-custom-supplier-with-aws

ref
ECS Task Metadata Endpoint
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-metadata-endpoint-v4-fargate.html
ECS Container Crednetials
https://docs.aws.amazon.com/sdkref/latest/guide/feature-container-credentials.html

## 如何在 Fargate 上 debug

如同 docker exec
也有
> kubectl exec --cluster my-cluster ---pod pod_id bash


aws ecs execute-command 的指令對「特定 container 執行 command」

前提
- 你的 IAM role 要有權限: ecs:ExecuteCommand
- ECS task (service) 執行時要 enable execute command


```
aws ecs run-task --cluster staging-bq-loader \
  --count 1 \
  --enable-execute-command \
  --launch-type FARGATE \
  --network-configuration='{"awsvpcConfiguration": { "subnets": ["subnet-04d4af22eeb7f80c1"], "securityGroups": ["sg-0a49fac1f0dd21edd"], "assignPublicIp": "DISABLED" }}' \
  --overrides='{"containerOverrides": [{ "name": "generate-jobs", "command": ["sleep"] }] }' \
  --task-definition arn:aws:ecs:us-west-1:912510024466:task-definition/staging-bq-loader-generate-jobs:68
```

得到 task_arn
```
task_arn=arn:aws:ecs:us-west-1:912510024466:task/staging-bq-loader/0579b0afb3a04bb49c5eaf3c8428393d
```

```
aws ecs execute-command --cluster staging-bq-loader --task $task_arn --container generate-jobs --interactive --command "/bin/sh"
```


Ref:
https://www.internetkatta.com/debugging-into-aws-ecs-task-containers-what-you-need-to-know
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-exec.html
https://docs.aws.amazon.com/zh_tw/AmazonECS/latest/developerguide/ecs-exec.html

---
tags:
  - archive
  - 2022
created: 2022-01-01
updated: 2025-01-23
status: archived
---

Recap 一下今天有關 BQ Sender 的分享
# Setup GCP

用 Terraform 建立此次建置 BQ Sender 在 GCP 所需的 Infra 資源
包括 Network(VPC)/Subnet, Cloud Storage, Cloud DNS, Service Accounts, KMS

# 建立 GKE Cluster

簡述 Kubernetes vs Google Kubenetes Engine 的關係，以及 GKE Standard vs Autopilot 模式
使用了 Workload Identity ，它提供了給第三方使用的識別系統(ex GithubAction to AWS/GCP)，
並在 bq-sender 專案中 GCP 與 K8S 之間的 service account 如何透過 workload identity 綁定，順帶提到了 Zero Trust

# 部署 BQ Sender 到 GKE
列舉了在 bq-sender 中所使用的 K8S Resources,
特別是 StatefullSet，為了讓 fluentd 服務的 pod 擁有固定識別，以確保它的 buffer 機制正常運作
另還有常用的資源如 ConfigMap (存放設定值), Secret (存放敏感資料), Service (定義對外服務接口)

特別說明 kubectl apply -f 這個指令的使用
以及 k8s 本身資源文件的在實務上的使用缺陷，因而導入了 kustomize 這個工具來達到 多環境/樣版化/參數化的需求
包括 kustomize build 以及 kustomize edit 指令的用法

另 BQ Sender 使用的 fluentd 中啟用了 + TLS 加密，所以會需要
建立一組 certificate -> 將證照與金鑰上傳至 AWS  SSM
(補充：這個組 cert 也會需要在資料發送端使用，即在 mikoto build app 端的 fluentd image 時要使用到，因此存一份在 SSM)
這裡利用 kubesec + GCP KMS Key 來加密 certificate 產生可以被 commit 的 secret.enc.yaml
在部署時同樣 kubesec 解密轉回 secret.yaml 然後 apply 到 k8s cluster

最後，在 Build BQ Sender 使用的 fluentd container image 發生了 ruby version conflict 問題
原因來自 fluentd/v1.10 的 base image 所用的 tzinfo 套件，跟一些社群開發的 fluentd plugin 用的 gem 有版本衝突
導修 bundler 解析後所使用的 activesupport 版本會與 base image 所用的 ruby 版本有相依性問題
後來解決版法是直接調整我們使用的 fluentd-bigquery plugin 中的 activesupport 版本

資訊量有點多，而各位大部分還沒接觸過 Kubernetes，可能會比較難意會
許多東西我也是剛接觸也可能認知不全面，歡迎大家討論指教

# 參考網址資料

## Mikoto BQ Sender Repo
https://github.com/aktsk/bigquery-sender/tree/mikoto
https://github.com/aktsk/bigquery-sender/blob/mikoto/docs/mikoto.md

## Fluentd Bigquery Plugin (forked from other repo)
https://github.com/aktsk/fluent-plugin-bigquery

## Kubesec
https://github.com/shyiko/kubesec

## Fluentd tzinfo issue
https://docs.fluentd.org/quickstart/faq#fluentd-raises-tzinfo-conflict-error-after-installed-plugins

## Other plugin has same issue
https://github.com/fluent/fluent-plugin-sql/pull/101/files

# 其他資源

## Autopilot
https://cloud.google.com/blog/products/containers-kubernetes/introducing-gke-autopilot
https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview
https://cloud.google.com/kubernetes-engine/docs/how-to/creating-an-autopilot-cluster

## Workload Identity
https://cloud.google.com/iam/docs/workload-identity-federation
https://cloud.google.com/blog/products/identity-security/google-cloud-zero-trust-security-talks-available-on-demand
https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity

## Zero Trust
https://cloud.google.com/blog/topics/developers-practitioners/what-zero-trust-identity-security

## StatefulSet
https://www.baeldung.com/ops/kubernetes-deployment-vs-statefulsets
https://medium.com/@tasslin/kubernetes%E7%9A%84deployments%E8%88%87-statefulsets%E8%88%87daemonsets%E4%B9%8B%E9%96%93%E5%B7%AE%E7%95%B0-829fce141fad

## Fluentd + TLS
https://chsasank.com/secure-fluentd-python.html

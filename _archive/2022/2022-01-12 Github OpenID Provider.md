---
tags:
  - archive
  - 2022
created: 2022-01-01
updated: 2025-01-23
status: archived
---

@tw_ishin_se
這是在 Company AWS 上設定 Github OpenID 的 PR
https://github.com/Akatsuki-Taiwan/company-service-stack/pull/6

先推這個版本上來，我覺得還有很多改良的空間
CFn 的部分，要怎麼方便管理 policy，方便再未來增加新 project 的 bucket 跟 distribution
Github Action 的部分，應該要把 deploy.sh 的程式移到 workflow 裡，還有讓 workflow 方便共用，一些參數可以自動取得或設成 input

相關參考資源
Github, 官方介紹
https://github.blog/changelog/2022-01-13-github-actions-update-on-oidc-based-deployments-to-aws/
https://github.blog/changelog/2021-10-27-github-actions-secure-cloud-deployments-with-openid-connect/
https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services
https://docs.github.com/ja/enterprise-cloud@latest/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect

AWS, 建立 Provider 與 IAM Role
https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html
https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc_verify-thumbprint.html
https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html

一些 Example, (大多是用 terraform 操作 :joy: )
https://scalesec.com/blog/identity-federation-for-github-actions-on-aws/
https://awsteele.com/blog/2021/09/15/aws-federation-comes-to-github-actions.html
https://blog.tedivm.com/guides/2021/10/github-actions-push-to-aws-ecr-without-credentials-oidc/
https://medium.com/engineers-haven/github-actions-aws-oidc-2a35a6664d25

# GCP - Keyless Authentication with Github Action
https://cloud.google.com/blog/products/identity-security/enabling-keyless-authentication-from-github-actions
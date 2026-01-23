---
tags:
  - infrastructure
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Service Mesh

Created: 2022年5月27日 上午10:06

What is Service Mesh

- 由於分散式系統面對的問題，系統之間包括：
    - visibility
    - connectivity
    - security requirement
- 元件之間會需要透過 untrusted network 溝通，以及邊界所需的負載平衡機制和 protocols、彈性、以及 sender 與 receiver 之間安全性變得至關重要
- 以往會將這些機制設計在 application layer (例如 token 驗証？)
- Service Mesh 將這些特性抽離 app 並提移動到 infra 層的設計，不再需要對 application 變動

Features

- Resilient Connectivity
- L7 Traffic Management
- Identity-based Security
- Observability & Tracing
- Transparency

[Service Mesh 介紹](https://jason-kao-blog.medium.com/service-mesh-%E4%BB%8B%E7%B4%B9-56301d70ade4)

[](https://ithelp.ithome.com.tw/articles/10217196)

[https://github.com/sivaramanselvam2704/ecs-service-connect/blob/main/ecs-service-connect.yaml](https://github.com/sivaramanselvam2704/ecs-service-connect/blob/main/ecs-service-connect.yaml)

On AWS ECS it’s served as `ECS Service Connect`

[New – Amazon ECS Service Connect Enabling Easy Communication Between Microservices | Amazon Web Services](https://aws.amazon.com/tw/blogs/aws/new-amazon-ecs-service-connect-enabling-easy-communication-between-microservices/)

[Use service discovery to connect Amazon ECS services with DNS names - Amazon Elastic Container Service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-discovery.html)

[Use Service Connect to connect Amazon ECS services with short names - Amazon Elastic Container Service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-connect.html)
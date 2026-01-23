---
tags:
  - aws
  - infrastructure
created: 2024-01-01
updated: 2025-01-23
status: active
---

What is ECS
ECS Task Definition
Why we need Fargate


Run Task
Execute Command

```

aws ecs run-task --task-definition bulma-app-RunCommandTaskDefinition-Ycq1hFzcc39B:1 \
  --cluster bulma-app \
  --enable-execute-command
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-01cb1875f18f4d44b],securityGroups=[sg-0721240b95493a232],assignPublicIp=DISABLED}" \
  --overrides '{"containerOverrides": [{"name": "bulma-run-command", "command": ["rake", "assets:cache_client_assets_urls"]}]}'

aws ecs describe-tasks --cluster bulma-app --tasks <task-id>

aws ecs execute-command --cluster bulma-app --task <task-id> --container bulma-run-command --interactive --command /bin/bash

```

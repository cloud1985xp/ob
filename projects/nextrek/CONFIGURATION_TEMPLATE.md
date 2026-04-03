# Dynamic Branch Deployment Configuration Template

此檔案提供配置範本和取得所需 AWS 資源 ARN 的指令。

## 快速配置指令集

複製以下指令到終端機執行，取得所需的所有 ARN 值：

```bash
# 設定變數
export AWS_REGION=ap-northeast-1
export PROJECT_PREFIX=nk
export ENVIRONMENT=staging
export ALB_NAME="${PROJECT_PREFIX}-${ENVIRONMENT}-app"
export TARGET_GROUP_NAME="${PROJECT_PREFIX}-${ENVIRONMENT}-app"

echo "=========================================="
echo "AWS 資源查詢結果"
echo "=========================================="
echo ""

# 1. ALB ARN
echo "1. ALB ARN:"
ALB_ARN=$(aws elbv2 describe-load-balancers \
  --region ${AWS_REGION} \
  --names ${ALB_NAME} \
  --query 'LoadBalancers[0].LoadBalancerArn' \
  --output text 2>/dev/null || echo "NOT_FOUND")
echo "   ${ALB_ARN}"
echo ""

# 2. Listener ARN (HTTPS)
echo "2. Listener ARN (HTTPS):"
LISTENER_ARN=$(aws elbv2 describe-listeners \
  --region ${AWS_REGION} \
  --load-balancer-arn ${ALB_ARN} \
  --query 'Listeners[?Protocol==`HTTPS`].ListenerArn | [0]' \
  --output text 2>/dev/null || echo "NOT_FOUND")
echo "   ${LISTENER_ARN}"
echo ""

# 3. Target Group ARN
echo "3. Target Group ARN:"
TARGET_GROUP_ARN=$(aws elbv2 describe-target-groups \
  --region ${AWS_REGION} \
  --names ${TARGET_GROUP_NAME} \
  --query 'TargetGroups[0].TargetGroupArn' \
  --output text 2>/dev/null || echo "NOT_FOUND")
echo "   ${TARGET_GROUP_ARN}"
echo ""

# 4. VPC ID
echo "4. VPC ID:"
VPC_ID=$(aws elbv2 describe-target-groups \
  --region ${AWS_REGION} \
  --target-group-arns ${TARGET_GROUP_ARN} \
  --query 'TargetGroups[0].VpcId' \
  --output text 2>/dev/null || echo "NOT_FOUND")
echo "   ${VPC_ID}"
echo ""

# 5. Task Definition
echo "5. Base Task Definition Family:"
TASK_DEFINITION="${PROJECT_PREFIX}-${ENVIRONMENT}-app"
echo "   ${TASK_DEFINITION}"
echo ""

# 6. Container Name (從 task definition 取得)
echo "6. Container Name:"
LATEST_TASK_DEF=$(aws ecs list-task-definitions \
  --region ${AWS_REGION} \
  --family-prefix ${TASK_DEFINITION} \
  --sort DESC \
  --max-items 1 \
  --query 'taskDefinitionArns[0]' \
  --output text 2>/dev/null)

if [ "${LATEST_TASK_DEF}" != "None" ] && [ -n "${LATEST_TASK_DEF}" ]; then
  CONTAINER_NAME=$(aws ecs describe-task-definition \
    --region ${AWS_REGION} \
    --task-definition ${LATEST_TASK_DEF} \
    --query 'taskDefinition.containerDefinitions[0].name' \
    --output text 2>/dev/null || echo "NOT_FOUND")
  echo "   ${CONTAINER_NAME}"

  CONTAINER_PORT=$(aws ecs describe-task-definition \
    --region ${AWS_REGION} \
    --task-definition ${LATEST_TASK_DEF} \
    --query 'taskDefinition.containerDefinitions[0].portMappings[0].containerPort' \
    --output text 2>/dev/null || echo "NOT_FOUND")
  echo "   Container Port: ${CONTAINER_PORT}"
else
  echo "   NOT_FOUND (no task definition found)"
fi
echo ""

# 7. Cluster Name
echo "7. ECS Cluster Name:"
CLUSTER_NAME="${PROJECT_PREFIX}-${ENVIRONMENT}-app"
echo "   ${CLUSTER_NAME}"
echo ""

echo "=========================================="
echo "配置總結"
echo "=========================================="
echo ""
echo "請將以下值更新到 workflow 檔案中："
echo ""
echo "deploy_dynamic_branch.yml:"
echo "---"
cat <<EOF
env:
  BASE_ALB_ARN: ${ALB_ARN}
  BASE_LISTENER_ARN: ${LISTENER_ARN}
  BASE_TASK_DEFINITION: ${TASK_DEFINITION}
  BASE_TARGET_GROUP_ARN: ${TARGET_GROUP_ARN}
  VPC_ID: ${VPC_ID}
  CONTAINER_NAME: ${CONTAINER_NAME}
  CONTAINER_PORT: ${CONTAINER_PORT}
EOF
echo ""
echo "cleanup_dynamic_branch.yml:"
echo "---"
cat <<EOF
env:
  BASE_LISTENER_ARN: ${LISTENER_ARN}
EOF
echo ""
echo "=========================================="
```

## 手動驗證步驟

### 1. 驗證 ALB 設定

```bash
# 列出所有 ALB
aws elbv2 describe-load-balancers \
  --region ap-northeast-1 \
  --query 'LoadBalancers[*].[LoadBalancerName,LoadBalancerArn]' \
  --output table

# 檢查指定 ALB 的詳細資訊
aws elbv2 describe-load-balancers \
  --region ap-northeast-1 \
  --names nk-staging-app
```

### 2. 驗證 Listener 設定

```bash
# 列出 ALB 的所有 listeners
aws elbv2 describe-listeners \
  --region ap-northeast-1 \
  --load-balancer-arn <ALB_ARN> \
  --query 'Listeners[*].[Protocol,Port,ListenerArn]' \
  --output table

# 檢查 HTTPS listener 的規則
aws elbv2 describe-rules \
  --region ap-northeast-1 \
  --listener-arn <LISTENER_ARN> \
  --query 'Rules[*].[Priority,Conditions[0].Values[0]]' \
  --output table
```

### 3. 驗證 Target Group 設定

```bash
# 列出所有 target groups
aws elbv2 describe-target-groups \
  --region ap-northeast-1 \
  --query 'TargetGroups[*].[TargetGroupName,TargetGroupArn,Port,Protocol]' \
  --output table

# 檢查 target group 健康狀態
aws elbv2 describe-target-health \
  --region ap-northeast-1 \
  --target-group-arn <TARGET_GROUP_ARN>
```

### 4. 驗證 VPC 設定

```bash
# 列出所有 VPC
aws ec2 describe-vpcs \
  --region ap-northeast-1 \
  --query 'Vpcs[*].[VpcId,Tags[?Key==`Name`].Value|[0],CidrBlock]' \
  --output table

# 從 target group 取得 VPC
aws elbv2 describe-target-groups \
  --region ap-northeast-1 \
  --target-group-arns <TARGET_GROUP_ARN> \
  --query 'TargetGroups[0].[VpcId,TargetGroupName]' \
  --output table
```

### 5. 驗證 ECS 設定

```bash
# 列出所有 ECS clusters
aws ecs list-clusters \
  --region ap-northeast-1

# 列出 cluster 中的 services
aws ecs list-services \
  --region ap-northeast-1 \
  --cluster nk-staging-app

# 列出 task definitions
aws ecs list-task-definitions \
  --region ap-northeast-1 \
  --family-prefix nk-staging-app \
  --sort DESC \
  --max-items 5

# 查看 task definition 詳細資訊
aws ecs describe-task-definition \
  --region ap-northeast-1 \
  --task-definition nk-staging-app
```

## 常見問題排查

### ALB 不存在

如果 ALB 不存在，可能需要先建立：

```bash
# 列出所有 ALB 看看是否有類似的名稱
aws elbv2 describe-load-balancers \
  --region ap-northeast-1 \
  --query 'LoadBalancers[*].LoadBalancerName' \
  --output text
```

### Listener ARN 找不到

確認 listener 是否存在且使用 HTTPS：

```bash
# 查看所有 listeners
aws elbv2 describe-listeners \
  --region ap-northeast-1 \
  --load-balancer-arn <ALB_ARN>
```

### VPC ID 無法取得

可以從多個來源取得 VPC ID：

```bash
# 方法 1: 從 ALB 取得
aws elbv2 describe-load-balancers \
  --region ap-northeast-1 \
  --names nk-staging-app \
  --query 'LoadBalancers[0].VpcId' \
  --output text

# 方法 2: 從 target group 取得
aws elbv2 describe-target-groups \
  --region ap-northeast-1 \
  --names nk-staging-app \
  --query 'TargetGroups[0].VpcId' \
  --output text

# 方法 3: 列出所有 VPC
aws ec2 describe-vpcs \
  --region ap-northeast-1
```

### Container Name/Port 無法取得

從最新的 task definition 查看：

```bash
# 取得最新的 task definition ARN
TASK_DEF=$(aws ecs list-task-definitions \
  --region ap-northeast-1 \
  --family-prefix nk-staging-app \
  --sort DESC \
  --max-items 1 \
  --query 'taskDefinitionArns[0]' \
  --output text)

# 查看 container 資訊
aws ecs describe-task-definition \
  --region ap-northeast-1 \
  --task-definition ${TASK_DEF} \
  --query 'taskDefinition.containerDefinitions[*].[name,image,portMappings[0].containerPort]' \
  --output table
```

## 配置檢查清單

在執行 workflow 前，確認以下項目：

- [ ] ALB ARN 正確且 ALB 存在
- [ ] Listener ARN 正確且為 HTTPS listener
- [ ] Target Group ARN 正確且可存取
- [ ] VPC ID 正確
- [ ] Base Task Definition family 存在
- [ ] Container Name 正確
- [ ] Container Port 正確 (通常是 3001)
- [ ] Domain suffix 正確 (nextrek.dev)
- [ ] AWS Region 正確 (ap-northeast-1)
- [ ] GitHub Secrets 已設定:
  - [ ] `AWS_ACCESS_KEY_ID`
  - [ ] `AWS_SECRET_ACCESS_KEY`

## 測試配置

建議先用簡單的 tag 測試：

```bash
# 1. 測試部署
gh workflow run deploy_dynamic_branch.yml \
  -f tag=test001 \
  -f branch=develop

# 2. 檢查結果
# 等待 workflow 完成，檢查是否成功

# 3. 測試存取
curl -I https://test001.nextrek.dev

# 4. 清理測試
gh workflow run cleanup_dynamic_branch.yml \
  -f tag=test001 \
  -f confirm=yes
```

## 需要的 AWS IAM 權限

GitHub Actions 使用的 IAM user/role 需要以下權限：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:PutImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ecs:RegisterTaskDefinition",
        "ecs:DeregisterTaskDefinition",
        "ecs:DescribeTaskDefinition",
        "ecs:ListTaskDefinitions",
        "ecs:CreateService",
        "ecs:UpdateService",
        "ecs:DeleteService",
        "ecs:DescribeServices",
        "ecs:ListServices"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "elasticloadbalancing:CreateTargetGroup",
        "elasticloadbalancing:DeleteTargetGroup",
        "elasticloadbalancing:DescribeTargetGroups",
        "elasticloadbalancing:DescribeTargetGroupAttributes",
        "elasticloadbalancing:ModifyTargetGroupAttributes",
        "elasticloadbalancing:DescribeTargetHealth",
        "elasticloadbalancing:RegisterTargets",
        "elasticloadbalancing:DeregisterTargets",
        "elasticloadbalancing:CreateRule",
        "elasticloadbalancing:DeleteRule",
        "elasticloadbalancing:DescribeRules",
        "elasticloadbalancing:ModifyRule",
        "elasticloadbalancing:DescribeListeners",
        "elasticloadbalancing:DescribeLoadBalancers"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "iam:PassRole"
      ],
      "Resource": [
        "arn:aws:iam::532533425670:role/stg-common-role-ECSRole-*"
      ]
    }
  ]
}
```

# Dynamic Branch Deployment

這套 CI/CD workflows 讓你可以在 AWS ECS Staging 環境上部署任何 branch 到獨立的 service，並透過專屬的 subdomain 存取。

## 功能特色

- ✅ 從任意 branch 建立獨立的 ECS service
- ✅ 自動建立專屬的 ALB listener rule 和 target group
- ✅ 使用 tag 作為 subdomain (例如: `t20250123.nextrek.dev`)
- ✅ 可重複使用的 build image workflow
- ✅ 完整的清理機制
- ✅ 只能在 staging 環境執行，保護 production

## 架構說明

### Workflows

1. **`build_image.yml`** - 可重用的 Docker image 建置 workflow
   - 可被其他 workflows 呼叫
   - 也可獨立執行
   - 支援自訂 branch 和 image tag

2. **`deploy_dynamic_branch.yml`** - 主要部署 workflow
   - 建置 Docker image (或使用現有 image)
   - 建立 ECS task definition
   - 建立 ECS service
   - 建立 ALB target group
   - 新增 ALB listener rule

3. **`cleanup_dynamic_branch.yml`** - 清理 workflow
   - 刪除 ECS service
   - 刪除 ALB listener rule
   - 刪除 target group
   - 反註冊 task definitions

### Actions

- **`action/deploy-dynamic-branch/`** - 部署邏輯
  - `action.yml` - Action 定義
  - `deploy.sh` - 主要部署腳本
  - `functions.sh` - 共用函數

- **`action/cleanup-dynamic-branch/`** - 清理邏輯
  - `action.yml` - Action 定義
  - `cleanup.sh` - 清理腳本

## 初始設定

在使用前，需要更新以下設定值：

### 1. 更新 `deploy_dynamic_branch.yml` 中的環境變數

```yaml
env:
  # ALB Configuration (Staging)
  BASE_ALB_ARN: arn:aws:elasticloadbalancing:ap-northeast-1:532533425670:loadbalancer/app/nk-staging-app/CHANGE_ME
  BASE_LISTENER_ARN: arn:aws:elasticloadbalancing:ap-northeast-1:532533425670:listener/app/nk-staging-app/CHANGE_ME/CHANGE_ME

  # ECS Configuration (Staging)
  BASE_TASK_DEFINITION: nk-staging-app
  BASE_TARGET_GROUP_ARN: arn:aws:elasticloadbalancing:ap-northeast-1:532533425670:targetgroup/nk-staging-app/CHANGE_ME

  # Network Configuration
  VPC_ID: vpc-CHANGE_ME

  # Domain Configuration
  DOMAIN_SUFFIX: nextrek.dev

  # Container Configuration
  CONTAINER_NAME: nextrek-app
  CONTAINER_PORT: 3001
```

### 2. 更新 `cleanup_dynamic_branch.yml` 中的環境變數

```yaml
env:
  # ALB Configuration (Staging)
  BASE_LISTENER_ARN: arn:aws:elasticloadbalancing:ap-northeast-1:532533425670:listener/app/nk-staging-app/CHANGE_ME/CHANGE_ME

  # Domain Configuration
  DOMAIN_SUFFIX: nextrek.dev
```

### 3. 取得所需的 AWS ARN 值

#### 取得 ALB ARN
```bash
aws elbv2 describe-load-balancers \
  --region ap-northeast-1 \
  --names nk-staging-app \
  --query 'LoadBalancers[0].LoadBalancerArn' \
  --output text
```

#### 取得 Listener ARN
```bash
aws elbv2 describe-listeners \
  --region ap-northeast-1 \
  --load-balancer-arn <ALB_ARN> \
  --query 'Listeners[?Protocol==`HTTPS`].ListenerArn' \
  --output text
```

#### 取得 Target Group ARN
```bash
aws elbv2 describe-target-groups \
  --region ap-northeast-1 \
  --names nk-staging-app \
  --query 'TargetGroups[0].TargetGroupArn' \
  --output text
```

#### 取得 VPC ID
```bash
aws ec2 describe-vpcs \
  --region ap-northeast-1 \
  --filters "Name=tag:Name,Values=*staging*" \
  --query 'Vpcs[0].VpcId' \
  --output text
```

或從現有的 target group 取得：
```bash
aws elbv2 describe-target-groups \
  --region ap-northeast-1 \
  --target-group-arns <TARGET_GROUP_ARN> \
  --query 'TargetGroups[0].VpcId' \
  --output text
```

## 使用方式

### 1. 部署 Branch

#### 首次部署

首次使用某個 tag 進行部署時，系統會建立所有必要的 AWS 資源。

**使用 GitHub UI**:
1. 前往 Actions 頁面
2. 選擇 "Deploy Dynamic Branch to Staging"
3. 點擊 "Run workflow"
4. 輸入參數：
   - **tag**: 部署識別碼 (例如: `pr123`, `dev-feature`)
   - **branch**: 要部署的 branch 名稱
   - **skip_build**: 保持 false

**使用 GitHub CLI**:
```bash
gh workflow run deploy_dynamic_branch.yml \
  -f tag=pr123 \
  -f branch=feature/new-feature
```

#### 重複部署 (Re-deployment)

對同一個 tag 進行重複部署時，系統會：
- ✅ 重新 build Docker image (使用更新後的程式碼)
- ✅ 建立新版本的 task definition
- ✅ 更新 ECS service (滾動部署新的 tasks)
- ⏭️ 重用現有的 target group
- ⏭️ 重用現有的 ALB rule
- ✅ 保持相同的 URL

**使用場景**:
- PR 有新的 commits，需要更新測試環境
- 修改了程式碼，需要重新部署
- 想切換到不同的 branch 但保持相同 URL

**使用 GitHub CLI**:
```bash
# 重複部署 (build 新 image)
gh workflow run deploy_dynamic_branch.yml \
  -f tag=pr123 \
  -f branch=feature/new-feature-updated

# 重複部署 (使用已存在的 image)
gh workflow run deploy_dynamic_branch.yml \
  -f tag=pr123 \
  -f branch=feature/new-feature \
  -f skip_build=true
```

**重複部署優點**:
- 更快的部署速度 (~5-7 分鐘 vs ~8-10 分鐘)
- 不產生額外的 AWS 資源
- URL 保持不變，不需要更新分享的連結
- 節省成本

### 2. 存取部署的應用

部署完成後，可透過以下 URL 存取：

```
https://{tag}.nextrek.dev
```

例如：
- Tag `t20250123` → `https://t20250123.nextrek.dev`
- Tag `pr456` → `https://pr456.nextrek.dev`

### 3. 清理部署

測試完成後，清理資源：

#### 使用 GitHub UI

1. 前往 Actions 頁面
2. 選擇 "Cleanup Dynamic Branch Deployment"
3. 點擊 "Run workflow"
4. 輸入參數：
   - **tag**: 要清理的部署識別碼
   - **confirm**: 輸入 `yes` 確認刪除

#### 使用 GitHub CLI

```bash
gh workflow run cleanup_dynamic_branch.yml \
  -f tag=t20250123 \
  -f confirm=yes
```

## Tag 命名規則

Tag 必須符合以下規則：
- 只能包含英文字母、數字和連字號 (`-`)
- 必須以英文字母或數字開頭
- 最長 32 個字元
- 建議格式：
  - 日期型: `t20250123`
  - PR 型: `pr456`
  - 功能型: `dev-feature`

## 建立的 AWS 資源

每次部署會建立以下資源：

1. **ECS Task Definition**: `nextrek-{tag}`
   - 從 base task definition 複製
   - 更新 container image

2. **ECS Service**: `nextrek-{tag}`
   - 在 `nk-staging-app` cluster 中
   - 使用新的 task definition
   - 連接到新的 target group

3. **Target Group**: `nextrek-{tag}`
   - 從 base target group 複製配置
   - 註冊 ECS tasks

4. **ALB Listener Rule**:
   - Host header: `{tag}.nextrek.dev`
   - Forward to target group

## 故障排除

### 部署失敗

1. **檢查 workflow logs**:
   ```bash
   gh run list --workflow=deploy_dynamic_branch.yml --limit=1
   gh run view <run-id> --log
   ```

2. **常見問題**:
   - Tag 格式不正確
   - AWS credentials 過期
   - VPC/Subnet 配置錯誤
   - ALB listener 優先權衝突

### 清理失敗

1. **Target Group 無法刪除**:
   - 等待 ECS service 完全停止
   - 手動 deregister targets
   - 重新執行 cleanup workflow

2. **手動清理**:
   ```bash
   # 刪除 ECS service
   aws ecs delete-service --cluster nk-staging-app --service nextrek-{tag} --force

   # 刪除 target group (需等待 service 刪除完成)
   aws elbv2 delete-target-group --target-group-arn <arn>

   # 刪除 ALB rule
   aws elbv2 delete-rule --rule-arn <arn>
   ```

## 安全性考量

1. **僅限 Staging**: Workflows 限制只能在 staging 環境執行
2. **需要確認**: 清理操作需要輸入 `yes` 確認
3. **AWS Credentials**: 使用 GitHub Secrets 管理 AWS credentials
4. **網路隔離**: 使用現有的 staging VPC 和 security groups

## 最佳實務

1. **Tag 命名**: 使用有意義的 tag 名稱，例如日期、PR 編號或功能名稱
2. **及時清理**: 測試完成後立即清理資源，避免成本浪費
3. **監控資源**: 定期檢查是否有遺留的部署
4. **日誌記錄**: 保留部署和清理的 workflow logs

## 成本考量

每個動態部署會產生以下成本：
- ECS Fargate task (依實際運行時間計費)
- ALB target group (每小時約 $0.008)
- 資料傳輸費用

建議：
- 不使用時立即清理
- 設定 CloudWatch alarm 監控異常使用
- 考慮使用 EC2 instances 而非 Fargate (如果有長期測試需求)

## 未來改進

可能的增強功能：
- [ ] 自動過期清理 (例如: 7 天後自動刪除)
- [ ] 支援多個 container 更新
- [ ] 支援資料庫 migration
- [ ] 整合 Slack 通知
- [ ] 支援自訂環境變數
- [ ] 支援 blue-green deployment
- [ ] 加入 Datadog APM service

## 相關資源

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [AWS ECS Documentation](https://docs.aws.amazon.com/ecs/)
- [AWS ALB Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)

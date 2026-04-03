# Quick Start Guide - Dynamic Branch Deployment

快速開始使用動態分支部署功能。

## 📋 TL;DR

```bash
# 1. 取得 AWS 配置
./.github/workflows/scripts/get-aws-config.sh staging nextrek

# 2. 更新 workflow 檔案中的配置 (參考上述輸出)

# 3. 部署一個 branch
gh workflow run deploy_dynamic_branch.yml \
  -f tag=test001 \
  -f branch=develop

# 4. 存取部署的應用
open https://test001.nextrek.dev

# 5. 測試完成後清理
gh workflow run cleanup_dynamic_branch.yml \
  -f tag=test001 \
  -f confirm=yes
```

## 🗂️ 建立的檔案結構

```
.github/workflows/
├── build_image.yml                    # 可重用的 Docker image 建置 workflow
├── deploy_dynamic_branch.yml          # 主要部署 workflow
├── cleanup_dynamic_branch.yml         # 清理 workflow
├── DYNAMIC_BRANCH_DEPLOYMENT.md       # 完整文檔
├── CONFIGURATION_TEMPLATE.md          # 配置範本和指令
├── QUICK_START.md                     # 此檔案
└── scripts/
    └── get-aws-config.sh              # AWS 配置取得工具

action/
├── deploy-dynamic-branch/
│   ├── action.yml                     # 部署 action 定義
│   ├── deploy.sh                      # 部署主腳本
│   └── functions.sh                   # 共用函數
└── cleanup-dynamic-branch/
    ├── action.yml                     # 清理 action 定義
    └── cleanup.sh                     # 清理腳本
```

## 🚀 設定步驟

### Step 1: 取得 AWS 配置

執行配置工具取得所需的 AWS ARN 值：

```bash
./.github/workflows/scripts/get-aws-config.sh staging nextrek
```

這會輸出所有需要的配置值。

### Step 2: 更新 Workflow 配置

將 Step 1 輸出的配置值更新到以下檔案：

1. `.github/workflows/deploy_dynamic_branch.yml` - 找到 `env:` 區塊並更新
2. `.github/workflows/cleanup_dynamic_branch.yml` - 找到 `env:` 區塊並更新

需要更新的值：
- `BASE_ALB_ARN`
- `BASE_LISTENER_ARN`
- `BASE_TASK_DEFINITION`
- `BASE_TARGET_GROUP_ARN`
- `VPC_ID`
- `CONTAINER_NAME`
- `CONTAINER_PORT`

### Step 3: 提交變更

```bash
git add .github/workflows/ action/
git commit -m "feat: Add dynamic branch deployment workflows"
git push
```

### Step 4: 測試部署

使用簡單的 tag 進行第一次測試：

```bash
# 部署測試
gh workflow run deploy_dynamic_branch.yml \
  -f tag=test001 \
  -f branch=develop

# 查看執行狀態
gh run list --workflow=deploy_dynamic_branch.yml --limit=5

# 查看詳細 log
gh run view --log
```

### Step 5: 驗證部署

部署完成後（約 5-10 分鐘），驗證：

```bash
# 檢查 URL 是否可存取
curl -I https://test001.nextrek.dev

# 在瀏覽器中開啟
open https://test001.nextrek.dev
```

### Step 6: 清理測試

測試完成後清理資源：

```bash
gh workflow run cleanup_dynamic_branch.yml \
  -f tag=test001 \
  -f confirm=yes
```

## 💡 常用命令

### 部署命令

```bash
# 首次部署 (建立所有資源)
gh workflow run deploy_dynamic_branch.yml \
  -f tag=pr123 \
  -f branch=feature/new-feature

# 重複部署到同一個 tag (更新程式碼)
# - 重新 build image
# - 更新 task definition
# - 更新 ECS service (不重建 target group, ALB rule)
gh workflow run deploy_dynamic_branch.yml \
  -f tag=pr123 \
  -f branch=feature/new-feature-updated

# 重複部署但跳過 build (使用已存在的 image)
gh workflow run deploy_dynamic_branch.yml \
  -f tag=pr123 \
  -f branch=feature/new-feature \
  -f skip_build=true

# 只建置 image (不部署)
gh workflow run build_image.yml \
  -f tag=pr123 \
  -f branch=feature/new-feature
```

### 查詢命令

```bash
# 列出最近的執行
gh run list --workflow=deploy_dynamic_branch.yml --limit=10

# 查看執行詳情
gh run view <run-id>

# 查看 log
gh run view <run-id> --log

# 即時監看執行
gh run watch <run-id>
```

### 清理命令

```bash
# 清理單一部署
gh workflow run cleanup_dynamic_branch.yml \
  -f tag=pr123 \
  -f confirm=yes

# 列出所有動態部署的 services (需要 AWS CLI)
aws ecs list-services \
  --cluster nk-staging-app \
  --region ap-northeast-1 \
  | grep nextrek-
```

## 🏷️ Tag 命名建議

好的 tag 命名範例：

```bash
# 日期型 - 適合臨時測試
tag=t20250109

# PR 型 - 適合 PR review
tag=pr456

# 功能型 - 適合長期開發分支
tag=dev-payment
tag=feat-oauth

# 環境型 - 適合特定測試環境
tag=qa-regression
tag=demo-client
```

注意事項：
- 只能包含 `a-z`, `A-Z`, `0-9`, `-`
- 必須以英數字開頭
- 最長 32 字元
- 避免使用底線 `_` (AWS 限制)

## 📊 部署流程概覽

```
User Input (tag + branch)
        ↓
1. Build Docker Image
   - Checkout branch
   - Build image
   - Tag: {tag}, {tag}-{commit}
   - Push to ECR
        ↓
2. Create Task Definition
   - Copy from base task def
   - Update image to new tag
   - Register new task def
        ↓
3. Create Target Group
   - Copy config from base
   - Name: nextrek-{tag}
   - Register in VPC
        ↓
4. Create ECS Service
   - Name: nextrek-{tag}
   - Use new task definition
   - Link to target group
   - Start 1 task
        ↓
5. Add ALB Rule
   - Host: {tag}.nextrek.dev
   - Forward to target group
   - Auto-assign priority
        ↓
✅ Deployment Complete
   Access at: https://{tag}.nextrek.dev
```

## 🔄 重複部署 (Re-deployment)

系統支援對同一個 tag 進行重複部署，這在以下情況非常有用：
- 修改了 branch 的程式碼，需要更新已部署的環境
- PR 有新的 commits，需要更新測試環境
- 需要切換到不同的 branch 但保持相同的 URL

### 重複部署流程

```
同一個 tag 的第二次部署
        ↓
1. Build New Image (如果需要)
   - 使用更新後的 branch
   - 新的 commit hash
        ↓
2. Create New Task Definition
   - 新版本號
   - 使用新的 image
        ↓
3. Reuse Existing Resources
   ✓ Target Group (不重建)
   ✓ ALB Rule (不重建)
        ↓
4. Update ECS Service
   - 使用新的 task definition
   - Force new deployment
   - ECS 滾動更新 tasks
        ↓
✅ Re-deployment Complete
   Same URL: https://{tag}.nextrek.dev
   Updated code is rolling out
```

### 重複部署範例

```bash
# 1. 首次部署 PR
gh workflow run deploy_dynamic_branch.yml \
  -f tag=pr123 \
  -f branch=feature/payment

# 2. PR 有新的 commits，重新部署更新
gh workflow run deploy_dynamic_branch.yml \
  -f tag=pr123 \
  -f branch=feature/payment

# 3. 想測試不同的 branch 但保持相同 URL
gh workflow run deploy_dynamic_branch.yml \
  -f tag=pr123 \
  -f branch=feature/payment-hotfix
```

### 重複部署的優點

- ✅ **保持 URL 不變** - 不需要更新分享的連結
- ✅ **不重建資源** - 更快的部署速度
- ✅ **節省成本** - 不會產生額外的 ALB rules 或 target groups
- ✅ **方便迭代** - 快速測試程式碼變更

### 首次部署 vs 重複部署

| 操作 | 首次部署 | 重複部署 |
|------|---------|---------|
| Build Image | ✅ | ✅ (除非 skip_build) |
| Create Task Def | ✅ | ✅ (新版本) |
| Create Target Group | ✅ | ⏭️ (重用現有) |
| Create ECS Service | ✅ | ⏭️ (更新現有) |
| Create ALB Rule | ✅ | ⏭️ (重用現有) |
| Deployment Time | ~8-10 分鐘 | ~5-7 分鐘 |

### 監控重複部署

```bash
# 查看 ECS service 的部署狀態
aws ecs describe-services \
  --cluster nk-staging-app \
  --services nextrek-pr123 \
  --region ap-northeast-1 \
  --query 'services[0].deployments'

# 查看正在運行的 tasks
aws ecs list-tasks \
  --cluster nk-staging-app \
  --service-name nextrek-pr123 \
  --region ap-northeast-1
```

## 🔧 疑難排解

### 問題: Workflow 找不到

**原因**: Workflow 檔案未推送到 GitHub

**解決**:
```bash
git add .github/workflows/
git commit -m "Add dynamic branch deployment workflows"
git push
```

### 問題: 部署失敗 - "Task definition not found"

**原因**: BASE_TASK_DEFINITION 配置錯誤

**解決**:
```bash
# 列出可用的 task definitions
aws ecs list-task-definition-families --region ap-northeast-1

# 更新 workflow 中的 BASE_TASK_DEFINITION
```

### 問題: 部署失敗 - "Target group creation failed"

**原因**: VPC_ID 錯誤或網路配置問題

**解決**:
```bash
# 從 base target group 取得 VPC ID
aws elbv2 describe-target-groups \
  --target-group-arns <BASE_TARGET_GROUP_ARN> \
  --query 'TargetGroups[0].VpcId'
```

### 問題: URL 無法存取 - 504 Gateway Timeout

**原因**: ECS task 未健康或 health check 失敗

**檢查**:
```bash
# 檢查 ECS service 狀態
aws ecs describe-services \
  --cluster nk-staging-app \
  --services nextrek-{tag} \
  --region ap-northeast-1

# 檢查 target health
aws elbv2 describe-target-health \
  --target-group-arn <TARGET_GROUP_ARN> \
  --region ap-northeast-1
```

### 問題: 清理失敗 - "Target group still in use"

**原因**: ECS service 尚未完全停止

**解決**:
```bash
# 等待幾分鐘後重試
# 或手動檢查並刪除
aws ecs describe-services \
  --cluster nk-staging-app \
  --services nextrek-{tag} \
  --region ap-northeast-1
```

## 📚 更多資訊

詳細文檔：
- [DYNAMIC_BRANCH_DEPLOYMENT.md](./DYNAMIC_BRANCH_DEPLOYMENT.md) - 完整功能文檔
- [CONFIGURATION_TEMPLATE.md](./CONFIGURATION_TEMPLATE.md) - 配置範本和 AWS 查詢指令

相關連結：
- [GitHub Actions 文檔](https://docs.github.com/en/actions)
- [AWS ECS 文檔](https://docs.aws.amazon.com/ecs/)
- [AWS ALB 文檔](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)

## 🎯 最佳實踐

1. **Tag 命名**
   - 使用有意義的名稱
   - 包含 PR 編號或功能名稱
   - 保持簡短易記

2. **及時清理**
   - 測試完成立即清理
   - 避免累積過多 services
   - 節省 AWS 成本

3. **監控資源**
   - 定期檢查未清理的 services
   - 設定 CloudWatch alarms
   - 追蹤部署成本

4. **安全性**
   - 只在 staging 環境使用
   - 不要部署敏感資料
   - 定期輪換 AWS credentials

5. **測試流程**
   - 先測試簡單的 tag
   - 驗證所有功能正常
   - 記錄常見問題和解決方案

## ✅ 檢查清單

部署前確認：
- [ ] AWS 配置已正確設定
- [ ] GitHub Secrets 已設定 (AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY)
- [ ] Base resources 存在 (ALB, Target Group, Task Definition)
- [ ] Tag 命名符合規則
- [ ] Branch 存在且可存取

部署後驗證：
- [ ] Workflow 執行成功
- [ ] ECS service 正常運行
- [ ] Target group 健康檢查通過
- [ ] URL 可正常存取
- [ ] 應用功能正常

清理前確認：
- [ ] 已完成測試
- [ ] 已記錄需要的資訊
- [ ] 確認要刪除的 tag
- [ ] 備份重要資料 (如有需要)

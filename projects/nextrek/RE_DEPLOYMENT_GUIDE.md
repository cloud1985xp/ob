# Re-deployment Guide - 重複部署指南

## 概述

重複部署功能允許你對同一個 tag 多次執行部署 workflow，用於更新已部署的應用程式碼，而不需要重新建立 AWS 資源。

## 🎯 為什麼需要重複部署

### 常見使用場景

1. **PR 持續更新**
   - PR 有新的 commits
   - 需要更新測試環境供 reviewer 測試
   - 不想改變已分享的 URL

2. **快速迭代開發**
   - 正在開發新功能
   - 需要頻繁部署測試
   - 保持同一個測試環境

3. **Bug 修復**
   - 發現測試環境有問題
   - 修復後重新部署
   - 繼續使用相同的測試環境

4. **切換 Branch**
   - 想測試不同的 branch
   - 但保持相同的 URL
   - 例如測試 hotfix branch

## 🔄 工作原理

### 首次部署 vs 重複部署

| 操作 | 首次部署 | 重複部署 |
|------|---------|---------|
| **Docker Image** | ✅ Build 新 image | ✅ Build 新 image |
| **Task Definition** | ✅ 建立 | ✅ 建立新版本 |
| **Target Group** | ✅ 建立 | ⏭️ 重用現有 |
| **ECS Service** | ✅ 建立 | ✅ 更新 (force new deployment) |
| **ALB Rule** | ✅ 建立 | ⏭️ 重用現有 |
| **URL** | 新的 `{tag}.nextrek.dev` | 相同 `{tag}.nextrek.dev` |
| **部署時間** | ~8-10 分鐘 | ~5-7 分鐘 |
| **AWS 成本** | 建立新資源 | 無額外資源成本 |

### 重複部署流程

```
User: 執行相同 tag 的 workflow
        ↓
System: 檢測資源是否存在
        ↓
   ┌────┴────┐
   │首次部署?│
   └────┬────┘
        │
    No  │ (已存在)
        ↓
1. Build New Image
   - 使用更新的 branch
   - 新的 commit hash
   - Tag: {tag} & {tag}-{commit}
        ↓
2. Create New Task Def
   - Family: nextrek-{tag}
   - New revision number
   - Uses new image URI
        ↓
3. Reuse Resources
   ✓ Target Group exists
   ✓ ALB Rule exists
   (log: "reusing...")
        ↓
4. Update ECS Service
   - Set new task definition
   - Force new deployment
   - ECS rolling update
        ↓
✅ Re-deployment Complete
   URL stays the same
   New code is rolling out
```

## 💻 使用方式

### 基本範例

```bash
# 1. 首次部署
gh workflow run deploy_dynamic_branch.yml \
  -f tag=pr123 \
  -f branch=feature/payment

# 2. PR 有新 commits，重複部署
gh workflow run deploy_dynamic_branch.yml \
  -f tag=pr123 \
  -f branch=feature/payment

# 3. 切換到不同 branch
gh workflow run deploy_dynamic_branch.yml \
  -f tag=pr123 \
  -f branch=feature/payment-hotfix
```

### 進階選項

#### 跳過 Build (使用現有 image)

```bash
# 如果 image 已經存在且不需要更新
gh workflow run deploy_dynamic_branch.yml \
  -f tag=pr123 \
  -f branch=feature/payment \
  -f skip_build=true
```

**使用時機**:
- Image 已經 build 好了
- 只想重啟 service
- 測試 ECS 配置變更

## 🔍 識別重複部署

### 在 Workflow Logs 中

首次部署的輸出：
```
Step 3: Setting up target group...
Creating new target group: nextrek-pr123

Step 4: Setting up ECS service...
Creating new ECS service: nextrek-pr123

Step 5: Setting up ALB listener rule...
Creating ALB rule with priority 5...

Initial Deployment Complete!
All resources have been created successfully.
```

重複部署的輸出：
```
Step 3: Setting up target group...
✓ Target group already exists (reusing): nextrek-pr123
  ARN: arn:aws:elasticloadbalancing:...

Step 4: Setting up ECS service...
✓ ECS service already exists (updating): nextrek-pr123
  Cluster: nk-staging-app
  New task definition: nextrek-pr123:2

  Forcing new deployment to use updated image...
  ✓ Service updated and redeploying

Step 5: Setting up ALB listener rule...
✓ ALB rule already exists (reusing): pr123.nextrek.dev
  Rule ARN: arn:aws:elasticloadbalancing:...

Re-deployment Complete!
The service has been updated with the new image.
ECS is now rolling out the new task definition.

Note: The new tasks are being deployed. It may take
a few minutes for the updated application to be available.
```

### 在 GitHub Actions Summary

重複部署會顯示：
```markdown
## Deployment Summary
- Status: ✅ Success
- Tag: pr123
- Branch: feature/payment-updated
- Environment: staging

### 🎉 Deployment URL
https://pr123.nextrek.dev

> Note: If this is a re-deployment, ECS is rolling out the new tasks.
> It may take a few minutes for the updated application to be available.

### Re-deploy to Update
To update this deployment with code changes, run the workflow again...
```

## 📊 監控重複部署

### 查看 ECS Service 狀態

```bash
# 查看 service 的部署歷史
aws ecs describe-services \
  --cluster nk-staging-app \
  --services nextrek-pr123 \
  --region ap-northeast-1 \
  --query 'services[0].deployments' \
  --output table

# 輸出範例:
# -----------------------------------------------------------------------
# |                          Deployments                               |
# +--------+------------------+-------------+-----------+---------------+
# | Status | TaskDefinition   | RunningCount| DesiredCount| CreatedAt  |
# +--------+------------------+-------------+-----------+---------------+
# | PRIMARY| nextrek-pr123:3  | 1           | 1         | 2025-01-09... |
# | ACTIVE | nextrek-pr123:2  | 0           | 0         | 2025-01-09... |
# +--------+------------------+-------------+-----------+---------------+
```

### 查看 Task Definition 版本

```bash
# 列出所有版本
aws ecs list-task-definitions \
  --family-prefix nextrek-pr123 \
  --region ap-northeast-1

# 輸出範例:
# arn:aws:ecs:ap-northeast-1:...:task-definition/nextrek-pr123:1
# arn:aws:ecs:ap-northeast-1:...:task-definition/nextrek-pr123:2
# arn:aws:ecs:ap-northeast-1:...:task-definition/nextrek-pr123:3
```

### 查看運行的 Tasks

```bash
# 查看目前運行的 tasks
aws ecs list-tasks \
  --cluster nk-staging-app \
  --service-name nextrek-pr123 \
  --region ap-northeast-1

# 查看 task 詳細資訊
aws ecs describe-tasks \
  --cluster nk-staging-app \
  --tasks <task-arn> \
  --region ap-northeast-1
```

## ⏱️ 部署時間線

### 重複部署的典型時間線

```
T+0:00  - Workflow 開始
T+0:30  - Build image 開始
T+3:00  - Build 完成，Push to ECR
T+3:30  - Register new task definition
T+4:00  - Update ECS service (觸發滾動更新)
T+4:30  - 新 task 開始啟動
T+5:00  - Health check 開始
T+6:00  - 新 task healthy，註冊到 target group
T+6:30  - 舊 task 開始 drain
T+7:00  - 舊 task 停止
T+7:30  - 部署完成 ✅

總時間: ~7-8 分鐘
```

## 🎯 最佳實踐

### 1. 適當使用重複部署

✅ **適合**:
- PR 有新 commits
- Bug 修復
- 功能迭代
- 配置調整

❌ **不適合**:
- 不同的功能需要不同的 tag
- 需要同時測試多個版本
- 需要 A/B 測試

### 2. Tag 命名策略

```bash
# Good: 使用 PR 號碼
tag=pr123

# Good: 使用功能名稱
tag=feat-payment

# Bad: 使用時間戳 (不利於重複部署)
tag=t20250109-1430

# Bad: 使用 commit hash (每次都不同)
tag=abc1234
```

### 3. 版本控制

```bash
# 在 PR 描述中記錄部署資訊
## 測試環境
- URL: https://pr123.nextrek.dev
- 最後部署: 2025-01-09 14:30
- Commit: abc1234
- Task Def: nextrek-pr123:5
```

### 4. 清理時機

重複部署不會產生額外資源，但仍需要定期清理：

```bash
# PR merged 後立即清理
gh workflow run cleanup_dynamic_branch.yml \
  -f tag=pr123 \
  -f confirm=yes
```

## 🐛 疑難排解

### 問題: 重複部署後還是看到舊版本

**原因**: 瀏覽器快取或 ECS task 尚未更新完成

**解決**:
1. 強制重新整理瀏覽器 (Ctrl+Shift+R / Cmd+Shift+R)
2. 等待 2-3 分鐘讓 ECS 滾動更新完成
3. 檢查 ECS service 的部署狀態

```bash
aws ecs describe-services \
  --cluster nk-staging-app \
  --services nextrek-pr123 \
  --region ap-northeast-1 \
  --query 'services[0].deployments'
```

### 問題: 重複部署失敗 - "Service update failed"

**原因**: Task definition 可能有錯誤

**解決**:
1. 檢查 task definition 內容
2. 確認 image 存在於 ECR
3. 查看 ECS service events

```bash
aws ecs describe-services \
  --cluster nk-staging-app \
  --services nextrek-pr123 \
  --region ap-northeast-1 \
  --query 'services[0].events[:10]'
```

### 問題: 重複部署很慢

**可能原因**:
- Build 時間長
- Health check 失敗重試
- Task 啟動慢

**優化建議**:
1. 使用 Docker layer caching
2. 檢查 health check 配置
3. 考慮使用 `skip_build=true` (如果 image 未變更)

## 📈 使用統計

追蹤重複部署的使用情況：

```bash
# 查看某個 tag 的所有 task definition 版本
aws ecs list-task-definitions \
  --family-prefix nextrek-pr123 \
  --region ap-northeast-1 \
  | jq '.taskDefinitionArns | length'

# 輸出: 5 (表示部署了 5 次)
```

## 🔗 相關文檔

- [QUICK_START.md](./QUICK_START.md) - 快速開始指南
- [DYNAMIC_BRANCH_DEPLOYMENT.md](./DYNAMIC_BRANCH_DEPLOYMENT.md) - 完整部署文檔
- [README.md](./README.md) - 系統概述

## 💡 小技巧

### 快速重複部署命令

建立 shell alias 簡化命令：

```bash
# 加到 ~/.bashrc 或 ~/.zshrc
alias redeploy='function _redeploy() {
  gh workflow run deploy_dynamic_branch.yml -f tag=$1 -f branch=$2;
}; _redeploy'

# 使用
redeploy pr123 feature/payment
```

### 監控部署進度

```bash
# 即時監看 workflow
gh run list --workflow=deploy_dynamic_branch.yml --limit=1 | \
  grep -o 'ID:[0-9]*' | cut -d: -f2 | \
  xargs gh run watch
```

---

**最後更新**: 2025-01-09
**版本**: v1.1.0

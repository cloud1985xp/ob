# Dynamic Branch Deployment System

完整的 GitHub Actions CI/CD 系統，用於在 AWS ECS Staging 環境上部署任意 branch 到獨立的 service，並透過專屬 subdomain 存取。

## 🎯 功能概述

這套系統可以：
- ✅ **動態部署** - 從任意 branch 建立獨立的 ECS service
- ✅ **自動路由** - 使用 tag 作為 subdomain (例如: `pr123.nextrek.dev`)
- ✅ **重複部署** - 同一個 tag 可重複部署更新，保持 URL 不變
- ✅ **隔離環境** - 每個部署完全獨立，互不影響
- ✅ **簡單清理** - 一鍵清理所有相關資源
- ✅ **安全保護** - 僅限 staging 環境，無法影響 production

## 📁 檔案結構

### Workflows (`.github/workflows/`)

| 檔案 | 用途 | 觸發方式 |
|------|------|----------|
| `build_image.yml` | 建置 Docker image 並推送到 ECR | workflow_call / workflow_dispatch |
| `deploy_dynamic_branch.yml` | 主要部署 workflow | workflow_dispatch (手動) |
| `cleanup_dynamic_branch.yml` | 清理資源 workflow | workflow_dispatch (手動) |

### Actions (`action/`)

| Action | 說明 |
|--------|------|
| `deploy-dynamic-branch/` | 部署邏輯的實作 |
| `cleanup-dynamic-branch/` | 清理邏輯的實作 |

### 文檔 (`.github/workflows/`)

| 檔案 | 內容 |
|------|------|
| `README.md` | 此檔案 - 系統概述 |
| `QUICK_START.md` | 快速開始指南 |
| `DYNAMIC_BRANCH_DEPLOYMENT.md` | 完整功能文檔 |
| `CONFIGURATION_TEMPLATE.md` | 配置範本和 AWS 查詢指令 |

### 工具 (`scripts/`)

| 工具 | 說明 |
|------|------|
| `get-aws-config.sh` | 自動取得 AWS 配置的輔助工具 |

## 🚀 快速開始

### 1. 取得配置

```bash
./.github/workflows/scripts/get-aws-config.sh staging nextrek
```

### 2. 更新配置

將輸出的配置值更新到：
- `.github/workflows/deploy_dynamic_branch.yml`
- `.github/workflows/cleanup_dynamic_branch.yml`

### 3. 部署測試

```bash
gh workflow run deploy_dynamic_branch.yml \
  -f tag=test001 \
  -f branch=develop
```

### 4. 存取應用

```
https://test001.nextrek.dev
```

### 5. 清理資源

```bash
gh workflow run cleanup_dynamic_branch.yml \
  -f tag=test001 \
  -f confirm=yes
```

詳細步驟請參考 [QUICK_START.md](./QUICK_START.md)

## 📖 文檔導覽

### 新手入門
👉 從 [QUICK_START.md](./QUICK_START.md) 開始

包含：
- 設定步驟
- 常用命令
- 疑難排解
- 檢查清單

### 完整文檔
📚 參考 [DYNAMIC_BRANCH_DEPLOYMENT.md](./DYNAMIC_BRANCH_DEPLOYMENT.md)

包含：
- 架構說明
- 詳細配置
- 建立的資源
- 安全性考量
- 最佳實務

### 配置指南
⚙️ 查看 [CONFIGURATION_TEMPLATE.md](./CONFIGURATION_TEMPLATE.md)

包含：
- 配置範本
- AWS 查詢指令
- 手動驗證步驟
- IAM 權限需求

## 🛠️ 系統架構

```
┌─────────────────────────────────────────────────────────┐
│                     GitHub Actions                       │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  1. build_image.yml                                     │
│     - Checkout branch                                    │
│     - Build Docker image                                 │
│     - Push to ECR with tag                              │
│                                                          │
│  2. deploy_dynamic_branch.yml                           │
│     - Call build_image.yml                              │
│     - Create task definition                            │
│     - Create target group                               │
│     - Create ECS service                                │
│     - Add ALB listener rule                             │
│                                                          │
│  3. cleanup_dynamic_branch.yml                          │
│     - Delete ECS service                                │
│     - Delete ALB rule                                   │
│     - Delete target group                               │
│     - Deregister task definitions                       │
│                                                          │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                         AWS                              │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ECR: nextrek:{tag}                                     │
│  ├─ Docker image storage                               │
│  └─ Tagged with branch + commit                        │
│                                                          │
│  ECS: nextrek-{tag}                                     │
│  ├─ Task Definition: nextrek-{tag}                      │
│  └─ Service: nextrek-{tag}                              │
│      └─ Tasks: 1 running                                │
│                                                          │
│  ALB: nk-staging-app                                    │
│  ├─ Listener (HTTPS)                                    │
│  │   └─ Rule: {tag}.nextrek.dev → Target Group         │
│  └─ Target Group: nextrek-{tag}                         │
│      └─ Targets: ECS tasks                              │
│                                                          │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
                   https://{tag}.nextrek.dev
```

## 🔐 安全性

### 環境限制
- ✅ 僅限 staging 環境
- ❌ 無法在 production 執行
- ✅ Workflow 內建環境驗證

### 權限管理
- GitHub Secrets 儲存 AWS credentials
- IAM role 限制最小權限
- 無法存取 production 資源

### 清理確認
- 必須輸入 `yes` 確認刪除
- 防止誤刪重要資源

## 💰 成本考量

每個動態部署的成本：
- **ECS Fargate**: ~$0.04/hour (1 vCPU, 2GB)
- **ALB Target Group**: ~$0.008/hour
- **資料傳輸**: 依實際使用量

**建議**:
- 及時清理不使用的部署
- 避免長期運行多個環境
- 設定 CloudWatch alarm 監控成本

## 📊 使用場景

### PR Review
```bash
# 部署 PR 供 reviewer 測試
gh workflow run deploy_dynamic_branch.yml \
  -f tag=pr456 \
  -f branch=feature/new-feature

# 分享 URL: https://pr456.nextrek.dev
```

### QA 測試
```bash
# 建立專屬 QA 環境
gh workflow run deploy_dynamic_branch.yml \
  -f tag=qa-sprint5 \
  -f branch=release/v2.5
```

### Demo 展示
```bash
# 建立 demo 環境給客戶
gh workflow run deploy_dynamic_branch.yml \
  -f tag=demo-client \
  -f branch=main
```

### 開發分支
```bash
# 長期開發分支測試
gh workflow run deploy_dynamic_branch.yml \
  -f tag=dev-payment \
  -f branch=develop/payment-gateway
```

## 🔄 工作流程範例

### 典型的 PR 流程

```bash
# 1. 建立 PR
git checkout -b feature/new-feature
# ... 開發 ...
git push origin feature/new-feature

# 2. 建立 PR 在 GitHub

# 3. 部署 PR 環境供 review
gh workflow run deploy_dynamic_branch.yml \
  -f tag=pr123 \
  -f branch=feature/new-feature

# 4. 在 PR 中分享測試 URL
# Comment: "測試環境: https://pr123.nextrek.dev"

# 5. Reviewer 測試

# 6. PR merged 後清理
gh workflow run cleanup_dynamic_branch.yml \
  -f tag=pr123 \
  -f confirm=yes
```

## 🔍 監控與除錯

### 查看 Workflow 執行

```bash
# 列出最近的執行
gh run list --workflow=deploy_dynamic_branch.yml --limit=10

# 查看詳細資訊
gh run view <run-id>

# 查看完整 log
gh run view <run-id> --log

# 即時監看
gh run watch <run-id>
```

### 檢查 AWS 資源

```bash
# ECS Service
aws ecs describe-services \
  --cluster nk-staging-app \
  --services nextrek-pr123 \
  --region ap-northeast-1

# Target Health
aws elbv2 describe-target-health \
  --target-group-arn <arn> \
  --region ap-northeast-1

# ALB Rules
aws elbv2 describe-rules \
  --listener-arn <arn> \
  --region ap-northeast-1
```

### 常見問題

| 問題 | 解決方案 |
|------|----------|
| 504 Gateway Timeout | 檢查 ECS task health check |
| Task 無法啟動 | 檢查 task definition 和 ECR image |
| ALB rule 不生效 | 確認 DNS 和 listener 配置 |
| 清理失敗 | 等待 service 停止後重試 |

詳細疑難排解請參考 [QUICK_START.md](./QUICK_START.md#-疑難排解)

## 📈 未來改進計畫

- [ ] 自動過期清理 (7 天後自動刪除)
- [ ] Slack 通知整合
- [ ] 資料庫 migration 支援
- [ ] 自訂環境變數注入
- [ ] Blue-green deployment 支援
- [ ] Datadog APM 整合
- [ ] 成本追蹤 dashboard

## 🤝 貢獻

改進建議或問題回報：
1. 建立 GitHub Issue
2. 提交 Pull Request
3. 更新相關文檔

## 📝 版本記錄

### v1.0.0 (2025-01-09)
- ✨ 初始版本
- ✅ 支援動態 branch 部署
- ✅ 自動 ALB 路由
- ✅ 完整清理機制
- ✅ 完整文檔

## 📞 支援

如遇問題：
1. 查閱 [QUICK_START.md](./QUICK_START.md) 疑難排解章節
2. 檢查 [DYNAMIC_BRANCH_DEPLOYMENT.md](./DYNAMIC_BRANCH_DEPLOYMENT.md) 詳細文檔
3. 查看 workflow logs
4. 聯絡 DevOps 團隊

---

**快速連結**:
- [快速開始](./QUICK_START.md)
- [完整文檔](./DYNAMIC_BRANCH_DEPLOYMENT.md)
- [配置指南](./CONFIGURATION_TEMPLATE.md)

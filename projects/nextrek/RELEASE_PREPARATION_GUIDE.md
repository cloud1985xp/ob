# Release Preparation Workflow Guide

自動化的 release 準備流程，包含 GitHub Release 建立、安全掃描和 Slack 通知。

## 🎯 功能概述

這套系統可以：
- ✅ **自動建立 Release** - 在 GitHub 上建立 release 並自動產生 release notes
- ✅ **安全掃描** - 使用 Brakeman 進行代碼安全性掃描
- ✅ **Slack 通知** - 自動發送 release 資訊和掃描結果到 Slack
- ✅ **可重複使用** - Brakeman 掃描可被其他 workflow 呼叫

## 📁 檔案結構

### Workflows (`.github/workflows/`)

| 檔案 | 用途 | 觸發方式 |
|------|------|----------|
| `release_preparation.yml` | 主要 release 準備 workflow | workflow_dispatch (手動) |
| `brakeman_scan.yml` | 可重複使用的安全掃描 workflow | workflow_call / workflow_dispatch |

## 🚀 快速開始

### 前置需求

確保 GitHub Secrets 已設定：
- `SLACK_BOT_TOKEN` - Slack Bot Token
- 更新 `release_preparation.yml` 中的 Slack channel ID

### 執行 Release 準備

#### 使用 GitHub CLI

```bash
# 基本用法
gh workflow run release_preparation.yml \
  -f version_tag=v2.3.3

# 從特定 branch 執行
gh workflow run release_preparation.yml \
  -f version_tag=v2.3.3 \
  --ref release/2.3.3
```

#### 使用 GitHub UI

1. 前往 GitHub Repository
2. 點選 **Actions** 標籤
3. 選擇 **Release Preparation** workflow
4. 點選 **Run workflow**
5. 填入參數：
   - **version_tag**: 例如 `v2.3.3`
   - **Use workflow from**: 選擇要發布的 branch
6. 點選 **Run workflow**

## 📋 Workflow 執行流程

```
┌─────────────────────────────────────────────────────────┐
│                Release Preparation                       │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  1. create-release                                      │
│     - Checkout code                                     │
│     - Create GitHub Release                             │
│     - Auto-generate release notes                      │
│     - Tag with version_tag                              │
│                                                          │
│  2. security-scan                                       │
│     - Call brakeman_scan.yml                            │
│     - Run Brakeman security scan                        │
│     - Generate reports (JSON + Text)                    │
│     - Upload artifacts                                  │
│                                                          │
│  3. notify-slack                                        │
│     - Download Brakeman report                          │
│     - Parse scan results                                │
│     - Send formatted Slack message with:                │
│       • Release tag and branch                          │
│       • Release URL                                     │
│       • Security scan results                           │
│       • Release notes                                   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## 📊 Slack 通知內容

Slack 訊息包含以下資訊：

### 成功通知範例

```
✅ Release 準備成功

Release Tag: v2.3.3
Branch: release/2.3.3
Release URL: 查看 Release
Workflow: #123

Security Scan:
• Warnings: 0
• Errors: 0
• Security Warnings: 0
• 查看掃描結果

Release Notes:
```
[自動產生的 release notes]
```

Triggered by: username | Repo: owner/nextrek
```

### 失敗/警告通知範例

```
❌ Release 準備失敗或有警告

[相同格式，但顏色為紅色，並顯示錯誤/警告數量]
```

## ⚙️ 配置說明

### 必要配置

#### 1. Slack Channel ID

編輯 `release_preparation.yml`:

```yaml
- name: Send Slack Notification
  uses: slackapi/slack-github-action@v1.18.0
  env:
    SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
  with:
    channel-id: 'C01234ABCDE'  # 👈 替換為你的 Slack channel ID
```

#### 2. 取得 Slack Channel ID

方法 1 - 從 Slack UI:
1. 開啟 Slack channel
2. 點選 channel 名稱
3. 往下捲動找到 "Channel ID"
4. 複製 ID (格式: C01234ABCDE)

方法 2 - 從 Slack URL:
```
https://app.slack.com/client/T0XXXXXX/C01234ABCDE
                                      ^^^^^^^^^^^
                                      這就是 Channel ID
```

#### 3. GitHub Secrets

確保已設定以下 secrets:
- `SLACK_BOT_TOKEN` - 你的 Slack Bot Token
- `GITHUB_TOKEN` - 自動提供，無需設定

## 🔍 Brakeman 掃描

### 獨立執行掃描

Brakeman 掃描也可以獨立執行：

```bash
# 使用 GitHub CLI
gh workflow run brakeman_scan.yml

# 或從 GitHub UI
Actions → Brakeman Security Scan → Run workflow
```

### 掃描報告

掃描完成後會產生兩種格式的報告：
- `brakeman-report.json` - JSON 格式，用於程式解析
- `brakeman-report.txt` - 文字格式，用於人工閱讀

報告保留 30 天，可從 workflow run 的 Artifacts 下載。

### 其他 Workflow 呼叫

其他 workflow 可以重複使用 Brakeman 掃描：

```yaml
jobs:
  security-scan:
    uses: ./.github/workflows/brakeman_scan.yml
```

## 🔐 安全性考量

### Release 權限
- 需要有 repository 的 write 權限才能建立 release
- 使用 `GITHUB_TOKEN` 進行認證
- Release 會自動與上一個 release 比較產生 notes

### Brakeman 掃描
- 掃描結果會上傳為 workflow artifacts
- 可在 GitHub Actions 中查看詳細報告
- 發現的問題會在 Slack 通知中顯示

### Slack Token
- `SLACK_BOT_TOKEN` 儲存在 GitHub Secrets
- 不會在 logs 中顯示
- 僅用於發送通知訊息

## 📖 使用場景

### Release 前準備

```bash
# 1. 建立 release branch
git checkout -b release/2.3.3

# 2. 更新版本號
# 編輯 VERSION 或相關檔案

# 3. 推送到 GitHub
git push origin release/2.3.3

# 4. 執行 release 準備
gh workflow run release_preparation.yml \
  -f version_tag=v2.3.3 \
  --ref release/2.3.3

# 5. 檢查 Slack 通知
# 確認 release 建立成功且沒有安全問題

# 6. 如有需要，處理掃描發現的問題
# 修正後重新執行 workflow

# 7. 繼續進行部署流程
```

### Hotfix Release

```bash
# 1. 建立 hotfix branch
git checkout -b hotfix/2.3.4

# 2. 修正問題並推送
git push origin hotfix/2.3.4

# 3. 快速 release
gh workflow run release_preparation.yml \
  -f version_tag=v2.3.4 \
  --ref hotfix/2.3.4
```

### 安全掃描檢查

```bash
# 定期執行安全掃描 (不建立 release)
gh workflow run brakeman_scan.yml
```

## 🔍 監控與除錯

### 查看 Workflow 執行

```bash
# 列出最近的執行
gh run list --workflow=release_preparation.yml --limit=10

# 查看特定執行的詳細資訊
gh run view <run-id>

# 查看完整 log
gh run view <run-id> --log

# 即時監看
gh run watch
```

### 下載 Brakeman 報告

```bash
# 列出 artifacts
gh run view <run-id> --log

# 下載報告
gh run download <run-id> -n brakeman-report
```

### 查看 Release

```bash
# 列出所有 releases
gh release list

# 查看特定 release
gh release view v2.3.3

# 查看 release notes
gh release view v2.3.3 --json body --jq .body
```

## ❌ 常見問題

### Release 建立失敗

**問題**: Release 建立失敗，顯示 "Reference already exists"

**解決方案**:
```bash
# 檢查 tag 是否已存在
git tag -l "v2.3.3"

# 如果存在，刪除舊 tag (謹慎使用!)
git tag -d v2.3.3
git push origin :refs/tags/v2.3.3

# 重新執行 workflow
```

### Slack 通知未收到

**問題**: Workflow 成功但沒收到 Slack 通知

**檢查清單**:
1. ✅ 確認 `SLACK_BOT_TOKEN` secret 已設定
2. ✅ 確認 channel ID 正確
3. ✅ 確認 Bot 已加入該 channel
4. ✅ 檢查 workflow logs 中的錯誤訊息

**加入 Bot 到 Channel**:
```
在 Slack channel 中輸入:
/invite @YourBotName
```

### Brakeman 掃描失敗

**問題**: Brakeman 掃描失敗

**解決方案**:
```bash
# 本地執行測試
bundle exec brakeman

# 檢查 Ruby 版本是否正確
ruby --version  # 應為 3.3.7

# 檢查 Brakeman 版本
gem list brakeman
```

### Release Notes 為空

**問題**: Release notes 沒有內容

**原因**: 這是第一個 release，或沒有上一個 release 可比較

**解決方案**: 手動編輯 release 加入說明，或確保之前有 release 存在

## 🎨 自訂化

### 修改 Slack 訊息格式

編輯 `release_preparation.yml` 中的 `payload` 區塊：

```yaml
payload: |
  {
    "attachments": [
      {
        "color": "${{ steps.status.outputs.color }}",
        "blocks": [
          # 在這裡自訂 Slack Block Kit 格式
        ]
      }
    ]
  }
```

參考: [Slack Block Kit Builder](https://app.slack.com/block-kit-builder)

### 修改 Brakeman 掃描選項

編輯 `brakeman_scan.yml`：

```yaml
- name: Run Brakeman Security Scan
  uses: reviewdog/action-brakeman@v2
  with:
    brakeman_version: 6.0.1
    # 加入其他選項
    brakeman_flags: '--confidence-level=2'
```

### 加入其他安全掃描工具

可以在 `release_preparation.yml` 中加入其他掃描 job：

```yaml
jobs:
  create-release:
    # ...

  security-scan:
    uses: ./.github/workflows/brakeman_scan.yml

  dependency-check:
    name: Dependency Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Bundler Audit
        run: |
          gem install bundler-audit
          bundle audit check --update
```

## 📈 最佳實務

### Release 流程

1. ✅ 在 release branch 上執行 workflow
2. ✅ 確認安全掃描通過
3. ✅ 檢查 release notes 正確性
4. ✅ 必要時手動編輯 release 補充資訊
5. ✅ 通知團隊 release 已準備好
6. ✅ 繼續執行部署流程

### 版本號命名

建議遵循 [Semantic Versioning](https://semver.org/):

- `v1.0.0` - Major release
- `v1.1.0` - Minor release (new features)
- `v1.1.1` - Patch release (bug fixes)
- `v2.0.0-beta.1` - Pre-release

### 安全掃描

- 📅 定期執行 Brakeman 掃描 (例如每週)
- 🔍 在每次 release 前執行
- 🚨 發現 High 等級問題時立即處理
- 📝 記錄已知問題和處理計畫

## 🔗 相關資源

- [GitHub Actions 文檔](https://docs.github.com/en/actions)
- [Brakeman 文檔](https://brakemanscanner.org/docs/)
- [Slack API 文檔](https://api.slack.com/)
- [Slack Block Kit Builder](https://app.slack.com/block-kit-builder)
- [Semantic Versioning](https://semver.org/)

## 📝 版本記錄

### v1.0.0 (2025-01-09)
- ✨ 初始版本
- ✅ 自動建立 GitHub Release
- ✅ Brakeman 安全掃描
- ✅ Slack 通知整合
- ✅ 可重複使用的 Brakeman workflow

## 📞 支援

如遇問題：
1. 檢查本文檔的常見問題章節
2. 查看 workflow logs
3. 檢查 GitHub Actions status page
4. 聯絡 DevOps 團隊

---

**快速連結**:
- [Workflows 主文檔](./README.md)
- [Dynamic Branch Deployment](./DYNAMIC_BRANCH_DEPLOYMENT.md)
- [Quick Start](./QUICK_START.md)

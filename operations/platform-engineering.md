---
tags:
  - sre
  - operations
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Platform Engineering

> Platform engineering is the discipline of designing and building toolchains and workflows that enable self-service capabilities for software engineering organizations in the cloud-native era. Platform engineers provide an integrated product most often referred to as an “Internal Developer Platform” covering the operational necessities of the entire lifecycle of an application
> 

如何讓一個剛到部、合格的開發人員，能過在一定時間之內、完成一個完整應用程式 (如 Hello World)，從開發環境、部署到測試環境、到正式環境，整個過程的 Lead-Time。
.
這些過程，包含:
.

1. 定義好應用程式的商業指標、系統指標、SLO，其他依賴關係
2. 寫好一個完整的應用程式的開發 (API, CRUD, Database)、完成 Unit Test。有些公司需要做 Security Scan。
3. 完成 Build Procedure，打包 Artifact，放到 Repos.
4. 測試環境：在測試環境完成整合測試，包含在測試環境的建置、整合測試、完成監控指標的定義與測試
5. 測試環境：模擬使用者流量，驗證在所有指標數據正確
6. 正式環境：上到 N 個正式環境的部署，在正式環境，可以使用應用，同時已經有監控指標，包含業務指標、系統指標，以及串接各種 Alert System.

如何讓一個需求，經由 Eng 在能夠在一定時間內，

一新的 masterdata record 進來

可以被展開

可以追蹤變動

可以有被翻譯欄位
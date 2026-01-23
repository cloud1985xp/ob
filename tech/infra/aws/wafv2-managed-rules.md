---
tags:
  - aws
  - infrastructure
created: 2024-01-01
updated: 2025-01-23
status: active
---

應該大多常見的攻擊手法可以透過 AWS WAF Managed Rules 來阻擋

https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-list.html

https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups-use-case.html#aws-managed-rule-groups-use-case-sql-db

主要使用
AWSManagedRulesCommonRuleSet
AWSManagedRulesKnownBadInputsRuleSet
AWSManagedRulesSQLiRuleSet

另外對作業系統可再進一步使用
AWSManagedRulesLinuxRuleSet
或
AWSManagedRulesUnixRuleSet

# AI Explain:

要應對 Cross-Site Scripting (XSS) 攻擊，AWS WAF 託管規則群組 (Managed Rule Groups) 中最常用且最推薦的是 **`AWSManagedRulesCommonRuleSet`**。

這個規則集是 AWS 提供的核心規則集，旨在保護應用程式免受 OWASP Top 10 中常見的威脅，其中包括 XSS 攻擊。

### AWSManagedRulesCommonRuleSet 的運作方式

`AWSManagedRulesCommonRuleSet` 內包含了一系列專門用於偵測 XSS 攻擊的規則。這些規則會檢查 HTTP 請求中的多個部分，包括：

- **`CrossSiteScripting_BODY`**: 檢查請求主體 (request body) 是否包含 XSS 攻擊的特徵。
    
- **`CrossSiteScripting_QUERYARGUMENTS`**: 檢查 URL 查詢參數是否包含惡意腳本。
    
- **`CrossSiteScripting_URIPATH`**: 檢查 URL 路徑 (URI path) 是否包含 XSS 攻擊碼。
    
- **`CrossSiteScripting_COOKIE`**: 檢查 Cookie 內容是否包含惡意腳本。
    

當 HTTP 請求的任何一個部分符合這些規則的模式時，`AWSManagedRulesCommonRuleSet` 會根據其預設動作來處理該請求，通常是直接阻擋 (Block) 這些惡意請求，從而保護你的應用程式。

### 如何使用

你可以在你的 CloudFormation 範本中，像你之前處理 SQL 注入規則一樣，將 `AWSManagedRulesCommonRuleSet` 規則群組加入到你的 Web ACL 中：

YAML

```
      Rules:
        - Name: AWSManagedCommonRuleSet
          Priority: 1
          Statement:
            ManagedRuleGroupStatement:
              VendorName: AWS
              Name: AWSManagedRulesCommonRuleSet
          Action:
            Block: {}
          VisibilityConfig:
            CloudWatchMetricsEnabled: true
            MetricName: AWSManagedCommonRuleSetMetric
            SampledRequestsEnabled: true
```

**提醒：**

- 請務必確認你的 **`Scope`** 設定是否與你的應用程式資源相符 (`CLOUDFRONT` 或 `REGIONAL`)。
    
- 由於託管規則集可能會偶爾產生誤判 (false positives)，建議在正式環境部署前，先將規則的 `Action` 設為 `Count` 模式，觀察一段時間的日誌，確認沒有阻擋到正常流量，再將其切換為 `Block`。
    

總結來說，若要防範 XSS 攻擊，`AWSManagedRulesCommonRuleSet` 是 AWS WAF 託管規則群組中一個功能全面且易於使用的首選。


AWS WAF 的 `AWSManagedRulesLinuxRuleSet` 和 `AWSManagedRulesUnixRuleSet` 都是專門設計來保護執行 Linux 或 Unix 作業系統的 Web 伺服器，抵禦針對這些環境的常見攻擊。它們主要針對的是攻擊者試圖利用作業系統層級的漏洞來執行惡意指令或存取敏感資料。

---

### AWSManagedRulesLinuxRuleSet

**適用情況：**

這個規則集適用於你的 Web 伺服器在 **Linux 環境**下運作時。它包含了一系列針對 Linux Shell 和檔案系統的攻擊模式，特別是那些試圖利用 Shell 執行指令的攻擊。

**能阻擋的攻擊範例：**

`AWSManagedRulesLinuxRuleSet` 主要偵測和阻擋那些包含 Linux 指令的惡意請求。例如，攻擊者可能會嘗試在 HTTP 請求的參數中注入命令，試圖讓伺服器執行。

- **阻擋 Shell 指令注入 (Shell Command Injection)：**
    
    - **惡意請求範例**：`GET /index.php?file=;cat%20/etc/passwd`
        
    - **說明**：攻擊者在 `file` 參數中，利用分號 (`;`) 試圖在後台執行 `cat /etc/passwd` 指令，以讀取 Linux 系統上的使用者帳號資訊。這個規則集會偵測到 `cat /etc/passwd` 這類的模式並阻擋請求。
        
- **阻擋惡意路徑遍歷 (Path Traversal)：**
    
    - **惡意請求範例**：`GET /download.php?file=../../../../etc/passwd`
        
    - **說明**：攻擊者利用 `../` 符號試圖在伺服器檔案系統中向上層目錄移動，進而存取敏感檔案。這個規則集能識別這類的路徑遍歷模式並阻止其執行。
        

---

### `AWSManagedRulesUnixRuleSet`

**適用情況：**

這個規則集和 `AWSManagedRulesLinuxRuleSet` 類似，但它更廣泛地針對 **Unix-like 作業系統**，例如 Linux、FreeBSD、OpenBSD 等。它包含的規則模式更廣泛，旨在涵蓋各種 Unix 環境下常見的 Shell 攻擊。

**能阻擋的攻擊範例：**

`AWSManagedRulesUnixRuleSet` 和 Linux 規則集有重疊，但也會偵測一些更廣泛的 Unix 語法。它也能有效阻擋指令注入和檔案存取攻擊。

- **阻擋 Shell 指令注入 (Shell Command Injection)：**
    
    - **惡意請求範例**：`POST /search?query=%60sleep%2010%60`
        
    - **說明**：攻擊者利用反引號 (`` ` ``) 試圖在伺服器上執行 `sleep 10` 指令，這是一種常見的偵測伺服器是否可被注入的方式（如果伺服器回應延遲了 10 秒，就可能存在漏洞）。`AWSManagedRulesUnixRuleSet` 會識別這類語法並阻擋。
        
- **阻擋遠端指令執行 (Remote Code Execution, RCE)：**
    
    - **惡意請求範例**：`GET /exec?cmd=echo%20'hello'|sh`
        
    - **說明**：攻擊者利用管道符號 (`|`) 將指令傳送給 Shell 執行。這個規則集會偵測到包含 `|sh` 或其他 Shell 執行語法的惡意請求並進行阻擋。
        

---

### 總結與建議

這兩個規則集的功能有很大的重疊，它們的目標都是保護底層的作業系統不受命令注入和檔案系統攻擊。

- 如果你的伺服器明確是 **Linux**，可以使用 `AWSManagedRulesLinuxRuleSet`。
    
- 如果你的伺服器是 **Unix-like** 系統，或者你不確定底層是哪一個，`AWSManagedRulesUnixRuleSet` 是一個更通用的選擇。
    

然而，AWS 官方建議，通常 **`AWSManagedRulesCommonRuleSet`** 已經包含了對 Shell 指令注入和路徑遍歷的基本防護。如果你需要更深層次、更全面的 Linux/Unix 環境保護，再考慮額外啟用這兩個專用規則集。這麼做可以提供額外的防禦層級，但同時也可能增加誤判的風險，因此建議在啟用後密切監控 CloudWatch Metrics 和日誌。

# Example

https://ireznykov.com/2022/02/21/how-to-create-regional-web-acl-wafv2-with-cloudformation/
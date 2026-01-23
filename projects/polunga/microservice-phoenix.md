---
tags:
  - polunga
  - project
  - elixir
created: 2024-01-01
updated: 2025-01-23
status: active
---


- Microservice
- Headless Architecture / SPA
- Authentication (and Authorization)
- Session, Cookies and JWT
- API Gateway

微服務和 headless 是不同的概念，雖然它們可以相輔相成，但並非互相依賴。以下是它們的基本概念和區別：


## Microservice and Headless Architecture

### 微服務架構

微服務架構是將應用拆分為一系列小型的、獨立部署的服務，每個服務專注於某一特定功能，並透過 API 進行通信。這樣的架構可以提高應用的靈活性、擴展性和維護性。微服務架構的主要目的是讓每個服務能夠獨立開發、部署和更新，無需依賴其他服務的狀態。

### Headless Architecture

Headless 架構則是指將前端（UI）與後端（業務邏輯）分離。後端只提供 API，前端可以根據需求在不同平台（如 Web、移動應用等）上進行呈現。這樣的架構非常適合於多終端支援和頻繁的前端更新，因為前端和後端是分離開發的，且後端只提供數據，無需負責前端展示。

- SPA

在 Web 微服務架構中，是否採用 headless 取決於需求：

- 如果應用需要支援多平台展示，並且需要前後端分離，則 headless 架構可能會很適合。
- 如果應用只需要簡單的 Web 接口，並沒有多平台需求，headless 就不是必需的。

因此，headless 不是建置微服務的必然選擇，但在特定情況下，它能夠提高靈活性，尤其是在多端支援和頻繁的前端需求變更時。

## Microservice

## 什麼是微服務架構？

微服務架構是一種將應用程式建構為一組小型、獨立服務的軟體開發方法。這些服務圍繞特定的業務功能設計，並且可以獨立部署、更新和擴展。 這種方法與傳統的單體式架構形成鮮明對比，在單體式架構中，應用程式的各個部分緊密耦合在一起，形成一個大型的程式碼庫。

### 微服務架構的主要特點

- **獨立部署：**每個微服務都可以獨立部署，而無需重新部署整個應用程式。 這是微服務架構的主要優勢之一，因為它允許團隊更快、更頻繁地發佈新功能和錯誤修復。
- **圍繞業務領域建模：**每個微服務都應該專注於特定的業務功能。 這有助於確保每個微服務都具有明確的責任，並且易於理解和維護。
- **擁有自己的數據：**每個微服務應該負責管理自己的數據。 這有助於確保數據一致性，並防止不同服務之間的數據依賴性。
- **小型且專注：**微服務應該保持較小的規模，並專注於單一功能。 這有助於提高可維護性和可擴展性。

### 微服務架構的優點

- **提高敏捷性和速度：**獨立部署允許團隊更快、更頻繁地發佈新功能。
- **提升可擴展性：**每個微服務都可以獨立擴展，以滿足特定的需求。
- **提高彈性和容錯能力：**如果一個微服務發生故障，其他微服務仍然可以繼續運作。
- **技術異質性：**團隊可以為不同的微服務選擇最適合的技術。
- **更有效的團隊協作：**小型且專注的服務更容易理解和維護，這使得團隊更容易協作。

### 微服務架構的挑戰

- **複雜性增加：**管理多個微服務比管理一個單體式應用程式更複雜。
- **分散式系統的挑戰：**分散式系統引入了新的挑戰，例如網路延遲、容錯和數據一致性。
- **部署和維護的開銷：**部署和維護多個微服務需要更多的工具和流程。
- **服務間通訊：**微服務需要相互通訊，這可能會導致效能問題和複雜性。

### 微服務架構的應用場景

微服務架構適用於各種應用程式，但它特別適合於以下場景：

- **大型且複雜的應用程式：**微服務可以將大型應用程式分解成更易於管理的小型部分。
- **快速發展的應用程式：**微服務架構可以支持快速迭代和頻繁發佈。
- **需要高可擴展性和彈性的應用程式：**微服務可以根據需求獨立擴展，並提供更高的容錯能力。

### 總結

微服務架構是一種強大的軟體開發方法，它可以提供許多優勢。但是，它也引入了新的挑戰，需要仔細考慮和規劃才能成功實施。

# Q: 我們需要"真的"微服務嗎？
- 目的？想要達成的目標是什麼
- Polunga & Worker & CS Tool
- Team/Organization Sight


**請注意，以上信息僅來自您提供的資料來源和我們的對話紀錄。**

https://medium.com/design-microservices-architecture-with-patterns/headless-architecture-with-separated-ui-for-backend-and-frontend-f9789920e112
https://medium.com/design-microservices-architecture-with-patterns/microservices-architecture-problems-and-solutions-with-pattern-and-principles-b673f342dc10


https://www.sitecore.com/resources/insights/ecommerce/headless-ecommerce-vs-microservices

## 微服務架構中的身份驗證  Authentication in Microservice

https://frontegg.com/blog/authentication-in-microservices
https://dev.to/behalf/authentication-authorization-in-microservices-architecture-part-i-2cn0

https://gamma.app/docs/Authentication-in-Microservices-Approaches-and-Techniques-un51umyqrwfnofp?mode=doc

微服務架構是由多個獨立組件透過 API 整合而成的應用程式。由於每個微服務執行不同的功能，因此在處理微服務的請求時，驗證每個請求的來源至關重要。這確保只有合法的服務和使用者才能存取每個微服務。

### 微服務驗證的挑戰 / Challenges

- **集中式依賴：**每個微服務都必須個別處理身份驗證和授權邏輯。雖然可以在所有微服務中使用相同的程式碼，但這需要所有微服務都支援特定的語言或框架。
- **違反單一職責原則：**微服務應該只執行一項功能。新增全域身份驗證和授權邏輯會讓微服務執行額外的功能，使其可靠性降低且更難管理。
- **複雜性：**微服務中的身份驗證和授權可能會導致非常複雜的情況。這表示使用者、微服務和第三方系統都可能存取每個微服務。這種複雜性會讓實作和維護變得困難。

### 微服務架構的身份驗證策略 / Approaches

- **邊緣層級授權：**這是一種簡單的策略，授權只發生在邊緣，通常使用 API 閘道器。您可以使用 API 閘道器集中處理所有下游微服務的身份驗證和授權。閘道器會為每個微服務強制執行身份驗證和存取控制。但是，如果攻擊者突破閘道器，他們就可以自由存取任何微服務，因此這種策略的安全性較低。此外，如果系統很複雜，有很多角色和存取控制規則，將所有授權決策都推送到 API 閘道器可能會變得難以管理。最後，由於 API 閘道器通常由營運和維護團隊設定，開發團隊無法直接變更權限，這可能會導致溝通和流程的負擔。
- **服務層級授權：**這種策略允許每個微服務直接進行身份驗證和授權。優點是每個微服務可以更好地控制其存取控制策略的執行。
- **外部實體身份傳播：**這種策略可以在考慮使用者情境的同時做出授權決策。例如，它可以根據使用者 ID、使用者角色或群組、使用者位置、時間或其他參數來更改授權決策。

### 微服務架構的身份驗證技術 ? Techiques

- Session-based 
- Token-based
- OAuth
- API Gateway

- **單一登入 (SSO)：**SSO 允許使用者或實體登入一次即可存取多個系統。在微服務架構中，SSO 可以用於驗證終端使用者或驗證需要連接到其他微服務或透過 API 請求存取的微服務。您可以使用身份和存取管理 (IAM) 解決方案來設定使用者資料庫並定義使用者面對微服務的權限。
- **JSON Web Token (JWT)：**JWT 提供一種機制，可以將一組聲明或屬性以加密和安全的方式從客戶端傳輸到微服務應用程式。JWT 也可用於保護服務之間的通訊，或在微服務之間傳遞終端使用者上下文和資料。
- **OAuth API 身份驗證：**OAuth 2.0 提供了一種業界標準協定，用於在分散式系統中授權使用者。在微服務架構中，OAuth 2.0 客戶端憑證流程支援 API 客戶端和 API 伺服器之間安全的伺服器到伺服器通訊。

### **其他注意事項**

- **將身份驗證與授權分開**是一種良好的做法。身份驗證確認使用者是誰，而授權決定使用者可以做什麼。將這兩者分開可以讓您的系統更靈活、更安全。
- **選擇正確的存取管理控制取決於應用程式的需求**。不同的應用程式可能有不同的安全需求，因此沒有單一的解決方案適合所有情況。
- **在微服務架構中實作和強制執行 IAM 並非易事**。您需要仔細考慮如何設計您的系統，以便在不犧牲安全性的情況下保持靈活性和可擴展性。

**請注意，以上資訊僅來自您提供的資料來源和我們的對話紀錄。**

## Token Based
- 一般作法是將 token 放在 Header: X-Authorization = Bear {token}
- 但在非 headless 架構下，我們必須把它放在 cookie 裡
- XSS

JWT
https://dev.to/behalf/authentication-authorization-in-microservices-architecture-part-i-2cn0

Cookies
https://www.writesoftwarewell.com/how-http-cookies-work-rails/

```
cookies[:jwt] = { 
  value: token, 
  httponly: true, 
  secure: Rails.env.production?, 
  same_site: :strict, 
  domain: :all # 或者 'example.com' 
}
```

- HttpOnly
- SameSite
- Secure
- Domain
- Expires

## Session Based

https://www.g9labs.com/2016/06/24/migrating-to-phoenix-with-rails-session-sharing/

### OAuth

## API Gateway

![[Pasted image 20241112115337.png]]

https://microservices.io/patterns/apigateway.html

### KONG
![[Pasted image 20241112143032.png]]
- Lua


### AWS API Gateway
- Lambda
![[Pasted image 20241112135045.png]]







- ollama / n8n
- Swarm
- Compute use
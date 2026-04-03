現在的系統有實作產品訂閱功能，包括處理訂閱的建立、續訂等操作，以及與第三方金流的串接整合


我將這套系統拆分出來成為一個獨立的支付系統服務。
一來降低了主系統的複雜度與耦合度，
二來讓支付系統能夠專注於處理支付相關的邏輯和流程，提升整體的可維護性和擴展性。

並且最重要的未來會再開發更多的產品，都可以直接使用這個獨立的支付系統服務，避免重複造輪子。

請幫我從需求討論、分析設計，架構規劃，介面設計到程式開發實作出這整套支付系統服務。

過程中我們可以持續討論細節與調整需求，以確保最終的系統符合預期目標。

幾個重點我希望特別注意：

- 循序漸進：從簡單的功能開始，逐步增加複雜度，確保每個階段都能穩定運作。
- 模組化設計：將系統拆分為多個模組，方便未來的擴展和維護。
- 先從同一個專案開始實作，將功能拆分減少耦合，再未來獨立成另外的一個專案
- 完整的測試覆蓋：確保每個功能都有對應的單元測試和整合測試，提升系統的穩定性。
- 未來新增加的產品或服務都能直接使用這個支付系統服務。而且可能會用不同的程式語言、技術實作。


需求：

- 支援多種第三方金流服務，目前系統至少有 tappay 與 ezpay，未來可能支援更多（如 Stripe、PayPal 等）
- 支援各種訂閱型態，包括試用、月費、季費、年費、或多樣客制的 pricing schedule。
- 可以追蹤各產品的客戶訂閱狀態，包含啟用、取消、過期等。且保留客戶管理功能的彈性；可能從系統上做客戶關係管理，也可能串接第三方 CRM / 客服系統。
- 為了方便用戶訂閱使用各項產品，需要實作 SSO（Single Sign-On）功能，讓用戶能夠使用單一帳號登入並訂閱不同的產品。

若還有任何需求或細節需要補充，請隨時告訴我。




  1. 用戶身份
  - Nextrek 用戶與 Nexign 用戶是 1:1 對應嗎？
  - 還是 Nexign 不管理用戶，只接收 Nextrek 傳來的 user reference？

  2. 部署架構
  - Nexign 會獨立部署（不同 server/container）？
  - 還是與 Nextrek 同一個部署單位？

  3. 使用場景
  - Nexign 只服務 Nextrek，還是未來可能有其他系統使用？

  4. 資料所有權
  - 訂閱/付款資料是否需要同步回 Nextrek？
  - 還是 Nextrek 只需要知道「這個用戶有沒有有效訂閱」？

  ---
  這些問題會影響整合架構的設計。你對這幾點有什麼想法？

1. 這裡的用戶是指？

我的認知是指 SSO 登入的用戶身份，那就是 1:1 對應的關係。
但不是每個用戶都一定會有訂閱行為，

而訂閱的對象在 Nextrek 是綁定在 Group 層級的
所以就我的理解 Group 也會有 1:1 的對應關係在 Nexign 上。

2. Nexign 會獨立部署在不同的 server/container 上。
包括 ECS Cluster, ALB, S3 等等都會是獨立的資源
只有 RDS 會跟 Nextrek 共用同一個 VPC 裡的資料庫資源。

3. Nexign 目前只服務 Nextrek，但未來有可能會有其他系統使用這個支付系統服務。

4. 我的認知是不用，訂閱/付款資料未來都只會存在 Nexign 這邊。
App(Nextrek) 端只需要透過 API 取得用戶的訂閱狀態即可。

關於1的部分請再多說明你的想法





我想再確認各項設計

一、訂閱管理的設計

我想試著從 Nexign 這邊的，作為一個與允許與不同 app 整合的的獨立抽象服務的角度來看
來看 Group 和 User 的角色定位

照前面的討論，訂閱是綁定在 Group 層級的

Group 代表的是可被訂閱的產品單位 -> 這樣的形容沒錯吧
- 對應到一項產品(app) 的使用單位
- 應用了一種訂閱方案
- 可以有一個或多個 User 管理者/擁有者(或擁有者是唯一，但管理者可以有多個)
- Group 有 subscription、orders、payments 等紀錄，以及 payment methods 等設定

User 則是
1. 對 group 有權理權，來管理 group 的訂閱
2. User 也有可能管理多個來自不同 Account 的 group 的訂閱
3. 更主要是做為 SSO 的登入身份，不是每位 user 都需要有 group 的訂閱有互動行為
- User 可能只是單純登入 Nexign 來管理自己的帳號資訊
- 也可能是登入 Nexign 來管理自己所具有管理權的 group 的訂閱

其中一個疑慮是 Account 這層概念在 Nexign 似乎沒有功能上的意義
但對 app 端來說 Account 有可能是必要的概念，例如 Nextrek 端的 Account 代表一個企業客戶，有指定的 domain 名稱
又或者在 Nexign 端 account 會是一個非必要的概念，就像 Google Cloud 上的無組織帳號一樣？

又像是如果 app 端的設計是給個人用戶使用的話，Account 這層就可以省略

二、SSO 的機制
Nexign 的SSO對象是 User
Nextrek 或其他 App(假設還有另一個 App Nexalon)，的使用者都可以透過 Nexign 的 SSO 登入

但這裡會有兩種流程

1. User 從 App 端(Nextrek/Nexalon) 操作，會先跳轉到 Nexign 的 SSO 頁面進行登入(OAuth2/OIDC 流程)，登入成功後再跳轉回 App 端
在轉跳前會在 App 端紀錄 return url，所以登入成功後能夠正確跳轉回去，這應該沒問題

2. User 從 Nexign 直接登入，這個流程會比較複雜，因為 Nexign 需要知道 User 是要登入到哪個 App
或是出現列表讓 User 選擇要登入到哪個 App
但也就是說 Nexign 需要知道有哪些 App 是允許這個使用者登入的

這部分會涉及訂閱的細節，例如 Group A 有訂閱 Nextrek 和 Nexalon 兩個 App
但只有 Group A 有 allocate seat 給 User 1 使用

又或者 Group A 訂閱了 Nextrek，但沒有訂閱 Nexalon，但 Group B 訂閱了 Nexalon，
User 1 則有被 GroupA & Group B 同時 allocate seat，那他的登入後選項就會有 Nextrek Group A 和 Nexalon Group B 兩個選項

換問話說 Nexign 需要知道 User 所屬的 Group 訂閱了哪些 App 且授權給哪些 User 使用
但實務上，使用者又管理 seat 的 allocation 通常是在 App 端進行的。

像是 Google Cloud 或是 Onelogin 的這類的服務，又是怎麼設計的來處理這個問題的呢？




# 需釐清 API Authentication 的問題

為了目前同時有 jwt token 與 oauth 兩種 api 驗証方式？我看到

Nexign 在做 API 驗証的時候 ，同時支援 jwt token 與 oauth 兩種方式
而 Nextrek 端目前是使用的 NexignClient 也實作了兩種方式
但實際上只有用 API(JWT?) TOKEN 一種方式在呼叫 Nexign 的 API 而己

這樣的設計是有什麼特別的考量嗎？



# 將 Nextrek 的訂閱管理功能，改成由 Nexign 來處理

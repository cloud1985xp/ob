製作 Github Action 的 workflow，包括相關可能需要的 shell script 腳本，來完成以下的 ci 目標

目的：
在 AWS ECS 上用 CI 選擇的目標 branch 運行起一個獨立的 app service
然後在 ALB 上將特定 domain 的流量導向這個新建立的 service，以讓 QA 可以測試這個 branch 版本的程式

需求：
當使用者輸入一個 tag 和選擇 branch，執行 workflow
能自動在 Github Action 已授權指定的 aws 帳號，對 ec2、ecs、ecr 等服務進行操作，完成以下事項：

- 用指定的 branch build docker image 並用給的 tag name 打上 image tag，push image 到 ecr
  - 可參考現有的 workflow：.github/workflows/build_and_deploy.yml 中 build image 的部分
  - 可以把這個 build image 的步驟獨立成一個 workflow，讓其他 workflow 方便去執行它來進行 build image push to ECR 的動作

- 用一個給定的ecs task definition 做樣版(或從既有的複製)，但將指定的 container 替換成上述的 image tag，定義成另一個 task definition
  - 或產生成新版本的即可
  - 或者如果有辦法不用產生新的 task definition，直接在在下一步運行時 overwrite image，請確認是否可行
- 在 ecs 上建立一個 service，用這個 tag 當 suffix 命名，ex: tag=t20250123，命名為 app-t20250123，並用上一步的 task definition 啟動 ECS Fargate Task
- 在 alb 上增加一個 listener rule，會將 request 中，hostname 的 subdomain 符合這個 tag 的請求導向上述 ecs 建立的 service
- 請補充要達到上述目標會需要建立的其他 aws resources
- 請將對應的 ALB, Task Definition 等定義為參數方便我調整、維護，你可以先用 sample 值
- 請參考現有 .github/workflows 裡現有的設計，來盡可能取得需要的設定值，例如 AWS Account, Secret, 環境變數 等等
- 目標的環境必定是 staging，不允許影響 production



請製作一個 github action 的 workflow，來進行 application 的 release 準備，需完成以下目的

- 接受一個參數：version_tag
- 用指定執行的 branch 在 Github 上建立一個 release，
  - 讓 Github release 自動與上一次的 release 比較來產生 release content/note
  - 並設定在 release publish 時用 version_tag 參數值來加上 tag
- 執行代碼掃描
  - 使用 brakeman 進行代碼掃描
  - 可以將這個動作建立成另一個獨立可重複呼叫的 workflow
  - 可以使用現成的 github action 來執行
- 發送 slack 通知
  - 建立 release 以及執行代碼完成後，發送 slack 訊息通知
  - 可以參考 build_and_deploy.yml 這個 workflow 裡的設定取得 slack token
  - Slack 訊息的頻道則另外定義在這個 workflow 裡即可，先用 sample 值，我可以自己修改
  - Slack 訊息的內容包含以下資訊：
    - 這次 release 的 tag_name 與 branch
    - release note
    - 附上連結連到這個 release
    - 如果可以，我希望能將 brakeman 執行的結果也附上
      - 或是附上 brakeman github action 執行的 link

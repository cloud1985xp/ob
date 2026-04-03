# 新增 Create Account Workflow

請建立一個新的 github workflow 來建立一個全新的 account 和制作 seed data

接受一個必填的 input 參數，做為 account_name


然後去使用 ecs-run-task 的 action 來運行以下指令

```
seed:group ACCOUNT={account_name}
```

- environment 必定是 staging
- launch_type 必定是 FARGATE
- launch_size 必定是 large

請參考現有的 run_rake.yml workflow
以及 action/ecs-run-task/run.sh

實際上執行的完整 run.sh 指令範例：

```
./run.sh -p nextrek -e staging -t FARGATE -S large seed:group ACCOUNT={account_name}
```

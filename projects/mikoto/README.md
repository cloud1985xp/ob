---
tags:
  - mikoto
  - project
created: 2024-01-01
updated: 2025-01-23
status: active
---

# 20240122 PresentBox Problem

- Do we have enough time to prepare/test/review
- Prepare/test/review should be done separately(person)
  - It's recommended to let other junior to develop and verify by senior
  - It's not tested by QA
  - It didn't verify well when doing test


# Key Rotation

- Use jenkins job to replace updating banner/gl-op-tools/adjust credentials on 1password?

# FutureVuls

cd /opt/vuls-sass
./vuls scan
ls results
./vuls help

# v2.12 Upgrade Dev Environments

▼dev01：目前裝 8/18dp 的內容
　期望日程： 8/22(二) ~ 8/24(四) 之間
　原因：
　　1. 希望可以等到 8/18 部署到外部後，稍作觀察再做更新
　　2. 預計下次要再使用到這個環境是 8/28(一) 當天

▼qa：目前裝 8/25dp 的內容
　期望日程： 8/29(二) ~ 8/30(三) 之間
　原因：
　　1. 希望可以等到 8/25 部署到外部後，稍作觀察再做更新
　　2. 預計下次要再使用到這個環境是 9/01(五) 當天


# Jenkins
docker run -m 12g --restart always -d -p 80:8080 -p 50000:50000 -u root \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /var/jenkins_home:/var/jenkins_home \
  -v /root/.aws:/root/.aws \
  -v /root/.ssh:/root/.ssh \
  -v /root/.config:/root/.config \
  -v /root/.gitconfig:/root/.gitconfig \
  -v /root/.rbenv:/root/.rbenv \
  -v /root/.mikoto-gl-ops-credentials.yml:/root/.mikoto-gl-ops-credentials.yml \
  -v /mnt:/mnt \
  --env JAVA_OPTS="-Dhudson.model.WorkspaceCleanupThread.disabled=true" -t mikoto-dev-jenkins

JENKINS_CONTAINER_NAME=mikoto_jenkins
docker run -d \
    -u root \
    -p 80:8080 \
    -p 50000:50000 \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /data/var/jenkins_home:/var/jenkins_home \
    -v /root/.aws:/root/.aws \
    -v /root/.gitconfig:/root/.gitconfig \
    -v /root/.ssh:/root/.ssh \
    --env JAVA_OPTS="-Dhudson.model.WorkspaceCleanupThread.disabled=true" \
    --restart always \
    --name $JENKINS_CONTAINER_NAME \
    jenkinsci/blueocean:latest


saml-sp-metadata.xml
jenkins.model.JenkinsLocationConfiguration.xml

New Built AMI
ami-056eca4d3fd21e72f

Version
from: Jenkins 2.303.2
to: Jenkins 2.346.3

New jenkins
admin
496c4bbf3ba644b8ba2a8c24646f7bd5

# Restore Memcached Data of Admin Settings

Click repair for each item in each page:

- Maintenance
- Super user
- Client Version
- Asset Version -> no need, it got update after deployed

# Jeff Talk
- Operation Issue
  - Deployment
  - Jenkins Operation
  - Testing Issue
- Infra
  - Update AMI
  - CFn / TF
- Version Update
  - Server Code
  - Master Data
- Documentation
- Elixir
- Tooling

3/3 release v2.7
2/24 - 3/10 process v2.10 masterdata
2/22 v2.11-jp release

# Tips
deploy staging
need to check client/master data version
could be found at:
https://qa-api.mikoto-gl-dev.aktsk.com/avalon#/operation/asset_master_version

# 2022-23 Q3Q4
Stablility of Tool
Airtest usage percentage
contact chalern for understang the operation issu

# Mac EC2 Instance
Could use ssm session to build a tunnel for vnc login
- https://github.com/aws-samples/amazon-ec2-mac-instance-jenkins-agent#configuration-of-the-agent

# 2022-23 RI
- Spot Instances
- Merge shard in US, EU
- Merge rds for dev environment
- Remove avalon
- Consider to apply AWS Mac Mini

# Recalculate KPI
- Delete Existed Data of the Date
- Check daily script to see what tables need to delete data
  - daily
  - last_login ip country
- Rerun script of them to calculate
- Rerun daily report task to re-send data to googlesheet / slack


# Kaizen
- [x] Stress Env
- Seed CS Master DB usually failed
- Staging AWS Problem
- ISHIN Restart Server

# v2.0.0 deployment

Because it has user DB migration, we run it separately
- But Akash said we run Production_deploy_assets to deploy server <- which I am not understadning what 'deploy assets' including app server or not


## MKT MasterData
data folder -> final export version
config.yml
environment.yml

submodule as base data
gl planner write excel files under masterdata folder
  - extra data
  - overwrite base data

{envname} folder
  csv
  json
  meta

schema
base_schema <- pull from jp
additional_schema

# Oct -> 2.4.0
Goal: enter the game on penetration
before: 11 Aug
- prepare masterdata branch test on this clean branch on penetration than give them to use that for debug
- 9 August the deadline

11 Aug start to setup debug data on dev02

dev02


# Issue when Merge v2.4.0 infra

- Found jp.bq_batch has cronjob setting but not used in gl version
  -> Found it's replaced in mikoto-gl-ops to run the task scheduled by jenkins
  -> notify_masterdata_event =
  - #masterdata_event
    - Add process to re-clone master repo periodicaly, but GL has move it to jenkins and jenkins will fetch the repo when start running the task
    - Add error message in case of the notification process failure
  - #kpi_notification
    - Added error handling, but GL has processed it in jenkins pipeline
## Client Jenkins
  - Double Check if GL is using client-jenkins? -> 目前看沒有建立 client-jenkins stack (only server-jenkins in AWS-dev)
  - [TODO] Change Slave connection protocol from JNLP to WebSocket
  - [TODO] In Cloudformation,
    - Originall client-jenkins uses EPI, now replaced by building ALB/ELB
    - client-jenkins add parameterts of jenkins certificate (also used by server jenkins)

- Add ClientJenkinsLoadBalancerSecurityGroup

## CloudFormation

### envfiles, in mikoto-dev / mikoto-*

- Add `INFO_CLOUDFRONT_IPV6_ENABLED`
- Add `HAS_DEPLOY_PARAM_ACCESSOR` (v2.3.0)
- Update `UPDATE_ASSETS_VERSION_FUNCTION_VERSION` -> global maybe unchanged
- Add `ENABLE_SANDBOX_ASSETBUNDLE`

### common imagebuilder
- Typo in components, but GL ver didnt use imagebuilder (no stack)

### common lambda
- Add new function resources (v2.3.0)
  - patch_protectwise_sensor (Enabled only in AWS-dev)
  - patch_bq_batch (Enabled only in AWS-dev)
  - layers/ec2_layer sub resource of new lambda functions above(patch protectwise/bq_batch)
  - layers/ssm_layer sub resource of new lambda functions above
- Removed ECS Task Schedule Handler

### Common CodeBuild

- Add project of deploy-params-accessor, configured by HAS_DEPLOY_PARAM_ACCESSOR
  - It's a tool to access googlesheet to retrieve deploy params, but GL does not use the googlesheet to manage ddeployment

### Common Info
- Use INFO_CLOUDFRONT_IPV6_ENABLED to toggle IPV6, enabled in production

### Server-Jenkins Stack

- Add DiskAlarm

### SSM Stack

- Add 3 Documents
  - check_bq_batch_status_after
  - check_bq_batch_status_before
  - check_protectwise_sensor_status

### Environment Lambda

- Add Additional Lambda Function of AutoScalingGroup/CW Event/Alarm which determined by Environment setting (ENABLE_SCHEDULE_GACHA_AUTO_SCALE)
- Update function of update_assets_version/app.rb
  -> Change to also update assets version which stored in Redis (it doesn't store in redis before?)

### Environment Subnet

- Add Peering Connection configured by ENABLE_PROTECTWISE_VPC_PEERING_INACCOUNT
  -> Check does GL need this or what's the affects of multi-regions

# v2.0.0 Release
- 7/14 release
- staing 7/10, 7/11
- Switch bq-sender dataset from mirror to real kpi logs
- resume mirror: nev02, feature01,

# Copy Cluster
- from Asia
- db type
- does JP staging has player/guild combined RDS cluster?


# Deploy Staging
## For test some infra
- Revision can use last deploytment tag, ex 20220610-v1.26.0-hotfix
- Target use the default

# Live Deploy

= Production_deploy_assets
  = Only deploy assets
  JENKINS_BRANCH=global-master
  SOURCE_ENV= staging-all
  TARGET_ENV= production-all
  - DEPLOY_ASSET
  - DEPLOY_MASTER
  SERVER_BRANCH=global-master
  ADMIN_BRANCH=global-master
  AVALON_BRANCH=global-master
  MASTERDATA_REVISION=TBD, usually YYYYmmdd-v{version}, ex: 20220602-v1.26.1
    * For hotfix, it appends -hotfix at suffix, like 20220610-v1.26.1-hotfix
  ECS_MINIMUM_HEALTHY_PERCENT=50
  CLIENT_BRANCH=gl/master

# Infra-formation, Merge new version Back to global-gl

- Because jenkins was running global-master before, after merged we need to test the changed jenkins job again
- Can test through deploy_manager
- Same client related tasks was in Dashboard > Client > client_tw in operation folder in version upgrade folder, ex: v2.0.0 the version upgrade folder was created by client engineer, usually copied from last one relation generator EKS / gke Gacha Module @ Client Designer Tool -> Banner Generator -> Jilian / Cloud BQ-Sender 後續 M1 Problem
  - 日本已經都換 m1
  - Jenkins Mac


Kevin 交接
- 怎麼得知部署要用的 branc

penetration -> dev02 -> develop(AS + EU) -> develop(US) -> feature01
  -> dev01 -> qa-us -> (stress)
  (prod-account)-> review-us -> staging(ALL) -> production(ALL)

[] common
[x] penetration
[x] dev02
[] develop(AS+EU)
  [] common-as
  [] common-eu
    - Problem of running codebuild, using AWS CLI v2
[] develop(US)
[] feature01

```
US
bundle exec rake stack:mikoto-global-dev:common:airtest-bucket:diff
bundle exec rake stack:mikoto-global-dev:common:api-gateway:diff

bundle exec rake stack:mikoto-global-dev:common:cfn-check:diff
bundle exec rake stack:mikoto-global-dev:common:chatbot:diff
bundle exec rake stack:mikoto-global-dev:common:server-jenkins:diff
bundle exec rake stack:mikoto-global-dev:common:waf:diff
bundle exec rake stack:mikoto-global-dev:common:sync-service:diff
bundle exec rake stack:mikoto-global-dev:common:ssm:diff
bundle exec rake stack:mikoto-global-dev:common:protect-wise:diff
  bundle exec rake stack:mikoto-global-dev:common:bq-batch:diff
bundle exec rake stack:mikoto-global-dev:common:info:diff
bundle exec rake stack:mikoto-global-dev:common:lambda:diff
bundle exec rake stack:mikoto-global-dev:common:codebuild:diff
bundle exec rake stack:mikoto-global-dev:common:bucket:diff
bundle exec rake stack:mikoto-global-dev:common:role:diff

AS
x bundle exec rake stack:mikoto-global-dev-as:common:airtest-bucket:diff
- bundle exec rake stack:mikoto-global-dev-as:common:cfn-check:diff
x bundle exec rake stack:mikoto-global-dev-as:common:chatbot:diff
- bundle exec rake stack:mikoto-global-dev-as:common:waf:diff
x bundle exec rake stack:mikoto-global-dev-as:common:ssm:diff
- bundle exec rake stack:mikoto-global-dev-as:common:protect-wise:diff
- bundle exec rake stack:mikoto-global-dev-as:common:info:diff
- bundle exec rake stack:mikoto-global-dev-as:common:lambda:diff
x bundle exec rake stack:mikoto-global-dev-as:common:codebuild:diff
x bundle exec rake stack:mikoto-global-dev-as:common:bucket:diff
x bundle exec rake stack:mikoto-global-dev-as:common:role:diff

EU

x bundle exec rake stack:mikoto-global-dev-eu:common:airtest-bucket:diff
- bundle exec rake stack:mikoto-global-dev-eu:common:cfn-check:diff
x bundle exec rake stack:mikoto-global-dev-eu:common:chatbot:diff
- bundle exec rake stack:mikoto-global-dev-eu:common:waf:diff
x bundle exec rake stack:mikoto-global-dev-eu:common:ssm:diff
- bundle exec rake stack:mikoto-global-dev-eu:common:protect-wise:diff
- bundle exec rake stack:mikoto-global-dev-eu:common:info:diff
- bundle exec rake stack:mikoto-global-dev-eu:common:lambda:diff
x bundle exec rake stack:mikoto-global-dev-eu:common:codebuild:diff
x bundle exec rake stack:mikoto-global-dev-eu:common:bucket:diff
x bundle exec rake stack:mikoto-global-dev-eu:common:role:diff



rake stack:mikoto-global-dev:common:ci:diff
rake stack:mikoto-global-dev:common:client-jenkins:diff
rake stack:mikoto-global-dev:common:diff
rake stack:mikoto-global-dev:common:imagebuilder:diff
rake stack:mikoto-global-dev:common:ptolemy:diff
rake stack:mikoto-global-dev:common:redash-akg:diff


SKIP_NETWORK=yes bundle exec rake stack:dev02:app:diff
SKIP_NETWORK=yes bundle exec rake stack:dev02:batch:diff
SKIP_NETWORK=yes bundle exec rake stack:dev02:bucket:diff
SKIP_NETWORK=yes bundle exec rake stack:dev02:codebuild:diff
SKIP_NETWORK=yes bundle exec rake stack:dev02:dynamo:diff
SKIP_NETWORK=yes bundle exec rake stack:dev02:elasticache:diff
SKIP_NETWORK=yes bundle exec rake stack:dev02:lambda:diff
SKIP_NETWORK=yes bundle exec rake stack:dev02:rds:diff
bundle exec rake stack:dev02:network:diff
```

Actual Only Need
```
bundle exec rake stack:dev02:network:update
SKIP_NETWORK=yes bundle exec rake stack:dev02:app:update
SKIP_NETWORK=yes bundle exec rake stack:dev02:codebuild:update
SKIP_NETWORK=yes bundle exec rake stack:dev02:dynamo:update

bundle exec rake stack:development-as:network:diff
SKIP_NETWORK=yes bundle exec rake stack:development-as:app:diff
SKIP_NETWORK=yes bundle exec rake stack:development-as:codebuild:diff
SKIP_NETWORK=yes bundle exec rake stack:development-as:dynamo:diff

bundle exec rake stack:development-eu:network:update
SKIP_NETWORK=yes bundle exec rake stack:development-eu:app:update
SKIP_NETWORK=yes bundle exec rake stack:development-eu:codebuild:update
SKIP_NETWORK=yes bundle exec rake stack:development-eu:dynamo:update

bundle exec rake stack:development:network:diff
SKIP_NETWORK=yes bundle exec rake stack:development:app:diff
SKIP_NETWORK=yes bundle exec rake stack:development:codebuild:diff
SKIP_NETWORK=yes bundle exec rake stack:development:dynamo:diff
```

info -> 多環境共用

dev01
  ->app ALB
  ->auth ALB(internal)
TxCache -> prevent duplicate request/response


## Admin Tool
https://penetration-api.mikoto-gl-dev.aktsk.com/admin/client/version
Admin: HTTP Basic Auth
pk_fire
icestorm


## TO ASK
Branch Management
Server Log - Backtrace?
### mDiff
- googlesheet priv/goth... can be changed?

## Check Client Version at Admin Tool
ex:
https://penetration-api.mikoto-gl-dev.aktsk.com/admin
https://penetration-api.mikoto-gl-dev.aktsk.com/admin/client/version

## Find Master Data in Avalon
ex:
https://development-api.mikoto-gl-dev.aktsk.com/avalon#/content/full_master_data/
https://development-api.mikoto-gl-dev.aktsk.com/avalon#/content/full_master_data/kakusei_material_set?search=id%3A30001


## Loading Test

Locust is setup under environment
-> need to update host before

-> update alb sg (ingress list)
-> update waf

sleep(1000) -> careful

when removing locust
it would happend dependency issue of security group
because the cfn rake always runs network update before other stack
the solution is add flag to skip network update, e.g.:

> SKIP_NETWORK=yes bundle exec rake stack:penetration:app:update

# Update AWS Log

bundle exec rake stack:mikoto-global-dev-as:common:lambda:update
bundle exec rake stack:mikoto-global-dev-as:common:role:update

bundle exec rake stack:mikoto-global-dev:common:lambda:update
bundle exec rake stack:mikoto-global-dev:common:role:update

bundle exec rake stack:penetration:app:update
bundle exec rake stack:dev01:app:update
bundle exec rake stack:dev02:app:update
bundle exec rake stack:qa-us:app:update
bundle exec rake stack:feature01-us:app:update
bundle exec rake stack:stress:app:update

bundle exec rake stack:mikoto-global-prod-us:common:lambda:update
bundle exec rake stack:mikoto-global-prod-us:common:role:update

rake stack:staging-as:app:diff
rake stack:staging-eu:app:diff
rake stack:staging-us:app:diff

rake stack:production-as:app:diff
rake stack:production-eu:app:diff
rake stack:production-us:app:diff

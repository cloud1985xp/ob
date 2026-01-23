---
tags:
  - interview
  - hr
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Template - Junior

Created: 2022年4月11日 下午1:30

Junior Insight

60%

- 考題過
- Rails
    - AR
    - MVC
    - Restful
    - 
    - Migration
    - Deployment
- Ruby
- DB
    - definition
    - Transaction

- 如何檢別出技能是否達到標準
    - 目試前請他進行
        - RSpec 題目
    - 面試前我們採行的動作
        - 批改作業
        - 看 repo (若有提供)
    - 面試
        - Rails 架構
        - Request - Response Workflow
    - 物件導向？
    - 計概網概
    - 程式設計邏輯
    - 伺服端領域
    - 資料庫基礎
    - 有沒有準備考古題
    - 程式題
- 技能都差不多的情況下，如何觀察出其適不適合
    - 成為工程師的動機
        - 
    - 是否具備工程師特質
        - 自主學習
        - 邏輯思考、反應
        - 拆解問題、深入淺出
            - 例出實例需求，詢問會怎麼實作
    - 對未學到//不熟悉的技術的態度
        - 課堂中沒深談後來自己深入研究的？
        - 還有什麼主題是自己想要再深入研究的？為什麼
    - 對工程師職涯的認知與規劃
    - 軟實力
        - 溝通
        - 團隊合作
        - 性格

### What do you know about server engineer at Akatsuki

- Why want this job
- What expect in this job
- What's the challenges in this job
- How do you imagine developing a game server with ruby/rails
- How you learn and get growth after leave Camp?

Depends:

- Project exp. sharing
- Self Evaluation: What’s the difference / distance with you become a qualified engineer?
- Rebase?
- Web development vs Current Requirement
- Next step you want?

# Tech

### Ruby , Rails

- Ruby, Rake, Rack, Rails
- Single quote vs double quote?
- Extend vs Include
- Lambda vs Proc vs block
- String vs Symbol

Consider following codes

[Q1](https://gist.github.com/cloud1985xp-aktsk/bb4aaceee1b0235c719a26997070f444)

```ruby
class MyController < ApplicationController
  def options
    options = {}
    available_option_keys = [:first_option, :second_option, :third_option]
    all_keys = params.keys.map(&:to_sym)
    set_option_keys = all_keys & available_option_keys
    set_option_keys.each do |key|
	    options[key] = params[key]
	  end
		options
	end
end
```

What’s the issue with the controller code below? How would you fix it?

[Q2](https://gist.github.com/cloud1985xp-aktsk/6719f2bf368001869f36febdefb7ee3e)

```ruby
class CommentsController < ApplicationController
  def users_comments
    posts = Post.all
    comments = posts.map(&:comments).flatten
    @user_comments = comments.select do |comment|
      comment.author.username == params[:username]
    end
  end
end
```

- How do you explain class and module, when/how you decide to use them
- How do you explain Duck-Typing
- How do you explain the ORM / MVC
- How do you explain RESTFul
- CSRF issue
    - Why POST, PUT, DELETE
- Session
- Observer vs Callbacks
- &symbol operator in block, how it actual work
- Instance method public/private or protect
- RSpec concept, introduction
- Cache

### ActiveRecord

- delete vs destroy
- What is migration, what can it do, why
- include vs join
- lazy load
- magic finder
- Polymorphic and STI

### Open Question

- Experience about of Working with legacy codes
- Share the experience about upgrading ruby / rails & gems
- Experience about Server operation / deployment?
- Experience about API development
- Experience about using Redis
    - For what? Any usage of specific data struct of it?
    - What If the built-in data struct didn't meet the requirements
- Any pattern used in you rails application development

### Infra. / DevOps

- Container
    - Why use container
    - List 3 tips when designer container/dockerfile
- 2 layers vs 3 layers
- CI & CD
- Experience of Automation
- Experience of
    - Docker & Container, how explain it
    - Selenium
    - Chef, Anaisble or Puppet, configuration management / provision deployment tool
    - IaC (Infrastructure as Code)
- Monitoring, tool? explain about Continuous Monitoring
- Server operation experience, AWS / GCP?
    - Used what services?
- Please explain SLI, SLO and SLA (Server Level Agreement)
    - How will you measure the reliability of the service
- Cloud services vs non-cloud
    - What are the pros and cons of using cloud applications

## Network

- TCP vs UDP
- Experience of setting http service
- Stateless characteristic of HTTP
- Network class/mask
- Websocket
- [J] How DNS work

### Database

- RDB ACID, vs NoSQL BASE
    - Struct vs Unstruct
    - Transaction
    - Scenario
- Indexing, Table Lock
- [J] Normalization, Unnormalizaition
- Any way to improve performance of your sql

### CS

- OOP
    - 3 characteristic
    - SOLID / KISS / YAGNI / DRY
    - 

### Software Development Teamwork

- Git , concept or usage
    - rebase, patch
    - working directory, stage, repository
- Interpersonal skills that help you communicate and collaborate across teams and roles.
- Github Flow / Git Flow
- Code Review

### Quiz

- Design an APIs for quest start&finish, with following features:
    - Prevented duplicate sessions.
    - Encrypted Content
    - Protected to quick calling
- Deal with bot user, how to prevent repeated API access
- Cross shard do data transfer, ex: transfer points
- Ranking system for user data under multiple databases
- Design a API service system architecture and describe how it considered:
    - Performance
    - Scalibility
    - Reilability
- Server symptom vs cause, consider following symptoms, what's the possible causes
    - Service got too many 500, 40x errors?
    - ELB latency got increased
    - computing instance cpu rate got highly increased
    - Db cpu rate got highly increased
        - Cache down
    - DB connection increases or decrease
    

## Personality and Culture

- Blameless: 是否曾經有同伴犯錯，如何面對，怎麼收拾
- Think out of box: 跳脫框架或全局思維

### Implementation

- Cross shard point transfer
- Large User Scale overall ranking

### Code Review
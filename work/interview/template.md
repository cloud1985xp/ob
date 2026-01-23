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

# Template

Created: 2019年7月26日 下午3:35

### What do you know about server engineer at Akatsuki

- Why want this job
- What expect in this job
- What's the challenges in this job
- How do you imagine develop game with ruby/rails

### Ruby , Rails

- How do you imagine develop game with ruby/rails
- Lambda vs Proc vs block
- String vs Symbol
- How do you explain class and module, when/how you decide to use them
- How do you explain the ORM / MVC
- How do you explain Duck-Typing
- include, extend and prepend
- Ruby Object Model, Inherit Chain, Singleton Class
- Meta Programming, open class
- ActiveRecord
    - include vs join
    - lazzy load
    - magic finder
- Experience about of Working with legacy codes

```go
Share the experience about upgrading ruby / rails & gems
```

- Experience about Server operation / deployment?
- Experience about API development
- Experience about using Redis
    - For what? Any usage of specific data struct of it?
    - What If the built-in data struct didn't meet the requirements
- How do you explain RESTFul
- RSpec
- Any pattern used in you rails application development

## Personality and Culture

- Blameless: 是否曾經有同伴犯錯，如何面對，怎麼收拾
- Think out of box: 跳脫框架或全局思維

### Network

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

### SRE / Infra. / DevOps

- 2 layers vs 3 layers
- CI & CD
- Experience of
    - Automation
    - Loading Test
    - Docker & Container, how explain it
    - Selenium
    - Chef, Anaisble or Puppet, configuration management / provision deployment tool
    - IaC (Infrastructure as Code)
    - Log System Building / Management
- Share and Explain Sidecar Pattern
    - And sidecarless
- Monitoring, tool? explain about Continuous Monitoring
    - How to build and improve observibility
- Server operation experience, AWS / GCP?
    - Used what services?
- Please explain SLI, SLO and SLA (Server Level Agreement)
    - How will you measure the reliability of the service
- Configuration Management
    - 
- Cloud services vs non-cloud
    - What are the pros and cons of using cloud applications
- Other domain interested?

### Quiz

- Design an APIs for quest start&finish, with following features:
    - Prevented duplicate sessions.
    - Encrypted Content - 防側錄
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
    

### Implementation

- Cross shard point transfer

### Code Review

## Backend Only

https://github.com/arialdomartini/Back-End-Developer-Interview-Questions

- What is CAP Theorem
- What is RESTful
- What are NoSQL databases, compare to RDB?
    - ACID
    - BASE
- What is SQL injection, how to prevent it
- What is CI (and CD,CD?)
- Functional Test vs Acceptance Test
- What it HTTP Idempotent
- What is Containerization
- What DB migration
    - Split Sharding
- What is your approach to debugging
- Test Coverage Criteria
    - Function coverage
    - Statement coverage
    - Branch coverage
    - Condition coverage
- Tell me about the largest web application you have ever worked on? What coding were you responsible for?
- What's your preferred type of development environment?
- Which programming languages do you want to work with and why?
- How you choose your next programming language? candidates?
- What is **Scalability** (and HA) how do you build scalability into a software programming
- Microservices
    - multiple smaller independently deployable services integrated into single application
    - versatility of the operation since individual services mby be implemented by difference language
    - leverage the strengths of each language to each service

# **Soft Skills Questions**

Your soft skills are your personality traits. All questions for this segment are designed to assess your ability to work with others and grow within the organization. Check out some sample soft skills questions mentioned below:

1. What do you think contributes to a successful project?
2. How do you keep up with the latest technologies and trends?
3. Tell me about a time when you had to work with someone difficult? How did you handle it?
4. What is the most difficult change you’ve encountered in your career?
5. How do you handle situations where there is a lot of tension between you and a colleague?

## Senior DevOps

current 2+3 people

2 main projects

dev and ops

- Merge codes
- Tooling development

CI/CD workflow

IaC / infra management, version update

performance tuning / loading

release and deployment

operation issue

monitoring

on call

### SRE

- Solution, plan
- Culture building

CI/CD Exp , DevOps Tools

Data Engineering

維運事件的經驗

Security

### **Why do you think that you will become a Site Reliability Engineer?**

I have a practical understanding and working knowledge in DevOps with a deep understanding of:

- The inter-relationship of SRE with DevOps and other popular frameworks
- The underlying principles behind SRE
- Service Level Objectives (SLO’s) and their user focus
- Service Level Indicators (SLI’s) and the modern monitoring landscape
- Error budgets and the associated error budget policies
- Toil and its effect on an organization’s productivity
- Some practical steps that can help to eliminate toil
- Observability as something to indicate the health of a service
- SRE tools, automation techniques, and the importance of security
- Anti-fragility, our approach to failure and failure testing
- The organizational impact that introducing SRE brings

### **9. Define Service Level Indicators**

A Service Level Indicator (SLI) is a measure of the service level provided by a service provider to a customer. SLIs form the basis of Service Level Objectives (SLOs), which in turn form the basis of Service Level Agreements (SLAs). An SLI can also be called an SLA metric.

Although every system is different in the services provided, common SLIs are used pretty often. Common SLIs include latency, throughput, availability, and error rate; others include durability (in storage systems), end-to-end latency (for complex data processing systems, especially pipelines), and correctness.

What’s SLx, 舉幾個 SLI

### **17. What is observability, how to improve organizations’ systems observability?**

Observability is basically a conversation around the measurement and instrument of an organization.

**To improve an organization’s observability, you need to:**

- Understand what types of data flow from an environment, and which of those data types are relevant and useful to your observability goals
- Get a clear vision of what a team cares about and figure out how your strategy is making sense of data by distilling, curating, transforming it into actionable insights into the performance of your systems.
- Observability offer potentially useful clues about an organization’s DevOps maturity level.

[DevOps Interview Questions](https://www.interviewbit.com/devops-interview-questions/)

### **5. What is the importance of having configuration management in DevOps?**

Configuration management (CM) helps the team in the automation of time-consuming and tedious tasks thereby enhancing the organization’s performance and agility.

It also helps in bringing consistency and improving the product development process by employing means of design streamlining, extensive documentation, control, and change implementation during various phases/releases of the project.

### **6. What does CAMS stand for in DevOps?**

CAMS stands for Culture, Automation, Measurement, and Sharing. It represents the core deeds of DevOps.

### **7. What is Continuous Integration (CI)?**

Continuous Integration (CI) is a software development practice that makes sure developers integrate their code into a shared repository as and when they are done working on the feature. Each integration is verified by means of an automated build process that allows teams to detect problems in their code at a very early stage rather than finding them after the deployment.

![Untitled](Template%20-%20DevOps/Untitled.png)

Based on the above flow, we can have a brief overview of the CI process.

- Developers regularly check out code into their local workspaces and work on the features assigned to them.
- Once they are done working on it, the code is committed and pushed to the remote shared repository which is handled by making use of effective version control tools like git.
- The CI server keeps track of the changes done to the shared repository and it pulls the changes as soon as it detects them.
- The CI server then triggers the build of the code and runs unit and integration test cases if set up.
- The team is informed of the build results. In case of the build failure, the team has to work on fixing the issue as early as possible, and then the process repeats.

### **10. What are the three important DevOps KPIs?**

Few KPIs of DevOps are given below:

- Reduce the average time taken to recover from a failure.
- Increase Deployment frequency in which the deployment occurs.
- Reduced Percentage of failed deployments.

### **21. Can you say something about the DevOps pipeline?**

A pipeline, in general, is a set of automated tasks/processes defined and followed by the software engineering team. DevOps pipeline is a pipeline which allows the DevOps engineers and the [software developers](https://www.scaler.com/courses/full-stack-developer/) to efficiently and reliably compile, build and deploy the software code to the production environments in a hassle free manner.

Following image shows an example of an effective DevOps pipeline for deployment.

![](https://s3.ap-south-1.amazonaws.com/myinterviewtrainer-domestic/public_assets/assets/000/000/086/original/DevOps_pipeline.jpg?1614935146)

The flow is as follows:

- Developer works on completing a functionality.
- Developer deploys his code to the test environment.
- Testers work on validating the feature. Business team can intervene and provide feedback too.
- Developers work on the test and business feedback in continuous collaboration manner.
- The code is then released to the production and validated again.

how you build kubernetes cluster, self-host?

- logging
- uses any monitoring tool? (not seen in resume
- how to do configuration management

DevOps culture

Experience about Jenkins?

- explain it's architecture? : master-slave

IaC

- programmable infrastructure
- imperative vs declarative

Deployment?

- Blue/Green and Canary Deployment
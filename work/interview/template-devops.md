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

# Template - DevOps

Created: 2023年6月9日 上午9:05

主要工作：

每間公司可能涵蓋的職責範圍不同，先對焦

- 你所認為的 vs 我們需要的

**Application development**

- Code building
- Code coverage
- Unit testing
- Packaging
- Deployment

**Infrastructure**

- Provisioning
- Configuration
- Orchestration
- Deployment

## Tech

- System administration / operation
- Build and deployment, containerization, cloud computing, monitoring, logging tools.
- Scripting / Automation

Soft Skills

- Workflow communication
- Agile, effective communicator

### 分享你在導入 DevOps 的原則或覺得須具備的特性, ex:

- Cattle, not pets
- Release early, release often…

Or Important DevOps KPIS you think to evaluate effective of DevOps

ex:

The three important KPIs are as follows:

- Meantime to failure recovery: This is the average time taken to recover from a failure.
- Deployment frequency: The frequency in which the deployment occurs.
- Percentage of failed deployments: The number of times the deployment fails.

And (Same)

- Reduce the average time taken to recover from a failure.
- Increase Deployment frequency in which the deployment occurs.
- Reduced Percentage of failed deployments.

夠不夠 geak ？

- 關注的語言
- 最近接觸的 topic、技術
- 接收新知的頻率、管道
- 維運事件的經驗
    - 如何定義成功、成就感的來源

Share some special experience of AWS/GCP

- ex: Traffic mirror?

## Shares Experience Of

Experience of 

- Automation
- CI & CD Experience, What’s Your Pipeline
- IaC Experience
    - Introduce terraform?
- Provision Tool: Chef, Ansible or Puppet, configuration management / provision deployment tool
    - Like How you explain / introduce it
- Docker & Container, how explain it
- Log System Building / Management
- Mixed Platform
- Loading Test
- Selenium
- 

Tell me about the largest web application you have ever worked on? What coding were you responsible for?

Development Skill? 

Learning New Programming Language

Data Engineering

Security

## Quiz

### HTTP Request Lifetime

### Deployment Strategies

- Big Bang
- Rolling
- Blue Green
- Canary
- Versioned
- Feature Toggle

### Describe the branching strategies you have used.

- Release branching - We can clone the develop branch to create a Release branch once it has enough functionality for a release. This branch kicks off the next release cycle, thus no new features can be contributed beyond this point. The things that can be contributed are documentation generation, bug fixing, and other release-related tasks. The release is merged into master and given a version number once it is ready to ship. It should also be merged back into the development branch, which may have evolved since the initial release.
- Feature branching - This branching model maintains all modifications for a specific feature contained within a branch. The branch gets merged into master once the feature has been completely tested and approved by using tests that are automated.
- Task branching - In this branching model, every task is implemented in its respective branch. The task key is mentioned in the branch name. We need to simply look at the task key in the branch name to discover which code implements which task.

### Design a API service system architecture and describe how it considered:

- Performance
- Scalibility
- Reilability

### Please explain SLI, SLO and SLA (Server Level Agreement)

- How will you measure the reliability of the service

### Server symptom vs cause, consider following symptoms, what's the possible causes

- Service got too many 500, 40x errors?
- ELB latency got increased
- computing instance cpu rate got highly increased
- Db cpu rate got highly increased
    - Cache down
- DB connection increases or decrease

### Docker

- Docker command
    - —rm -it
    - EXPOSE vs publish

### Network

- TCP vs UDP
- Experience of setting http service
- Stateless characteristic of HTTP
- Network class/mask
- Websocket
- [J] How DNS work

## Others

- 2 layers vs 3 layers
- Share and Explain Sidecar Pattern
    - And sidecarless
- Monitoring, tool? explain about Continuous Monitoring
    - How to build and improve observibility
- Server operation experience, AWS / GCP?
    - Used what services?
- 
- Configuration Management
- Cloud services vs non-cloud
    - What are the pros and cons of using cloud applications
- Other domain interested?

從教訓面

- 表達能力，能否言簡意賅的表達問題，描述目標、溝通，言之有物
- 紀律性
- 誠實
- 互動、主動、反應

是否對這位職位展現出熱情

是否確保過去專案中，個人貢獻與他人的區分開來

從需求面，他會要

- 負責處理 infra 更新、調整
- 管理 Jenkins
    - CI/CD Pipeline
- Tooling
- 負載測試
- 建置新 infra
- 

## Personality and Culture

- Blameless: 是否曾經有同伴犯錯，如何面對，怎麼收拾
- Think out of box: 跳脫框架或全局思維

What's your preferred type of development environment?

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
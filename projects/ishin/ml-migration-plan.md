---
tags:
  - ishin
  - project
created: 2024-01-01
updated: 2025-01-23
status: active
---


- Jenkins login credentials, on 1password?
- Jenkins slave login credentials, on 1password
	- how this service been provisioned? any script to setup it?
- Which jenkins job has highest priorty to migrate, in active & using by ISHIN

Quesiton
- Why so many jobs seems be using by ISHIN but use muse/neon-workflow?
	- should we checkout to muse/ishin-workflow or has independent repo of ishin-neon on Github Enterprise.

Step 1
- Move local-4090 to ishin-slave
- Migrate jenkins jobs: the Jenkinsfile to ishin-jenkins
	- Rewrite job
	- 

---
tags:
  - polunga
  - project
  - roadmap
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Rewrite Query Caching Logic

Created: 2019年7月19日 上午10:06
Type: Task 🔨
Status: Not Started
Epic: 無標題 (https://www.notion.so/551a7f7e7e94405c98da0518a4e24ad0?pvs=21)
Sprint: Sprint 22, Sprint 23, Sprint 24
Priority: P1 🔥
Product Manager: Ivan Zhao
Engineers: Brian Park

<aside>
💡 Create a new item in your database and choose `Task` from the list of template options to generate the format below. Learn more about database templates [here](https://www.notion.so/454ed5ab5bd24226b58d176697bd7e10?pvs=21).

</aside>

# Overview

## Problem statement

Describe the problem you're trying to solve by doing this work.

Our Postgres queries *are* currently cached, but the performance is subpar and is causing slow load times on the front end.

## Proposed work

High-level overview of what we're building and why we think it will solve the problem.

Research and implement a new caching strategy.

# Success criteria

The criteria that must be met in order to consider this project a success. 

- Load times reduced at least 20%, ideally more
- CPU usage decreased

# User stories

How the product should work for various user types.

## As a user

- I want my search queries to return results quickly
- So I can get my work done faster

## As a front end engineer

- I expect my queries to be cached on the back end
- So the front end can be as fast as possible

# Scope

## Requirements

Current project requirements.

- Plan and implement new database caching strategy
- Create a testing and deployment plan
- Improve monitoring tools

## Future work

Future requirements.

- Monitor caching performance

## Non-requirements

List anything that is out of scope.

- Front end caching
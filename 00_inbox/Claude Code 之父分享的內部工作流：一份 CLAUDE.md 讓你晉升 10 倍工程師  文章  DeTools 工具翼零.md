---
title: "Claude Code 之父分享的內部工作流：一份 CLAUDE.md 讓你晉升 10 倍工程師 | 文章 | DeTools 工具翼零"
source: "https://tools.wingzero.tw/article/sn/3788"
author:
  - "[[DeTools 工具翼零]]"
published: 2026-02-24
created: 2026-04-04
description: "總是在糾正 AI 寫出的 Bug？快套用 Claude Code 創作者親授的 CLAUDE.md 終極設定檔！內含工作流編排、子代理策略與自我完善迴圈等 6 大核心引擎，強制 AI 驗證程式碼並記住專案規範。立即複製這份「10 倍工程師」的最佳實踐，打造越用越聰明的專屬開發助手！"
tags:
  - "clippings"
---
\## Workflow Orchestration  
  
\### 1. Plan Node Default  
\- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)  
\- If something goes sideways, STOP and re-plan immediately - don't keep pushing  
\- Use plan mode for verification steps, not just building  
\- Write detailed specs upfront to reduce ambiguity  
  
\### 2. Subagent Strategy  
\- Use subagents liberally to keep main context window clean  
\- Offload research, exploration, and parallel analysis to subagents  
\- For complex problems, throw more compute at it via subagents  
\- One tack per subagent for focused execution  
  
\### 3. Self-Improvement Loop  
\- After ANY correction from the user: update \`tasks/lessons.md\` with the pattern  
\- Write rules for yourself that prevent the same mistake  
\- Ruthlessly iterate on these lessons until mistake rate drops  
\- Review lessons at session start for relevant project  
  
\### 4. Verification Before Done  
\- Never mark a task complete without proving it works  
\- Diff behavior between main and your changes when relevant  
\- Ask yourself: "Would a staff engineer approve this?"  
\- Run tests, check logs, demonstrate correctness  
  
\### 5. Demand Elegance (Balanced)  
\- For non-trivial changes: pause and ask "is there a more elegant way?"  
\- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"  
\- Skip this for simple, obvious fixes - don't over-engineer  
\- Challenge your own work before presenting it  
  
\### 6. Autonomous Bug Fixing  
\- When given a bug report: just fix it. Don't ask for hand-holding  
\- Point at logs, errors, failing tests - then resolve them  
\- Zero context switching required from the user  
\- Go fix failing CI tests without being told how  
  
\## Task Management  
  
1\. \*\*Plan First\*\*: Write plan to \`tasks/todo.md\` with checkable items  
2\. \*\*Verify Plan\*\*: Check in before starting implementation  
3\. \*\*Track Progress\*\*: Mark items complete as you go  
4\. \*\*Explain Changes\*\*: High-level summary at each step  
5\. \*\*Document Results\*\*: Add review section to \`tasks/todo.md\`  
6\. \*\*Capture Lessons\*\*: Update \`tasks/lessons.md\` after corrections  
  
\## Core Principles  
  
\- \*\*Simplicity First\*\*: Make every change as simple as possible. Impact minimal code.  
\- \*\*No Laziness\*\*: Find root causes. No temporary fixes. Senior developer standards.  
\- \*\*Minimat Impact\*\*: Changes should only touch what's necessary. Avoid introducing bugs.
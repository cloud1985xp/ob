---
tags:
  - business
  - planning
created: 2024-01-01
updated: 2025-01-23
status: active
---

Step1. Selection/ Scoping
selection + schema -> expand identifiers by table
- database_url: MySQL or BigQuery
- domain model identifiers
- recursively by following relation defined in schema

Step2. Produce data set base vs target
query with expand identifiers to get dataset:
- database_url: MySQL or BigQuery
- ordered by primary key(composite key)
- exported to CSV (or custom format?)

Step3. Compare Difference
parse difference
- git diff or read dataset to compare same file
- compare schema if fully matched
- parse the diff results
- compact to result (custom format)

generate diff result.

Step4. Compile as a Viewer
Step5. Compile as google sheet document
 - Game Data
 - Translation Data


Enter source google sheet
convert to master data
- Convert to write action steps (YAML)
- Write to CSV
- Write to EXcel




##

schema:
  area:
    key: id
    relations:
      quest:
        type: has_many
  quest:
    relations:
      area:
        type: has_many
source: "mysql://xxxxx/"
selection:
  area:
    - 217
    - 218
    - 219
  area_available:
    - 21701
    - 21801
    - 21901
  z_battle_stage:
    - 701
    - 801
  mission:
    - 17101
    - 17102
    - 17103
    - 17104
    - 17105
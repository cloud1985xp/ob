---
tags:
  - bigquery
  - database
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# BigQuery

Created: 2022年6月16日 上午11:39

## Lag/Lead Over Partition By

```sql
SELECT
TIMESTAMP_DIFF(
  timestamp,
  LAG(timestamp) OVER(PARTITION BY user_id ORDER BY timestamp),
  MILLISECOND
) AS diff
FROM sign_in_logs
```
---
tags:
  - learning
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# HYPE Note

- learned unique_index with nulls_distinct: false
    - Need pg15+

learned #hd

learned #get_in

learned :on_conflict and returning: true

Unresolve Question

- Ecto many to many 無法透過 association 名稱，一定要直接寫明 join table name?
- Ecto change belongs_to 要把 association 清除，無法透過只改 foreign_key value 的方式，一定要用 put_assoc(nil) ？
    - [https://github.com/elixir-ecto/ecto/issues/3007](https://github.com/elixir-ecto/ecto/issues/3007)
    - [https://elixirforum.com/t/ecto-changeset-ignore-nil-fields-in-update/40654](https://elixirforum.com/t/ecto-changeset-ignore-nil-fields-in-update/40654)
    - [https://elixirforum.com/t/how-to-update-a-foreign-key-set-to-nil/54170/5](https://elixirforum.com/t/how-to-update-a-foreign-key-set-to-nil/54170/5)

learned LiveView

learning libcluster, multipass
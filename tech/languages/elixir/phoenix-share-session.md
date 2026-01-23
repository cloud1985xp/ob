---
tags:
  - elixir
  - language
  - phoenix
created: 2025-01-01
updated: 2025-01-23
status: active
---

## Allow Share Session between Different Apps

session
- session_key
- signing_salt
live_view
- singing_salt

## Share Session Token in Cache
## Cachex
https://blog.appsignal.com/2024/03/05/powerful-caching-in-elixir-with-cachex.html


要啟用 distributed ，需要在啟動時 options 設定 router
https://hexdocs.pm/cachex/Cachex.html#start_link/2-options
https://hexdocs.pm/cachex/distributed-caches.html

Router 應會自動用 node() + Node.list() 來取得所有 node list

https://hexdocs.pm/cachex/Cachex.Router.Ring.html#init/2-options
並且要將 monitor 設成 true

Nebulex
https://medium.com/erlang-battleground/distributed-caching-in-elixir-using-nebulex-9af589186caa

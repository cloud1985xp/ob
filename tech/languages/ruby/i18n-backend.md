---
tags:
  - ruby
  - rails
  - language
created: 2024-01-01
updated: 2025-01-23
status: active
---


可能帶來的問題
- Memory Usage > JP
	- 在資源的運用計算上必須額外考量，不能只參照日本
- Unicorn kill 理論上會比較頻繁


I18n::Backend::Simple
I18n::Backend::Cache

現行的 Backend
- 一次會把所有 locales file 載入 @translations

新的作法
- 縮短到載入時間約 3 分鐘
- 支援 express load

副作用
當 Cache missed 時，會重新 open file 讀取 yml 資料

Local:
- Memcached  limit_maxbytes

Change memcached config by:
> ~/Library/LaunchAgents/homebrew.mxcl.memcached.plist

Memcached command:
```
echo 'flush_all' | nc localhost 11211
echo "stats" | nc -w 1 localhost 11211 | awk '$2 == "bytes" { print $2" "$3 }'
echo "stats" | nc -w 1 localhost 11211 | awk '$2 == "limit_maxbytes" { print $2" "$3 }'
```


# Reference
- Ruby object space to evaluate memory usage
	- https://stackoverflow.com/questions/10068018/memory-size-of-a-hash-or-other-object
	- https://ruby-doc.org/stdlib-2.6.4/libdoc/objspace/rdoc/ObjectSpace.html#method-c-memsize_of
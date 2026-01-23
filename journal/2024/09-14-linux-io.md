---
tags:
  - journal
  - 2024
created: 2024-01-01
updated: 2025-01-23
status: active
---


Unix: Every is file
Linux: Everything is file descriptor

read command
read fd, buffer


user space : read
  get 
-> kernel space: 依給定的 fd
   wait for data
     -> copy data from kernel to user
 kernel 與 user space 是 isolated 的，從記憶體上是分開的，不讓 user 影響 kernel
 也因為這樣的隔離，必須讀取到的資料複製到 user space
第2個參數是由 user 端去配置緩衝區，因為需要複製資料到 user space
過程中 user space 只能等待，blocking I/O
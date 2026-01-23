---
tags:
  - ishin
  - project
created: 2024-01-01
updated: 2025-01-23
status: active
---



計算 RSS
```
cd /proc/{pid}
sudo cat smaps | grep Rss | awk '{ s+=$2 } END {printf "%.0f", s}'
```

同理可算 PSS/USS
公式：
```
USS = sum of /proc/<pid>/smaps Private_clean + Private_dirty
PSS = sum of /proc/<pid>/smaps Pss
RSS = sum of /proc/<pid>/smaps Rss
```

https://stackoverflow.com/questions/30890684/how-does-smem-calculate-rss-uss-and-pss
https://changkun.de/blog/posts/pss-uss-rss/



或直接裝 smem 來計算
https://www.cnblogs.com/yychuyu/p/17146289.html

apt install smem


- 記憶體是夠用的
- 現在的 unicorn memory boundary，無論 30/36 worker，被 killed 的頻率都很低
- 關鍵反而是在 cpu cores?
	- 可能要 check 一下日本之前的狀況
	- 日版卻是改用 m series 對策？


## Unicorn-Killer 計算 memory 的方式

是用 get_process_mem 這個 gem

https://github.com/zombocom/get_process_mem/blob/main/lib/get_process_mem.rb

方法其實就是去計算 proc/{pid}/smaps 裡的數值


REF
有人問一樣的問題：
https://stackoverflow.com/questions/52034729/verify-that-object-is-shared-in-memory-between-ruby-unicorn-processes



# References

https://unix.stackexchange.com/questions/323693/how-to-know-shared-memory-between-two-processes/323736#323736
https://unix.stackexchange.com/questions/305606/linux-inactive-memory/305752#305752

smem
https://linux.die.net/man/8/smem
https://www.cnblogs.com/yychuyu/p/17146289.html
https://segmentfault.com/a/1190000040077427
https://stackoverflow.com/questions/30890684/how-does-smem-calculate-rss-uss-and-pss
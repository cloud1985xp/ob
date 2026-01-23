---
tags:
  - ruby
  - rails
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# RubyConf 2019

Created: 2019年7月27日 上午10:04

Pipeline operator

Compact ⇒ fragmentation

Unicorn ⇒ CoW (Copy on Write)

Child process copy memory status of parent process (fixed width)

Ruby Heaps

Malloc Heap , Ruby's object heap

GC.compact

malloc → pages → slots

Compaction Algorithm

1. move object, 2. update reference (very complicated..) ⇒ use C extensions?

3. GC empty slots

Known Typed Object can  GC by ruby

but unknown Typed? ex Yajl (implement by C

mark as rb_gc_mark gc_mark_no_pin

use pinning bits make it cannot move

C extention compact it self , compaction callback 

when ruby obj & c obj point to same object.

C object marked pointed object?

debugging GC

maximum chaos

2 space collector / zombie objects address / sanitizer

Updated object ID in hash table

~10% smaller heap

CocoaPods

Make mobile developer more productivity

bundler + rubygems for iOS

Profiling

- method calls
- call stack sample
- memory allocate
- disk io
- network io
- databse

rubyprof

 Tracing Profilers

rbspy

 sample profiler

chronometer
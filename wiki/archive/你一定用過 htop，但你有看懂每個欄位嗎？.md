---
title: "你一定用過 htop，但你有看懂每個欄位嗎？"
source: "https://medium.com/starbugs/do-you-understand-htop-ffb72b3d5629"
author:
  - "[[Larry Lu]]"
published: 2021-05-16
created: 2026-04-07
description: "你一定用過 htop，但你有看懂每個欄位嗎？ 身為一個工程師，不管你寫的是前端、後端、全端還是什麼端，一定多少用過 …"
tags:
  - "clippings"
---
[Sitemap](https://medium.com/sitemap/sitemap.xml)

Get unlimited access to the best of Medium for less than $1/week.[Become a member](https://medium.com/plans?source=upgrade_membership---post_top_nav_upsell-----------------------------------------)

[

Become a member

](https://medium.com/plans?source=upgrade_membership---post_top_nav_upsell-----------------------------------------)

## [Starbugs Weekly 星巴哥技術專欄](https://medium.com/starbugs?source=post_page---publication_nav-feef767a6d9-ffb72b3d5629---------------------------------------)

[![Starbugs Weekly 星巴哥技術專欄](https://miro.medium.com/v2/resize:fill:76:76/1*Kma_T7CmrGKYhgVzjiLiXA.jpeg)](https://medium.com/starbugs?source=post_page---post_publication_sidebar-feef767a6d9-ffb72b3d5629---------------------------------------)

一群技術人想要寫出一些好文章所建立的技術專欄。每週二一篇原創文章、一封電子報，歡迎大家訂閱！主網站： [https://weekly.starbugs.dev/。](https://weekly.starbugs.dev/%E3%80%82)

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*DTNt1QCzCpy0YgIgILlyLg.png)

身為一個工程師，不管你寫的是前端、後端、全端還是什麼端，一定多少用過 htop，就算真的沒用過也會聽同事說過。htop 是一個 process manager，他可以讓你看到執行中的 process、系統資源的使用量，也可以讓你輕鬆 kill 掉任何一個 process，總之，你想得到的功能統統都有～

雖然大家都說 htop 很好用，但許多人打開 htop 也只看得懂 CPU、Mem、PID、Command 這些簡單的欄位，對於 Load average、NI、State、SHR 就沒那麼熟悉

不過 htop 都幫你列出來了，看不懂好像又有點可惜對吧！所以這篇文章要跟大家鉅細彌遺的介紹 htop 每個欄位的意義，也許哪天有需要時就能派上用場呢！

## CPU

首先來說說最重要的 CPU，在 htop 最上方會列出各 CPU 的使用率，值得注意的是這邊顯示的是 **CPU 的邏輯核心數** ，譬如說你的電腦有四核心 **八執行緒** ，意思是同時可以執行八個 thread，那這邊就會顯示八個 CPU 哦～

![](https://miro.medium.com/v2/resize:fit:1362/format:webp/1*Y9NM4kTtBEppLelmeQ4Azg.png)

另外不知道有沒有人發現這個使用率的 bar 包含了 **紅色** 跟 **綠色** ，有時甚至還會有 **藍色** ，那其實是有意義的哦：

- **紅色** 代表的是 **kernel thread** 佔用的 CPU，像是系統需要自動做 process scheduling、memory management 等等，是整個系統中最重要、優先權也最高的任務
- **綠色** 代表的是 **normal priority thread** ，排程的優先權比 kernel thread 低一些，一般來說使用者執行的程式如果沒有特別調優先權的話，都會歸在這一類
- **藍色** 的話就是 **low priority thread** ，因為優先權比較低，分配到的 CPU 自然也比較少，適合「我 ok，你先跑」那類比較無關緊要的 process，如果 CPU 已經被操到快不行了，或是 memory 真的不夠用了，第一個殺掉的也是這類 process

## Memory & Swap

![](https://miro.medium.com/v2/resize:fit:1346/format:webp/1*SdWoFDelfkaJvmdmOjzqTQ.png)

緊接在 CPU 下面的是 memory 跟 swap 的使用量，memory 這邊應該大家都看得懂，值得一提的是他的顏色也是有意義的：

- **綠色** 指的是被 process 佔用的記憶體，譬如說你開的瀏覽器、VSCode、終端機等等程式，還有正在執行的 htop 都算是這一類
- **藍色** 則是 **buffer pages** ，他是用來儲存一些 metadata。譬如說當你第一次執行 `ls -l` 時系統會去硬碟撈看 **這個資料夾有哪些檔案、每個檔案的權限** 等等，然後幫你存在 buffer pages，當你短時間內再執行 `ls -l` 時就不用再進入硬碟（因為硬碟很慢），直接從 buffer 拿即可
- **橘色** 的 **cache pages** 跟 buffer 很像，只不過 **buffer 存的是 metadata，而 cache 存的是檔案內容** 。像你第一次下 `cat index.js` 時就會把內容讀取到 cache pages，如果你 cat 之後發現程式碼太長，決定先看前十行就好了，那再下 `head -n 10 index.js` 就會從 cache pages 直接讀取

這也代表說 **記憶體使用量並非越低越好** ，畢竟閒在那邊也沒啥用，不如讓系統把閒置的部分拿去當 buffer 跟 cache，讀取時能不碰硬碟就不碰硬碟，才可以讓程式執行得更快

> 所以千萬不要相信什麼「記憶體清理大師」可以幫你提升效能，說真的沒變慢就不錯了XD，隨便把 buffer 跟 cache 都清掉了只會加重系統負擔。記憶體管理就交給系統來，十之八九都可以管理得還不錯

![](https://miro.medium.com/v2/resize:fit:1346/format:webp/1*SdWoFDelfkaJvmdmOjzqTQ.png)

而 swap 的部分雖然上圖完全沒用到，但還是解釋一下： **swap 的機制跟剛剛提到的 cache & buffer 正好相反** ，萬一你實在開了太多程式，而且每個程式都跟 chrome 一樣狂吃猛吃， **導致 memory 快要不夠了**

**那系統就會把記憶體裡面一些東西 swap 到硬碟上** ，等真的需要那些東西時再從硬碟拿回來。雖然這樣做看似有更多記憶體可以用，但代價就是程式速度會慢上許多，因為硬碟實在是太慢了

> 順帶一提 swap 的發音是「司哇ㄆㄜ」，不要再唸成「司位ㄆㄜ」了～

## Load Average

接著來看看螢幕右上方那堆神奇的數字

首先 **Tasks** 欄位的 `488, 1994 thr; 3 running` 代表的是目前總共有 488 的 process、1994 個 thread，其中 3 個 thread 正在執行（這數字最大就是你的邏輯核心數）

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*WN1L1_kbk8jK2fuwgYwlXw.png)

而 **Load Average(LA)** 是用來判斷斷目前系統有多繁忙，三個數字代表的是系統在 **最近 1分鐘、5分鐘、15 分鐘內，平均有多少個 thread 需要 CPU**

以上圖來說， **近一分鐘內平均有 5.9 個 thread 需要使用 CPU 進行運算** ，但無奈我只有 4 個邏輯核心，所以 CPU 是忙到翻過來（因為我正在編譯 Rust 執行檔XD），而最近十五分鐘的 LA 是 3.49，其實也算是高了

一般來說電腦完全沒在用時 LA 會低於 1，而平常在上網、聽音樂、做簡報則是會介於 1 到 2 之間

所以如果覺得自己寫的程式跑很慢，不妨先看看 LA 確認瓶頸是不是在 CPU，如果 LA 很低但程式卻慢得誇張，那很可能 **程式並沒有善用多核心，或是瓶頸卡在硬碟跟網路 IO** ；如果 LA 已經很高了但還是覺得太慢，那就只能改善演算法、或是換更快的 CPU 了

## PID/USER

上面講完之後來看看下面這一大塊，這部分 **每個 row 都是一個 process** ，而 PID 就是每個 process 的 ID

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*pwf3sNWA2shXjgwhK4Rysg.png)

那知道 PID 之後可以幹嘛呢？其實他的用途挺多的，譬如說你可以用 `kill -KILL <pid>` 來殺掉某個 process；或是使用 `kill -STOP` 來暫停 process 再用 `kill -CONT` 讓他繼續執行（這超好用 👍，但好像很多人都不知道）

而 USER 欄位沒什麼好解釋的XD，就是 **把這個 process 跑起來的人。** 不管程式是誰寫的，只要是我把他跑起來，USER 那欄就會顯示我的名字

## PRI & NI

接下來的 Priority 跟 Nice 兩個都是跟優先權有關的指標，注意 **數字越小表示優先權越高，也就可以分配到越多 CPU 時間**

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Y1dau0jk7lDXC0hHI5gp_w.png)

其中 PRI 是由系統幫你決定的，無法自行修改，像上圖 mdbulkimport 的 PRI 值是 17，而 `ping 8.8.8.8` 則是 24，代表系統認為 mdbulkimport 比 ping 來得重要

> 註：mdbulkimport 是 mac spotlight 功能的一部分

而 nice 值的部分預設是 0，可以用 `renice -n 19 -p <pid>` 調整到最低優先權 19，想要調高的話最高也可以調到 -20雖然 nice 值可以隨自己高興調高調低，但 **系統不見得都會聽你的** 。有的系統比較友善會願意參考你設的 nice 值，但也有一些只看 PRI 系統根本不在乎 nice，你設你的我排我的XD，所以不要太期待提高優先權可以為效能帶來多大的變化，想要提升效能還是乖乖把程式寫好比較重要～

## VIRT/RES/SHR

這三個數字都是跟記憶體有關的，分別代表 Virtual memory、Resident 跟 Shared memory

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*gjN0MDsr5kg9hgpQMtVzTQ.png)

Virtual memory 的概念比較複雜一點，基本上你可以把他想成 **process 可以存取到的 memory 總和** 。譬如說 `head -n index.js` 內部運作的方式是先把 index.js 打開，然後讀取前十行

雖然他只讀取前十行， **但 head process 已經把檔案打開了，他其實有權限 access 到整個檔案的內容（只是它沒有這麼做），所以 virtual memory 會把整個檔案的大小算進去**

而 Resident 正好相反，他指的是 **物理上你到底佔用了多少記憶體** 。以同樣的例子來說，若你只讀取前十行， **那系統就只把前十行從硬碟讀進記憶體，RES 也就只算那十行**

因此在 **htop 裡面 RES 一定會小於 VIRT（如下圖），而且通常是遠小於** ，因為 VIRT 會把一堆哩哩叩叩的東西都算進去，所以就算看到 VIRT 很肥也完全不用擔心

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*gjN0MDsr5kg9hgpQMtVzTQ.png)

而 Shared memory 的話 **顧名思義就是可以跟別人分享的 memory** ，像程式執行時很常會用的 ==glibc== ，或是在讀取 read-only 檔案時， **這些東西都只需要讀進記憶體一次就可以了，所以就會被算進 SHR 裡面**

雖說能跟其他 process 共用記憶體是好事，但這種事也強求不來，所以一般我都只看 RES，很少在管 SHR 是多少

## State

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*i6NnubhankPL-nee4F_iTQ.png)

接著來看這小小一個 S，代表的是 process 的 state，比較常見的有以下幾個

- R: **意思是 process 正在跑或是在 running queue 裡等待 CPU 排程**

如果你的程式長時間處於 R 狀態但還是跑很慢，代表可能是演算法太慢了，或是 CPU 實在太忙一直把你丟在 queue 裡面，可以再透過 CPU% 來確認是哪個問題

- S: **目前正在睡覺，有事做才會醒來**

通常定時執行的、要使用者互動的程式如 ping 跟 VSCode 很常會處於 S 狀態（上圖）， **畢竟 ping 一秒只送一個封包，但現今的 CPU 一秒可以跑十億個 cycle** ，所以 CPU 當然是送出封包後就馬上把你踢到旁邊去睡覺，等一秒後你睡醒了再回來

VSCode 的話是因為 **你總不可能一秒打十億個字吧** ，所以沒事做的時候他會馬上被 CPU 踢出去，等你打字了他再被叫回來動一下，然後馬上又被踢出去（怎麼感覺有點慘XD）

- D: 這個也是在睡覺，只不過等待的一定是 IO，譬如說讀取檔案、寫入資料庫等等

如果你的 process 長時間處於 D 狀態，代表 IO 所佔的時間比較多，三不五時就被 CPU 踢出去睡覺。因此想改善效能也要從 IO 著手，譬如用 redis 做快取或是換 SSD 等等，若只是單純更換程式語言或是升級 CPU 可能沒什麼效果

## CPU%/MEM%

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*lcjZ888wF35qUuYn_72B-Q.png)

CPU% 意思是你在 **這段時間平均用了幾顆 CPU** ，因為 htop 預設 3 秒更新一次，假如前 1.5 秒你用了一顆，後 1.5 都沒用， **那平均就是 50%** ；如果你這三秒用好用滿四個核心，那就是漂亮的 400%（上圖 Rust 編譯器的其中一個 process 就有用到 331%）

因為 CPU% 是很短期的數據，所以當你突然覺得電腦當當的快不行時， **直接看 CPU% 就可以知道是誰在作怪** ，然後看是要用 signal 把他暫停還是乾脆直接殺掉

MEM% 也很類似，就是使用記憶體的比例，要注意的是 **他是用 RES 來做計算** ，所以如果電腦配有 4GB 的記憶體、某個 process 的 RES 是 1GB，那他就是用掉實體記憶體的 25%

## Time+

最後要來說說 Time+，這個時間很有意思， **他代表的並不是程式從啟動到現在總共經過了多久，而是這個程式總共佔用了多少 CPU Time**

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*OKEwTc1DML1DnsUVhOowBw.png)

譬如說上圖我在編譯 Rust 時，雖然我才剛跑十秒而已，但 CPU Time 已經有 28 秒， **代表說這個 process 在十秒內平均使用了 2.8 顆 CPU，也就是平均的 CPU% 是 280% 左右**

相反的如果像 `ping 8.8.8.8` 這種幾乎不需要 CPU 運算的 process（就只是把封包送出去而已），那即便跑了十多分鐘，佔用的 CPU Time 也不過 12 秒而已，平均起來的 CPU% 大概是 2% 左右

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*z_wCrLymdo89Oe-6x9A1LQ.png)

因此如果想知道 **長期而言** 哪個程式最佔 CPU 的話，就看 Time+ 的數值；如果是想看 **短期、目前正在暴衝** 的程式，那就是看先前提到的 CPU%

## 總結

今天介紹了怎麼用 htop 看 **系統的負載狀態** 、 **各種記憶體使用量** 、以及 **長短期的 CPU 使用率** ，看到這我想大家都累了，一下子資訊量太大說不定也忘得差不多了XD

但也沒關係，看完腦袋中有個概念就好了，先把文章收藏起來，哪天發現又看不太懂 htop 時，再回來當技術文件翻一下也可以哦～

### 延伸閱讀

- [Linux 的記憶體快取功能：系統把記憶體用光了？](https://blog.gtwang.org/linux/linux-cache-memory-linux/)
- [Operating System — Virtual Memory](https://www.tutorialspoint.com/operating_system/os_virtual_memory.htm)
- [htop explained](https://peteris.rocks/blog/htop)

[![Starbugs Weekly 星巴哥技術專欄](https://miro.medium.com/v2/resize:fill:96:96/1*Kma_T7CmrGKYhgVzjiLiXA.jpeg)](https://medium.com/starbugs?source=post_page---post_publication_info--ffb72b3d5629---------------------------------------)

[![Starbugs Weekly 星巴哥技術專欄](https://miro.medium.com/v2/resize:fill:128:128/1*Kma_T7CmrGKYhgVzjiLiXA.jpeg)](https://medium.com/starbugs?source=post_page---post_publication_info--ffb72b3d5629---------------------------------------)

[Last published Dec 6, 2023](https://medium.com/starbugs/%E4%B8%80%E8%B5%B7%E8%B7%B3%E6%A7%BD%E5%90%A7-%E8%8B%B1%E5%9C%8B%E8%BB%9F%E9%AB%94%E5%B7%A5%E7%A8%8B%E5%B8%AB%E6%B1%82%E8%81%B7%E7%B6%93%E9%A9%97%E5%88%86%E4%BA%AB-00e0ef3afe14?source=post_page---post_publication_info--ffb72b3d5629---------------------------------------)

一群技術人想要寫出一些好文章所建立的技術專欄。每週二一篇原創文章、一封電子報，歡迎大家訂閱！主網站： [https://weekly.starbugs.dev/。](https://weekly.starbugs.dev/%E3%80%82)

[![Larry Lu](https://miro.medium.com/v2/resize:fill:96:96/1*ti9wWeiCLA2dhU9ewpFhtQ.jpeg)](https://medium.com/@larry850806?source=post_page---post_author_info--ffb72b3d5629---------------------------------------)

[![Larry Lu](https://miro.medium.com/v2/resize:fill:128:128/1*ti9wWeiCLA2dhU9ewpFhtQ.jpeg)](https://medium.com/@larry850806?source=post_page---post_author_info--ffb72b3d5629---------------------------------------)

[393 following](https://medium.com/@larry850806/following?source=post_page---post_author_info--ffb72b3d5629---------------------------------------)

我是 Larry 盧承億，傳說中的 0.1 倍工程師。我熱愛技術、喜歡與人分享，專長是 JS 跟 Go，平常會寫寫技術文章還有參加各種技術活動，歡迎大家來找我聊聊～

## Responses (3)

Aaron Kuo

What are your thoughts?  

```hs
實用！謝謝分享！
```

3

```hs
感謝分享!
```

```hs
線程 -> 執行緒
```
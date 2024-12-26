并发与并行，傻傻分不清楚？

## 

​[视频解说](https://www.bilibili.com/video/BV1RK4y1R7Kf?p=12)​[](https://fengmuzi2003.gitbook.io/csapp3e/di-12-zhang-bing-fa-bian-cheng#shi-pin-jie-shuo)

## 

导读[](https://fengmuzi2003.gitbook.io/csapp3e/di-12-zhang-bing-fa-bian-cheng#dao-du)

处理器发展的趋势：多核时代早已到来！免费的午餐已经结束了！

参考文章（Herb Sutter）：[The Free Lunch Is Over: A Fundamental Turn Toward Concurrency in Software](http://www.gotw.ca/publications/concurrency-ddj.htm)​

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-Mb_0l1DxbrQroqoosCU%2F-Mb_18hVbbEG2x_KH-n8%2Fimage.png?alt=media&token=01f3e734-7a11-46a5-a5f5-9100b77976c3)

多核处理器成为上述变革的主流解決方案，想要压榨出更多的处理器效能，软件必须跟上硬件的设计！

## 

学习方式[](https://fengmuzi2003.gitbook.io/csapp3e/di-12-zhang-bing-fa-bian-cheng#xue-xi-fang-shi)

​[CMU教授的视频教程 - Lecture23：并发编程](https://www.bilibili.com/video/BV1a54y1k7YE?p=29)​

​[CMU教授的视频教程 - Lecture24：同步（基础）](https://www.bilibili.com/video/BV1a54y1k7YE?p=30)​

​[CMU教授的视频教程 - Lecture25：同步（进阶）](https://www.bilibili.com/video/BV1a54y1k7YE?p=31)​

​[CMU教授的视频教程 - Lecture26：线程级并行](https://www.bilibili.com/video/BV1a54y1k7YE?p=32)​

## 

重点解读[](https://fengmuzi2003.gitbook.io/csapp3e/di-12-zhang-bing-fa-bian-cheng#zhong-dian-jie-du)

​[【大佬讲座】谷歌大神 Rob Pike：并发 vs 并行 (傻傻分不清楚？)](https://www.bilibili.com/video/BV1EN411o7FY/)​

Concurrency （并发）指程序架构，将程序拆成多个可独立运行的部分，不一定要同时运行，Parallelism（并行） 指程序执行，同时执行多个程序。

Concurrency 可能会用到 Parallelism，但不一定要用 Parallelism 才能实现 Concurrency。在 Concurrent 中，工作可拆分成「独立执行」的部份，于是「可以」让很多事情一起做，但「不一定」要真的同时做，参见下图。

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-Mb_6O3J2tKkCGRwXx-i%2F-Mb_8rPEKSwKYp6bHv-6%2Fimage_%E5%89%AF%E6%9C%AC.png?alt=media&token=72531224-2848-440d-bc49-901f13d95b0e)

以Rob Pike的地鼠烧书为例：1） 如何分割各个地鼠负责的工作， 2）各个地鼠如何进行高效的协调与沟通

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-Mb_BzY5PTH2-L6LUtOZ%2F-Mb_FD1xMZ4FrgkZ_jnO%2Fimage.png?alt=media&token=7b2d15b7-c8ea-414c-bc65-96a81ac88192)

那么我们究竟应该如何创建一个并行程序呢？以下流程图示取自CMU：15-418 并行计算架构与编程。

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MbdnJ0wgiJIU8pUDaUX%2F-MbdwBz1e5u_auLO2PVN%2Fimage%20(2).png?alt=media&token=9c61601f-9e46-43e0-acfc-753f140b3dea)

## 

延伸阅读[](https://fengmuzi2003.gitbook.io/csapp3e/di-12-zhang-bing-fa-bian-cheng#yan-shen-yue-du)

- 卡内基梅隆大学：[并行计算架构与编程 (15-418 / 15-618)](https://www.bilibili.com/video/BV1aa4y1s7EH/)​
    

- 伊利诺伊大学：[异构并行编程（胡文美教授）- 重点讲解CUDA编程](https://www.bilibili.com/video/BV1z541137iG/)
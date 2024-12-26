务必搞清楚你自己正在和整个系统的哪一层打交道

## 

​[视频解说](https://www.bilibili.com/video/BV1RK4y1R7Kf?p=10)​[](https://fengmuzi2003.gitbook.io/csapp3e/di-10-zhang-xi-tong-ji-io#shi-pin-jie-shuo)

## 

导读[](https://fengmuzi2003.gitbook.io/csapp3e/di-10-zhang-xi-tong-ji-io#dao-du)

今天我们回头看当初Unix的设计哲学：一切都是文件，都可以用 “打开open –> 读写write/read –> 关闭close” 模式来操作，真的是一个很了不起的决策，历久弥新！

## 

学习方式[](https://fengmuzi2003.gitbook.io/csapp3e/di-10-zhang-xi-tong-ji-io#xue-xi-fang-shi)

​[CMU教授的视频教程 - Lecture16：系统级I/O](https://www.bilibili.com/video/BV1a54y1k7YE?p=20)​

**系统编程的书籍**：推荐两本书（附录中可以直接下载电子书）：

一本是《UNIX环境高级编程》，简称APUE，是已故知名技术作家Richard Stevens的传世之作。

另外一本是《Linux编程接口》，简称TLPI，是针对Linux环境的编程完全手册。

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MZ1O__ZRLUpBVgj2SzF%2F-MZ1WGzhQub2gv_TmgYK%2Fimage.png?alt=media&token=6f301f86-0921-4126-9671-abfe8b48c79f)

## 

重点导读[](https://fengmuzi2003.gitbook.io/csapp3e/di-10-zhang-xi-tong-ji-io#zhong-dian-dao-du)

**学习系统I/O，标准I/O，最好的办法是：自己动手写一个终端编辑器（功能类似vim）**

可以参考Redis 的作者antirez编写的Kilo，之所以称为Kilo是因为它不到1024行的代码， 他花了几个小时就编写出了文本编辑器的原型，同时antirez表示编写该编辑器的原因很简单：[仅为了乐趣](http://antirez.com/news/108) 。

PS. 这里还有一份很棒的入门指南：[Build Your Own Text Editor](https://viewsourcecode.org/snaptoken/kilo/)​

## 

延伸阅读[](https://fengmuzi2003.gitbook.io/csapp3e/di-10-zhang-xi-tong-ji-io#yan-shen-yue-du)

- UNIX环境高级编程（APUE） 视频课程：[史蒂文斯理工(2020年最新课程)](https://www.bilibili.com/video/BV1e54y1t73h/)​
    

- Linux编程接口（TLPI） 极简版视频课程：[《Linux/UNIX系统编程手册》 6小时简易教程 (代码演示)](https://www.bilibili.com/video/BV1uQ4y1d7vQ/)​
    

- 本人拙作（视频讲座）： [《Unix环境高级编程》系列视频课程：文件IO和标准IO视频课程](https://edu.51cto.com/course/11557.html)
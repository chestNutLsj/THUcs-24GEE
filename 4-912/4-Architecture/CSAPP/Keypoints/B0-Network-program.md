实现古人“天涯若比邻”的梦想

## 

​[视频解说](https://www.bilibili.com/video/BV1RK4y1R7Kf?p=11)​[](https://fengmuzi2003.gitbook.io/csapp3e/di-11-zhang-wang-luo-bian-cheng#shi-pin-jie-shuo)

## 

导读[](https://fengmuzi2003.gitbook.io/csapp3e/di-11-zhang-wang-luo-bian-cheng#dao-du)

今天互联网中的大千世界都立足于TCP/IP协议之上，Socket甚至已经成为了网络编程的同义词。

历史：1983年BSD 4.2版本实现了TCP/IP协议，由于UC Berkeley强大的技术实力和良好的声誉，使得BSD 成为最流行的UNIX发行版，很多其他操作系统的网络部分都是基于BSD的源代码开发的，所以BSD加速了互联网前进的步伐，它是BSD对UNIX最重要的贡献之一，Socket的英文原意为 “插孔” 或者 “插座”，通常称作"套接字"，它允许两个进程进行通信，这两个进程可能运行在同一个机器上，也可能运行在不同机器上。

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-McXW-xU66gVyYM9dksv%2F-McXWCnCVD48bpvy_7GR%2Fimage.png?alt=media&token=bf24a7fe-805b-4974-ae04-5b29b0cdcdfe)

从应用程序员的角度来看，它是应用层与TCP/IP协议族通信的中间软件抽象层，是一组接口API。

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-McXW-xU66gVyYM9dksv%2F-McXW6BmZLpJ7SpFK02i%2Fimage.png?alt=media&token=264ced8e-2de9-4dec-90a1-8e9ac6dd7c9a)

## 

学习方式[](https://fengmuzi2003.gitbook.io/csapp3e/di-11-zhang-wang-luo-bian-cheng#xue-xi-fang-shi)

​[CMU教授的视频课程 - Lecture21：网络编程 （上）](https://www.bilibili.com/video/BV1a54y1k7YE?p=27)，[CMU教授的视频课程 - Lecture22：网络编程 （下）](https://www.bilibili.com/video/BV1a54y1k7YE?p=28)​

提及网路编程，W. Richard Stevens对我们这些学习Unix/Linux的程序员的影响是巨大的，每每捧读他写的书都会被感动，不仅被他那丰富的知识所折服，更是被他那一丝不苟，严谨治学的态度所折服。

“他不清楚的，他下决心要弄明白。他知道的，他要努力传授给所有感兴趣的人们！”这就是Stevens！

他的个人网站至今还能访问：[http://www.kohala.com/start/](http://www.kohala.com/start/)，让我们向他致敬！

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MZ6sXtXqTJOov0eIOtF%2F-MZ6wEVDOG_1KQYWkRfk%2F1_%E5%89%AF%E6%9C%AC.jpg?alt=media&token=191c2904-f0b6-43c7-a027-b7563e6cf334)

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MZ6q7Hxk5iUdlmHg6ry%2F-MZ6s9dokT0ABjxiP6wc%2Fu%3D2214001228%2C2595822860%26fm%3D224%26gp%3D0_%E5%89%AF%E6%9C%AC.jpg?alt=media&token=414695c3-c49b-446e-ae2b-f4e4a4d6d619)

​

## 

实验解读[](https://fengmuzi2003.gitbook.io/csapp3e/di-11-zhang-wang-luo-bian-cheng#shi-yan-jie-du)

自己动手写一个小的Web服务器

​

## 

延伸阅读[](https://fengmuzi2003.gitbook.io/csapp3e/di-11-zhang-wang-luo-bian-cheng#yan-shen-yue-du)

- ​[计算机网络 (自顶向下方法)：配套视频教程](https://www.bilibili.com/video/BV1gZ4y1c7Ju/)​
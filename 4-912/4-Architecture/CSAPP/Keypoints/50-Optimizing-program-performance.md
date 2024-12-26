​[视频解说](https://www.bilibili.com/video/BV1RK4y1R7Kf?p=5)
## 导读

程序优化涉及的范围实在是太广了，几乎每个层面都可以进行优化，比如撰写<编译器友好型>以及<缓存友好型>的程序，针对不同的目标硬件平台还可能进行特定的优化，等等，优化的难点在于你需要对系统有充分理解，当然了在你做优化之前**首先要保证原始程序功能正确（并且有回归测试）**，否则一切都是徒劳。

首先需要理解，哪些因素会影响程序的性能

![[50-Optimizing-program-performance-what-impose?.png]]

**温馨提醒**: Nothing can fix a dumb algorithm! == 没有什么能修正一个愚蠢的算法 !

**历史**：第一个编译器是由美国女性计算机科学家葛丽丝·霍普（Grace Hopper）于1952年为A-0 系统编写的。但是1957年由任职于IBM的美国计算机科学家约翰·巴科斯（John Warner Backus）领导的FORTRAN则是第一个被实现出具备完整功能的编译器。1960年，COBOL成为一种较早的能在多种架构下被编译的语言。首个能编译自己源程序的LISP编译器是在1962年由麻省理工学院的Hart和Levin制作的。

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MVpQcaewFVD97nO_IkO%2F-MVpRctEbtdAxWKF_Oef%2Fmaxresdefault.jpg?alt=media&token=baa921ca-bad4-48a1-8273-66fe446a06c9)

题外话：1947年9月9日，葛丽丝·霍普（Grace Hopper）还发现了第一个电脑上的bug。当在Mark II计算机上工作时，整个团队都搞不清楚为什么电脑不能正常运作了。经过大家的深度挖掘，发现原来是一只飞蛾意外飞入了一台电脑内部而引起的故障。这个团队把错误解除了，并在日记本中记录下了这一事件。也因此，人们逐渐开始用“Bug”（原意为“虫子”）来称呼计算机中隐藏的错误。

**编译器领域最早成名的教材**： _Compilers: Principles,Techniques,and Tools，_中文名（龙书）：_编译原理_

本书的一大特点就是抽象难懂，主要讨论了编译器设计的重要主题，包括词法分析、语法分析、语义分析、中间代码生成、代码优化、代码生成等过程。

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MVsyuz18TSwOiZCrltQ%2F-MVt07nkwuQI_FXvP0i7%2F1001948-20171123164637571-2133970350_%E5%89%AF%E6%9C%AC.jpg?alt=media&token=ebe603b5-a47e-466a-ac31-25f491b0dc3f)

2020年图灵奖宣布授予哥伦比亚大学计算机科学名誉教授 Alfred Vaino Aho 和斯坦福大学计算机科学名誉教授 Jeffrey David Ullman，以表彰他们在编程语言实现（programming language implementation）领域基础算法和理论方面的成就。

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-M_ufl_8rv5kmaR6bcMa%2F-M_ugEhl2vBOk-9hayT_%2Fimage.png?alt=media&token=04328abc-5f2d-40dc-a284-a1b266094859)

**从教科书回到现实世界（工业强度的现代编译器）**：[GCC](https://gcc.gnu.org/)，[LLVM](https://llvm.org/)​

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-M_ugLSj6v7caKD75n6h%2F-M_umJNUDWeMPXdpWw96%2Fimage.png?alt=media&token=69311138-c584-41d0-b1c5-71c75311907c)

例如，GCC实现的时候做了两层的中间码，分别是GIMPLE中间码，RTL中间码。代码生成阶段要考虑对应的指令集架构的寄存器使用，以及考虑流水线的调度。

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-M_uJ87D84WoI9L9Wrwb%2F-M_uTav6aNVXfEf46IWC%2Fimage.png?alt=media&token=98926bbb-e3e7-4db5-b988-5c347e6f9257)



## 学习方式

​[CMU教授的视频教程 - Lecture10：程序性能优化 (Program Optimization)](https://www.bilibili.com/video/BV1a54y1k7YE?p=13)​

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-Ma6GX5L3O18fm4k1eBi%2F-Ma6IS3tlQ8Szu1aW9-O%2F%E6%8D%95%E8%8E%B7.PNG?alt=media&token=db603c9e-9131-4e87-946b-8a1ca5479d98)

系统性能优化的诸多着眼点

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-Ma6GX5L3O18fm4k1eBi%2F-Ma6KgoEFTAGrKG4eOfJ%2F%E6%8D%95%E8%8E%B7_%E5%89%AF%E6%9C%AC.png?alt=media&token=e4374d36-62a5-421e-9240-5c9be992f3ab)

编译器优化思路以及它的局限性

## 重点示例

例1：编译器聪明但保守，有时候你写的程序会阻止编译器做优化（precedure side-effects）

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-Ma6NUam9RoSlWM9ndeI%2F-Ma6Sb-bxAgH7QWW4T_b%2F%E6%8D%95%E8%8E%B7_%E5%89%AF%E6%9C%AC.png?alt=media&token=7dfcd956-2c82-49db-be14-0d7beac8aa48)

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-Ma6NUam9RoSlWM9ndeI%2F-Ma6SxSp_c4dpZpWcDFN%2Fimage.png?alt=media&token=a23127ff-3dd0-448b-bdca-64d7660bf8c2)

例2：编译器聪明但保守，有时候你写的程序会阻止编译器做优化（memory alising）

```
int fn (int *a, int *b){
	*a = 3;
	*b = 4;

	return (*a + 5);
}

// 编译器会把以上代码优化成下面的样子么？不会！谁知道程序员会不会这么调用 f(&x,&x);
int fn (int *a, int *b){
	*a = 3;
	*b = 4;

	return (3 + 5);
}

// 但是你可以帮助编译器，使用C99的restrict类型限定符，但还是需要开发者确保两个指针不指向同一数据

// https://gcc.gnu.org/onlinedocs/gcc/Restricted-Pointers.html

int fn (int *__restrict__ a, int *__restrict__ b){
	*a = 3;
	*b = 4;

	return (*a + 5); // 这里会被优化为 return (3 + 5)
}
```

例3：善用编译器的特定选项或者开关（当然也需要处理器支持），利用有限的资源创造出最大的价值。

请参考教学投影片（p22~）以及CMU教授的视频讲解，与原始程序相比，性能达到了数十倍（甚至百倍）的提升，其中使用到了许多优化方面的技巧，比如 code motion，loop unrolling，auto vectorization，etc，相信会对你有所启发。
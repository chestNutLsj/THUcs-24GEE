---
tags:
  - 计算机组成原理
date: 2023-10-21
publish: "true"
---
## 数据表示

### 数据表示
需要在计算机中表示的对象：
- 程序、整数、浮点数、字符（串）、逻辑值

通过编码表示，表示方式
- 用数字电路的两个状态表示，存放在机器字中
- 由上一层的抽象计算机来识别不同的内容

编码原则
- 少量简单的基本符号
- 一定的规则
- 表示大量复杂的信息
- 计算性能与存储空间要均衡

编码表示：
- 基本元素：0、1 两个基本符号
- 字符表示


数值型数据表示
- 定点数
	- 无符号整数 0，1，…，2^(n-1)
	- 有符号整数
	- 定点小数
- 浮点数
- 逻辑值 0->false, 1->true 
	- 与运算
	- 或运算
	- 非运算
	- 异或运算
- 颜色（RGB）
- 位置/地址/指令：n 位只能代表 2^n 个不同的对象


### 字符表示
- 字符：26 字符-> $\lceil \log_{2}26\rceil=5$ 位
	- 大/小写+其它符号 -> 7bits -> ASCII 码
	- 世界上其它语言的文字 -> 16 bits（Unicode）
	- UTF-8：变长字符编码，字符长度由首字节确定
		- ![[12-Data-Expression-and-EDC-utf8.png]]
		- 首字节外，均以“10”开始，可自同步
		- 可扩展性强
		- 成为互联网上占统治地位的字符集
	- 点阵字体：点阵本质是单色位图，如果为 0，对应位置没有点，为 1 则显示点
		- 文字编码->查找字体文件->找到点阵->显示
	- 矢量字体：
		- ![[12-Data-Expression-and-EDC-truetype.png]]
		- 一个字可以用多条曲线来表示，每条曲线保存其关键点
		- 显示字的时候，取出这些关键点，采用平滑的曲线将这些关键点连接起来，并填充闭合空间以显示
		- 需要放大或者缩小的时候，按照比例改变关键点的相对位置即可
		- 不会失真

## 整数与浮点数

- 数值范围：指一种类型的数据所能表示的最大值和最小值
- 数据精度：指实数所能给出的有效数字位数；对浮点数来说，精度不够会造成误差，误差大量积累会出问题
- 机内处理：数值范围和数据精度概念不同。在计算机中，它们的值与用多少个二进制位表示某种类型的数据，以及怎么对这些位进行编码有关

### 整数
- 原码，反码，补码
	- 正数的原码、反码、补码表示均相同，符号位为 0，数值位同数的真值。
	- 零的原码和反码均有 2 个编码，补码只 1 个码
	- 负数的原码、反码、补码表示均不同
		- 符号位为 1，数值位：原码为数的绝对值
		- 反码为除符号位外，每一位均取反
		- 补码为反码再在最低位+1
		- 只有一个负数的原码与补码是相同的：1100 0000 0000 0000 0000 0000 0000 0000
			- ![[12-Data-Expression-and-EDC-question.png]]
	- 由 `[X]补` 求 `[-X]补`：每一位取反后再在最低位+1
- 符号扩展
- 大端，小端
- 加法，减法，乘法，除法


### 浮点数
#### 表示方法
图片来源：[^1]
$$
V=(-1)^{s}\times M\times 2^{E}
$$
- s: 符号位
- M: 尾数，二进制小数，范围为 1~2-ε 或 0~1-ε
- E: 阶码，对浮点数进行加权，权重是 2^E

![[12-Data-Expression-and-EDC-ieee754.png]]
- 符号位单独编码；
- 阶码字段 $exp=e_{k-1}...e_{1}e_{0}$，k 位（在单精度中 8 位，双精度中 11 位）
- 小数字段 $frac=f_{n-1}...f_{1}f_{0}$，n 位（单精度 23 位，双精度 52 位）

#### 规格化与非规格化
![[12-Data-Expression-and-EDC-normalized-ieee754.png]]
- 规格化：阶码不全为 0 或 1
	- 阶码被解释为以偏置形式表示的有符号整数：E=e-Bias, 
	- e 是无符号数，其位表示为 $e_{k-1}...e_{1}e_{0}$
	- Bias 是 $2^{k-1}-1$，对单精度就是 $2^{8-1}-1=127$，对双精度是 $2^{11-1}-1=1023$ 
	- 综上，对规格化数，单精度的指数范围为 -126~+127, 双精度的指数范围 -1022~+1023
	- frac 字段为小数 $frac=0.f_{n-1}...f_{1}f_{0}$，而尾数 M=1+f，表达一个 $1.f_{n-1}f_{n-2}...f_{1}f_{0}$ 的二进制小数，即尾数总是处于 $0\le M<2$ 之间，那么小数点前的 1 可以忽略。
- 非规格化：阶码全 0
	- 此时阶码值为 E=1-Bias
	- 尾数为 M=frac，不包含开头的 1
	- 用途：表示数值 0（除符号位外全 0 表示 0），同样有+0.0 和-0.0 之分
	- 另外可以表示非常接近于 0.0 的数
- 特殊值：阶码全 1:
	- 尾数全 0 —— 无穷大
	- 尾数非 0 —— NaN

#### 表示的值的范围
![[12-Data-Expression-and-EDC-6bit-ieee754.png]]
- 6bit 的位中，3 个阶码，2 个尾数：$S_{5}E_{4}E_{3}E_{2}M_{1}M_{0}$
- 则非规格化数的尾数部分为 0.11，0.10，0.01，0.00，用以表示 0 或极小值
- 阶码的取值范围为 `[0,1,2,3,4,5,6,7] - Bias = [0..7] - (2^(3-1) -1) = [-3,-2,-1,0,1,2,3,4]`，其中-3 的表示是 000——非规格化，4 的表示是 111——特殊值

![[12-Data-Expression-and-EDC-ieee-range.png]]
- 可以看到非规格化数 0 0000 111 到规格化数 0 0001 000 的平滑转变；

![[12-Data-Expression-and-EDC-floating-numbers-example.png]]
- 注意 1 的表示方式，其中阶码 M=exp-Bias=2^(k-1)-1-Bias=0

> [! note] 一些结论
> 对于 n 位小数的浮点格式，其不能准确描述的最小正整数为：2^(n+1)+1
> 对于单精度格式，n=23，这个值为 2^24+1=16777216

#### 舍入方式
![[12-Data-Expression-and-EDC-floating-rounding.png]]
- 向偶数舍入，又称向最接近值舍入，默认的方式。将数字向上或向下舍入，使得==结果的最低有效数字是偶数==；
- 其它三种舍入方式得到实际值的确界 guaranteed bound
- Round-to-even rounding can be applied even when we are not rounding to a whole number. We simply consider whether the least significant digit is even or odd. For example, suppose we want to round decimal numbers to the nearest hundredth. We would round 1.2349999 to 1.23 and 1.2350001 to 1.24, regardless of rounding mode, since they are not halfway between 1.23 and 1.24. On the other hand, we would round both 1.2350000 and 1.2450000 to 1.24, since 4 is even.
- Similarly, round-to-even rounding can be applied to binary fractional numbers. We consider least significant bit value 0 to be even and 1 to be odd. In general, the rounding mode is only significant when we have a bit pattern of the form $XX ... X.YY ... Y 100 ...$, where X and Y denote arbitrary bit values with the rightmost Y being the position to which we wish to round. Only bit patterns of this form denote values that are halfway between two possible results. 
	- As examples, consider the problem of rounding values to the nearest quarter (i.e., 2 bits to the right of the binary point.) 
	- We would round $10.00011_{2} (2 \frac{3}{32} )$ down to $10.00_2 (2)$, and $10.00110_{2} (2 \frac{3}{16} )$ up to $10.01_{2} (2 \frac{1}{4} )$, because these values are not halfway between two possible values. 
	- We would round $10.11100_{2} (2 \frac{7}{8} )$ up to $11.00_{2} (3)$ and $10.10100_{2} (2 \frac{5}{8} )$ down to $10.10_{2} (2 \frac{1}{2} )$, since these values are halfway between two possible results, and we prefer to have the least significant bit equal to zero.

#### 浮点运算（加法、乘法）
We saw earlier that integer addition, both unsigned and two’s complement, forms an abelian group. Addition over real numbers also forms an abelian group, but we must consider what effect rounding has on these properties.
> 补码、无符号加法构成阿贝尔群 (满足交换律)，实数运算也形成阿贝尔群，但由于精度的舍入问题，浮点数不满足结合性。

- Let us define $x +^{f} y$ to be Round (x + y). This operation is defined for all values of x and y, although ==it may yield infinity even when both x and y are real numbers due to overflow==. 
- The operation is commutative, with $x +^{f} y = y +^{f} x$ for all values of x and y. On the other hand, the operation is not associative. 
> 浮点数加法满足交换律，但不满足结合律，如果由于交换导致结合顺序发生变化，则要加括号。

For example, with single precision floating point the expression (3.14+1e10)-1e10 evaluates to 0.0—the value 3.14 is lost due to rounding. On the other hand, the expression 3.14+(1e10 - 1e10) evaluates to 3.14. 

As with an abelian group, most values have inverses under floating-point addition, that is, $x +^f −x = 0$. The exceptions are infinities (since $+∞ − ∞ = NaN$), and NaNs, since $NaN +^f x = NaN$ for any x. 

The lack of associativity in floating-point addition is the most important group property that is lacking. It has important implications for scientific programmers and compiler writers. For example, suppose a compiler is given the following code fragment:
```
x = a + b + c
y = b + c + d
```
The compiler might be tempted to save one floating-point addition by generating the following code:
```
t = b + c
x = a + t
y = t + d
```
However, this computation might yield a different value for x than would the original, since it uses a different association of the addition operations. In most applications, the difference would be so small as to be inconsequential. Unfortunately, compilers have no way of knowing what trade-offs the user is willing to make between efficiency and faithfulness to the exact behavior of the original program. As a result, they tend to be very conservative, avoiding any optimizations that could have even the slightest effect on functionality.
> 浮点加法不具有结合性，这一点要万分小心，尽量不要做不必要的优化。

On the other hand, floating-point addition satisfies the following monotonicity property: if a ≥ b, then x +f a ≥ x +f b for any values of a, b, and x other than NaN. This property of real (and integer) addition is not obeyed by unsigned or two’s complement addition.
> 浮点加法具有单调性，只要没有 NaN 参与运算。

Let us define $x *^f y$ to be Round(x × y). This operation is closed under multiplication (although possibly yielding infinity or NaN), it is commutative, and it has 1.0 as a multiplicative identity. On the other hand, it is not associative, due to the possibility of overflow or the loss of precision due to rounding. 
> 浮点数乘法是封闭的（尽管可能溢出或 NaN），满足交换律，不满足结合律。

For example, with single-precision floating point, the expression `(1e20*1e20)*1e-20` evaluates to $+∞$, while `1e20*(1e20*1e-20)` evaluates to 1e20. 

In addition, floating-point multiplication does not distribute over addition. For example, with single-precision floating point, the expression `1e20*(1e20- 1e20)` evaluates to 0.0, while `1e20*1e20-1e20*1e20` evaluates to NaN. On the other hand, floating-point multiplication satisfies the following monotonicity properties for any values of a, b, and c other than NaN:
```
a>=b and c>=0 -> a *f c >= b *f c
a>=b and c<=0 -> a *f c <= b *f c
```
> 浮点乘法在加法上不具有分配性
> 浮点乘法具有单调性
> 无符号数或补码不具有单调性

#### C 语言中的浮点数

Unfortunately, since the C standards do not require the machine to use IEEE floating point, there are no standard methods to change the rounding mode or to get special values such as −0, +∞, −∞, or NaN. Most systems provide a combination of include (.h) files and procedure libraries to provide access to these features, but the details vary from one system to another. 

For example, the GNU compiler gcc defines program constants `INFINITY` (for +∞) and `NAN` (for NaN) when the following sequence occurs in the program file: 
```
#define _GNU_SOURCE 1 
#include <math.h>
```

When casting values between int, float, and double formats, the program changes the numeric values and the bit representations as follows (assuming data type int is 32 bits): 
- From int to float, the number ==cannot overflow, but it may be rounded==. 
- From ==int or float to double, the exact numeric value can be preserved== because double has both greater range (i.e., the range of representable values), as well as greater precision (i.e., the number of significant bits). 
- From double to float, the value ==can overflow to +∞ or −∞==, since the range is smaller. Otherwise, it may be rounded, because ==the precision is smaller==.
- From float or double to int, ==the value will be rounded toward zero==. For example, 1.999 will be converted to 1, while −1.999 will be converted to −1. Furthermore, the value may overflow. The C standards do not specify a fixed result for this case. Intel-compatible microprocessors designate the bit pattern `[10 ... 00]` ($TMin_w$ for word size w) as an integer indefinite value. Any conversion from floating point to integer that cannot assign a reasonable integer approximation yields this value. Thus, the expression (int) +1e10 yields -21483648, generating a negative value from a positive one.

```
int x;
float f;
double d; //除了NaN、∞都能随便取，判断以下结果

x == (int)(double)x // true

x == (int)(float)x // false

d == (double)(float)d //false

f == (float)(double)f //true

f == -(-f)    //true

1.0/2 == 1/2.0  //true，先转浮点数再计算

d*d >= 0.0  //true，符号位负负得正

(f+d)-f == d  //false，d可能被舍入
```

## 检错纠错

- 使编码具有某种特征，通过检查这种特征是否存在来判断编码是否正确
- 出错时，如果还能支出是哪位出错，则可以纠正错误

### 码距

码距(最小码距)的概念：是指任意两个合法码之间至少有几个二进制位不相同。
- 仅有一位不同的编码是无纠错能力的。例如用 4 位二进制表示 16 种状态，则 16 种编码都用到了，此时码距为 1。任意一个编码状态的四位码中的一位或者几位出错，都会变成另外一个合法码。这种编码无检错能力。
- 若用 4 个二进制位表示 8 种合法的状态，就可以只使用其中的 8 个编码来表示，另外 8 个为非法编码。此时可以使合法码的码距为 2。任何一位出错后都会成为非法码，有发现一位出错的能力
- 合理增大码距，能提高发现错误的能力，但表示一定数量的合法码所使用的二进制位数要变多，增加了电子线路的复杂性和数据存储、数据传送的数量。

- 奇偶校验码的码距为 2
- 海明码的码距为 4

### 奇偶校验码
流程：数据编码 -> 数据传输 -> 数据译码

- 奇校验，偶校验
- 用于并行码检错
- 原理：在 k 位数据码之外增加 1 位校验位，
	- 使 K+1 位码字中取值为 1 的位数总保持为偶数（偶校验）或奇数（奇校验）。
	- ![[12-Data-Expression-and-EDC-parity-check.png]]

### 海明校验码

- 用于多位并行数据检错纠错处理
- 实现目标：为 k 个数据位设立 r 个校验位，使 k+r 位的码字同时具有这样两个特性：
	- 能发现并改正 k+r 位中任何一位出错
	- 能发现 k+r 位中任何二位同时出错，但已无法改正

- 策略：
	- 合理的使用 k 位数据形成 r 个校验位的值，保证用 k 个数据中不同的数据位组合来形成每个校验位的值，使任何一个数据位出错时，将影响 r 个校验位中不同的校验位组合起变化。这样一来，就可以通过检查哪种校验位组合起了变化，来推断是那个数据位错误造成的，对该位求反则实现纠错
	- 有的时候两位出错与某种情况的一位出错对校验位组合的影响相同，必须加以区分与解决
	- 位数 r 和 k 的关系：2^r≥k+r+1，即用 2^r 个编码分别表示 k 个数据 位，r 个校验位中哪一位出错，都不错
	- 2^(r-1)≥k+r，用 r-1 位校验码为出错位编码，再单独设一位用以区分 1 位还是 2 位同时出错，更实用

#### 海明码的实现

k=3, r=4: 满足 2^(4-1)=8>3+4，即 r 中 3 位用于出错位编码，1 位区分 1 位错还是 2 位错：

| D3  | D2  | D1  | P4  | P3  | P2  | P1  |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|  1  |  1  |  1  |  1  |  1  |  1  |  1  |
|  1  |  1  |  0  |  0  |  1  |  0  |  0  |
|  1  |  0  |  1  |  0  |  0  |  1  |  0  |
|  0  |  1  |  1  |  0  |  0  |  0  |  1  | 

编码方案：
- $P1=D2\oplus D1$
- $P2=D3\oplus D1$
- $P3=D3\oplus D2$
- $P4=P3\oplus P2\oplus P1\oplus D3\oplus D2\oplus D1$ 

译码方案：
- $S1=P1\oplus D2\oplus D1$
- $S2=P2\oplus D3\oplus D1$
- $S3=P3\oplus D3\oplus D2$
- $S4=P4\oplus P3\oplus P2\oplus P1\oplus D3\oplus D2\oplus D1$

#### 海明码举例

如何分配不同的数据位组合来形成每个校验位的值？
- P1 P2 D1 P3 D2 D3 P4
- 1  2  3  4  5  6    

步骤：
1. 从 1~6 按次序排列数据位、校验位，将校验位 P1、P2、P3 依次安排在 2 的幂次方位。P4 为总校验位，暂不考虑。
2. 为各校验位分配数据位组合：
	1) 看数据位的编号分别为 3、5、6，它们是校验位编号的组合：`3=1+2`、`5=1+4`、`6=2+4` 
	2) 1 出现在 3 和 5 中，则 P1 负责对 D1 和 D2 进行校验。
	3) 2 出现在 3 和 6 中，则 P2 负责对 D1 和 D3 进行校验。
	4) 4 出现在 5 和 6 中，则 P3 负责对 D2 和 D3 进行校验。
3. 写出各校验位的编码逻辑表达式：
	1) 结果是: P1 = D2 ⊕ D1；P2 = D3 ⊕ D1；P3 = D3 ⊕ D2
	2) 用其他各校验位及各数据位进行异或运算求校验位 P4 的值，用于区分无错、奇数位错、偶数位错 3 种情况：总校验位 P4 = P3 ⊕ P2 ⊕ P1 ⊕ D3 ⊕ D2 ⊕ D1

海明码译码方案：
- 对接收到数据位再次编码，用得到的结果和传送过来的校验位的值相比较，二者相同表明无错，不同是有 1 位错了。
- 或者将校验位与对应数据位进行异或，获得 S4~S1 值，排查是哪一位错了，就看 S4~S1 这4 位的编码值

![[12-Data-Expression-and-EDC-hamming-code.png]]

![[12-Data-Expression-and-EDC-more-hamming-code.png]]

### 循环冗余校验码

[^1]: [Fetching Title#8faa](https://waynerv.com/posts/csapp-ieee-floating-intro/)
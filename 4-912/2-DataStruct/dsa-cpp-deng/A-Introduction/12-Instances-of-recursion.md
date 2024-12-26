---
publish:
---
## Hanoi 问题
描述：法国数学家 Edouard Lucas 于 1883 提出的 Hanoi 塔问题，可形象地描述如下：
- 有 n 个中心带孔的圆盘贯穿在直立于地面的一根柱子上，各圆盘的半径自底而上不断缩小；
- 需要利用另一根柱子将它们转运至第三根柱子，但在整个转运的过程中，游离于这些柱子之外的圆盘不得超过一个 (即每次只能取一个)，且每根柱子上圆盘半径都须保持上小下大。
试将上述转运过程描述为递归形式，并而实现一个递归算法。

### 递归版
将三根柱子分别记作 X、Y 和 Z，则整个转运过程可递归描述为：
- 为将 X 上的 n 个盘子借助 Y 转运至 Z，只需（递归地）
	- 将 X 上的 n - 1 个盘子借助 Z 转运至 Y 
	- 再将 X 上最后一个盘子直接转移到 Z 
	- 最后再将 Y 上的 n - 1 个盘子借助 X 转运至 Z 

按照这一理解，即可如下所示实现对应的递归算法：
```
// 按照Hanoi规则，将柱子Sx上的n只盘子，借助柱子Sy中转，移到柱子Sz上

void hanoi ( int n, Stack<Disk>& Sx, Stack<Disk>& Sy, Stack<Disk>& Sz ) {
	if ( n > 0 ) { //没有盘子剩余时，不再递归
		hanoi ( n - 1, Sx, Sz, Sy ); //递归：将Sx上的n - 1只盘子，借助Sz中转，移到Sy上
		move ( Sx, Sz ); //直接：将Sx上最后一只盘子，移到Sz上
		hanoi ( n - 1, Sy, Sx, Sz ); //递归：将Sy上的n - 1只盘子，借助Sx中转，移到Sz上
	}
}

```

关于时间复杂度，该算法对应的边界条件和递推式为：
T(1) = O(1)
T(n) = 2T(n - 1) + O(1)

若令: S(n) = T(n) + O(1)
则有: S(1) = O(2) 
     S(n) = 2∙S (n - 1) 
          = 2^2 ∙ S(n - 2) 
          = 2^3 ∙ S(n - 3)
          = ... = 2^(n-1) ∙ S(1) = 2n 

故有： T(n) = O(2^n)

### 迭代版 

```
// Iterative Algorithm
1. Calculate the total number of moves required 
	i.e."pow(2, n)- 1" here n is number of disks.

2. If number of disks (i.e. n) is even then interchange destination pole and auxiliary pole.

3. for i = 1 to total number of moves:
    if i%3 == 1:
	    legal movement of top disk between source pole and destination pole
    if i%3 == 2:
	    legal movement top disk between source pole and auxiliary pole    
    if i%3 == 0:
        legal movement top disk between auxiliary pole and destination pole
```

![[12-Instances-of-recursion-hanoi.png]]
- Let us understand with a simple example with 3 disks: So, total number of moves required = 2^3-1 = 7
- When i=1, (i % 3 == 1) legal movement between ‘A’ and ‘C’
- When i=2, (i % 3 == 2) legal movement between ‘A’ and ‘B’
- When i=3, (i % 3 == 0) legal movement between ‘B’ and ‘C’
- When i=4, (i % 3 == 1) legal movement between ‘A’ and ‘C’
- When i=5, (i % 3 == 2) legal movement between ‘A’ and ‘B’
- When i=6, (i % 3 == 0) legal movement between ‘B’ and ‘C’
- When i=7, (i % 3 == 1) legal movement between ‘A’ and ‘C’
- So, after all these destination poles contains all the in order of size.
- After observing above iterations, we can think that after a disk other than the smallest disk is moved, the next disk to be moved must be the smallest disk because it is the top disk resting on the spare pole and there are no other choices to move a disk.

```
// C++ Program for Iterative Tower of Hanoi using STL

#include <iostream>
#include <vector>
#include <stack>
using namespace std;

char rod[]={'S', 'A', 'D'};
vector<stack<int>> stacks(3); // 3 stacks for 3 rods

void moveDisk(int a, int b)
{
	if (stacks[b].empty() || (!stacks[a].empty() && stacks[a].top() < stacks[b].top()))
	{
		cout << "Move disk " << stacks[a].top() << " from rod " << rod[a] << " to rod " << rod[b] << "\n";
		stacks[b].push(stacks[a].top());
		stacks[a].pop();
	}
	else
		moveDisk(b, a);
}

void towerOfHanoi(int n)
{
	cout << "Tower of Hanoi for " << n << " disks:\n";

	int src = 0, aux = 1, dest = 2;
	for (int i = n; i > 0; i--)
		stacks[src].push(i);

	int totalMoves = (1 << n) - 1;
	if (n % 2 == 0)
		swap(aux, dest);

	for (int i = 1; i <= totalMoves; i++)
	{
		if (i % 3 == 0)
			moveDisk(aux, dest);
		else if (i % 3 == 1)
			moveDisk(src, dest);
		else
			moveDisk(src, aux);
	}
}

int main()
{
	int n = 3; // number of disks
	towerOfHanoi(n);
	return 0;
}

```

### References

**Related Articles** 

- [Recursive Functions](https://www.geeksforgeeks.org/recursive-functions/)
- [Tail recursion](https://www.geeksforgeeks.org/tail-recursion/)
- [Quiz on Recursion](https://www.geeksforgeeks.org/algorithms-gq/recursion-gq/)

**References:**   
[http://en.wikipedia.org/wiki/Tower_of_Hanoi#Iterative_solution](http://en.wikipedia.org/wiki/Tower_of_Hanoi#Iterative_solution)

## 八皇后问题
![[43-Probing-backtracking#八皇后问题]]

## 迷宫问题
![[43-Probing-backtracking#迷宫寻径]]

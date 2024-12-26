---
publish: "true"
tags:
  - DSA
  - 邓俊辉
  - Cpp
---
## ADT
![[30-List-listnode-ADT.png]]

```cpp
template <typename T> using ListNodePosi = ListNode<T>*; //列表节点位置（C++.0x）
template <typename T> struct ListNode { //简洁起见，完全开放而不再严格封装
	T data; //数值
	ListNodePosi<T> pred; //前驱
	ListNodePosi<T> succ; //后继
	ListNode() {} //针对header和trailer的构造
	ListNode(T e, ListNodePosi<T> p = NULL, ListNodePosi<T> s = NULL): data(e), pred(p), succ(s) {} //默认构造器
	ListNodePosi<T> insertAsPred( T const & e ); //前插入
	ListNodePosi<T> insertAsSucc( T const & e ); //后插入
};
```

![[30-List-list-ADT.png]]

![[30-List-struct.png]]
- 头节点的 order 设置为-1，尾节点的 order 设置为 n，都是哨兵，所谓 order 可以用以列表模拟 vector 的循秩访问，只是时间复杂度为 $O (rank)$；
- 创建列表时，列表为空，此时只需要连接头尾节点，各自设置好指针域；

## 无序列表
### 增
![[30-List-insert.png]]
- $O (1)$, 需要查找时间

### 删
![[30-List-remove.png]]
- $O (1)$，但是需要查找时间

### copy and delete
![[30-List-copy&delete.png]]
- $O (n)$

### 查
自后向前：
![[30-List-find.png]]
- $O (n)$

### 去重
![[30-List-deduplicate.png]]
- $O (n^2)$

### 遍历
自前向后：
![[30-List-traverse.png]]
- $O (n \times opt)$

## 有序向量
### 唯一化
![[30-List-uniquify.png]]
- $O (n)$

### 查找
![[30-List-search.png]]
- 最好 $O (1)$，最坏 $O (n)$
- 按照循位置访问的方式，物理存储地址与其逻辑次序无关，而只能依据元素间的引用顺序访问
- 如何提高查找效率？——引入其它数据结构
	- **使用哈希表：** 如果需要频繁地查找链表中的特定元素，可以考虑将链表与哈希表结合使用。可以使用元素的某个唯一属性（例如关键字）作为哈希表的键，将元素的指针存储在哈希表中。这样，可以通过哈希表快速定位到链表中的元素，而不需要遍历整个链表。
	- **缓存：** 如果你的应用程序对链表的访问具有一定的局部性，可以考虑使用缓存来存储最近访问的链表节点。这可以减少对链表的实际访问次数，提高查找效率。
	- **分割链表：** 如果链表非常长而且你知道要查找的元素位于链表的某个特定部分，你可以考虑将链表分割为多个较短的链表。这样，你可以只查找特定部分的链表，从而减少查找的时间复杂度。—— [[70-Hash#跳转表|跳转表]]
	- **使用平衡二叉搜索树：** 如果你的链表需要支持高效的插入和删除操作，考虑使用平衡二叉搜索树来代替链表。平衡二叉搜索树具有快速的查找、插入和删除性能。

## 排序
### 选择排序
#### 基本思想

选择未排序区间的最大值，移动到已排序区间的最小位置
![[30-List-selection-sort-shift.png]]

```cpp
// selectionSort
template <typename T> void List<T>::selectionSort( ListNodePosi<T> p, Rank n ) {
	ListNodePosi<T> head = p->pred, tail = p;
	for ( Rank i = 0; i < n; i++ ) tail = tail->succ; //待排序区间为(head, tail)
	while ( 1 < n ) { //反复从（非平凡）待排序区间内找出最大者，并移至有序区间前端
		insert( remove( selectMax( head->succ, n ) ), tail ); //可能就在原地...
		tail = tail->pred; n--; //待排序区间、有序区间的范围，均同步更新
	}
}

template <typename T> //从起始于位置p的n个元素中选出最大者，1 < n
ListNodePosi<T> List<T>::selectMax( ListNodePosi<T> p, Rank n ) { //Θ(n) 
	ListNodePosi<T> max = p; //最大者暂定为p
	for ( ListNodePosi<T> cur = p; 1 < n; n-- ) //后续节点逐一与max比较
	if ( ! lt( (cur = cur->succ)->data, max->data ) ) //data >= max
		max = cur; //则更新最大元素位置记录
	return max; //返回最大节点位置
}
```

选择排序的选择、置放策略：
![[30-List-selectionSort-situation1.png]]

未排序区间的搜索策略：
![[30-List-selectionSort-situation2.png]]

#### 稳定性
![[30-List-selectionSort-stability.png]]

#### 性能
共迭代 n 次，在第 k 次迭代中
- `selectMax()` 为 $Θ(n - k)$   //算术级数
- `swap()` 为 $O (1)$            //或 remove () + insert ()
故总体复杂度应为 $Θ(n^2)$

尽管如此，==元素的移动操作远远少于起泡排序== //实际更为费时
也就是说，$Θ(n^2)$ 主要来自于元素的比较操作 //成本相对更低

>[!tip] 可否... 每轮只做 $o (n)$ 次比较，即找出当前的最大元素？
  可以！利用高级数据结构，`selectMax()` 可改进至 $O (\log n)$ ——大顶堆。
  当然，如此立即可以得到 $O (n\log n)$ 的排序算法——堆排序。

### 插入排序
#### 基本思想
从未排序的区间中取一个元素，不失一般性就取区间开头，然后在已就序区间选择合适位置插入：
![[30-List-insertionSort-idea.png]]
- 不变性：序列总能视作两部分：`S[0, r) + U[r, n)`
- 初始时：`|S| = r = 0`
- 如何查找？顺序还是二分？（此处是列表，故只能顺序，但若是向量，可以考虑二分，但是向量的插入操作的时间复杂度为 $O (r)$）

#### 实现
```cpp
template <typename T> void List<T>::insertionSort( ListNodePosi<T> p, Rank n ) { 
	for ( Rank r = 0; r < n; r++ ) { //逐一引入各节点，由Sr得到Sr+1
		insert( search( p->data, r, p ), p->data ); //查找 + 插入
		p = p->succ; remove( p->pred ); //转向下一节点
	} //n次迭代，每次O(r + 1)
} //仅使用O(1)辅助空间，属于就地算法
```

- 紧邻于 search ()接口返回的位置之后插入当前节点，总是保持有序
- 验证各种情况下的正确性，体会哨兵节点的作用： Sr 中含有/不含与 p 相等的元素；Sr 中的元素均严格小于/大于 p

#### 性能
- 插入排序是==就地算法==：空间复杂度 O (1)
- ==在线算法==：不必等数据传输完全完成，即可进行排序
- 具有==输入敏感性==：输入数据序列越有序，时间复杂度越低——$O (n)$
- 优化方向：在有序前缀中的查找定位，采用二分查找——但这仍然是常系数意义上的优化，在数据不大时体现不出来，并且列表本身不支持二分查找


若各元素的取值系独立均匀分布，平均要做多少次元素比较？
- 考查 `e=[r]` 刚插入完成的那一时刻，此时的有序前缀 `[0, r]` 中，谁是 `e` ？
- 观察：其中的 r+1 个元素均有可能，且概率均为 $\frac{1}{r+1}$ 
- 因此，在刚完成的这次迭代中为引入 S\[r\]所花费时间的数学期望为 $1+\sum\limits_{k=0}^{r} \frac{k}{r+1}=1+ \frac{r}{2}$
- 于是，总体时间的数学期望为 $\sum\limits_{r=0}^{n-1}(1+ \frac{r}{2})=O(n^{2})$
- 再问：在 n 次迭代中，平均有多少次无需交换呢？—— [[31-List-Exercise#3-10 插入排序性能分析|习题 3-10：插入排序性能分析]]

### 归并排序
#### 思想
![[30-List-mergeSort-illustration.png]]

```cpp
// 主算法
template <typename T> void List<T>::mergeSort( ListNodePosi<T> & p, Rank n ) {
	if ( n < 2 ) return; //待排序范围足够小时直接返回，否则...
	ListNodePosi<T> q = p; Rank m = n >> 1; //以中点为界
	for ( Rank i = 0; i < m; i++ ) q = q->succ; //均分列表：O(m) = O(n)
	mergeSort( p, m ); mergeSort( q, n – m ); //子序列分别排序
	p = merge( p, m, *this, q, n – m ); //归并
} //若归并可在线性时间内完成，则总体运行时间亦为O(nlogn)
```
^da908a

#### 二路归并

```cpp
template <typename T> ListNodePosi<T> //this.[p +n) & L.[q +m)：归并排序时，L == this
List<T>::merge( ListNodePosi<T> p, Rank n, List<T>& L, ListNodePosi<T> q, Rank m ) {
	ListNodePosi<T> pp = p->pred; //归并之后p或不再指向首节点，故需先记忆，以便返回前更新
	while ( ( 0 < m ) && ( q != p ) ) //小者优先归入
		if ( ( 0 < n ) && ( p->data <= q->data ) ) { p = p->succ; n--; } //p直接后移
		else { insert( L.remove( ( q = q->succ )->pred ) , p ); m--; } //q转至p之前
	return pp->succ; //更新的首节点
} //运行时间O(n + m)，线性正比于节点总数

```

## 循环节
### 定义
任何一个元素之间可以定义次序的序列 `A[0, n)` ，都可以分解为若干个循环节：
- 任何一个序列 A，都对应于一个有序序列 `S[0, n)`
- 元素 `A[k]` 在 S 中对应的秩，记作 $r (A[k])=r (k) \in [0, n)$
- 元素 `A[k]` 所属的循环节是：
$A[k],A[r(k)],A[r(r(k))],A[r(r(r(k)))],...,A[r(...(r(r(k))...)]=A[k]$
- 每个循环节，长度均不超过 $n$
- 循环节之间，互不相交

### 实例
![[30-List-repetend-instance.png]]

1. J -> C -> P -> E -> A -> J
2. N -> L -> B -> N
3. ~~P -> E -> A -> J -> C -> P~~ (P 是 1 中的一个节点，故自身的循环节就是 1)
4. M -> K -> H -> O -> F -> I -> D -> M
5. G -> G (自成循环节)

### 单调性
采用交换法，每迭代一步，M 都会脱离原属的循环节，自成一个循环节:
![[30-List-repetend.png]]
### 无效交换
问：M 已经就位，无需交换 —— 这种情况会出现几次？
答：最初有 $c$ 个循环节，就出现 $c$ 次 —— 最大值为 $n$，期望 $Θ(\log n)$

## 逆序对
### 定义
考查序列 A\[0, n\)，设元素之间可比较大小，则
$<i,j> \text{is called an inversion iff.}\ 0\le i<j<n\text{ and }A[i]>A[j]$

为便于统计，可将逆序对统一记到后者的账上 (例如 5>3，则在 3 的逆序数标记上加一)：
- `A[] = { 5, 3, 1, 4, 2 }` 中，共有 0 + 1 + 2 + 1 + 3 = 7 个逆序对
- `A[] = { 1, 2, 3, 4, 5 }` 中，共有 0 + 0 + 0 + 0 + 0 = 0 个逆序对
- `A[] = { 5, 4, 3, 2, 1 }` 中，共有 0 + 1 + 2 + 3 + 4 = 10 个逆序对

显然，逆序对总数 $I=\sum\limits_{j}I(j)\le \binom{n}{2}=O(n^{2})$ 。
### 冒泡排序与逆序对

在序列中交换一对逆序元素，==逆序对总数必然减少==：
- 在序列中交换一对紧邻的逆序元素，逆序对总数恰好减一
- 因此对于 Bubblesort 算法而言，交换操作的次数恰等于输入序列所含逆序对的总数

### 插入排序与逆序对

针对 `e=A[r]` 的那一步迭代，恰好需要做 $I (r)$ 次比较 
若共含 $I$ 个逆序对，则 
- 关键码比较次数为 $O (I)$
- 运行时间为 $O (n+I)$ —— [[31-List-Exercise#3-11 逆序对与插入排序分析|习题 3-11：逆序对与插入排序分析]]

### 如何计算逆序对数？

蛮力算法需要 $Ω(n^2)$ 时间；而借助归并排序，仅需 $O (n\log n)$ 时间：
![[30-List-inversion-counts.png]]

## 游标(静态链表)
某些语言不支持指针类型，即便支持，频繁的动态空间分配也影响总体效率。此时，又当如何实现列表结构呢？
- 利用线性数组，以游标方式模拟列表 
	- `elem[]`：对外可见的数据项 
	- `link[]`：数据项之间的引用 
- 维护逻辑上互补的列表 data 和 free 
- 在插入或删除元素时，应如何调整？

![[30-List-cursor.png]]

### 操作实例
![[30-List-cursor-instance1.png]]

![[30-List-cursor-instance2.png]]

习题：[[31-List-Exercise]]
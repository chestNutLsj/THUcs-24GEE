---
url: https://zhuanlan.zhihu.com/p/291206708
---
## 什么是 Top k？

在算法领域，有一个经典的问题，用一句话就可以描述清楚：“从长度为 N 的无序数组中找出前 k 大的数。” 这就是所谓的 Top k 问题。

它之所以经典，是因为问题足够抽象，而其应用又足够广泛，可以作为许多高级算法的基础组成部分。此外，Top k 的实现方式众多，应用不同数据结构可以得到不同复杂度的解法。本文从经典解法开始，一步步引出该问题的终极解法——BFPRT。

## 经典解法
### 排序
从最容易想到的解法开始，我们首先考虑排序。对 N 个数排序，然后取前 k 个，即可达到目标。无论用归并排序、选择排序、快速排序还是其它任何类型的排序算法，这种解法的时间复杂度下界是 $\Omega \left( nlgn \right)$ ，因为这是任何排序算法的最低耗时。

### 大顶堆
接下来，考虑到我们的目标是最大的 k 个元素，对完整的数组排序有点大材小用了。回想之前介绍过的数据结构堆，如果我们把所有 N 个元素建最大堆，此时堆顶元素即为当前堆中最大的元素。取出堆顶元素，把最后一个结点补充到堆顶，维护堆，使之重新成为最大堆，再取出堆顶元素，如此循环往复，取够 k 个元素，即可得到 Top k 的集合。这一方案分为两个阶段，建堆的时间复杂度为 $O \left( n \right)$ ，k 次维护堆的时间复杂度为 $O \left( k lgn \right)$ ，总时间复杂度为 $O \left( n + klgn \right)$ 。

### 小顶堆
这种方法在没有对完整数组排序的情况下，得到了有序的 Top k 个元素。但实际上，该问题并不要求得到的 Top k 个元素有序，只需要知道哪些元素是 Top k 就可以了。在这一视角下，Top k 问题等价于顺序统计量（Order Statistics）问题，即找出无序数组中第 k 大的元素。一旦找出该元素，只需要遍历一遍数组，即可得到 Top k。我们仍然考虑使用堆来解决这一问题。与之前不同的是，这次使用最小堆，并且最小堆的大小为 k。在遍历数组的过程中，令最小堆始终保存当前已遍历的数据中最大的 k 个元素。具体过程也很简单，先从 N 个数据中取出前 k 个建堆，接下来继续遍历数组，每次都将新的元素和堆顶元素比较，如果大于堆顶元素，则说明当前堆中的 k 个元素已经不是最大的 k 个了，因为至少堆顶元素小于新元素。因此，将堆顶元素丢弃，新元素放入堆顶，执行维护堆操作，形成新的最小堆。依次遍历下去，最终得到的 k 个堆元素即为 Top k。这一方案也包括两个阶段，建堆的时间复杂度为 $O \left( k \right)$ ， $n - k$ 次维护堆的时间复杂度为 $O \left( \left( n - k \right) lg k \right)$ ，总时间复杂度为 $O \left( k + \left( n-k \right) lgk \right)$ 。

比较这两种基于堆的方案，可以发现，当 k 远小于 N 时，两者都近似等于 $O \left( n \right)$ 。由于 Top k 算法的时间复杂度下界是 $\Omega \left( n \right)$ （至少要遍历一遍数组），这两种基于堆的方案在 k 比较小时可以说是最佳方案了。不过，我们当然会问，如果 k 不太小呢，比如从无序数组中找出中位数，此时 $k = N / 2$ ，这种情况下基于堆的方案都变成了 $O \left( nlgn \right)$ ，似乎不太理想。

### 快速选取
既然如此，我们需要重新寻找能在任意大小 k 的情况下都有 $O \left( n \right)$ 时间复杂度的算法。为了尽可能节约时间，这种算法一定不会对最大的 k 个数排序，也不会对剩下的数排序，而只是寻找恰好位于分界点处的第 k 大的数。这使我们联想到快速排序算法。快速排序的一趟划分，可以将整个数组分为三部分，小于主元的部分、主元以及大于主元的部分。如果大于主元的部分长度恰好是 k，那么这部分元素就是 Top k。否则，如果大于主元的部分长度大于 k，那么我们就需要在这部分再做一次划分，找到其中的 Top k。如果大于主元的部分长度小于 k，那么这部分属于 Top k，但不完整，假设其长度是 d，那么我们还需要从小于主元的部分中找到 Top k-d。可以发现，这就形成了一个递归算法，而且应用了快速排序中的划分流程，但与快速排序不同的是，快速排序的每次划分需要同时对两侧进行递归调用，而这里只需要对其中的一侧进行递归调用。这一算法称为快速选择算法（Quick Select）。

我们简单分析一下该算法的时间复杂度。在最佳情况下，每次划分都是绝对均匀的划分，于是每次递归都将数组规模降低为之前的一半。假设直到递归到数组规模为 1 时才结束递归，那么整个过程的时间复杂度为 $O \left( \sum_{k = 0}^{lgn}{n/2^k} \right) = O \left( n \right)$ 。在最坏情况下，每次划分都是绝对不均匀的划分，导致每次递归只能将数组规模降低 1 个单位，这种情况的时间复杂度显然是 $O \left( n^2 \right)$ 。在实际中，我们当然期望不要落入最坏情况，解决方案与快速排序一样，随机选取主元即可。可以从概率的层面证明，随机化快速选择算法的期望时间复杂度为 $O \left( n \right)$ 。

## BFPRT
> Top k 的终极解法——BFPRT，也称为 median of medians 算法。它在最坏情况下的时间复杂度仍然是 $O \left( n \right)$ ，不存在退化问题。

BFPRT 这一名称来源于该算法的五位作者的首字母，在维基百科上，该算法被称为 Median of medians，因为中位数在这里起到了至关重要的角色。

在快速选择算法中，我们分析了最佳和最坏情况的时间复杂度。如果每次选择主元，都恰好选中中位数，那么自然就会落入最佳情况。BFPRT 正是利用了这一点，在快速选择算法的基础上，额外增加了计算近似中位数的步骤。

之所以计算近似中位数，是因为计算准确的中位数显然具有和 Top k 问题相同的时间复杂度，得不偿失。我们需要以较小的代价得到一个比较接近中位数的数。

首先，将 N 个数据每五个分为一组，共得到 $N/5$ 组。注意，这里为了方便起见，我们不去考虑余数的问题，那些常数项在分析时间复杂度时并不重要。使用插入排序对每一组排序，找出每一组的中位数，共得到 $N/5$ 个中位数。接下来这步很奇幻，我们递归调用 BFPRT 算法计算这 $N/5$ 个数的中位数。没错，调用我们正在描述的这个算法本身，因为中位数就是第 $N/2$ 大的数，BFPRT 的目标就是计算这个。虽然目前还看不出递归调用到底如何起作用，不过不必担心，递归算法的神奇之处正在于此。假设这个中位数真的计算出来了，那么这个数就可以称为中位数的中位数（Median of medians），也是整个数据的近似中位数。

我们来看看这样得到的近似中位数到底近似到什么程度。将数据五个一组逐列放置，得到如下图所示的排布。其中箭头从较大的元素指向较小的元素。中间白色的元素是所有中位数， $x$ 标记的元素是中位数的中位数。

![[topk problem-illustration.png]]

可以断定，所有能够连通到 $x$ 元素的元素都大于 $x$ ，即图中灰色背景包围的部分。回到我们的目标，如何评估中位数的近似程度呢？如果能够找出有多少比例的元素必然大于中位数，以及多少比例的元素必然小于中位数，就可以得知中位数所处的区间。由于近似中位数的计算对两侧是一视同仁的，所以中位数所处区间的中点一定是真实的中位数位置，那么该区间的范围越小，则估计的近似中位数越准确。

借助上图，我们可以估计出大于 $x$ 的元素至少有几个。图中灰色区域是一个矩形，它的水平边长为 $\frac{N}{5} \cdot\frac{1}{2}$ ，竖直边长为 $3$ ，因此包含的元素数量为 $\frac{3N}{10}$ 。当然，这里计算的时候我们仍然不考虑舍入误差，因为误差是常数规模的，不会随着 $N$ 增大，在计算时间复杂度时并不影响大局。既然大于 $x$ 的元素至少有 $\frac{3N}{10}$ 个，那么同理，小于 $x$ 的元素也至少有 $\frac{3N}{10}$ 个。于是，近似中位数被限制在整个数组的 30%~70% 范围内。

接下来的计算步骤和快速选择算法一样，选取刚才计算出的近似中位数作为主元，递归调用 BFPRT，直到找到 Top k 的数。

现在，大家应该能体会到作者的良苦用心了。BFPRT 算法通过计算中位数的中位数这一手段将主元限制在一个接近中位数的范围内，从而避免最坏情况的发生。

不过，另一个问题随之而来，额外增加了中位数的中位数这一步骤，难道不会使时间复杂度暴增吗？让我们来分析一下。

### 时间复杂度分析

对于一个递归算法，要想分析时间复杂度，首先要写出递归式。记 BFPRT 算法的时间复杂度为 $T\left(n\right)$ 。根据前面的描述，BFPRT 做了两部分工作，先递归调用它自身计算中位数的中位数，再递归调用它自身寻找 Top k。

具体来说，第一步先对 $N/5$ 个数组执行插入排序，找出每一组的中位数，该过程用时 $O\left( n \right)$ 。然后对这 $N/5$ 个数递归调用 BFPRT，找到中位数，该过程用时 $T\left( \frac{n}{5}\right)$ 。

第二步，选取刚才找到的中位数作为主元，执行一次划分过程，用时 $O\left(n\right)$ 。接下来，根据快速选择算法的要求，我们需要判断选择主元的左侧还是右侧进一步递归。显然，选择哪一侧并不重要，重要的是选择的那一侧数据规模的上限是多少。前面刚刚分析过近似中位数所处的区间在 30%~70% 范围内，那么最坏情况下，近似中位数恰好是 30% 或者 70% 分位点，于是左右两侧的数据规模将分别是 $\frac{3N}{10}$ 和 $\frac{7N}{10}$ 。我们当然选择分析最坏情况，认为每次都对 $\frac{7N}{10}$ 的那一侧进行递归，于是该过程用时 $T\left( \frac{7N}{10} \right)$ 。

最后，将这两步的时间复杂度加起来，即可得到 BFPRT 的递归式。

$$T\left(n \right) = T \left( \frac{n}{5} \right) + T\left( \frac{7n}{10} \right) + O\left( n \right) \\
$$

这是一个双分支的递归式，没法用之前介绍过的主方法求解，只能用代入法来证明我们的猜测。假设 $T\left(n\right) \le cn$ ，代入上面的递归式，得

$$ \begin{aligned} T\left (n\right) &= T \left ( \frac{n}{5} \right) + T\left ( \frac{7n}{10} \right) + O\left ( n \right) \\ &\le \frac{cn}{5} + \frac{7cn}{10} + an \\ &= \frac{9cn}{10} + an \\ &= cn + \left ( -\frac{cn}{10} + an \right) \end{aligned} \\ $$

如果最后一行的第二项小于等于 0，上式即可推导出 $T\left(n\right) \le cn$ ，从而猜测成立。 $-\frac{cn}{10} + an \le0$ 等价于 $c \ge 10a$ ，这是可以满足的，因为对于任意的常数 $a$ ，我们都可以找到符合条件的 $c$ 使其成立。

终于，我们证明了 BFPRT 算法的时间复杂度为 $O \left( n\right)$ ，而且是最坏情况的时间复杂度。

### 继续深入

事情到这里还没完，作为一个细心的读者，你可能会好奇，中位数的中位数算法看起来怪怪的，为什么偏偏选择五个数一组呢？三个一组、七个一组不行吗？另外，为什么非要在中位数的中位数中递归调用 BFPRT 呢，这种交叉调用有一种说不出的怪异感。

先来研究第一个问题，五个一组是不是必须的。在算法流程中将五个一组修改成三个一组或七个一组很容易，关键在于它们导致的整体算法的时间复杂度是否仍然是 $O \left( n \right)$ 。如果选择三个一组，重复之前的分析，灰色矩形区域的水平边长为 $\frac{N}{3}\cdot \frac{1}{2}$ ，竖直边长为 $2$ ，因此包含的元素数量为 $\frac{N}{3}$ 。于是，近似中位数所处的区间为 33%~67%，最坏情况下第二步递归调用的规模为 $\frac{2N}{3}$ 。递归式可以表示为 $T\left(n\right) = T\left( \frac{n}{3} \right) + T \left( \frac{2n}{3} \right) + O \left( n \right)$ 。仍然假设 $T\left(n\right) \le cn$ ，带入递归式，得

$$ \begin{aligned} T\left (n\right) &= T\left ( \frac{n}{3} \right) + T \left ( \frac{2n}{3} \right) + O \left ( n \right)\\ &\le \frac{cn}{3} + \frac{2cn}{3} + an \\ &= cn + an \end{aligned} \\ $$

看看我们发现了什么？由于 $an \gt 0$ ，导致无法推出 $T\left(n\right) \le cn$ ，所以 $T\left(n\right) = O\left(n\right)$ 的猜测不成立。也就是说，三个一组不可行。

那七个一组呢？再来一遍同样的分析，可以发现七个一组是可以保证线性时间复杂度的。而且，不光七个一组，只要大于等于五就能达到这个结果。不过，可以想象，对五个一组做插入排序会比对七个一组做插入排序更快，这会体现在常数因子 $a$ 上。因此五个一组是最好的选择。至于为什么不考虑四个一组、六个一组等等，只是因为偶数项的组计算中位数比较麻烦，而且容易造成偏差，所以就不考虑了。

接下来，我们思考一下第二个问题，是否一定要在计算近似中位数的过程中递归调用 BFPRT 呢？说实话，这个问题困扰了我很久。最初我认为根本不需要递归调用 BFPRT，而是不断递归调用中位数子程序就行。之所以这样想，是因为即使递归调用了 BFPRT，它内部还是要先调用中位数子程序，而且我们只是要计算中位数，后面的划分过程根本是不需要的。然而事实证明我错了，这里有个很微妙的地方，如果只是递归调用中位数子程序，它在每次运行时都将当前数据分组，计算每一组的中位数，然后返回新的更小规模的数组，直到数据规模为 1。似乎这样的确找到了我们需要的中位数，但是，这样找到的其实是中位数的近似中位数，而不是中位数的中位数。什么意思呢？标准的 BFPRT 算法要求采用中位数的中位数作为主元，这样才能确保后续的划分将数据规模降低到 $\frac{7N}{10}$ ，对应于上面的图片， $x$ 务必是所有白色元素的真实中位数，而不是近似中位数（虽然 $x$ 是所有元素的近似中位数，但它是白色元素的真实中位数，这一点需要明确），这样才能保证阴影区域的元素大于 $x$ 。然而，单纯递归调用中位数子程序找到的显然是中位数的近似中位数，因为每次递归都在近似，只有在中位数子程序中递归调用 BFPRT，根据 BFPRT 的定义，它找到的才是准确的中位数，才能满足我们的目标。这个问题在算法导论上没有非常明确的指出，从而导致了我以及网上很多人对此不明所以。

BFPRT 的这一编程范式称为互递归（mutual recursion），两个函数分别递归调用另一个函数，它在函数式编程和编译器中有不少应用。笔者也是第一次遇到这个怪物，被它折磨的够呛，期待下次见到一些不同的应用。

### 代码实现

我在网上搜索过一些 BFPRT 算法的实现，很可惜，有不少犯了刚才提到的第二点错误，没有在中位数子程序中调用 BFPRT。这个错误很隐蔽，因为无论怎么写，最终的结果一定是对的，只不过时间复杂度没达到我们的目标，这是无法通过调试发现的。下面是我实现的 BFPRT 算法，流程参考了算法导论和维基百科中的描述。

```
/**
 * The recursive procedure in BFPRT.
 *
 * @param arr The array from which select.
 * @param p   The start index of the sub-array from which select.
 * @param r   The end index of the sub-array from which select.
 * @param i   The order of the element to be selected.
 * @return The selected element.
 */
protected int bfprt(int[] arr, int p, int r, int i) {
	if (p == r) {
		return arr[p];
	}
	int pivot = medianOfMedians(arr, p, r);
	int[] pivotRange = partition(arr, p, r, pivot);
	if (p + i >= pivotRange[0] && p + i <= pivotRange[1]) {
		return arr[pivotRange[0]];
	} else if (p + i < pivotRange[0]) {
		return bfprt(arr, p, pivotRange[0] - 1, i);
	} else {
		return bfprt(arr, pivotRange[1] + 1, r, i + p - pivotRange[1] - 1);
	}
}

/**
 * Compute the median of the medians of the input array.
 *
 * @param arr The array to be computed.
 * @param p   The start index of the sub-array.
 * @param r   The end index of the sub-array.
 * @return The median of the medians of the sub-array.
 */
protected int medianOfMedians(int[] arr, int p, int r) {
	int num = r - p + 1;
	int extra = num % 5 == 0 ? 0 : 1;
	int[] medians = new int[num / 5 + extra];
	for (int i = 0; i < medians. length; i++) {
		medians[i] = computeMedian (arr, p + i * 5, Math.min (p + i * 5 + 4, r));
	}
	return bfprt (medians, 0, medians. length - 1, medians. length / 2);
}

/**
 * Partition the array into two parts.
 * <p>
 * After this method, elements less than pivot are put left, pivots are put middle, others are put right.
 *
 * @param arr The array to be sorted.
 * @param p   The start index of the sub-array to be sorted.
 * @param r   The end index of the sub-array to be sorted.
 * @return A two elements array. The first element indicates the index of the first pivot, the second element
 * indicates the index of the last pivot.
 */
protected int[] partition (int[] arr, int p, int r, int pivot) {
	int small = p - 1;
	int equal = 0;
	int temp;
	for (int j = p; j <= r; j++) {
		if (arr[j] < pivot) {
			small++;
			temp = arr[small];
			arr[small] = arr[j];
			if (equal > 0) {
				arr[j] = arr[small + equal];
				arr[small + equal] = temp;
			} else {
				arr[j] = temp;
			}
		} else if (arr[j] == pivot) {
			equal++;
			temp = arr[j];
			arr[j] = arr[small + equal];
			arr[small + equal] = temp;
		}
	}
	return new int[]{small + 1, small + equal};
}

/**
 * Compute the median of the given array.
 *
 * @param arr   Array to be computed.
 * @param begin The begin index of the range to be computed.
 * @param end   The end index of the range to be computed.
 * @return The median of the array in the specified range.
 */
private int computeMedian (int[] arr, int begin, int end) {
	Arrays.sort (arr, begin, end + 1);
	return arr[begin + (end - begin) / 2];
}
```

这里面有个需要特别注意的地方，`partition`方法与快速排序中的有所区别。在快速排序中，`partition`的主元只有一个，如果出现等于主元的元素， 会把它放在左侧（当然也可以放在右侧）。而这里则是把所有等于主元的元素都放在中间，这样做可以更快地缩小子数组的规模，更快找到目标元素。不过这会导致代码稍微复杂一些，涉及到三个位置的元素互换顺序，需要小心处理。

### 关于实际应用的讨论

虽说 BFPRT 是 Top k 的终极解法，但实际上众多库函数并不直接采用这一算法。原因是其中隐含的常数因子比随机化快速选择算法大很多，而且，随机化快速选择算法在实际中表现很好，因为在随机化算法中，很难出现导致其进入最坏情况的案例。于是，有人提出了 Introselect 算法，该算法是随机化快速选择算法与 BFPRT 的结合。它内部默认采用随机化快速选择算法，同时监控是否出现退化现象，一旦发现退化，立即切换到 BFPRT。显然，Introselect 的核心是如何检测退化，这可以通过检查连续的若干次迭代是否在一定比例上降低了数据规模来实现，因为正常情况下，数据规模应该是成比例下降的，而退化情况下，数据规模则按照等差级数下降。

在实际中，也可以将 BFPRT 替换为我们最初介绍的堆选择算法，当快速选择出现退化时，切换到堆选择算法。当然，这种方案在 $k$ 较小时比较高效。

本文代码的完整版见 [BFPRT.java](https://github.com/jingedawang/Algorithms/blob/master/src/select/BFPRT.java)。欢迎关注我的 GitHub 仓库 **[Algorithms](https://github.com/jingedawang/Algorithms)** 获取更多资料。

## 参考资料

[Median of medians](https://en.wikipedia.org/wiki/Median_of_medians) Wikipedia

[Time Bounds for Selection](http://people.csail.mit.edu/rivest/pubs/BFPRT73.pdf) Blum, Floyd, Pratt, Rivest, Tarjan

[Quickselect](https://en.wikipedia.org/wiki/Quickselect) Wikipedia

[Introselect](https://en.wikipedia.org/wiki/Introselect) Wikipedia
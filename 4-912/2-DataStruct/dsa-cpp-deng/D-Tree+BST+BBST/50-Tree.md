---
publish: 
tags:
  - DSA
  - 邓俊辉
  - Cpp
---
## 树的特征、相关概念定义
1. 半线性：确定某种次序后，具有线性特征；
2. 描述：T=(V; E) ，节点数 n=|V|，边数 e=|E|
3. 有根树：任一节点 $r \in V$ 指定为根，树都称为有根树：
	- ![[50-Tree-subtree.png]]

4. 节点和边的关系：
	1. $r_i$ 是 r 的孩子，$r_i$ 互称兄弟，
	2. r 称为父亲，$d=degree(r)$ 为 r 的 (出)度
	3. 可以证明，$e=\sum\limits_{v\in V}degree(v)=n-1=\Theta(n)$
	4. 指定 $T_i$ 为 T 的第 i 棵子树，$r_i$ 为 r 的第 i 个孩子，则称树 T 为有序树

5. 路径/通路 path：节点集 V 中 k+1 个节点，通过 k 条边相连
	1. 通路 $\pi=\{(v_{0},v_{1}),(v_{1},v_{2}),...,(v_{k-1},v_{k})\}$
	2. 路径长度：所含边数 $|\pi|=k$
	3. 环路 loop/cycle：$v_{k}=v_{0}$
6. 极小连通图：节点之间均有路径，且边数最少的情况
7. 极大无环图：不存在环路，且边数最多的情况

8. 树任一节点都与根存在唯一路径：path (v, r)=path (v)，
	1. 按照 |path (v)|的大小可以将节点作等价类划分——相同深度
	2. 节点 v 的深度 depth (v)=|path (v)|
	3. path (v)上节点，均为 v 的祖先（ancestor）, v 是它们的后代（descendent）
	4. 除自身以外，都是真（proper）祖先/后代
9. 根节点 r 是所有节点的公共祖先，深度为 0  ^d0cd9e
	1. 没有后代的节点称为叶子
	2. 叶子深度的最大者称为树的高度 height (v)=height (subtree (v))
	3. 空树的高度取作-1，
	4. 则 depth (v)+height (v)<=height (T) （==当且仅当 v 是最深路径上的节点时，取等号==）
	5. ![[50-Tree-height-depth-relation.png]]

## 树的 ADT 和表示
### ADT
- root () 根节点
- parent () 父节点
- firstChild () 长子
- nextSibling () 兄弟
- insert (i, e) 将 e 作为第 i 个孩子插入
- remove (i) 删除第 i 个孩子（及其后代）
- traverse () 遍历

### 父亲-孩子表示法
- 父亲表示：节点组织为一个序列，各自记录：data 本身信息、parent 父节点的秩或位置
	- 树根的父亲为其自身：R ~ parent (4) = 4（下面是旧图，按新 ppt 指向自身来）
	- ![[图05-03.多叉树的“父节点”表示法.png]]
	- 性能：
		- 空间占用 O (n)，
		- parent ()为 O (1),
		- root ()为 O (depth (v))
		- firstChild () 无法实现
		- nextSibling () 无法实现

- 孩子表示：同一节点的所有孩子，各成一个序列，各序列的长度，即对应节点的度数：
	- ![[图05-04.多叉树的“孩子节点”表示法.png]]
	- 性能：
		- parent ()无法实现
		- root ()无法实现
		- firstChild ()为 O (1)
		- nextSibling ()为 O (1)

- 因此综合考虑，使用父亲-孩子表示法：
	- ![[图05-05.多叉树的“父节点+孩子节点”表示法.png]]
	- 性能：
		- parent ()为 O (1)
		- root ()为 O (depth)
		- firstChild ()为 O (1)
		- nextSibling ()为 O (1)

### 长子-兄弟表示法
- 每个节点只设置 3 个引用：parent ()、firstChild ()和 nextSibling ()
- ![[图05-06.多叉树的“长子+兄弟”表示法（在(b)中，长子和兄弟指针分别以垂直实线和水平虚线示意）.png]]
- 性能：
	- parent ()为 O (1)
	- root ()为 O (层次遍历中的次序)
	- firstChild ()为 O (1)
	- nextSibling ()为 O (1)
	- O (degree (v)+1)的时间可以遍历所有 v 的孩子

## 二叉树
### 特点与类别
1. Binary Tree 的度数不超过 2，孩子左右区分
2. 多叉树都可以通过长子兄弟表示法转化为二叉树：长子~左孩子，兄弟~右孩子
3. 深度为 k 的节点，至多有 2^k 个

4. n 个节点，高度 h 的二叉树满足：$h+1\le n\le 2^{h+1}-1$
5. 度数、节点数、边数的关系： ^b26f71
	1. 设度数为 0、1、2 的节点各有 $n_0$， $n_1$， $n_2$ 个
	2. 则边数 e=n-1= $n_1+2n_2$，
		-  ![[50-Tree-edge-node-num-relation.png]]
	3. 叶节点数 $n_0$ = $n_2+1$
		1. $n_1$ 数量与 $n_0$ 数量无关，因为 $n_1$ 总可以回退到 $n_2$ 中
		2. h=0 时，$n_0$=1=$n_2$+1，此后 $n_0$ 与 $n_2$ 同步递增；
	4. 节点数 n= $n_0$ + $n_1$ + $n_2$ + 1 = $n_1$ + $2n_2$ + 1 
	5. $n_1$ =0 时，所有节点的度数都是偶数，此时 e=$2n_2$, $n_0$=$n_2+1$=(n+1)/2

5. 满二叉树：$n=2^{k+1}-1$，每一层都是 2^k 个节点 ^c031da
6. 真二叉树：引入 $n_1+2n_0$ 个外部节点，将所有节点度数转换为 2
	- ![[50-Tree-proper-binary-tree.png]]

### ADT: BinNode
```
template <typename T> using BinNodePosi = BinNode<T>*; //节点位置
template <typename T> struct BinNode {
	BinNodePosi<T> parent, lc, rc; //父亲、孩子
	T data; Rank height, npl; Rank size(); //高度、npl、子树规模
	BinNodePosi<T> insertAsLC( T const & ); //作为左孩子插入新节点
	BinNodePosi<T> insertAsRC( T const & ); //作为右孩子插入新节点
	BinNodePosi<T> succ(); //（中序遍历意义下）当前节点的直接后继
	template <typename VST> void travLevel( VST & ); //层次遍历
	template <typename VST> void travPre( VST & ); //先序遍历
	template <typename VST> void travIn( VST & ); //中序遍历
	template <typename VST> void travPost( VST & ); //后序遍历
};
```

![[50-Tree-binnode.png]]

引入新节点：
```
template <typename T>
BinNodePosi<T> BinNode<T>::insertAsLC( T const & e )
	{ return lc = new BinNode( e, this ); }
	
template <typename T>
BinNodePosi<T> BinNode<T>::insertAsRC( T const & e )
	{ return rc = new BinNode( e, this ); }
```
![[50-Tree-insertBinNode.png]]

### ADT: BinTree
```
#include "BinNode.h" //引入二叉树节点类
template <typename T> class BinTree { //二叉树模板类
protected:
   Rank _size; BinNodePosi<T> _root; //规模、根节点
   virtual Rank updateHeight( BinNodePosi<T> x ); //更新节点x的高度
   void updateHeightAbove( BinNodePosi<T> x ); //更新节点x及其祖先的高度
public:
   BinTree() : _size( 0 ), _root( NULL ) {} //构造函数
   ~BinTree() { if ( 0 < _size ) remove( _root ); } //析构函数
   Rank size() const { return _size; } //规模
   bool empty() const { return !_root; } //判空
   BinNodePosi<T> root() const { return _root; } //树根
   BinNodePosi<T> insert( T const& ); //插入根节点
   BinNodePosi<T> insert( T const&, BinNodePosi<T> ); //插入左孩子
   BinNodePosi<T> insert( BinNodePosi<T>, T const& ); //插入右孩子
   BinNodePosi<T> attach( BinTree<T>*&, BinNodePosi<T> ); //接入左子树
   BinNodePosi<T> attach( BinNodePosi<T>, BinTree<T>*& ); //接入右子树
   Rank remove ( BinNodePosi<T> ); //子树删除
   BinTree<T>* secede ( BinNodePosi<T> ); //子树分离
   template <typename VST> //操作器
   void travLevel( VST& visit ) { if ( _root ) _root->travLevel( visit ); } //层次遍历
   template <typename VST> //操作器
   void travPre( VST& visit ) { if ( _root ) _root->travPre( visit ); } //先序遍历
   template <typename VST> //操作器
   void travIn( VST& visit ) { if ( _root ) _root->travIn( visit ); } //中序遍历
   template <typename VST> //操作器
   void travPost( VST& visit ) { if ( _root ) _root->travPost( visit ); } //后序遍历
   template <typename VST> //操作器
   void traverse ( VST& ); //自定义遍历
   bool operator<( BinTree<T> const& t ) //比较器（其余自行补充）
      { return _root && t._root && lt( _root, t._root ); }
   bool operator==( BinTree<T> const& t ) //判等器
      { return _root && t._root && ( _root == t._root ); }
}; //BinTree
```

#### Insert/Attach
```
// Insert node
template <typename T> BinNodePosi<T> BinTree<T>::insert( T const& e )
   { _size = 1; return _root = new BinNode<T>( e ); } //将e当作根节点插入空的二叉树

template <typename T> BinNodePosi<T> BinTree<T>::insert( T const& e, BinNodePosi<T> x )
   { _size++; x->insertAsLC( e ); updateHeightAbove( x ); return x->lc; } // e插入为x的左孩子

template <typename T> BinNodePosi<T> BinTree<T>::insert( BinNodePosi<T> x, T const& e )
   { _size++; x->insertAsRC( e ); updateHeightAbove( x ); return x->rc; } // e插入为x的右孩子
```
![[50-Tree-insert-node.png]]

```
// 插入子树
template <typename T> //将S当作节点x的左子树接入二叉树，S本身置空
BinNodePosi<T> BinTree<T>::attach( BinTree<T>*& S, BinNodePosi<T> x ) { // x->lc == NULL
   if ( x->lc = S->_root ) x->lc->parent = x; //接入
   _size += S->_size; updateHeightAbove( x ); //更新全树规模与x所有祖先的高度
   S->_root = NULL; S->_size = 0; release( S ); S = NULL; return x; //释放原树，返回接入位置
}

template <typename T> //将S当作节点x的右子树接入二叉树，S本身置空
BinNodePosi<T> BinTree<T>::attach( BinNodePosi<T> x, BinTree<T>*& S ) { // x->rc == NULL
   if ( x->rc = S->_root ) x->rc->parent = x; //接入
   _size += S->_size; updateHeightAbove( x ); //更新全树规模与x所有祖先的高度
   S->_root = NULL; S->_size = 0; release( S ); S = NULL; return x; //释放原树，返回接入位置
}
```
![[50-Tree-insert-subtree.png]]

#### Update height
```
// 更新高度
#define stature(p) ((int)((p)?(p)-height:-1)) //空树高度为-1
template <typename T> Rank BinTree<T>::updateHeight( BinNodePosi<T> x ) //更新节点x高度
{ 
	return x->height = 1 + max( stature( x->lc ), stature( x->rc ) ); 
} //具体规则，因树而异，此处为常规二叉树，O(1)

template <typename T> void BinTree<T>::updateHeightAbove( BinNodePosi<T> x ) //更新高度
{ 
	while ( x ) { 
		updateHeight( x );
		x = x->parent; 
	} 
} //从x出发，覆盖历代祖先。可优化
```

#### Remove

```
template <typename T> //删除二叉树中位置x处的节点及其后代，返回被删除节点的数值
Rank BinTree<T>::remove( BinNodePosi<T> x ) { // assert: x为二叉树中的合法位置
   FromParentTo( *x ) = NULL; //切断来自父节点的指针
   updateHeightAbove( x->parent ); //更新祖先高度
   Rank n = removeAt( x ); _size -= n; return n; //删除子树x，更新规模，返回删除节点总数
}
template <typename T> //删除二叉树中位置x处的节点及其后代，返回被删除节点的数值
static Rank removeAt( BinNodePosi<T> x ) { // assert: x为二叉树中的合法位置
   if ( !x ) return 0; //递归基：空树
   Rank n = 1 + removeAt( x->lc ) + removeAt( x->rc ); //递归释放左、右子树
   release( x->data ); release( x ); return n; //释放被摘除节点，并返回删除节点总数
} // release()负责释放复杂结构，与算法无直接关系，具体实现详见代码包
```

#### Split subtree
```
template <typename T> BinTree<T>* BinTree<T>::secede( BinNodePosi<T> x ) {
	FromParentTo( * x ) = NULL; updateHeightAbove( x->parent );
	// 以上与BinTree<T>::remove()一致；以下还需对分离出来的子树重新封装
	BinTree<T> * S = new BinTree<T>; //创建空树
	S->_root = x; x->parent = NULL; //新树以x为根
	S->_size = x->size(); _size -= S->_size; //更新规模
	return S; //返回封装后的子树
}
```

## 遍历
![[50-Tree-traverse.png]]

```
template <typename T>
Rank BinNode<T>::size() { //后代总数,亦为子树规模
	Rank s = 1; //计入本身
	if (lc) s += lc->size(); //递归计入左子树规模
	if (rc) s += rc->size(); //递归计入右子树规模
	return s;
} //懒惰策略，O( n = |size| )
```

^b9085a

### 先序遍历
#### 递归版
```
// recursive version
template <typename T, typename VST> 
void traverse( BinNodePosi<T> x, VST & visit ) {
	if ( ! x ) return;
		visit( x->data );
	traverse( x->lc, visit );
	traverse( x->rc, visit );
} //O(n)

```
- 制约：使用默认的 Call Stack，允许的递归深度有限
- 如何化尾递归为迭代？
![[50-Tree-preorder.png]]

#### 先序遍历特点——藤缠树
![[图05-32.先序遍历过程：先沿左侧通路自顶而下访问沿途节点，再自底而上依次遍历这些节点的右子树.png]]
- 沿着左侧藤，整个遍历过程可分解为：
	- 自上而下访问藤上节点，
	- 再逆转方向，自下而上遍历各右子树，各右子树的遍历彼此递归地、独立地自成子任务；

#### 迭代版
``` hl:17
template <typename T, typename VST>
void travPre_I2( BinNodePosi<T> x, VST & visit ) {
	Stack < BinNodePosi<T> > S; //辅助栈
	while ( true ) { //以右子树为单位，逐批访问节点
		visitAlongVine( x, visit, S ); //访问子树x的藤蔓，各右子树（根）入栈缓冲
		if ( S.empty() ) break; //栈空即退出
		x = S.pop(); //弹出下一右子树（根），作为下一批的起点
	} //#pop = #push = #visit = O(n) = 分摊O(1) ，总数不过O(n)
}

//从当前节点出发，沿左分支不断深入，直至没有左分支的节点；沿途节点遇到后立即访问
template <typename T, typename VST>
static void visitAlongVine
( BinNodePosi<T> x, VST & visit, Stack < BinNodePosi<T> > & S ) { //分摊O(1)
	while ( x ) { //反复地
		visit( x->data ); //访问当前节点
		S.push( x->rc ); //右孩子（右子树）入栈（将来逆序出栈）
		x = x->lc; //沿藤下行
	} //只有右孩子、NULL可能入栈——增加判断以剔除后者，是否值得？——不必
}
```

^871d4f

![[50-Tree-prefix-visit-along-vine.png]]

#### 分析
- 正确性：
	- 无遗漏：
	- 根优先：任一子树中，只有根被访问后才会访问其他节点；
	- 先左后右：同一节点的左子树先于右子树被访问；
- 复杂度 O (n)
	- 每步迭代都有一个节点出栈并被访问；
	- 每个节点入/出栈一次且仅一次
	- 每步迭代只需要 O (1)时间
- 遗憾：
	- 这种用栈消除尾递归的方式难以推广到其它递归形式

### 中序遍历
#### 递归版
```
template <typename T, typename VST>
void traverse( BinNodePosi<T> x, VST & visit ) {
	if ( !x ) return;
	traverse( x->lc, visit );
	visit( x->data );
	traverse( x->rc, visit ); //tail
}
```

时间复杂度：$T (n)=T (a)+O (1)+T (n-a-1)=O (n)$

![[50-Tree-inorder.png]]

#### 中序遍历迭代版的思路

- 右子树的递归遍历是尾递归，但左子树不是；因此不能直接套用先序遍历中使用栈直接消除尾递归的思路；
- 因此==中序遍历的递归转迭代的解决思路是==：找到第一个被访问的节点，将其祖先用栈保存，于是化问题为依次对若干棵右子树的遍历问题（依从最"左"节点向上的次）
- 于是问题关键在于，中序遍历任一二叉树 T 时，首先被访问的节点是哪个？如何找到它？

![[图05-33.中序遍历过程：顺着左侧通路，自底而上依次访问沿途各节点及其右子树.png]]
- 沿着左侧藤，==遍历可自底向上分解为 d+1 步迭代：每访问藤上一个节点，再遍历其右子树==
- 各右子树的遍历彼此独立，递归地自成子任务；

#### 迭代版
```
template <typename T, typename V> 
void travIn_I1( BinNodePosi<T> x, V& visit ) {
	Stack < BinNodePosi<T> > S; //辅助栈
	while ( true ) { //反复地
		goAlongVine( x, S ); //从当前节点出发，逐批入栈
		if ( S.empty() ) break; //直至所有节点处理完毕
		x = S.pop(); //x的左子树或为空，或已遍历（等效于空），故可以
		visit( x->data ); //立即访问之
		x = x->rc; //再转向其右子树（可能为空，留意处理手法）
	}
}

template <typename T> //从当前节点出发，沿左分支不断深入，直至没有左分支
static void goAlongVine(BinNodePosi<T> x,Stack < BinNodePosi<T>> & S){
	while(x){ //当前节点入栈后随即向左侧分支深入，迭代直到无左孩子
		s.push(x);
		x=x->lc;
	}
}
```

![[50-Tree-inorder-recurrence.png]]

#### 分析
1. ==每个节点出栈时，其左子树或不存在，或已完全遍历，而右子树尚未入栈==；
2. 于是每当节点出栈，只需访问它，然后从其右孩子出发继续按先左后右子树的次序访问；
3. `goAlongVine()` 最多需要调用Ω(n)次，单次调用最多需要最多Ω(n)次 push 入栈。纵观整个遍历过程中所有对 `goAlongVine()` 的调用，实质的操作只有辅助栈的 push 和 pop：
	1. 每次调用 `goAlongVine()` 都恰有一次 pop，全程不超过 O (n)次
	2. `goAlongVine()` 过程中尽管 push 次数不定，但累计应于 pop 一个量级；

#### 更多递归实现
```
//travIn_I2
template <typename T, typename VST> //元素类型、操作器
void travIn_I2( BinNodePosi<T> x, VST& visit ) { //二叉树中序遍历算法（迭代版#2）
   Stack<BinNodePosi<T>> S; //辅助栈
   while ( true )
      if ( x ) {
         S.push( x ); //根节点进栈
         x = x->lc; //深入遍历左子树
      } else if ( !S.empty() ) {
         x = S.pop(); //尚未访问的最低祖先节点退栈
         visit( x->data ); //访问该祖先节点
         x = x->rc; //遍历祖先的右子树
      } else
         break; //遍历完成
}
```

```
//travIn_I3
template <typename T, typename VST> //元素类型、操作器
void travIn_I3( BinNodePosi<T> x, VST& visit ) { //二叉树中序遍历算法（迭代版#3，无需辅助栈）
   bool backtrack = false; //前一步是否刚从左子树回溯――省去栈，仅O(1)辅助空间
   while ( true )
      if ( !backtrack && HasLChild( *x ) ) //若有左子树且不是刚刚回溯，则
         x = x->lc; //深入遍历左子树
      else { //否则――无左子树或刚刚回溯（相当于无左子树）
         visit( x->data ); //访问该节点
         if ( HasRChild( *x ) ) { //若其右子树非空，则
            x = x->rc; //深入右子树继续遍历
            backtrack = false; //并关闭回溯标志
         } else { //若右子树空，则
            if ( !( x = x->succ() ) ) break; //回溯（含抵达末节点时的退出返回）
            backtrack = true; //并设置回溯标志
         }
      }
}
```

^22637f

```
//travIn_I4
template <typename T, typename VST> //元素类型、操作器
void travIn_I4( BinNodePosi<T> x, VST& visit ) { //二叉树中序遍历（迭代版#4，无需栈或标志位）
   while ( true )
      if ( HasLChild( *x ) ) //若有左子树，则
         x = x->lc; //深入遍历左子树
      else { //否则
         visit ( x->data ); //访问当前节点，并
         while ( !HasRChild( *x ) ) //不断地在无右分支处
            if ( ! ( x = x->succ() ) ) return; //回溯至直接后继（在没有后继的末节点处，直接退出）
            else visit ( x->data ); //访问新的当前节点
         x = x->rc; //（直至有右分支处）转向非空的右子树
      }
}
```

^de0c39

#### 后继与前驱
![[50-Tree-inorder-succ-pre.png]]
- 直接后继：最靠左的右后代，或最低的左祖先（将节点包含于其左子树中的最低祖先）

```
template <typename T> BinNodePosi<T> BinNode<T>::succ() { //定位节点v的直接后继
   BinNodePosi<T> s = this; //记录后继的临时变量
   if ( rc ) { //若有右孩子，则直接后继必在右子树中，具体地就是
      s = rc; //右子树中
      while ( HasLChild( *s ) ) s = s->lc; //最靠左（最小）的节点
   } else { //否则，直接后继应是“将当前节点包含于其左子树中的最低祖先”，具体地就是
      while ( IsRChild( *s ) ) s = s->parent; //逆向地沿右向分支，不断朝左上方移动
      s = s->parent; //最后再朝右上方移动一步，即抵达直接后继（如果存在）
   }
   return s;
}// 两种情况下运行时间分别为当前节点的高度和深度，总和不超过O(h)
```

^8c941b

### 后序遍历
#### 应用
```
//子树删除
template <typename T> //删除二叉树中位置x处的节点及其后代，返回被删除节点的数值
Rank BinTree<T>::remove( BinNodePosi<T> x ) { // assert: x为二叉树中的合法位置
   FromParentTo( *x ) = NULL; //切断来自父节点的指针
   updateHeightAbove( x->parent ); //更新祖先高度
   Rank n = removeAt( x ); 
   _size -= n;
   return n; //删除子树x，更新规模，返回删除节点总数
}

template <typename T> //删除二叉树中位置x处的节点及其后代，返回被删除节点的数值
static Rank removeAt( BinNodePosi<T> x ) { // assert: x为二叉树中的合法位置
   if ( !x ) return 0; //递归基：空树
   Rank n = 1 + removeAt( x->lc ) + removeAt( x->rc ); //递归释放左、右子树
   release( x->data ); release( x ); return n; //释放被摘除节点，并返回删除节点总数
} // release()负责释放复杂结构，与算法无直接关系，具体实现详见代码包

```

^bdee4c

事实上，之前提到的更新高度的函数 [[50-Tree#Update height|updateHeight]] 、更新节点的后代规模的函数 [[50-Tree#^b9085a|size]] 也是后序遍历。

#### 递归版
```
template <typename T, typename VST>
void traverse( BinNodePosi<T> x, VST & visit ) {
	if ( ! x ) return;
	traverse( x->lc, visit );
	traverse( x->rc, visit );
	visit( x->data );
}

```

- 时间复杂度：$T (n)=T (a)+T (n-a-1)+O (1)=O(n)$

![[50-Tree-postorder-instance.png]]

#### 后序遍历特点
- 左右子树的递归遍历都不是尾递归，因此解决办法是找到第一个被访问的节点，将其祖先及右兄弟用栈保存；
- 于是==后序遍历分解为依次对若干棵右子树遍历的问题，此处次序是沿左侧藤最底部左节点向上的次序==。此时问题的关键是找到首先被访问的节点；
- 从根出发下行，尽可能沿左分支前进，实不得以再沿右分支，左右分支都不存在，才返回；
- ==第一个找到的节点，是每个子树的叶子，并且是其中序遍历次序的最靠左者==；

![[50-Tree-postorder-leftmost-leaf.png]]

#### 迭代版
```
template <typename T, typename V> 
void travPost_I( BinNodePosi<T> x, V & visit ) {
	Stack < BinNodePosi<T> > S; //辅助栈
	if ( x ) S.push( x ); //根节点首先入栈
	while ( ! S.empty() ) { //x始终为当前节点
		if ( S.top() != x->parent ) //若栈顶非x之父（而为右兄），则
			gotoLeftmostLeaf( S ); //在其右兄子树中找到最靠左的叶子
		x = S.pop(); //弹出栈顶（即前一节点之后继）以更新x
		visit( x->data ); //并随即访问之
	}
}

template <typename T> //在以S栈顶节点为根的子树中，找到最高左侧可见叶节点
static void gotoLeftmostLeaf( Stack <BinNodePosi<T>> & S ) { //沿途所遇节点依次入栈
	while ( BinNodePosi<T> x = S.top() ) //自顶而下反复检查栈顶节点
		if ( HasLChild( * x ) ) { //尽可能向左。在此之前
			if ( HasRChild( * x ) ) //若有右孩子，则
				S.push( x->rc ); //优先入栈
			S.push( x->lc ); //然后转向左孩子
		} else //实不得已
			S.push( x->rc ); //才转向右孩子
	S.pop(); //返回之前，弹出栈顶的空节点
}
```

#### 实例
![[50-Tree-postorder-instance-1.png]]

#### 分析
- 正确性：
	- 每个节点出栈后，以其为根的子树已经完全遍历，并且如果其右兄弟存在，则必恰为栈顶；
	- 后续继续遍历子树r
- 效率：
	- 分摊分析与中序遍历类似，时间是 O (n)
	- 空间是 O (height)

#### 应用：表达式树
![[50-Tree-expression-tree.png]]
- 运算符一定是分支节点，操作数一定是叶子节点；
- 上图先是对原运算式添加括号，隔离运算符的优先级；
- 之后将操作数摘出，作为运算树的叶节点，将每对括号与运算符匹配；
- 最后根据深度决定优先级，去除括号，得到的就是 RPN 式，即对表达式树的后序遍历就是正确的运算：

![[50-Tree-expression-tree-rpn.png]]

### 层次遍历
#### 迭代版
```
template <typename T> template <typename VST>
void BinNode<T>::travLevel( VST & visit ) { //二叉树层次遍历
	Queue< BinNodePosi<T> > Q; Q.enqueue( this ); //引入辅助队列，根节点入队
	while ( ! Q.empty() ) { //在队列再次变空之前，反复迭代
		BinNodePosi<T> x = Q.dequeue(); visit( x->data ); //取出队首节点并随即访问
		if ( HasLChild( *x ) ) Q.enqueue( x->lc ); //左孩子入队
		if ( HasRChild( *x ) ) Q.enqueue( x->rc ); //右孩子入队
	}
}
```

#### 实例
![[50-Tree-travLevel-instance.png]]

#### 分析
- 正确性：
	- 实质上是树的广度优先遍历；
		- 每次迭代，入队节点都是出队节点的孩子，深度增加一层；
		- ==任何时刻，队列中各节点按照深度单调排列，并且相邻节点的深度相差不超过 1 层；==
		- 所有节点都迟早会入队，越高、靠左的节点，越早入队；
		- ==每个节点都入队、出队恰好一次，故总时间为 O (n)==
- 先序、中序、后续遍历都是树的深度优先遍历；


#### 完全二叉树
- 特点：
	- 完全二叉树是二叉树的紧凑表示，
	- 叶节点只在于最低两层，且底层叶子均居于次底层叶子的左侧；
	- 除末节点的父亲，其余 (内部)节点都有双子
	- **叶节点数不少于内部节点数，但至多多出一个**
![[50-Tree-complete-tree.png]]

![[50-Tree-complete-tree-instance.png]]

- 完全二叉树中的层次遍历：
	- 前 $\lceil \frac{n}{2}\rceil -1$ 步迭代中，均有右孩子入队；
	- 前 $\lceil \frac{n}{2}\rceil$ 步迭代中，均有左孩子入队；
	- 累计至少 n-1 次入队；
	- 因此辅助队列的规模先增后减，单峰且对称；
	- 最大规模为 $\lceil \frac{n}{2}\rceil$，前 $\lceil \frac{n}{2}\rceil -1$ 次均是出 1 入 2；
	- 最大规模可能出现 2 次
		- 单分支时会出现两次，即上图中考查 j 节点，j 出队时会使 i 入队，队内节点数相当于没变；

[[51-Tree-Exercise#5-18 层次遍历辅助队列容量问题]]

![[50-Tree-full-complete-tree.png]]

## 二叉树重构
### 结论速查
- 先序 or 后序 + 中序 -> 能够确定唯一二叉树
- 先序 + 后序 -> 能够确定层次遍历的序列，但不能确定唯一二叉树
- （先序 + 后序）x 真二叉树 -> 能够确定唯一二叉树
- 后序 + 层次 -> 先序 （反之，先序+层次不能推出后序）
- 层次 + 中序 -> 能够确定唯一二叉树

### 先序 or 后序 + 中序
![[50-Tree-inorder+preorder.png]]
- 先序能够确定节点及其后代，而中序可以给出左右子树的划分；

![[50-Tree-pre-or-post+inorder=uniq_tree.png]]
### 先序 + 后序
![[50-Tree-preorder+postorder.png]]
- 但是后序+先序不能确定、区分左右子树，原因在于左右子树可能为空，这样会导致歧义；
```
pre:  ABC
post: ABC

Tree:
		A
	B
C

or

A
	B
		C
```

![[50-Tree-pre-post??.png]]
- 因此如何消除空子树的歧义性，是从先序和后序中导出确定二叉树的关键所在：

### (先序+后序) x 真二叉树
![[50-Tree-preorder+postorderxtrue-bt.png]]

![[50-Tree-pre+postxtrue=uniq_tree.png]]

### 增强序列
- 假想地认为，每个 NULL 节点也是真实的节点，并在遍历时一并输出，每次递归到 NULL 则返回统一约定的元字符 `^`
- 若将遍历序列表示为一个 Iterator，则可将其定义为 `Vector< BinNode<T> * >`
- 于是在增强的遍历序列中，这类“节点”可统一记作 NULL。可归纳证明：**在增强的先序、后序遍历序列中**
	- 任一子树依然对应于一个子序列，而且
	- 其中的 NULL 节点恰比非 NULL 节点多一个
- 如此，通过对增强序列分而治之，即可重构原树
- 增强序列就成为了真二叉树，此时只需要通过先序和后序就能确定树，之后去掉增强节点即可；

> [! warning] 增强的中序遍历序列并不能保证无歧义
> 如下图所示：
> ![[50-Tree-inorder-augment.png]]

![[50-Tree-augment-sequence.png]]

## Huffman Tree
### 应用情景
文件编码时，字符通过二进制编码，组成数据文件的字符来自一个字符集Σ，字符通过赋值为互异的二进制串。因此随着字符集的增加，二进制串势必越来越长。

文件的大小=字符数量 x 各字符编码时的二进制串的长短。因此如何对各字符编码，使得文件最小？
### PFC 编码
![[50-Tree-pfc.png]]
- 将字符集 Σ 中的字符组织成一棵二叉树，以 0/1 表示左/右孩子，**各字符 x 分别存放于对应的叶子 v (x)中**
- 字符 x 的编码串 $rps(v(x))=rps(x)$ 则由根到叶子的通路唯一确定；
- **字符编码不是等长，且不同字符的编码互不为前缀**（Prefix-Free Code）（因为所有字符都在叶节点，而不是中间节点），故没有歧义；
- 但是缺点是，如果使用频繁的字符放在最底层，意味着这个字符本身占用的“长”二进制串将反复调用，使得文件大小不是最优。
- 考查平均编码长度：$ald(T)=\sum\limits_{x\in\Sigma}\frac{depth(v(x))}{|\Sigma|}$
- 最优编码树 $T_{opt}$ 即为使 $ald()$ 最小者，显然它一定存在且未必唯一。

### PFC 解码
反过来，依据 PFC 编码树可便捷地完成编码串的解码:
1. PFC 编码中的编码树为例，设对编码串"101001100"解码。**从前向后扫描该串，同时在树中相应移动**。
2. 起始时从树根出发，视各比特位的取值相应地向左或右深入下一层，直到抵达叶节点。比如，在扫描过第 1 位 "1"后将抵达叶节点'M'。此时，可以输出其对应的字符'M'，然后重新回到树根，并继续扫描编码串的剩余部分。
3. 再经过接下来的 3 位"010"后将抵达叶节点'A'，同样地输出字符'A' 并回到树根。
4. 如此迭代，即可无歧义地解析出原文中的所有字符。

实际上，这一解码过程甚至可以在二进制编码串的接收过程中实时进行，而不必等到所有比特位都到达之后才开始，因此这类算法属于==在线算法==。

### 最优编码树
叶子只能出现在倒数两层以内——否则，通过节点交换即可降低总的编码长度：
![[50-Tree-PFC-opt.png]]

- $\forall v\in T_{opt}, degree(v)=0\ \iff\ depth(v)\ge depth(T_{opt})-1$ (当且仅当的等价关系)
- 字符的出现概率或频度不尽相同，甚至，往往相差极大... 已知各字符的期望频率，如何构造最优编码树？

### 最优带权编码树
- 首先明确，文件长度 $\propto$ 平均带权深度 $weight_ald()=\sum\limits_{x}rps(x)\times w(x)$
- 如果综合考虑带权情况，完全树未必是最优编码树：
![[50-Tree-wald.png]]

- 因此，通过调整频率不同的字符所在编码树的深度，可以降低 wald (T)——频率高的字符，放在编码树的高处；

### Huffman 树
- Huffman 的贪心策略：频率低的字符优先引入，其位置亦更低：
	- 为每个字符创建一棵单节点的树，组成森林 F
	- 按照出现频率，对所有树排序
	- while ( F 中的树不止一棵 )
		- 取出==频率最小的两棵树==：T1 和 T2
		- 将它们==合并成一棵新树== T，并令：
			- lc (T) = T1 且 rc (T) = T2
			- w( root(T) ) = w( root(T1) ) + w( root(T2) )

- 尽管贪心策略未必总能得到最优解，但非常幸运，如上算法的确能够得到最优带权编码树之一。

![[50-Tree-huffman-pfc.png]]

### Huffman 树的特性
1. 双子性：
	- 每个内部节点都有两个孩子——==真二叉树==
	- 否则将 1 度节点替换成唯一的孩子，得到的新树 wald 更小（注意叶节点才保存字符信息）：
	- ![[50-Tree-huffman-true-tree.png]]

2. 不唯一性：
	- 对任一内部节点而言，左右子树互换后 wald 不变，故 Huffman 算法中兄弟子树的次序若随机选取，则可能出现歧义——不同的最优带权编码树；
	- 为了消除这种歧义，可以==要求左子树的频率不低于右子树==；
	- ![[50-Tree-huffman-!unique.png]]

3. 层次性：
	- 出现频率最低的字符 x 和 y ，必在某棵最优编码树中处于最底层，且互为兄弟；
	- 否则，任取一棵最优编码树，并在其最底层任取一对兄弟 a 和 b，a 和 x 、 b 和 y 交换之后，wald 也不会增加
	- ![[50-Tree-huffman-layer.png]]

### Huffman 树的最优性证明
对字符集|Σ|做数学归纳可证：Huffman 算法所生成的，必是一棵最优编码树！
- |Σ| = 2 时显然正确；
- 设算法在|Σ|<n 时均正确，则设|Σ|=n，取Σ中频率最低的 x、y，并不妨设其互为兄弟：
- 令 $\Sigma^{'}=(\Sigma \\ \{x,y\} \cup \{z\}, w(z)=w(x)+w(y)$，
- ![[50-Tree-huffman-justify.png]]
- 则对 Σ' 的任一编码树 T'，只要为 z 添加孩子 x 和 y 就可以得到Σ的一棵编码树 T，且 $wd(T)-wd(T^{'})=w(x)+w(y)=w(z)$
- 因此可见，如此对应的 T 和 T'，wd 之差与 T 的具体形态无关，因此只要 T'是 Σ' 的最优编码树，则 T 也必然是Σ的最优编码树之一；
- 事实上 Huffman 的每一步迭代，都相当于从最优编码树 T 转入另一棵最优编码树 T'

### Huffman 树的实现
```
#define N_CHAR (0x80 - 0x20) //仅以可打印字符为例

struct HuffChar { //Huffman（超）字符
	char ch; unsigned int weight; //字符、频率
	HuffChar ( char c = '^', unsigned int w = 0 ) : ch ( c ), weight ( w ) {};
	
	bool operator< ( HuffChar const& hc ) { return weight > hc.weight; } //比较器
	bool operator== ( HuffChar const& hc ) { return weight == hc.weight; } //判等器
};

// Huffman(子)树、森林
using HuffTree = BinTree<HuffChar>;

using HuffForest = List<HuffTree*>;

// 构造编码树：反复合并二叉树
HuffTree* generateTree( HuffForest * forest ) { 
	//Huffman编码算法
	while ( 1 < forest->size() ) { //反复迭代，直至森林中仅含一棵树
		HuffTree *T1 = minHChar( forest ), *T2 = minHChar( forest ); 
		HuffTree *S = new HuffTree(); //创建新树，然后合并T1和T2
		S->insert( HuffChar('^', T1->root()->data.weight + T2->root()->data.weight) );
		S->attach( T1, S->root() ); S->attach( S->root(), T2 );
		forest->insertAsLast( S ); //合并之后，重新插回森林
	} //assert: 森林中最终唯一的那棵树，即Huffman编码树
	return forest->first()->data; //故直接返回之
}

// 查找最小(超)字符：遍历List/Vector
HuffTree* minHChar( HuffForest* forest ) { 
	//此版本仅达到O(n)，故整体为O(n^2)
	ListNodePosi<HuffTree*> m = forest->first(); //从首节点出发，遍历所有节点
	for ( ListNodePosi<HuffTree*> p = m->succ; forest->valid( p ); p = p->succ )
		if( m->data->root()->data.weight > p->data->root()->data.weight ) //不断更新
			m = p; //找到最小节点（所对应的Huffman子树）
	return forest->remove( m ); //从森林中取出该子树，并返回
} //Huffman编码的整体效率，直接决定于minHChar()的效率
```

```
// 构造编码表：遍历二叉树
#include "Hashtable.h" //用HashTable实现
using HuffTable = Hashtable< char, char* >; //Huffman编码表
static void generateCT //通过遍历获取各字符的编码
( Bitmap* code, int length, HuffTable* table, BinNodePosi<HuffChar> v ) {
	if ( IsLeaf( * v ) ) //若是叶节点（还有多种方法可以判断）
	{ 
		table->put( v->data.ch, code->bits2string( length ) ); 
		return; 
	}
	if ( HasLChild( * v ) ) //Left = 0，深入遍历
	{ 
		code->clear( length ); 
		generateCT( code, length + 1, table, v->lc ); 
	}
	if ( HasRChild( * v ) ) //Right = 1
	{ 
		code->set( length ); 
		generateCT( code, length + 1, table, v->rc );
	}
} //总体O(n)
```

### Huffman 树构造的改进方案
1. 非升序向量：O (n^2)
	1. 初始化时，通过排序得到一个非升序向量 //O (nlogn) 
	2. 每次（从后端）取出频率最低的两个节点 //O (1)
	3. 将合并得到的新树插入向量，并保持有序 //O (n)
2. 非降序列表：O (n^2)
	1. 初始化时，通过排序得到一个非降序列表 //O (nlogn)
	2. 每次（从前端）取出频率最低的两个节点 //O (1)
	3. 将合并得到的新树插入列表，并保持有序 //O (n)
3. 优先级队列：O (nlogn)
	1. 初始化时，将所有树组织为一个优先级队列（第 12 章）//O (n) 
	2. 取出频率最低的两个节点，合并得到的新树插入队列 //O (logn) + O (logn)
4. 按频率排序后使用栈和队列：O (nlogn)
	1. 所有字符按频率非升序入栈 //O (nlogn)
	2. 维护另一（有序）队列... //O(n)
![[50-Tree-huffman-instance.png]]
![[50-Tree-huffman-improve-instance2.png]]

## 二叉树应用
### 图/树的直径、偏心率、半径、中心
![[50-Tree-app.png]]
- Diameter: 直径是指图中任意两个节点之间的最长路径的长度。
	- 换句话说，它是连接图中任意两个节点的最长边的长度。
	- 直径反映了图或树的整体大小。
- Eccentricity: 对于每个节点来说，其偏心率是指从该节点到图中所有其他节点的最长路径的长度中的最大值。
	- 偏心率衡量了一个节点到图中其他节点的距离，偏心率最小的节点被称为图的中心。
- Radius: 图或树的半径是指所有节点偏心率中的最小值。
	- 换句话说，它是图中所有节点到离它最远的节点的最短距离中的最小值。
	- 半径反映了图或树的紧凑程度，越小表示越紧凑。
- Center: 图或树的中心是指具有最小偏心率的节点集合。
	- 这些节点是离图中其他节点最近的节点，通常是半径的节点。
	- 中心是图或树的"核心"部分。

### 树的半径与BFS
![[50-Tree-diameter.png]]
- BFS 遍历是计算树直径的一种常用方法之一。
- 树的直径是指树中任意两个节点之间的最长路径的长度。通过 BFS 遍历，可以轻松地找到树的直径，以下是一个简单的步骤：
	1. 选择树中的任意节点作为起始节点。
	2. 使用 BFS 从选定的起始节点开始，一层一层地探索树的节点。在每一层中，记录最后一个被访问的节点。
	3. 当 BFS 完成后，最后一个被访问的节点就是树的直径的一个端点。
	4. 重新选择刚才找到的端点作为新的起始节点，再次进行 BFS。
	5. 最后一次 BFS 访问的节点将是树的直径的另一个端点。
	6. 最长路径就是这两个端点之间的路径，其长度即为树的直径。

BFS 遍历保证了从树的一侧到达另一侧，因此，通过上述过程，能够找到树的直径。BFS 的时间复杂度是 O(V + E)，其中 V 是节点数，E 是边数，对于树结构，E = V - 1，因此该方法的时间复杂度为 O(V)。

### 圆桌骑士问题
![[50-Tree-knights-of-round-table.png]]
[365. 圆桌骑士 - AcWing题库](https://www.acwing.com/problem/content/description/367/)
国王有时会在圆桌上召开骑士会议。
由于骑士的数量很多，所以每个骑士都前来参与会议的情况非常少见。
通常只会有一部分骑士前来参与会议，而其余的骑士则忙着在全国各地做英勇事迹。
骑士们都争强好胜，好勇斗狠，经常在会议中大打出手，影响会议的正常进行。
现在已知有若干对骑士之间互相憎恨。

为了会议能够顺利的召开，每次开会都必须满足如下要求：
1. 相互憎恨的两个骑士不能坐在相邻的两个位置。
2. 为了让投票表决议题时都能有结果（不平票），出席会议的骑士数必须是奇数。
3. 参与会议的骑士数量不能只有 1 名。

如果前来参加会议的骑士，不能同时满足以上三个要求，会议会被取消。
如果有某个骑士无法出席任何会议，则国王会为了世界和平把他踢出骑士团。

现在给定骑士总数 n，以及 m 对相互憎恨的关系，求至少要踢掉多少个骑士。

#### 输入格式

输入包含多组测试用例。

对于每个用例，第一行包含两个整数 n 和 m。

接下来 m 行，每行包含两个整数 a 和 b，表示骑士 a 和骑士 b 相互憎恨。

当遇到某行为 `0 0` 时表示输入终止。

#### 输出格式

每个测试用例输出一个整数，表示结果。

每个结果占一行。

#### 数据范围

n≤1000,m≤106

### 旅行骑士问题
![[50-Tree-traveling-knight.png]]
这个问题涉及到象棋中的骑士棋子，目标是找到一种方式，使得骑士按照象棋规则（骑士按“日”字格走），恰好访问棋盘上的每个格子一次，并最终回到起始位置。

这个问题通常被用来考察图论和搜索算法的性能，它涉及到数据结构，特别是用于存储骑士在棋盘上移动的数据结构。通常，这个问题可以使用图或矩阵来建模，其中每个格子是一个节点，骑士的合法移动是边，然后可以使用深度优先搜索（DFS）或其他搜索算法来找到解决方案。

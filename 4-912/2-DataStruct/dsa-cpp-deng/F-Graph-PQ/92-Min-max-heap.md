
| Algorithm | Average      | Worst case |
| --------- | ------------ | ---------- |
| Insert    | O (logn)     | O (logn)   |
| Delete    | O (logn)[^1] | O (logn)   |

## Introduction
In computer science, a min-max heap is a complete binary tree data structure which combines the usefulness of both a min-heap and a max-heap, that is, ==it provides constant time retrieval and logarithmic time removal of both the minimum and maximum elements in it[^2]==.

This makes the min-max heap a very useful data structure to implement a double-ended priority queue. Like binary min-heaps and max-heaps, ==min-max heaps support logarithmic insertion and deletion and can be built in linear time[^2]==. Min-max heaps are often represented implicitly in an array;[^2] hence it's referred to as an implicit data structure.

The **min-max heap property** is: each node at an even level in the tree is less than all of its descendants, while each node at an odd level in the tree is greater than all of its descendants.[^2]

The structure can also be generalized to ==support other order-statistics operations efficiently==, such as `find-median`, `delete-median`,[^2] `find(k)` (determine the *k*th smallest value in the structure) and the operation `delete(k)` (delete the *k*th smallest value in the structure), for any fixed value (or set of values) of k. These last two operations can be implemented in constant and logarithmic time, respectively. 

The notion of min-max ordering can ==be extended to other structures based on the max- or min-ordering==, such as leftist trees, generating a new (and more powerful) class of data structures.[^2] A min-max heap can also be useful when implementing an **external quicksort**.[^3]

## Description
A min-max heap is a *complete binary tree* containing alternating (交替的) min (or even) and max (or odd) levels. Even levels are for example 0, 2, 4, etc, and odd levels are respectively 1, 3, 5, etc. We assume in the next points that the root element is at the first level, i.e., 0.

![[92-Min-max-heap-instance.png]]
- Each node in a min-max heap has a data member (usually called *key*) whose value is used to determine the order of the node in the min-max heap.
- The root element is the smallest element in the min-max heap.
- One of the two elements in the second level, which is a max (or odd) level, is the greatest element in the min-max heap
- Let $x$ be any node in a min-max heap.
	- If $x$ is on a min (or even) level, then ${\displaystyle x.key}$ is the minimum key among all keys in the subtree with root $x$.
	- If $x$ is on a max (or odd) level, then ${\displaystyle x.key}$ is the maximum key among all keys in the subtree with root $x$.
- A node on a min (max) level is called a min (max) node.

A max-min heap is defined analogously; in such a heap, the maximum value is stored at the root, and the smallest value is stored at one of the root's children.[^2]

## Operations
In the following operations we assume that the min-max heap is represented in an array `A[1..N]`; The $ith$ location in the array will correspond to a node located on the level ${\displaystyle \lfloor \log i\rfloor }$ in the heap.

### Build
Creating a min-max heap is accomplished by an adaptation of Floyd's linear-time heap construction algorithm, which proceeds in a bottom-up fashion.[^2] A typical Floyd's build-heap algorithm[^4] goes as follows:

**function FLOYD-BUILD-HEAP**(_h_):
    **for** each index _i_ from ${\displaystyle \lfloor length(h)/2\rfloor }$ **down to** 1 **do:**
        **push-down**(_h_, _i_)
    **return** h

In this function, h is the initial array, whose elements may not be ordered according to the min-max heap property. The `push-down` operation (which sometimes is also called heapify) of a min-max heap is explained next.

### Push Down
The `push-down` algorithm (or trickle-down as it is called in [^2] ) is as follows:

```
function PUSH-DOWN(h, i):
    if i is on a min level then:
        PUSH-DOWN-MIN(h, i)
    else:
        PUSH-DOWN-MAX(h, i)
    endif
```

#### Push Down Min
```
function PUSH-DOWN-MIN(h, i):
    if i has children then:
        m := index of the smallest child or grandchild of i
        if m is a grandchild of i then:
            if h[m] < h[i] then:
                swap h[m] and h[i]
                if h[m] > h[parent(m)] then:
                    swap h[m] and h[parent(m)]
                endif
                PUSH-DOWN(h, m)
            endif
        else if h[m] < h[i] then:
            swap h[m] and h[i]
        endif 
    endif
```

#### Push Down Max
The algorithm for `push-down-max` is identical to that for push-down-min, but with all of the comparison operators reversed.
```
function PUSH-DOWN-MAX(h, i):
    if i has children then:
        m := index of the largest child or grandchild of i
        if m is a grandchild of i then:
            if h[m] > h[i] then:
                swap h[m] and h[i]
                if h[m] < h[parent(m)] then:
                    swap h[m] and h[parent(m)]
                endif
                PUSH-DOWN(h, m)
            endif
        else if h[m] > h[i] then:
            swap h[m] and h[i]
        endif 
    endif
```

#### Iterative Form
As the recursive calls in `push-down-min` and `push-down-max` are in tail position, these functions can be trivially converted to purely iterative forms executing in constant space:
```
function PUSH-DOWN-ITER(h, m):
    while m has children then:
        i := m
        if i is on a min level then:
            m := index of the smallest child or grandchild of i
            if h[m] < h[i] then:
                swap h[m] and h[i]
                if m is a grandchild of i then:
                    if h[m] > h[parent(m)] then:
                        swap h[m] and h[parent(m)]
                    endif
                else
                    break
                endif
            else
                break 
            endif
        else:
            m := index of the largest child or grandchild of i
            if h[m] > h[i] then:
                swap h[m] and h[i]
                if m is a grandchild of i then:
                    if h[m] < h[parent(m)] then:
                        swap h[m] and h[parent(m)]
                    endif
                else
                    break
                endif
            else
                break 
            endif
        endif
    endwhile
```

### Insertion
To add an element to a min-max heap perform following operations:

1. Append the required key to (the end of) the array representing the min-max heap. This will likely break the min-max heap properties, therefore we need to adjust the heap.
2. Compare the new key to its parent:
	1. If it is found to be less (greater) than the parent, then it is surely less (greater) than all other nodes on max (min) levels that are on the path to the root of heap. Now, just check for nodes on min (max) levels.
	2. The path from the new node to the root (considering only min (max) levels) should be in a descending (ascending) order as it was before the insertion. So, we need to make a binary insertion of the new node into this sequence. Technically it is simpler to swap the new node with its parent while the parent is greater (less).

This process is implemented by calling the `push-up` algorithm described below on the index of the newly-appended key.

### Push Up
The push-up algorithm (or bubble-up as it is called in [^2] ) is as follows:
```
function PUSH-UP(h, i):
    if i is not the root then:
        if i is on a min level then:
            if h[i] > h[parent(i)] then:
                swap h[i] and h[parent(i)]
                PUSH-UP-MAX(h, parent(i))
            else:
                PUSH-UP-MIN(h, i)
            endif
        else:
            if h[i] < h[parent(i)] then:
                swap h[i] and h[parent(i)]
                PUSH-UP-MIN(h, parent(i))
            else:
                PUSH-UP-MAX(h, i)
            endif
        endif
    endif
```

#### Push Up Min
```
function PUSH-UP-MIN(h, i):
    if i has a grandparent and h[i] < h[grandparent(i)] then:
        swap h[i] and h[grandparent(i)]
        PUSH-UP-MIN(h, grandparent(i))
    endif
```

#### Push Up Max
As with the `push-down` operations, `push-up-max` is identical to `push-up-min`, but with comparison operators reversed:
```
function PUSH-UP-MAX(h, i):
    if i has a grandparent and h[i] > h[grandparent(i)] then:
        swap h[i] and h[grandparent(i)]
        PUSH-UP-MAX(h, grandparent(i))
    endif
```

#### Iterative Form
As the recursive calls to `push-up-min` and `push-up-max` are in tail position, these functions also can be trivially converted to purely iterative forms executing in constant space:
```
function PUSH-UP-MIN-ITER(h, i):
    while i has a grandparent and h[i] < h[grandparent(i)] then:
        swap h[i] and h[grandparent(i)]
        i := grandparent(i)
    endwhile
```

#### Example
Here is one example for inserting an element to a Min-Max Heap.

Say we have the following min-max heap and want to insert a new node with value 6.

![[92-Min-max-heap-insert-instance.png]]

Initially, node 6 is inserted as a right child of the node 11. 6 is less than 11, therefore it is less than all the nodes on the max levels (41), and we need to check only the min levels (8 and 11). We should swap the nodes 6 and 11 and then swap 6 and 8. So, 6 gets moved to the root position of the heap, the former root 8 gets moved down to replace 11, and 11 becomes a right child of 8.

Consider adding the new node 81 instead of 6. Initially, the node is inserted as a right child of the node 11. 81 is greater than 11, therefore it is greater than any node on any of the min levels (8 and 11). Now, we only need to check the nodes on the max levels (41) and make one swap.

### Find Minimum
The minimum node (or a minimum node in the case of duplicate keys) of a Min-Max Heap is always located at the root. Find Minimum is thus a trivial constant time operation which simply returns the roots.

### Find Maximum
The maximum node (or a maximum node in the case of duplicate keys) of a Min-Max Heap that contains more than one node is always located on the first max level--i.e., as one of the immediate children of the root. Find Maximum thus requires at most one comparison, to determine which of the two children of the root is larger, and as such is also a constant time operation. If the Min-Max heap containd one node then that node is the maximum node.

### Remove Minimum
Removing the minimum is just a special case of removing an arbitrary node whose index in the array is known. In this case, the last element of the array is removed (reducing the length of the array) and used to replace the root, at the head of the array. `push-down` is then called on the root index to restore the heap property in ${\displaystyle O(\log _{2}(n))}$ time.

### Remove Maximum
Removing the maximum is again a special case of removing an arbitrary node with known index. As in the Find Maximum operation, a single comparison is required to identify the maximal child of the root, after which it is replaced with the final element of the array and `push-down` is then called on the index of the replaced maximum to restore the heap property.

## Extensions
The min-max-median heap is a variant of the min-max heap, suggested in the original publication on the structure, that supports the operations of an order statistic tree.

[^1]: Mischel. ["Jim"](https://stackoverflow.com/a/39393768). Stack Overflow. Retrieved 8 September 2016.
[^2]: ATKINSON, M. D; SACK, J.-R; SANTORO, N.; STROTHOTTE, T. (1986). Munro, Ian (ed.). [ "Min-Max Heaps and Generalized Priority Queues"](http://www.akira.ruc.dk/~keld/teaching/algoritmedesign_f03/Artikler/02/Atkinson86.pdf). Communications of the ACM. 29 (10): 996–1000. doi: 10.1145/6617.6621. S2CID 3090797.
[^3]: Gonnet, Gaston H.; Baeza-Yates, Ricardo (1991). Handbook of Algorithms and Data Structures: In Pascal and C. ISBN 0201416077.
[^4]: K. Paparrizos, Ioannis (2011). "A tight bound on the worst-case number of comparisons for Floyd's heap construction algorithm". arXiv: [1012.0956](https://arxiv.org/abs/1012.0956). Bibcode: [2010arXiv1012.0956P](https://ui.adsabs.harvard.edu/abs/2010arXiv1012.0956P).
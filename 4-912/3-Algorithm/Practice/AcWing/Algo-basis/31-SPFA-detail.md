The **Shortest Path Faster Algorithm (SPFA)** is an improved version of the Bellman–Ford algorithm that computes single-source shortest paths in a weighted directed graph. The algorithm works well on sparse random graphs, especially those that contain negative-weight edges. The worst-case complexity of SPFA is the same as that of Bellman–Ford, therefore Dijkstra's algorithm is preferred for graphs with nonnegative edge weights.

> SPFA 是 Bellman-Ford 算法的改进版，在带权有向图中计算单源最短路径表现出色，并且非常适合稀疏图，尤其是在包含负权边的图中。
> SPFA 的最坏时间复杂度与 Bellman-Ford 相同，

SPFA was first published by Edward F. Moore in 1959, as a generalization of breadth first search; SPFA is Moore's “Algorithm D.” The name “Shortest Path Faster Algorithm (SPFA)” was given by Fanding Duan, a Chinese researcher who rediscovered the algorithm in 1994.

## Algorithm

Given a weighted directed graph  $\displaystyle G=(V,E)$ and a source vertex  ![{\displaystyle s}](https://wikimedia.org/api/rest_v1/media/math/render/svg/01d131dfd7673938b947072a13a9744fe997e632), SPFA finds the shortest path from  ![{\displaystyle s}](https://wikimedia.org/api/rest_v1/media/math/render/svg/01d131dfd7673938b947072a13a9744fe997e632) to each vertex  ![{\displaystyle v}](https://wikimedia.org/api/rest_v1/media/math/render/svg/e07b00e7fc0847fbd16391c778d65bc25c452597) in the graph. The length of the shortest path from  ![{\displaystyle s}](https://wikimedia.org/api/rest_v1/media/math/render/svg/01d131dfd7673938b947072a13a9744fe997e632) to  ![{\displaystyle v}](https://wikimedia.org/api/rest_v1/media/math/render/svg/e07b00e7fc0847fbd16391c778d65bc25c452597) is stored in  ![{\displaystyle d(v)}](https://wikimedia.org/api/rest_v1/media/math/render/svg/a957838fc14ceeffef8dc6ba66fad2680cae3656) for each vertex  ![{\displaystyle v}](https://wikimedia.org/api/rest_v1/media/math/render/svg/e07b00e7fc0847fbd16391c778d65bc25c452597).

SPFA is similar to the Bellman-Ford algorithm in that each vertex is used as a candidate to relax its adjacent vertices. However, instead of trying all vertices blindly, SPFA maintains a queue of candidate vertices and adds a vertex to the queue only if that vertex is relaxed, repeating the process until no more vertices can be relaxed.

Below is the pseudo-code of the algorithm. Here  ![{\displaystyle Q}](https://wikimedia.org/api/rest_v1/media/math/render/svg/8752c7023b4b3286800fe3238271bbca681219ed) is a first-in, first-out queue of candidate vertices, and  ![{\displaystyle w(u,v)}](https://wikimedia.org/api/rest_v1/media/math/render/svg/c820425919af804281a4d42c9798951ba8f76140) is the edge weight of  ![{\displaystyle (u,v)}](https://wikimedia.org/api/rest_v1/media/math/render/svg/eadf12294edccd7a29c99cfc1765e4a14bf47e58).

![](//upload.wikimedia.org/wikipedia/commons/thumb/8/8e/SPFADemo.gif/220px-SPFADemo.gif)A demo of SPFA based on Euclidean distance. Red lines are the shortest path covering (so far observed). Blue lines indicate where relaxing happens, i.e., connecting  ![{\displaystyle v}](https://wikimedia.org/api/rest_v1/media/math/render/svg/e07b00e7fc0847fbd16391c778d65bc25c452597) with a node  ![{\displaystyle u}](https://wikimedia.org/api/rest_v1/media/math/render/svg/c3e6bb763d22c20916ed4f0bb6bd49d7470cffd8) in  ![{\displaystyle Q}](https://wikimedia.org/api/rest_v1/media/math/render/svg/8752c7023b4b3286800fe3238271bbca681219ed), which gives a shorter path from the source to  ![{\displaystyle v}](https://wikimedia.org/api/rest_v1/media/math/render/svg/e07b00e7fc0847fbd16391c778d65bc25c452597).

```
 **procedure** Shortest-Path-Faster-Algorithm(_G_, _s_)
  1    **for** each vertex _v_ ≠ _s_ in _V_(_G_)
  2        d(_v_) := ∞
  3    d(_s_) := 0
  4    push _s_ into _Q_
  5    **while** _Q_ is not empty **do**
  6        _u_ := poll _Q_
  7        **for each** edge (_u_, _v_) in _E_(_G_) **do**
  8            **if** d(_u_) + w(_u_, _v_) < d(_v_) **then**
  9                d(_v_) := d(_u_) + w(_u_, _v_)
 10                **if** _v_ is not in _Q_ **then**
 11                    push _v_ into _Q_

```

The algorithm can be applied to an undirected graph by replacing each undirected edge with two directed edges of opposite directions.

Proof of correctness
--------------------

It is possible to prove that the algorithm always finds the shortest path.

_Lemma_: Whenever the queue is checked for emptiness, any vertex currently capable of causing relaxation is in the queue.

_Proof_: To show that if  ![{\displaystyle dist[w]>dist[u]+w (u, w)}]( https://wikimedia.org/api/rest_v1/media/math/render/svg/b61ef4221bdb28de76bf3c11a5926e70e5648b88 ) for any two vertices  ![{\displaystyle u}](https://wikimedia.org/api/rest_v1/media/math/render/svg/c3e6bb763d22c20916ed4f0bb6bd49d7470cffd8) and  ![{\displaystyle w}](https://wikimedia.org/api/rest_v1/media/math/render/svg/88b1e0c8e1be5ebe69d18a8010676fa42d7961e6) at the time the condition is checked,  ![{\displaystyle u}](https://wikimedia.org/api/rest_v1/media/math/render/svg/c3e6bb763d22c20916ed4f0bb6bd49d7470cffd8) is in the queue. We do so by induction on the number of iterations of the loop that have already occurred. First we note that this holds before the loop is entered: İf  ![{\displaystyle u\not =s}](https://wikimedia.org/api/rest_v1/media/math/render/svg/f6878111853f76b3857258b6b06f9a34be53704d), then relaxation is not possible; relaxation is possible from  ![{\displaystyle u=s}](https://wikimedia.org/api/rest_v1/media/math/render/svg/4917c88a09cecda7fbc7d0d0229ff5828dcffbf6) , and this is added to the queue immediately before the while loop is entered. Now, consider what happens inside the loop. A vertex  ![{\displaystyle u}](https://wikimedia.org/api/rest_v1/media/math/render/svg/c3e6bb763d22c20916ed4f0bb6bd49d7470cffd8) is popped, and is used to relax all its neighbors, if possible. Therefore, immediately after that iteration of the loop,  ![{\displaystyle u}](https://wikimedia.org/api/rest_v1/media/math/render/svg/c3e6bb763d22c20916ed4f0bb6bd49d7470cffd8) is not capable of causing any more relaxations (and does not have to be in the queue anymore). However, the relaxation by  ![{\displaystyle x}](https://wikimedia.org/api/rest_v1/media/math/render/svg/87f9e315fd7e2ba406057a97300593c4802b53e4) might cause some other vertices to become capable of causing relaxation. If there exists some vertex  ![{\displaystyle x}](https://wikimedia.org/api/rest_v1/media/math/render/svg/87f9e315fd7e2ba406057a97300593c4802b53e4) such that  ![{\displaystyle dist[x]>dist[w]+w (w, x)}]( https://wikimedia.org/api/rest_v1/media/math/render/svg/4d5e6052f37edf9449d4fa29838ecad2d66fee05 ) before the current loop iteration, then  ![{\displaystyle w}](https://wikimedia.org/api/rest_v1/media/math/render/svg/88b1e0c8e1be5ebe69d18a8010676fa42d7961e6) is already in the queue. If this condition becomes true _during_ the current loop iteration, then either  ![{\displaystyle dist[x]}]( https://wikimedia.org/api/rest_v1/media/math/render/svg/6258d9b0506b6b01cf761ed3835bacb4daf617df ) increased, which is impossible, or  ![{\displaystyle dist[w]}]( https://wikimedia.org/api/rest_v1/media/math/render/svg/570e3acb315feb844890d768257b0d2ad5d48e5d ) decreased, implying that  ![{\displaystyle w}](https://wikimedia.org/api/rest_v1/media/math/render/svg/88b1e0c8e1be5ebe69d18a8010676fa42d7961e6) was relaxed. But after  ![{\displaystyle w}](https://wikimedia.org/api/rest_v1/media/math/render/svg/88b1e0c8e1be5ebe69d18a8010676fa42d7961e6) is relaxed, it is added to the queue if it is not already present.

_Corollary_: The algorithm terminates when and only when no further relaxations are possible.

_Proof_: If no further relaxations are possible, the algorithm continues to remove vertices from the queue, but does not add any more into the queue, because vertices are added only upon successful relaxations. Therefore the queue becomes empty and the algorithm terminates. If any further relaxations are possible, the queue is not empty, and the algorithm continues to run.

The algorithm fails to terminate if negative-weight cycles are reachable from the source. See here for a proof that relaxations are always possible when negative-weight cycles exist. In a graph with no cycles of negative weight, when no more relaxations are possible, the correct shortest paths have been computed (proof). Therefore, in graphs containing no cycles of negative weight, the algorithm will never terminate with incorrect shortest path lengths.

Complexity
----------

Experiments show that the average time complexity of SPFA is  ![{\displaystyle O(|E|)}](https://wikimedia.org/api/rest_v1/media/math/render/svg/976fe7f1e011d0dcdb3d6163754c877aaad5187f) on random graphs, and the worst-case time complexity is  ![{\displaystyle \Omega (|V|\cdot |E|)}](https://wikimedia.org/api/rest_v1/media/math/render/svg/12d61834ec510ea01ec45aa3bc15a990d7fb717d), which is equal to that of Bellman-Ford algorithm.

Optimization techniques
-----------------------

The performance of the algorithm is strongly determined by the order in which candidate vertices are used to relax other vertices. In fact, if  ![{\displaystyle Q}](https://wikimedia.org/api/rest_v1/media/math/render/svg/8752c7023b4b3286800fe3238271bbca681219ed) is a priority queue, then the algorithm resembles Dijkstra's. However, since a priority queue is not used here, two techniques are sometimes employed to improve the quality of the queue, which in turn improves the average-case performance (but not the worst-case performance). Both techniques rearrange the order of elements in  ![{\displaystyle Q}](https://wikimedia.org/api/rest_v1/media/math/render/svg/8752c7023b4b3286800fe3238271bbca681219ed) so that vertices closer to the source are processed first. Therefore, when implementing these techniques,  ![{\displaystyle Q}](https://wikimedia.org/api/rest_v1/media/math/render/svg/8752c7023b4b3286800fe3238271bbca681219ed) is no longer a first-in, first-out (FIFO) queue, but rather a normal doubly linked list or double-ended queue.

**Small Label First** (**SLF**) technique. In line 11, instead of always pushing vertex  ![{\displaystyle v}](https://wikimedia.org/api/rest_v1/media/math/render/svg/e07b00e7fc0847fbd16391c778d65bc25c452597) to the end of the queue, we compare  ![{\displaystyle d(v)}](https://wikimedia.org/api/rest_v1/media/math/render/svg/a957838fc14ceeffef8dc6ba66fad2680cae3656) to  ![{\displaystyle d{\big (}{\text{front}}(Q){\big )}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/603341fc75f8df19597faf42d1b44fc92d443fe8), and insert  ![{\displaystyle v}](https://wikimedia.org/api/rest_v1/media/math/render/svg/e07b00e7fc0847fbd16391c778d65bc25c452597) to the front of the queue if  ![{\displaystyle d(v)}](https://wikimedia.org/api/rest_v1/media/math/render/svg/a957838fc14ceeffef8dc6ba66fad2680cae3656) is smaller. The pseudo-code for this technique is (after pushing  ![{\displaystyle v}](https://wikimedia.org/api/rest_v1/media/math/render/svg/e07b00e7fc0847fbd16391c778d65bc25c452597) to the end of the queue in line 11):

```
**procedure** Small-Label-First(_G_, _Q_)
    **if** d(back(_Q_)) < d(front(_Q_)) **then**
        u := pop back of _Q_
        push u into front of _Q_

```

**Large Label Last** (**LLL**) technique. After line 11, we update the queue so that the first element is smaller than the average, and any element larger than the average is moved to the end of the queue. The pseudo-code is:

```
**procedure** Large-Label-Last(_G_, _Q_)
    _x_ := average of d(_v_) for all _v_ in _Q_
    **while** d(front(_Q_)) > _x_
        _u_ := pop front of _Q_
        push _u_ to back of _Q_

```

References
----------

1.  ^ Jump up to: a b
2.  **^** Pape, U. (1974-12-01). "Implementation and efficiency of Moore-algorithms for the shortest route problem". _Mathematical Programming_. **7** (1): 212–222. doi: 10.1007/BF01585517. ISSN 1436-4646.
3.  **^** Schrijver, Alexander (2012-01-01). "On the history of the shortest path problem". _ems. press_. Retrieved 2023-12-13.
4.  **^** Zhang, Wei; Chen, Hao; Jiang, Chong; Zhu, Lin. "Improvement And Experimental Evaluation Bellman-Ford Algorithm". _Proceedings of the 2013 International Conference on Advanced ICT and Education_.
5.  **^** Moore, Edward F. (1959). "The shortest path through a maze". _Proceedings of the International Symposium on the Theory of Switching_. Harvard University Press. pp. 285–292.
6.  **^** Duan, Fanding (1994). "关于最短路径的 SPFA 快速算法 [About the SPFA algorithm]". _Journal of Southwest Jiaotong University_. **29** (2): 207–212.
7.  **^** "Algorithm Gym :: Graph Algorithms".
8.  **^** "Shortest Path Faster Algorithm". _wcipeg_.
9.  **^** "Worst test case for SPFA". Retrieved 2023-05-14.

*   α–β pruning
*   A*
    *   IDA*
    *   LPA*
    *   SMA*
*   Best-first search
*   Beam search
*   Bidirectional search
*   Breadth-first search
    *   Lexicographic
    *   Parallel
*   B*
*   Depth-first search
    *   Iterative deepening
*   D*
*   Fringe search
*   Jump point search
*   Monte Carlo tree search
*   SSS*

Shortest path

*   Bellman–Ford
*   Dijkstra's
*   Floyd–Warshall
*   Johnson's
*   Shortest path faster
*   Yen's

Minimum spanning tree

*   Borůvka's
*   Kruskal's
*   Prim's
*   Reverse-delete

List of graph search algorithms <table class="nowraplinks hlist mw-collapsible autocollapse navbox-inner mw-made-collapsible mw-collapsed" style="border-spacing:0;background:transparent;color:inherit"><tbody><tr><th scope="col" class="navbox-title" colspan="3"><button type="button" class="mw-collapsible-toggle mw-collapsible-toggle-default mw-collapsible-toggle-collapsed" aria-expanded="false" tabindex="0">show</button><link rel="mw-deduplicated-inline-style" href="mw-data:TemplateStyles:r1129693374"><link rel="mw-deduplicated-inline-style" href="mw-data:TemplateStyles:r1063604349">Optimization: Algorithms, methods, and heuristics</th></tr><tr style="display: none;"><td colspan="2" class="navbox-list navbox-odd" style="width:100%;padding:0"><table class="nowraplinks mw-collapsible mw-collapsed navbox-subgroup mw-made-collapsible" style="border-spacing:0"><tbody><tr><th scope="col" class="navbox-title" colspan="2"><button type="button" class="mw-collapsible-toggle mw-collapsible-toggle-default mw-collapsible-toggle-collapsed" aria-expanded="false" tabindex="0">show</button>Unconstrained nonlinear</th></tr><tr style="display: none;"><td colspan="2" class="navbox-list navbox-odd" style="width:100%;padding:0"><table class="nowraplinks navbox-subgroup" style="border-spacing:0"><tbody><tr><th scope="row" class="navbox-group" style="width:1%">Functions</th><td class="navbox-list-with-group navbox-list navbox-odd" style="width:100%;padding:0"><ul><li>Golden-section search</li><li>Interpolation methods</li><li>Line search</li><li>Nelder–Mead method</li><li>Successive parabolic interpolation</li></ul></td></tr><tr><th scope="row" class="navbox-group" style="width:1%">Gradients</th><td class="navbox-list-with-group navbox-list navbox-odd" style="width:100%;padding:0"><table class="nowraplinks navbox-subgroup" style="border-spacing:0"><tbody><tr><th scope="row" class="navbox-group" style="width:1%">Convergence</th><td class="navbox-list-with-group navbox-list navbox-even" style="width:100%;padding:0"><ul><li>Trust region</li><li>Wolfe conditions</li></ul></td></tr><tr><th scope="row" class="navbox-group" style="width:1%">Quasi–Newton</th><td class="navbox-list-with-group navbox-list navbox-odd" style="width:100%;padding:0"><ul><li>Berndt–Hall–Hall–Hausman</li><li>Broyden–Fletcher–Goldfarb–Shanno and L-BFGS</li><li>Davidon–Fletcher–Powell</li><li>Symmetric rank-one (SR1)</li></ul></td></tr><tr><th scope="row" class="navbox-group" style="width:1%">Other methods</th><td class="navbox-list-with-group navbox-list navbox-even" style="width:100%;padding:0"><ul><li>Conjugate gradient</li><li>Gauss–Newton</li><li>Gradient</li><li>Mirror</li><li>Levenberg–Marquardt</li><li>Powell's dog leg method</li><li>Truncated Newton</li></ul></td></tr></tbody></table></td></tr><tr><th scope="row" class="navbox-group" style="width:1%">Hessians</th><td class="navbox-list-with-group navbox-list navbox-odd" style="width:100%;padding:0"><ul><li>Newton's method</li></ul></td></tr></tbody></table></td></tr></tbody></table></td><td class="noviewer navbox-image" rowspan="5" style="width:1px;padding:0 0 0 2px"><img alt="Graph of a strictly concave quadratic function with unique maximum." src="//upload.wikimedia.org/wikipedia/commons/thumb/7/72/Max_paraboloid.svg/150px-Max_paraboloid.svg.png" decoding="async" width="150" height="120" class="mw-file-element" srcset="//upload.wikimedia.org/wikipedia/commons/thumb/7/72/Max_paraboloid.svg/225px-Max_paraboloid.svg.png 1.5x, //upload.wikimedia.org/wikipedia/commons/thumb/7/72/Max_paraboloid.svg/300px-Max_paraboloid.svg.png 2x" data-file-width="700" data-file-height="560">Optimization computes maxima and minima.</td></tr><tr style="display: none;"><td colspan="2" class="navbox-list navbox-odd" style="width:100%;padding:0"><table class="nowraplinks mw-collapsible mw-collapsed navbox-subgroup mw-made-collapsible" style="border-spacing:0"><tbody><tr><th scope="col" class="navbox-title" colspan="2"><button type="button" class="mw-collapsible-toggle mw-collapsible-toggle-default mw-collapsible-toggle-collapsed" aria-expanded="false" tabindex="0">show</button>Constrained nonlinear</th></tr><tr style="display: none;"><td colspan="2" class="navbox-list navbox-odd" style="width:100%;padding:0"><table class="nowraplinks navbox-subgroup" style="border-spacing:0"><tbody><tr><th scope="row" class="navbox-group" style="width:1%">General</th><td class="navbox-list-with-group navbox-list navbox-odd" style="width:100%;padding:0"><ul><li>Barrier methods</li><li>Penalty methods</li></ul></td></tr><tr><th scope="row" class="navbox-group" style="width:1%">Differentiable</th><td class="navbox-list-with-group navbox-list navbox-even" style="width:100%;padding:0"><ul><li>Augmented Lagrangian methods</li><li>Sequential quadratic programming</li><li>Successive linear programming</li></ul></td></tr></tbody></table></td></tr></tbody></table></td></tr><tr style="display: none;"><td colspan="2" class="navbox-list navbox-odd" style="width:100%;padding:0"><table class="nowraplinks mw-collapsible mw-collapsed navbox-subgroup mw-made-collapsible" style="border-spacing:0"><tbody><tr><th scope="col" class="navbox-title" colspan="2"><button type="button" class="mw-collapsible-toggle mw-collapsible-toggle-default mw-collapsible-toggle-collapsed" aria-expanded="false" tabindex="0">show</button>Convex optimization</th></tr><tr style="display: none;"><td colspan="2" class="navbox-list navbox-odd" style="width:100%;padding:0"><table class="nowraplinks navbox-subgroup" style="border-spacing:0"><tbody><tr><th scope="row" class="navbox-group" style="width:1%">Convex minimization</th><td class="navbox-list-with-group navbox-list navbox-odd" style="width:100%;padding:0"><ul><li>Cutting-plane method</li><li>Reduced gradient (Frank–Wolfe)</li><li>Subgradient method</li></ul></td></tr><tr><th scope="row" class="navbox-group" style="width:1%">Linear and<br>quadratic</th><td class="navbox-list-with-group navbox-list navbox-odd" style="width:100%;padding:0"><table class="nowraplinks navbox-subgroup" style="border-spacing:0"><tbody><tr><th scope="row" class="navbox-group" style="width:1%">Interior point</th><td class="navbox-list-with-group navbox-list navbox-even" style="width:100%;padding:0"><ul><li>Affine scaling</li><li>Ellipsoid algorithm of Khachiyan</li><li>Projective algorithm of Karmarkar</li></ul></td></tr><tr><th scope="row" class="navbox-group" style="width:1%">Basis-exchange</th><td class="navbox-list-with-group navbox-list navbox-odd" style="width:100%;padding:0"><ul><li>Simplex algorithm of Dantzig</li><li>Revised simplex algorithm</li><li>Criss-cross algorithm</li><li>Principal pivoting algorithm of Lemke</li></ul></td></tr></tbody></table></td></tr></tbody></table></td></tr></tbody></table></td></tr><tr style="display: none;"><td colspan="2" class="navbox-list navbox-odd" style="width:100%;padding:0"><table class="nowraplinks mw-collapsible uncollapsed navbox-subgroup mw-made-collapsible" style="border-spacing:0"><tbody><tr><th scope="col" class="navbox-title" colspan="2"><button type="button" class="mw-collapsible-toggle mw-collapsible-toggle-default" aria-expanded="true" tabindex="0">hide</button>Combinatorial</th></tr><tr><td colspan="2" class="navbox-list navbox-odd" style="width:100%;padding:0"><table class="nowraplinks navbox-subgroup" style="border-spacing:0"><tbody><tr><th scope="row" class="navbox-group" style="width:1%">Paradigms</th><td class="navbox-list-with-group navbox-list navbox-odd" style="width:100%;padding:0"><ul><li>Approximation algorithm</li><li>Dynamic programming</li><li>Greedy algorithm</li><li>Integer programming<ul><li>Branch and bound/cut</li></ul></li></ul></td></tr><tr><th scope="row" class="navbox-group" style="width:1%">Graph algorithms</th><td class="navbox-list-with-group navbox-list navbox-odd" style="width:100%;padding:0"><table class="nowraplinks navbox-subgroup" style="border-spacing:0"><tbody><tr><th id="Minimum_spanning_tree" scope="row" class="navbox-group" style="width:1%">Minimum spanning tree</th><td class="navbox-list-with-group navbox-list navbox-even" style="width:100%;padding:0"><ul><li>Borůvka</li><li>Prim</li><li>Kruskal</li></ul></td></tr></tbody></table><table class="nowraplinks navbox-subgroup" style="border-spacing:0"><tbody><tr><th id="Shortest_path" scope="row" class="navbox-group" style="width:1%">Shortest path</th><td class="navbox-list-with-group navbox-list navbox-odd" style="width:100%;padding:0"><ul><li>Bellman–Ford<ul><li>SPFA</li></ul></li><li>Dijkstra</li><li>Floyd–Warshall</li></ul></td></tr></tbody></table></td></tr><tr><th scope="row" class="navbox-group" style="width:1%">Network flows</th><td class="navbox-list-with-group navbox-list navbox-even" style="width:100%;padding:0"><ul><li>Dinic</li><li>Edmonds–Karp</li><li>Ford–Fulkerson</li><li>Push–relabel maximum flow</li></ul></td></tr></tbody></table></td></tr></tbody></table></td></tr><tr style="display: none;"><td colspan="2" class="navbox-list navbox-odd" style="width:100%;padding:0"><table class="nowraplinks mw-collapsible mw-collapsed navbox-subgroup mw-made-collapsible" style="border-spacing:0"><tbody><tr><th scope="col" class="navbox-title" colspan="2"><button type="button" class="mw-collapsible-toggle mw-collapsible-toggle-default mw-collapsible-toggle-collapsed" aria-expanded="false" tabindex="0">show</button>Metaheuristics</th></tr><tr style="display: none;"><td colspan="2" class="navbox-list navbox-odd" style="width:100%;padding:0"><ul><li>Evolutionary algorithm</li><li>Hill climbing</li><li>Local search</li><li>Parallel metaheuristics</li><li>Simulated annealing</li><li>Spiral optimization algorithm</li><li>Tabu search</li></ul></td></tr></tbody></table></td></tr><tr style="display: none;"><td class="navbox-abovebelow" colspan="3"><ul><li>Software</li></ul></td></tr></tbody></table>
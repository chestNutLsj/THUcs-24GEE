---
publish:
---
Intuitively, the assertion "_f_(_x_) is _o_(_g_(_x_))" (read "_f_(_x_) is little-o of _g_(_x_)") means that _g_(_x_) grows much faster than _f_(_x_).

As before, let _f_ be a real or complex valued function and _g_ a real valued function, both defined on some unbounded subset of the positive real numbers, such that _g_(_x_) is strictly positive for all large enough values of _x_. 

One writes $\displaystyle f(x)=o(g(x))\quad {\text{ as }}x\to \infty$ , if for every positive constant ε there exists a constant $x_{0}$ such that ${\displaystyle |f(x)|\leq \varepsilon g(x)\quad {\text{ for all }}x\geq x_{0}.}$ [^1]

For example, one has ${\displaystyle 2x=o(x^{2})}$ and ${\displaystyle 1/x=o(1),}$ both as ${\displaystyle x\to \infty .}$

The difference between the definition of the big-O notation and the definition of little-o is that ==while the former has to be true for _at least one_ constant _M_, the latter must hold for _every_ positive constant _ε_, however small==[^2].

In this way, little-o notation makes a _stronger statement_ than the corresponding big-O notation: ==every function that is little-o of _g_ is also big-O of _g_, but not every function that is big-O of _g_ is also little-o of _g_==. For example, ${\displaystyle 2x^{2}=O(x^{2})}$ but ${\displaystyle 2x^{2}\neq o(x^{2})}$.

As _g_(_x_) is nonzero, or at least becomes nonzero beyond a certain point, the relation ${\displaystyle f(x)=o(g(x))}$ is equivalent to
${\displaystyle \lim _{x\to \infty }{\frac {f(x)}{g(x)}}=0}$ (and this is in fact how Landau originally defined the little-o notation).

Little-o respects a number of arithmetic operations. For example,
- if c is a nonzero constant and ${\displaystyle f=o(g)}$ then ${\displaystyle c\cdot f=o(g)}$, and
- if ${\displaystyle f=o(F)}$ and ${\displaystyle g=o(G)}$ then ${\displaystyle f\cdot g=o(F\cdot G).}$

It also satisfies a transitivity relation:
- if ${\displaystyle f=o(g)}$ and ${\displaystyle g=o(h)}$ then ${\displaystyle f=o(h).}$

[^1]: Landau, Edmund (1909). [Handbuch der Lehre von der Verteilung der Primzahlen](https://archive.org/stream/handbuchderlehre01landuoft#page/61/mode/2up) [Handbook on the theory of the distribution of the primes] (in German). Leipzig: B. G. Teubner. p. 61.
[^2]: Thomas H. Cormen et al., 2001, [Introduction to Algorithms, Second Edition, Ch. 3.1](http://highered.mcgraw-hill.com/sites/0070131511/) Archived 2009-01-16 at the Wayback Machine
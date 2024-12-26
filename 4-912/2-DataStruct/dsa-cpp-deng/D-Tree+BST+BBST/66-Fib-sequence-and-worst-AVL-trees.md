---
url: http://www.geekyarticles.com/2017/04/fibonacci-sequence-and-worst-avl-trees.html
---
## Fibonocci numbers

We all know about Fibonacci sequence. It is the famous sequence that starts with 1 and 1 as the first two elements. From there on, every element is the sum of the previous elements. So, it goes like 1,1,2,3,5,8,13,21,... We can call the $n^{th}$ element this series $F_n$. So, we have $F_1=1$, $F_2=1$ and $F_n = F_{n-1} + F_{n-2}$. 

Fibonooci numbers are very closely related to AVL trees, a kind of self-balancing binary search tree. We will explore this relation in this blog article and then establish the worst case search complexity of an AVL tree.

## A formula for Fibonacci numbers

Our problem is to find out any arbitrary element $F_{n}$ given $n$. Sure, one can just start from the beginning and keep computing each element until $F_n$ is reached, but this probably is not the most efficient method of computing $F_n$. It will be nice to have a formula for $F_n$. This is what we are going to find out.

We have $F_n = F_{n-1} + F_{n-2}$, but here $F_n$ is dependent on two different elements of the sequence. Our first step would be to find a sequence where each element is only dependent on just the previous element and still somehow lets us compute $F_n$. To do this, we first stack a Fibonacci sequence on top of another, shifted by one.

$$
\begin{array}{c c} &1, &1, &2, &3, &5, &8, &13, &21, ... \\ &0, &1, &1, &2, &3, &5, &8, &13, ... \end{array}
$$

Now, you can consider the the stacked thing is a sequence in which each column is an element. So, we have the following sequence

$$
\begin{bmatrix} 1 \\ 0 \end{bmatrix}, \begin{bmatrix} 1 \\ 1 \end{bmatrix}, \begin{bmatrix} 2 \\ 1 \end{bmatrix}, \begin{bmatrix} 3 \\ 2 \end{bmatrix}, \begin{bmatrix} 5 \\ 3 \end{bmatrix}, \begin{bmatrix} 8 \\ 5 \end{bmatrix}, \begin{bmatrix} 13 \\ 8 \end{bmatrix}, \begin{bmatrix} 21 \\ 13 \end{bmatrix},...
$$

This is of course a sequence of vectors. each of them of the form  $\begin{bmatrix} F_{n} \\ F_{n-1} \end{bmatrix}$. The previous term of this would be $\begin{bmatrix} F_{n-1} \\ F_{n-2} \end{bmatrix}$. Now we have a series for which we can compute the next term purely based on the previous term. The top component of the next term is the sum of the components in the previous term, and the bottom component of the next term is just the top component of the previous term. Or, written in matrix multiplication form,

$$
\begin{bmatrix} F_{n} \\ F_{n-1} \end{bmatrix} = \begin{bmatrix} 1 &1 \\ 1 &0 \end{bmatrix} \begin{bmatrix} F_{n-1} \\ F_{n-2} \end{bmatrix}
$$

Verify that the relation is correct. That is, if we actually do the multiplication, we will have, $\begin{bmatrix} F_{n} \\ F_{n-1} \end{bmatrix} = \begin{bmatrix} F_{n-1} + F_{n-2} \\ F_{n-1} \end{bmatrix}$, which is correct. Okay, so this turns out to be a geometric series in linear algebra, and that means, we can write,

$$
\begin{bmatrix} F_{n} \\ F_{n-1} \end{bmatrix} = \begin{bmatrix} 1 &1 \\ 1 &0 \end{bmatrix}^{n-1} \begin{bmatrix} F_1 \\ F_0 \end{bmatrix} => \begin{bmatrix} F_{n} \\ F_{n-1} \end{bmatrix} = \begin{bmatrix} 1 &1 \\ 1 &0 \end{bmatrix}^{n-1} \begin{bmatrix} 1 \\ 0 \end{bmatrix}
$$

$F_0$ is 0. This is easy to verify, because $F_0 + F_1 = F_2$ by definition, and hence, $F_0 + 1 = 1 \Rightarrow F_0 = 0$. This is sort of a formula, although in the matrix multiplication form. At least we can now use the exponentiation by squaring algorithm to compute the value of $F_n$ in $O(\log n)$ time instead of $O(n)$ time required for the naive computation algorithm. But, if we want a real formula, we will have to use eigenvectors.

## Eigenvectors
> 特征向量

For any square matrix, an eigenvector is defined as a column vector, that when multiplied by the matrix does not change direction, or changes direction to exactly opposite. Say we have a square matrix $A$, then an eigenvector $X$ is a non-zero vector such that $AX = \lambda X$ where $\lambda$ is a scalar called the eigenvalue. Eigenvectors don't have to exist for a matrix, but if they do, they will have this property. One thing to note here is that given any eigenvector $X$ of a matrix, then for any non-zero scalar $a$, $aX$ is also an eigenvector. It is easy to see why. $A (aX) = aAX = a\lambda X = \lambda (aX)$. Now, we have the following,

$$\begin{align*} & AX = \lambda X \\ \Rightarrow & AX - \lambda X = 0 \\ \Rightarrow & AX - \lambda IX = 0 \\ \Rightarrow & (A - \lambda I) X = 0 \\ \end{align*}$$

Now, I claim that this means that $det (A - \lambda I) = 0$. Why is this the case? There are many ways to think of it, the one I like is as follows. Say we write $(A - \lambda I)$ in terms of its row vectors $(A - \lambda I) = \begin{bmatrix} r_1 \\ r_2 \\ r_3\\ \vdots\\ r_d \end{bmatrix}$ where d is the number of dimensions.


So, $(A-\lambda I) X = \begin{bmatrix} r_1 X\\ r_2 X\\ r_3 X\\ \vdots\\ r_d X \end{bmatrix}$. That is, each component of the product vector is the dot product of the row vector and the vector $X$. Now, we are saying that each of these dot products is zero, because the result vector is supposed to be zero. When is a dot product zero? Either one of the vectors is zero or the vectors are perpendicular to each other. Now, we already know $X$ cannot be zero, that was our condition for it being an eigenvector. So, if $r_1$ is zero, the $det (A - \lambda I)$ has to be zero. Same goes for all rows. Now, if none of the rows are zero, they must be all perpendicular to $X$. If they are all perpendicular to $X$, they must all lie in the hyperplane perpendicular to $X$. The hyperplane is a $d-1$ dimensional space. So, since there are $d$ row vectors squeezed into a $d-1$ dimensional space, they must be linearly dependent. This means the determinant $det (A - \lambda I)$ is zero. So, we have the condition
$$\begin{align*} det (A - \lambda I) = 0 \end{align*}$$

This is called the characteristic equation for the eigenvalues. We can solve the characteristic equation to obtain the eigenvalues, from which we can find out the eigenvectors.

## Eigenspaces
> 特征空间

The eigenspace of a square matrix is the space spanned by the eigenvectors, that is to say that it the set of all vectors that can be expressed as the linear combination of the eigenvectors. The eigenspace is always a subspace of the n-dimentional space where n is the number of rows and columns of the square matrix. If a square matrix with n rows have n linearly independent eigenvectors, then the eigenspace is the same as the full n-dimentional space. This means, any vector that the matrix can operate on (that is multiplied by) can be expressed as a linear combination of the eigenvectors. This can come very handy, as we shall see in the next section.

## Back to Fibonacci numbers

Now, back to our Fibonacci sequence. We have the matrix $A=\begin{bmatrix} 1 &1 \\ 1 &0 \end{bmatrix}$. We are raising it to a power $n-1$. If $X$ is an eigenvector, then $A^{n-1}X = \lambda^{n-1}X$. So, it may make sense to find the eigenvectors and eigenvalues. The characteristic equation is as follows
$$\begin{align*} &det\left (\begin{bmatrix} 1 &1 \\ 1 &0 \end{bmatrix} - \lambda \begin{bmatrix} 1 &0 \\ 0 &1 \end{bmatrix}\right) = 0\\ \Rightarrow & det\left (\begin{bmatrix} 1-\lambda &1 \\ 1 &-\lambda \end{bmatrix}\right) = 0\\ \Rightarrow & -\lambda (1-\lambda) - 1 = 0\\ \Rightarrow & \lambda^2 -\lambda -1=0\\ \Rightarrow & \lambda = \frac{1 \pm \sqrt{5}}{2} \end{align*}$$

Good! Looks like we have found two eigenvalues. We will call them $\lambda_1$ and $\lambda_2$. $\lambda_1 = \frac{1 + \sqrt{5}}{2}$ and $\lambda_2 = \frac{1 - \sqrt{5}}{2}$. Let us find the corresponding eigenvectors $X_1$ and $X_2$. If we take $X = \begin{bmatrix} x_{1} \\ x_{2} \end{bmatrix}$ 

We have
$$\begin{align*} &AX = \lambda X\\ \Rightarrow & \begin{bmatrix} 1 &1 \\ 1 &0 \end{bmatrix} \begin{bmatrix} x_{1} \\ x_{2} \end{bmatrix} = \lambda \begin{bmatrix} x_{1} \\ x_{2} \end{bmatrix}\\ \Rightarrow & \begin{bmatrix} x_{1} + x_{2}\\ x_{1} \end{bmatrix} = \lambda \begin{bmatrix} x_{1} \\ x_{2} \end{bmatrix}\\ \end{align*}$$

So, we have $x_{1} + x_{2} = \lambda x_{1}$ and $x_{1} = \lambda x_{2}$. We can use any of these equations. Now, since we can scale the eigenvectors, we take $x_2=1$. Hence $x_1=\lambda$. So, for each $\lambda$ of our eigenvalues, we have an eigenvector $\begin{bmatrix} \lambda \\ 1 \end{bmatrix}$. In other words, our eigenvectors are $\begin{bmatrix} \lambda_1 \\ 1 \end{bmatrix}$ and $\begin{bmatrix} \lambda_2 \\ 1 \end{bmatrix}$. One can easily check that the other equation works as well.

Okay, so we found out the eigenvectors, but our initial vector is not one of them. So, how does that help? Simple, we try to represent our initial vector $\begin{bmatrix} 1 \\ 0 \end{bmatrix}$ as a linear combination of the two eigenvectors. So we solve the equations,
$$\begin{align*} c_1\begin{bmatrix} \lambda_{1} \\ 1 \end{bmatrix} + c_2\begin{bmatrix} \lambda_{2} \\ 1 \end{bmatrix} = \begin{bmatrix} 1 \\ 0 \end{bmatrix} \end{align*}$$

From the bottom part, we easily see $c_1 = -c_2$. So, we solve,
$$\begin{align*} &c_1 \lambda_1 - c_1 \lambda_2 = 1\\ \Rightarrow & c_1 = \frac{1}{\lambda_1 - \lambda_2}\\ \Rightarrow & c_2 = \frac{1}{\lambda_2 - \lambda_1} \end{align*}$$

Now, using the values of the eigenvalues we have calculated, we can derive that, $\frac{1}{\lambda_2 - \lambda_1} = \sqrt{5}$. Now, our initial equation was
$$\begin{align*} &\begin{bmatrix} F_{n} \\ F_{n-1} \end{bmatrix} = A^{n-1} \begin{bmatrix} 1 \\ 0 \end{bmatrix}\\ \Rightarrow &\begin{bmatrix} F_{n} \\ F_{n-1} \end{bmatrix} = A^{n-1} \left ( c_1\begin{bmatrix} \lambda_{1} \\ 1 \end{bmatrix} + c_2\begin{bmatrix} \lambda_{2} \\ 1 \end{bmatrix}\right)\\ \Rightarrow &\begin{bmatrix} F_{n} \\ F_{n-1} \end{bmatrix} = \left ( c_1A^{n-1}\begin{bmatrix} \lambda_{1} \\ 1 \end{bmatrix} + c_2A^{n-1}\begin{bmatrix} \lambda_{2} \\ 1 \end{bmatrix}\right)\\ \Rightarrow &\begin{bmatrix} F_{n} \\ F_{n-1} \end{bmatrix} = \left ( c_1\lambda_1^{n-1}\begin{bmatrix} \lambda_{1} \\ 1 \end{bmatrix} + c_2\lambda_2^{n-1}\begin{bmatrix} \lambda_{2} \\ 1 \end{bmatrix}\right)\\ \Rightarrow &\begin{bmatrix} F_{n} \\ F_{n-1} \end{bmatrix} = \left ( \begin{bmatrix} c_1\lambda_1^{n} \\ c_1\lambda_1^{n-1} \end{bmatrix} + \begin{bmatrix} c_2\lambda_2^{n} \\ c_2\lambda_2^{n-1} \end{bmatrix}\right)\\ \Rightarrow &F_n = c_1 \lambda_1^n + c_2 \lambda_2^n\\ \Rightarrow &F_n = \frac{1}{\sqrt{5}} \left (\frac{1+\sqrt{5}}{2}\right)^n - \frac{1}{\sqrt{5}} \left (\frac{1-\sqrt{5}}{2}\right)^n\\ \Rightarrow &F_n = \frac{1}{\sqrt{5}} \left\{\left (\frac{1+\sqrt{5}}{2}\right)^n - \left (\frac{1-\sqrt{5}}{2}\right)^n\right\} \end{align*}$$

This is an exact formula for Fibonacci numbers. It involves a lot of square roots, but for every integer $n$, they always cancel out and give and integer result equal to the corresponding Fibonacci number.

## The Golden Ratio

The golden ratio is the ratio, written as $\phi$ between two consecutive Fibonacci numbers. Well, not really. It is the limit of the ratio between two consecutive Fibonacci numbers as $n$ tends to infinity. That is to say,
$$\begin{align*} &\phi = \lim_{n\to\infty} \frac{F_n}{F_{n-1}}\\ =& \lim_{n\to\infty} \frac{\frac{1}{\sqrt{5}} \left\{\left (\frac{1+\sqrt{5}}{2}\right)^n - \left (\frac{1-\sqrt{5}}{2}\right)^n\right\}}{\frac{1}{\sqrt{5}} \left\{\left (\frac{1+\sqrt{5}}{2}\right)^{n-1} - \left (\frac{1-\sqrt{5}}{2}\right)^{n-1}\right\}}\\ =& \lim_{n\to\infty} \frac{ \left\{\left (\frac{1+\sqrt{5}}{2}\right)^n - \left (\frac{1-\sqrt{5}}{2}\right)^n\right\}}{ \left\{\left (\frac{1+\sqrt{5}}{2}\right)^{n-1} - \left (\frac{1-\sqrt{5}}{2}\right)^{n-1}\right\}} \end{align*}$$

Dividing both numerator and the denominator by $\left (\frac{1-\sqrt{5}}{2}\right)^{n}$, we get,
$$\begin{align*} &\phi = \lim_{n\to\infty} \frac{\left (\frac{1+\sqrt{5}}{1-\sqrt{5}}\right)^n - 1}{\left (\frac{1-\sqrt{5}}{2}\right)^{-1}\left\{\left (\frac{1+\sqrt{5}}{1-\sqrt{5}}\right)^{n-1} - 1\right\}}\\ = &\lim_{n\to\infty} \frac{\left (\frac{1+\sqrt{5}}{1-\sqrt{5}}\right)^n }{\left (\frac{1-\sqrt{5}}{2}\right)^{-1}\left\{\left (\frac{1+\sqrt{5}}{1-\sqrt{5}}\right)^{n-1}\right\}}\\ = &\lim_{n\to\infty} \frac{1+\sqrt{5}}{1-\sqrt{5}} \left (\frac{1-\sqrt{5}}{2}\right)\\ = &\frac{1+\sqrt{5}}{2} \end{align*}$$

## Generalizing the Fibonacci sequence

We can generalize the Fibonacci sequence in many ways. The one we are interested in is when any term is a linear combination of a fixed number of previous terms plus a constant. So, it is of the form $T_n = aT_{n-1} + cT_{n-2} + \ldots + sT_{n-p} + t$ where $a, b, c,\ldots, s$ and $p$ are constants. In all these cases, we can similarly form matrix equations just like the ones we have seen in case of Fibonacci sequence. The constant cannot be handled simply, but we add a constant component 1 to our initial column vector, and we can easily find a matrix from that. As an example, we will see the worst AVL tree.

## The worst AVL tree

AVL trees are self balancing binary search trees, that put some constraint on how much unbalanced the tree can be. I am not going to introduce AVL trees here, it can be looked up elsewhere. The property I am interested in is that the difference in the height of the left and right sub-trees of any sub-tree rooted at any node of the tree can be maximum 1. Given this condition, we are interested in the most unbalanced tree, that is the tree with the maximum height for a given number of nodes. Stated in another way, the worst AVL tree is the AVL tree with the minimum number of nodes for a given height $h$.

![[66-Fib-sequence-and-worst-AVL-trees-worst-avl.png]]

Suppose, the minimum number of nodes required for the tree to have a height $h$ is $f_h$. How to create a worst tree of height $h$ ? There has to be a root and one of it's sub-trees must have a height $h-1$. This height combined with 1 for the root, will create the height of $h$. The other sub-tree cannot differ by more than 1, so it must have a height $h-2$. Both of these sub-trees should have minimum number of nodes allowed for their heights. This is shown in Fig. 1. So, the two sub-trees will have $f_{h-1}$ and $f_{h-2}$ elements. And root adds another node. So,
$$\begin{align*} f_h = f_{h-1}+f_{h-2}+1 \end{align*}$$

We also have $f_1 = 1$ and $f_2 = 2$. This also gives us $f_0 = 0$. This formula looks almost like a Fibonacci sequence except for the plus one. We can stack up the values just as before, except we also write a row of just ones. The dummy row of ones is there so that we can add the constant 1 in a matrix transformation.
$$\begin{array}{c c} &1, &2, &4, &7, &12, &20, &33, &54, ... \\ &0, &1, &2, &4, &7, &12, &20, &33, ... \\ &1, &1, &1, &1, &1, &1, &1, &1, ... \\ \end{array}$$

Similar to the Fibonacci sequence, we can write a matrix formula for this ,
$$\begin{align*} &\begin{bmatrix} &f_{h} \\ &f_{h-1}\\ &1 \end{bmatrix} = \begin{bmatrix} &1 &1 &1 \\ &1 &0 &0 \\ &0 &0 &1 \end{bmatrix} \begin{bmatrix} &f_{h-1} \\ &f_{h-2} \\ &1 \end{bmatrix} \end{align*}$$

Verify that the equation does reflect the formula for the column vectors in the stack representation.

Now, we need to find the eigenvalues for the following matrix
$$\begin{align*} &B = \begin{bmatrix} 1 &1 &1 \\ 1 &0 &0 \\ 0 &0 &1 \end{bmatrix} \end{align*}$$

We start with the characteristic equation and solve, just as before.
$$\begin{align*} &det (B-\lambda I) = 0\\ \Rightarrow & det\left (\begin{bmatrix} 1 &1 &1 \\ 1 &0 &0 \\ 0 &0 &1 \end{bmatrix} - \lambda \begin{bmatrix} 1 &0 &0 \\ 0 &1 &0 \\ 0 &0 &1 \end{bmatrix}\right) = 0\\ \Rightarrow & det\left (\begin{bmatrix} 1-\lambda &1 &1 \\ 1 &-\lambda &0 \\ 0 &0 &1-\lambda \end{bmatrix}\right) = 0\\ \Rightarrow &-\lambda (1-\lambda)^2 - (1-\lambda) = 0\\ \Rightarrow & (1-\lambda)(\lambda^2 - \lambda -1) = 0 \end{align*}$$

Hence, either $(1-\lambda) = 0 \Rightarrow \lambda = 1$ or $(\lambda^2 - \lambda -1) = 0 \Rightarrow \lambda = \frac{1 \pm \sqrt{5}}{2}$. We take these values and name them as follows
$$\begin{align*} &\lambda_1 = \frac{1 + \sqrt{5}}{2}\\ &\lambda_2 = \frac{1 - \sqrt{5}}{2} \\ &\lambda_3 = 1 \end{align*}$$

Now, we have to find out the eigenvectors. Again we write an eigenvector as $X = \begin{bmatrix} x_1 \\ x_2 \\ x_3 \end{bmatrix}$ and hence we have,
$$\begin{align*} & BX = \lambda X \\ \Rightarrow & \begin{bmatrix} 1 &1 &1 \\ 1 &0 &0 \\ 0 &0 &1 \end{bmatrix} \begin{bmatrix} x_1 \\ x_2 \\ x_3 \end{bmatrix} = \lambda \begin{bmatrix} x_1 \\ x_2 \\ x_3 \end{bmatrix} \\ \Rightarrow &\begin{bmatrix} x_1 + x_2 + x_3\\ x_1 \\ x_3 \end{bmatrix} = \begin{bmatrix} \lambda x_1 \\ \lambda x_2 \\ \lambda x_3 \end{bmatrix} \end{align*}$$

For either $\lambda_1$ or $\lambda_2$, both of these are not zero or one, the bottom row gives $x_3=0$. If that be the case, we arrive at the equation,
$$\begin{align*} &\begin{bmatrix} x_1 + x_2\\ x_1 \\ \end{bmatrix} = \begin{bmatrix} \lambda x_1 \\ \lambda x_2 \\ \end{bmatrix} \end{align*}$$

This is the same equation that we got for the Fibonacci sequence and we know the vectors $X_1 = \begin{bmatrix} \lambda_1 \\ 1 \\ 0 \end{bmatrix}$ and $X_2 = \begin{bmatrix} \lambda_2 \\ 1 \\ 0 \end{bmatrix}$. We added the bottom row 0 because $x_3=0$. Now, for $\lambda_3 = 1$, the bottom row does not do anything. We have the equation,
$$\begin{align*} &\begin{bmatrix} x_1 + x_2 + x_3\\ x_1 \\ \end{bmatrix} = \begin{bmatrix} x_1 \\ x_2 \\ \end{bmatrix} \end{align*}$$

Substituting $x_1 = x_2$ from the bottom row into the equation in the top row, we have
$$\begin{align*} & x_2 + x_2 + x_3 = x_2 \Rightarrow & x_2 = -x_3 \end{align*}$$

In this case, we scale $x_2 = 1$ which gives $x_3=-1$. Hence, our eigenvector $X_3 = \begin{bmatrix} 1 \\ 1 \\ -1 \end{bmatrix}$ 

Our equation for finding $f_h$ in matrix form is
$$\begin{align*} \begin{bmatrix} f_h \\ f_{h-1} \\ 1 \end{bmatrix} = B^{h-1} \begin{bmatrix} 1 \\ 0 \\ 1 \end{bmatrix} \end{align*}$$

We need to express the vector $\begin{bmatrix} 1 \\ 0 \\ 1 \end{bmatrix}$ as a linear combination of the eigenvectors. Let the constants be $c_1$, $c_2$ and $c_3$. So, we have the equations
$$\begin{align*} &c_1 X_1 + c_2 X_2 + c_3 X_3 = \begin{bmatrix} 1 \\ 0 \\ 1 \end{bmatrix} \\ \Rightarrow & c_1 \begin{bmatrix} \lambda_1 \\ 1 \\ 0 \end{bmatrix} + c_2 \begin{bmatrix} \lambda_2 \\ 1 \\ 0 \end{bmatrix} + c_3 \begin{bmatrix} 1\\ 1 \\ -1 \end{bmatrix} = \begin{bmatrix} & 1 \\ & 0 \\ & 1 \end{bmatrix} \\ \end{align*}$$

From the bottom row, we have $c_3 = -1$. Substituting this in the other two equations, we have,
$$\begin{align*} & c_1 \begin{bmatrix} \lambda_1 \\ 1 \\ 0 \end{bmatrix} + c_2 \begin{bmatrix} \lambda_2 \\ 1 \\ 0 \end{bmatrix} - \begin{bmatrix} 1\\ 1 \\ -1 \end{bmatrix} = \begin{bmatrix} 1 \\ 0 \\ 1 \end{bmatrix} \\ \Rightarrow & c_1 \begin{bmatrix} \lambda_1 \\ 1 \\ 0 \end{bmatrix} + c_2 \begin{bmatrix} \lambda_2 \\ 1 \\ 0 \end{bmatrix} = \begin{bmatrix} 2 \\ 1 \\ 0 \end{bmatrix} \\ \Rightarrow & \begin{bmatrix} \lambda_1 &\lambda_2 \\ 1 &1 \\ \end{bmatrix}\begin{bmatrix} c_1 \\ c_2 \\ \end{bmatrix} = \begin{bmatrix} 2 \\ 1 \\ \end{bmatrix} \\ \end{align*}$$

Solving the above set of equations, we have $c_1 = \frac{2- \lambda_2}{\lambda_1 - \lambda_2} = \frac{3+\sqrt{5}}{2\sqrt{5}}$ and $c_2 = \frac{2- \lambda_1}{\lambda_2 - \lambda_1} = -\frac{3-\sqrt{5}}{2\sqrt{5}}$. Now, we have
$$\begin{align*} &\begin{bmatrix} f_{h} \\ f_{h-1} \\ 1 \end{bmatrix} = B^{h-1} \begin{bmatrix} 1 \\ 0 \\ 1 \end{bmatrix}\\ =& B^{h-1} (c_1 X_1 + c_2 X_2 + c_3 X_3)\\ =& B^{h-1} \left (c_1 \begin{bmatrix} \lambda_1 \\ 1 \\ 0 \end{bmatrix} + c_2 \begin{bmatrix} \lambda_2 \\ 1 \\ 0 \end{bmatrix} + c_3 \begin{bmatrix} 1\\ 1 \\ -1 \end{bmatrix} \right)\\ =& \left (c_1 B^{h-1} \begin{bmatrix} \lambda_1 \\ 1 \\ 0 \end{bmatrix} + c_2 B^{h-1} \begin{bmatrix} \lambda_2 \\ 1 \\ 0 \end{bmatrix} + c_3 B^{h-1} \begin{bmatrix} 1\\ 1 \\ -1 \end{bmatrix} \right)\\ =& \left (c_1 \lambda_1^{h-1} \begin{bmatrix} \lambda_1 \\ 1 \\ 0 \end{bmatrix} + c_2 \lambda_2^{h-1} \begin{bmatrix} \lambda_2 \\ 1 \\ 0 \end{bmatrix} + c_3 \lambda_3^{h-1} \begin{bmatrix} 1\\ 1 \\ -1 \end{bmatrix} \right)\\ =& \left ( \begin{bmatrix} c_1 \lambda_1^{h} \\ c_1 \lambda_1^{h-1} \\ 0 \end{bmatrix} + \begin{bmatrix} c_2 \lambda_2^{h} \\ c_2 \lambda_2^{h-1} \\ 0 \end{bmatrix} + \begin{bmatrix} c_3 \lambda_3^{h-1}\\ c_3 \lambda_3^{h-1} \\ -c_3 \lambda_3^{h-1} \end{bmatrix} \right)\\ \end{align*}$$

From the above, we can write,
$$\begin{align*} &f_h = c_1 \lambda_1^{h} + c_2 \lambda_2^{h} + c_3 \lambda_3^{h-1} \\ \Rightarrow & f_h = \frac{3+\sqrt{5}}{2\sqrt{5}} \left (\frac{1 + \sqrt{5}}{2}\right)^h -\frac{3-\sqrt{5}}{2\sqrt{5}} \left (\frac{1 - \sqrt{5}}{2}\right)^h - 1 \\ \Rightarrow & f_h =\frac{1}{\sqrt{5}} \left\{\frac{3+\sqrt{5}}{2} \left (\frac{1 + \sqrt{5}}{2}\right)^h -\frac{3-\sqrt{5}}{2} \left (\frac{1 - \sqrt{5}}{2}\right)^h\right\} - 1 \\ \Rightarrow & f_h =\frac{1}{\sqrt{5}} \left\{\left (\frac{1+\sqrt{5}}{2}\right)^2 \left (\frac{1 + \sqrt{5}}{2}\right)^h -\left (\frac{1-\sqrt{5}}{2}\right)^2 \left (\frac{1 - \sqrt{5}}{2}\right)^h\right\} - 1 \\ \Rightarrow & f_h =\frac{1}{\sqrt{5}} \left\{ \left (\frac{1 + \sqrt{5}}{2}\right)^{h+2} - \left (\frac{1 - \sqrt{5}}{2}\right)^{h+2}\right\} - 1 \\ \Rightarrow & f_h = F_{h+2} - 1 \\ \end{align*}$$

This is the formula for the exact number of nodes in the worst AVL tree of height h. Again, it has lots of square roots, but for integer h, it will always give an integer result. What this means is that the number of minimum number of nodes in a AVL tree of height $h$ is $F_{h+2} - 1 \approx k\phi^h -1 = \theta ({\phi}^h)$ where $k$ is a constant. This in turn means that if an AVL tree has n nodes, then the maximum height $h = \theta (\log_{\phi}n)$.
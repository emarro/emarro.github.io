---
title: Matrix Structures
author: emarro
math: true
tags: Linear Algebra Matrix Structure
---


I've been re-reading some old course notes and books from a previous course I took that left an impression on me,
this is the first in a series of write-ups and is my way of reviewing some of the key points of that course.
> This post assumes a background of some introductory Linear Algebra
{: .prompt-warning}

## Ways to Solve a Problem
We have a system of linear equations $Ax = b$, where we want to solve for $x$. How would you solve this?

...


...


...



Maybe we can find the inverse of $A^{-1}$ then $x = A^{-1}b$, or maybe we can run an algorithm like Gaussian Elimination to
find $x$. While these methods are sometimes correct, this was really a trick question since I told you nothing about $A$.
1. What shape is $A$? 
2. Is $$ A \in \mathbb{Z}^{n \times m}$$? 
3. Is $$A \in \mathbb{R}^{n \times m}$$?. 
4. Is $$A$$ even invertible? 
5. Is it overdetermined or underdetermined? 

I use to not think about the structure of my matrix much, until I
took a course on Applied Numerical Methods in grad school. If I had to boil down that course to a single sentence it would be: 

> How can I exploit the structure of some matrix(-ices) so that I can compute what I want in less time/space?
{: .prompt-tip}

This is really a fundamental aspect of computer science and applied mathematics, if I understand the structure/geometry/design patterns of something I know what parts can be sped up, what requires a lot of thought and care, and what parts are largely irrelevant and don't contribute much to the overall product/end-goal. 

## When __Not__ to Take the Inverse
### A Triangular Example
Let's consider a binary square lower triangular matrix,
$$
A = \begin{bmatrix} 1& 0& 0\\
                    1& 1& 0\\
                    1& 1& 1
           \end{bmatrix}
$$
.

We can see that for any $b \in \mathbb{R}^3$, we can solve $Ax = b$ in just a few operations. Writing out the formulas for each $x_i$, we can see that

$$
\begin{align}
    x_1 &= b_1 &\\
    x_2 &= b_2 - x_1 &= b_2 - b_1 \\
    x_3 &= b_3 - x_2 - x_1 &= b_3 - b_2 + b_1 - b_1 = b_3 - b_2  \\
\end{align}
$$

and we can solve this in $O(n)$ time.

### Solving With the Inverse

 A is invertible though, we can calculate 
$$A^{-1} = \begin{bmatrix}
    1& 0& 0 \\
    -1& 1& 0 \\
    0& -1& 1
\end{bmatrix} $$
and we can then take $A^{-1}b = x$, which gives us the exact same equations we derived above! With the exception that we didn't need to take the inverse at all if we just solved the equations directly. 

### The General Case
In general, for lower triangular $A \in \mathbb{1}^{n \times n}$, we can solve $Ax = b, b \in \mathbb{R}^n$ in $O(n)$ time. 

$$
\begin{align}
    x_1 &= b_1                   &=b_1 - 0\\
    x_2 &= b_2 - x_1             &= b_2 - b_1\\
    x_3 &= b_3 - x_2 - x_1       &= b_3 - b_2\\
    x_4 &= b_4 - x_3 - x_2 - x_1 &= b_4 - b_3\\
    \ldots \\
    x_n &= b_n + (-1) * \sum_{i=1}^{n-1}{x_i} &= b_n - b_{n-1} 
\end{align}
$$


> We can even formulate this as vector addition, let $b^{\prime} = \[0, b_1, b_2, \ldots, b_{n-1}\]$, then $x = b - b^{\prime}$. If we're clever with our memory and with SIMD instructions, this operation can be done __very__ quickly.  
{: .prompt-tip}

> This algorithmic pattern to solve lower triangular matricies shows up all the time and is known as __forward substitution__. This particular $A$ (where we only consider 1's) is simpler and faster to solve than the general case, but forward substitution and backward substitution (the upper triangular equivalent) are very useful tools to have for matrix computations.
{: .prompt-info}

### Takeaways
Solving a system of linear equations where $A$ is a binary triangular matrix can be done much faster than for many other classes of $A$. By taking advantage of the structure of $A$, we can formulate it as a vector addition (well, subtraction really) and avoid having to run some other, costlier algorithm. 

While taking the inverse would have also been correct here, it would have been strictly worse since it would have been a lot of extra work (roughly $O(n^3)$ more work) to give us a matrix that would allow us to do the computation that we figured out directly.
For large $n$, this might be the difference between getting and answer or failing to compute it all together. 
I hope this illustrates just how important the structure of the matrix, if we understand the structure of a matrix we may be able to come up with a solution that is a lot cheaper and faster than the blunt way of just taking the inverse. 
> While we looked at a contrived example where $A$ was a binary triangular matrix, we can also find a similar fast algorithm to solve $Ax = b$ where $A$ is a _real-valued_ triangular matrix. This implies that if we can find a fast way to transform a matrix into a triangular matrix, we can solve $Ax = b$ quickly. We'll see examples of this when we talk about matrix decompositions in the future.
{: .prompt-info}

#### Losing the Structure
And one thing I didn't draw attention to but that I also want to emphasis:
> $A^{-1}$ was no longer a triangular matrix, the inverse has a different structure
{: .prompt-danger}

While for this specific example $A^{-1}$ is a banded matrix and therefore still has structure we could exploit, there may be differences in the properties of $A$ and $A^{-1}$ that would make working with $A^{-1}$ significantly more difficult. Taking the inverse of a matrix can introduce pratical problems! For example, if $A$ is sparse, $A^{1}$ may not necessarily be sparse and so we might have trouble just storing $A^{-1}$, let alone doing any computations with it. 

Consider the following matrix and it's inverse:

$$
A = 
\begin{bmatrix}
1& 0& 0& 0& 0& \\
1& 1& 0& 0& 0& \\
0& 1& 1& 0& 0& \\
0& 0& 1& 1& 0& \\
0& 0& 0& 1& 1& \\
\end{bmatrix}

A^{-1} = 
\begin{bmatrix}
1& 0& 0& 0& 0& \\
-1& 1& 0& 0& 0& \\
1& -1& 1& 0& 0& \\
-1& 1& -1& 1& 0& \\
1& -1& 1& -1& 1& \\
\end{bmatrix}
$$

The matrix $A$ would be a fairly straight forward one to solve directly and only has 9 non-zero entries, whereas it's inverse has more non-zero entries ($15$). If we were working with large $n$ and could only work on $A$ because it was sparse, it is not necessarily guaranteed that $A^{-1}$ is also sparse and so we may not even be able to perform any operations with it. Even through in theory the answers are the same, the different practical considerations may cause significant issues if we try to work with $A^{-1}$ without some consideration as to whether not it is a good idea.

Taking the inverse of a matrix changes the structure of the matrix, and so we have to be very careful about why we are doing so!

## Future Topics
I just wanted to take today to write about the importance of matrix structure, as understanding the consequences of a matrix's structure is vital to understand how to solve it quickly. In the future, I want to write a bit about different methods for solving $$ Ax = b$$ and even $$\|{Ax - b}\|$$. I include a few of those topics below, and I'll often be following [Matrix Computation by Golub and Van Loan](https://books.google.com/books?hl=en&lr=&id=5U-l8U3P-VUC&oi=fnd&pg=PP1&dq=golub+van+loan&ots=7-JDKiVT7s&sig=6t-DebSKhXrn9G1nCgfAuXpzpRw#v=onepage&q&f=false) since I think that book is fantastic.
### Matrix Decompositions
Sometimes, a matrix is structured in such a way that we can decompose it into a combination of other matrices and/or vectors. These decompositions often come with advantages, where after we decompose a matrix we have a fast algorithm for solving the problem, so if we can decompose a matrix quickly we can also solve the problem quickly.
1. Lower-Upper (LU) decomposition (sometimes called LR decomposition) decomposes a matrix in a lower triangular matrix $L$ and an upper triangular matrix $U$. This decomposition is actually a common way for computers to solve a system of linear equations. 
2. Cholesky Decomp - If $A$ is positive-definite, the Cholesky decomp is kind of like a way faster version of the LU decomp and so admits even faster solutions to $Ax = b$
4. Schur Decomp - Takes a square $A$ and is often used to find the eigenvalues of $A$.
5. Singular Value Decomposition - Can be used to to find the singular values (i.e the root of the eigenvalues, $\lambda^{\frac{1}{2}}$) of $A \in \mathbb{C}^{m \times n}$.
### Methods for Sparse Matrices
What if most of the entries in $A$ are $0$? Can we take advantage of that to speed up computations?
### Methods for Overdetermined/Underdetermined systems
If we have multiple valid solutions/no solutions, what do we do?
### Iterative Methods
Mainly focusing on Krylov subspace methods, there are ways to solve problems with massive matrices where we can get very close approximations fairly quickly in instances where direct methods are very expensive to run if they run at all.
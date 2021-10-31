---
layout: page
title: How Computers Multiply HUGE Matrices
---
<script>
  document.getElementById("page-link-Linear Algebra").classList.add("active")
</script>

## Introduction: Computational Complexity of Matrix Multiplication

As you have learned and practiced matrix algebra, you may have been feeling that matrix multiplication could be extremely complex. It is okay to multiply $$2 \times 2$$ or $$3 \times 3$$ matrices by hand, but what about larger ones? Often in the real world, we would leave such a task to a computer, as there is a very clear and unambiguous algorithm for matrix multiplication. Today, there are a lot of cases where we would need to perform huge matrix-matrix or matrix-vector multiplications. In deep learning, for example, we could easily have millions of rows, each representing a data point, and tens or hundreds or thousands of columns, each representing a "characteristic" of the sample. This could be challenging even for a computer, and we will see why with some simple analysis of how complex matrix multiplication actually is.

For simplicity, let's assume we are multiplying two $$n \times n$$ matrices $$A$$ and $$B$$, where

$$
A = \left[ \begin{matrix}
a_{1, 1} & a_{1, 2} & \cdots & a_{1, n} \\
a_{2, 1} & a_{2, 2} & \cdots & a_{2, n} \\
\vdots & \vdots & \ddots & \vdots \\
a_{n, 1} & a_{n, 2} & \cdots & a_{n, n}
\end{matrix} \right],
B = \left[ \begin{matrix}
b_{1, 1} & b_{1, 2} & \cdots & b_{1, n} \\
b_{2, 1} & b_{2, 2} & \cdots & b_{2, n} \\
\vdots & \vdots & \ddots & \vdots \\
b_{n, 1} & b_{n, 2} & \cdots & b_{n, n}
\end{matrix} \right]
\text{, and }C = A \times B = \left[ \begin{matrix}
c_{1, 1} & c_{1, 2} & \cdots & c_{1, n} \\
c_{2, 1} & c_{2, 2} & \cdots & c_{2, n} \\
\vdots & \vdots & \ddots & \vdots \\
c_{n, 1} & c_{n, 2} & \cdots & c_{n, n}
\end{matrix} \right].
$$

Now let's recall the procedure of matrix multiplication. No matter what view you have in your mind, try to convince yourself that what we are effectively doing is that, **for each element $$c_{i, j}$$ in $$C$$, its value is given by the "dot product" of the $$i$$-th row of $$A$$ and the $$j$$-th column of $$B$$, i.e.,**

$$
\boxed{c_{i, j} = a_{i, 1} b_{1, j} + a_{i, 2} b_{2, j} + \cdots + a_{i, n} b_{n, j}}.
$$

Let's have a closer look at the equation above. We have $$n$$ pairs of values to multiply, and then we add all $$n$$ products together, so there are $$2n$$ operations involved. This is for one single value in $$C$$—there are a total of $$n \times n = n^2$$ of them. So we have a magnitude of $$n^2 \times 2n = 2n^3$$ arithmetic operations to perform. Within the computer, there might be much more low-level operations hidden in one single arithmetic operation, but for a high-level complexity analysis, they can be assumed to be approximately the same for each arithmetic operation. Then, this is literally just a constant we would multiply with $$2n^3$$. In computer science, we tend to drop such constants along with the constant $$2$$ and simply say that the **asymptotic computational complexity** is $$O(n^3)$$, read "big-$$O$$ of $$n$$-cubed". Intuitively, this means that the number of "operations" involved grows "*like*" $$n^3$$, and the exact coefficient in front of $$n^3$$ doesn't really matter that much when $$n$$ is large. (A formal definition of this big-$$O$$ notation, as well as two other similar ones, big-$$\Omega$$ and big-$$\Theta$$, will be introduced in some relevant computer science courses such as COMP 251 -- Algorithms and Data Structures.)

$$n^3$$ actually blows up pretty quickly. Say a computer can perform $$10^7$$ "average" arithmetic operations in one second. $$200^3$$ is approaching a second, $$1000^3$$ takes almost two minutes, and $$10000^3$$ takes half an hour already. You can see how it grows for even larger matrices. This is why we really want some ways to optimize it, and the next few sections will introduce some relevant techniques.

## Block Matrices

A very common technique we need to solve this problem is to divide a big matrix into *blocks*. Block matrices are not part of MATH 133 but are covered in section 2.3 of the textbook [1]. Intuitively, the idea is to divide the matrix into several parts, each of which can be viewed as a "sub"-matrix. For simplicity, in this write-up, we will only look at square matrices and leave out the more tricky cases. For example, we can divide a matrix

$$A = \left[ \begin{matrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 2 & -1 & 4 & 2 \\ 3 & 1 & -1 & 7 \end{matrix} \right]$$

into four blocks

$$
A_{1,1} = \left[ \begin{matrix} 1 & 0 \\ 0 & 1 \end{matrix} \right], A_{1,2} = \left[ \begin{matrix} 0 & 0 \\ 0 & 0 \end{matrix} \right], A_{2,1} = \left[ \begin{matrix} 2 & -1 \\ 3 & 1 \end{matrix} \right], \text{ and } A_{2,2} = \left[ \begin{matrix} 4 & 2 \\ -1 & 7 \end{matrix} \right].
$$

And now, we can write

$$A = \left[ \begin{matrix} A_{1,1} & A_{1,2} \\ A_{2,1} & A_{2,2} \end{matrix} \right].$$

That is the intuition for block matrices. In fact, you have already been exposed to the idea of block matrices, although this term was not used at that time. When we take a "row view" or "column view" of a matrix, i.e. viewing a matrix as something like $$\left[ \begin{matrix} \text{—} & v_1 & \text{—} \\ \text{—} & v_2 & \text{—} \\ & \vdots & \end{matrix} \right]$$ or $$\left[ \begin{matrix} \mid & \mid &  \\ v_1 & v_2 & \dots \\ \mid & \mid & \end{matrix} \right]$$, we are in fact dividing the matrix into blocks, where each block has a width or height of 1!

Now let's get back to our problem of matrix multiplication. Let's say we have two $$4 \times 4$$ matrices

$$
A = \left[ \begin{matrix} a_{1,1} & a_{1,2} & a_{1,3} & a_{1,4} \\ a_{2,1} & a_{2,2} & a_{2,3} & a_{2,4} \\ a_{3,1} & a_{3,2} & a_{3,3} & a_{3,4} \\ a_{4,1} & a_{4,2} & a_{4,3} & a_{4,4} \end{matrix} \right] = \left[ \begin{matrix} A_{1,1} & A_{1,2} \\ A_{2,1} & A_{2,2} \end{matrix} \right] \text{ and } B = \left[ \begin{matrix} b_{1,1} & b_{1,2} & b_{1,3} & b_{1,4} \\ b_{2,1} & b_{2,2} & b_{2,3} & b_{2,4} \\ b_{3,1} & b_{3,2} & b_{3,3} & b_{3,4} \\ b_{4,1} & b_{4,2} & b_{4,3} & b_{4,4} \end{matrix} \right] = \left[ \begin{matrix} B_{1,1} & B_{1,2} \\ B_{2,1} & B_{2,2} \end{matrix} \right],
$$

and we want to compute

$$
C = A \times B = \left[ \begin{matrix} c_{1,1} & c_{1,2} & c_{1,3} & c_{1,4} \\ c_{2,1} & c_{2,2} & c_{2,3} & c_{2,4} \\ c_{3,1} & c_{3,2} & c_{3,3} & c_{3,4} \\ c_{4,1} & c_{4,2} & c_{4,3} & c_{4,4} \end{matrix} \right] = \left[ \begin{matrix} C_{1,1} & C_{1,2} \\ C_{2,1} & C_{2,2} \end{matrix} \right].
$$

It turns out, as mentioned as Theorem 2.3.4 in the textbook [1], "the product AB can be computed by matrix multiplication using blocks as entries." What this means is that we can do $$C_{1,1} = A_{1,1}B_{1,1} + A_{1.2}B_{2,1}$$, $$C_{1,2} = A_{1,1}B_{2,1} + A_{1,2}B_{2,2}$$, etc., just as the normal matrix multiplication—of course, the products are themselves (smaller) matrix multiplications again! You can try to verify this by calculating a few elements in $$C$$ in terms of elements in $$A$$ and $$B$$ once using regular matrix multiplication and once using block matrix multiplication.

This immediately gives us a fairly nice idea of calculating huge matrix multiplication. We could divide the two matrices into smaller blocks, feed different combinations of blocks to a few different computers, let them each calculate a block multiplication simultaneously, and finally put everything back together. The benefit is that we can use multiple "*distributed*" machines to perform some subtasks *in parallel*, utilizing the available computing power and saving the total time needed.

## Cannon Algorithm

The Cannon Algorithm does exactly that. I will adapt the material in [2] for illustration. Imagine that we have two $$3 \times 3$$ block matrices

$$
A = \left[ \begin{matrix} A_{1,1} & A_{1,2} & A_{1,3} \\ A_{2,1} & A_{2,2} & A_{2,3} \\ A_{3,1} & A_{3,2} & A_{3,3} \end{matrix} \right], B = \left[ \begin{matrix} B_{1,1} & B_{1,2} & B_{1,3} \\ B_{2,1} & B_{2,2} & B_{2,3} \\ B_{3,1} & B_{3,2} & B_{3,3} \end{matrix} \right] \text{, and } C = A \times B = \left[ \begin{matrix} C_{1,1} & C_{1,2} & C_{1,3} \\ C_{2,1} & C_{2,2} & C_{2,3} \\ C_{3,1} & C_{3,2} & C_{3,3} \end{matrix} \right].
$$

For illustration, we would focus on the block $$C_{2, 3}$$. We know that $$C_{2, 3} = A_{2, 1} \times B_{1, 3} + A_{2, 2} \times B_{2, 3} + A_{2, 3} \times B_{3, 3}$$. The idea of Cannon Algorithm is that, we would keep "circular-shifting" the blocks so that, by adding the product of the blocks at position $$(2,3)$$ of $$A$$ and $$B$$ each time, we would get the correct $$C_{2, 3}$$.

To initialize, we would *skew* the matrices: For $$A$$, we would keep the first row untouched, shift every block on the second row to the left by one block, and shift every block on the third row to the left by two blocks. For $$B$$, we would keep the first column untouched, shift every block on the second column to the top by one block, and shift every block on the third column to the top by two blocks. We will then get

$$
\left[ \begin{matrix} A_{1,1} & A_{1,2} & A_{1,3} \\ A_{2,2} & A_{2,3} & A_{2,1} \\ A_{3,3} & A_{3,1} & A_{3,2} \end{matrix} \right] \text{ and } \left[ \begin{matrix} B_{1,1} & B_{2,2} & B_{3,3} \\ B_{2,1} & B_{3,2} & B_{1,3} \\ B_{3,1} & B_{1,2} & B_{2,3} \end{matrix} \right].
$$

Notice that, when we multiply the blocks at $$(2,3)$$ of both matrices, we have $$A_{2,1} \times B_{1,3}$$, which is exactly the first term of $$C_{2,3}$$. And similarly for other blocks.

Then, at each subsequent iteration, we will shift every block in $$A$$ one block to the left and every block in $$B$$ one block to the top. At the next step, we have

$$
\left[ \begin{matrix} A_{1,2} & A_{1,3} & A_{1,1} \\ A_{2,3} & A_{2,1} & A_{2,2} \\ A_{3,1} & A_{3,2} & A_{3,3} \end{matrix} \right] \text{ and } \left[ \begin{matrix} B_{2,1} & B_{3,2} & B_{1,3} \\ B_{3,1} & B_{1,2} & B_{2,3} \\ B_{1,1} & B_{2,2} & B_{3,3} \end{matrix} \right].
$$

Again, if we look at the the blocks at $$(2,3)$$ of both matrices, we have $$A_{2,2} \times B_{2,3}$$, which is exactly the second term of $$C_{2,3}$$.

We will do this shifting one more time to obtain the third term. Try to verify by yourself that this works! The same will hold for other blocks and this method can be generalized for block matrices of other dimensions.

## Strassen's Algorithm

Using block matrices, there are many more algorithms that can help us perform matrix multiplication more efficiently. One I would introduce is the Strassen's Algorithm which I saw in [COMP 251 -- Algorithms and Data Structures](https://mcgill.ca/study/2021-2022/courses/comp-251) [3]. This algorithm multiplies 2-by-2 blocks, but uses only 7 multiplications. (There are 11 additions and 7 subtractions, but they are way less expensive than multiplication.) I will not present the proof of its correctness as it might be way too complex, but the idea is to first set

$$P_1 = A_{1,1} \times (B_{1,2} - B_{2,2})$$,

$$P_2 = (A_{1,1} + A_{1,2}) \times B_{2,2}$$,

$$P_3 = (A_{2,1} + A_{2,2}) \times B_{1,1}$$,

$$P_4 = A_{2,2} \times (B_{2,1} - B_{1,1})$$,

$$P_5 = (A_{1,1} + A_{2,2}) \times (B_{1,1} + B_{2,2})$$,

$$P_6 = (A_{1,2} - A_{2,2}) \times (B_{2,1} + B_{2,2})$$,

$$P_7 = (A_{1,1} - A_{2,1}) \times (B_{1,1} + B_{1,2})$$,

Then,

$$C_{1,1} = P_5 + P_4 - P_2 + P_6$$,

$$C_{1,2} = P_1 + P_2$$,

$$C_{2,1} = P_3 + P_4$$,

$$C_{2,2} = P_1 + P_5 - P_3 - P_7$$.

It can be proved by what is called the Master Theorem that the time complexity of Strassen's Algorithm is $$O(n^{\log_2 7})$$, or approximately $$O(n^{2.81})$$, which is smaller than $$O(n^3)$$. Therefore, Strassen's Algorithm not only allows us to multiply matrices in parallel, but also makes matrix multiplication less complex overall.

## Acknowledgements and Further References

Hopefully, through this write-up, you have gained some intuitive understanding of how complex huge matrix multiplications are and computers handle them. I have to admit that, as a third-year undergraduate student, I myself am not an expert in distributed systems and algorithms, and this write-up is far from exhaustive for those matrix multiplication algorithms. Having a look further at the resources I referenced might be helpful if you are interested. And, for computer science or electrical/computer/software engineering students, you will see more about divide-and-conquer and computational complexity in [COMP 251 -- Algorithms and Data Structures](https://mcgill.ca/study/2021-2022/courses/comp-251), and there would likely be many more similar parallel/distributed algorithms in relevant courses like [COMP 512 -- Distributed Systems](https://mcgill.ca/study/2021-2022/courses/comp-512) and [ECSE 420 -- Parallel Computing](https://mcgill.ca/study/2021-2022/courses/ecse-420).

## References

[1] W. K. Nicholson. *Linear Algebra with Applications.* 2021-A. Lyryx Learning Inc., 2021. [E-book]. Available: <https://lyryx.com/linear-algebra-applications/>.
    
[2] S. B. Baden. "CSE 260 Lecture 13 -- Matrix Multiplication," 2012. [Slides]. University of California, San Diego. Available: <https://cseweb.ucsd.edu//classes/fa12/cse260-b/Lectures/Lec13.pdf>.
    
[3] J. Waldispühl. "COMP 251: Divide-and-Conquer (2)," 2020. [Slides]. McGill University.
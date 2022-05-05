---
layout: post
title:  "Solving Markov Chains"
date:   2022-04-30 21:00:00 +0800
categories: Linear Algebra
mathjax: true
---
My primary motivation for writing this blog post is laying the foundation how Markov Chains are used in Reinforcement Learning, Statistics and Biology etc.

Let's start with an intuition on what's a Markov Chain.
Assume that a physical or mathematical system goes through a process of change that allows it to exist in one of a finite number of states at any one time. For example, the weather in a given city could be in one of 4 possible states: Sunny, Cloudy, Snow, Stormy. We wish to know the probability of states in the long run.

If a Markov chain has $$k$$ possible states $$1, 2,... k$$, then the probability that the system is in state $$i$$ at any observation after being in state $$j$$ at the previous observation is denoted by $$p_{i,j}$$, and is known as the **transition probability** from state $$j$$ to state $$i$$ where $$p_{(final\ state, initial\ state)}$$.

The transition matrix of the Markov Chain is known as $$P = [p_{i,j}]$$. 

## I shall explain the concepts via an example, 

A population of ants is put into a maze with 3 compartments labeled a, b, and c. If the ant is in compartment a, an hour later, there is a 20% chance it will go to compartment b, and a 40% change it will go to compartment c. If it is in compartment b, an hour later, there is a 10% chance it will go to compartment a, and a 30% chance it will go to compartment c. If it is in compartment c, an hour later, there is a 50% chance it will go to compartment a, and a 20% chance it will go to compartment b.

Suppose 100 ants has been placed in compartment a. The state vectors for the Markov chain are 
$$x_{n} = \begin{pmatrix}
a_{n}\\ 
b_{n}\\ 
c_{n}\\
\end{pmatrix}$$
where $$a_{n}$$, $$b_{n}$$, and $$c_{n}$$ is the number of ants in compartment a, b, and c, respectively, after n hours. The initial state vector is $$x_{0} = \begin{pmatrix}
100\\ 
0\\ 
0\\
\end{pmatrix}$$

<img src="/assets/images/Solving_Markov_Chains/example.jpg" alt="button" style="width:320px;height:400px;">

The first step would be to form a system of linear equations,

$$a_{n+1} = 0.4a_{n} + 0.1b_{n} + 0.5c_{n}$$

$$b_{n+1} = 0.2a_{n} + 0.6b_{n} + 0.2c_{n}$$

$$c_{n+1} = 0.4a_{n} + 0.3b_{n} + 0.3c_{n}$$

The transition matrix, A is

$$New \ State
\stackrel{\mbox{Preceding State}}{
\begin{pmatrix}
 0.4&  0.1& 0.5\\ 
 0.2&  0.6& 0.2\\ 
 0.4&  0.3& 0.3\\
\end{pmatrix}
}$$

From this matrix, the probability is 0.4 ($$p_{1,1}$$) that an ant moves from compartment $$a$$ and stays in compartment $$a$$, the probability is 0.2 ($$p_{2,1}$$) that an ant moves from compartment $$a$$ to compartment $$b$$, the probability is 0.4 ($$p_{3,1}$$) that an ant moves from compartment a to compartment c, and so forth.

Observe that any columns in transition matrices sum to 1. This matrix property is called $$stochastic \ matrix$$

Let me define a **Lemma** stating that a square matrix **A** is diagonalizable if there exists an invertible matrix **P** such that $A = PDP^{-1}$ where D is a diagonal matrix. 

**Theorem 1**
*If P is the transition matrix of a Markov chain and $x^{(n)}$ is the state vector at the nth observation, then $x^{n+1} = Px^{(n)}$*

From this theorem 1, it follows that

$$
x^{1} = P x^{0}\\
x^{2} = P x^{1} = P^{2} x^{0}\\
x^{3} = P x^{2} = P^{3} x^{0}\\
\\
\vdots
\\
x^{n} = P x^{n-1} = P^{n} x^{0}\\
$$

Consequently,

$$
x^{1} = Px^{0} = \begin{bmatrix}
 0.4&  0.1& 0.5\\ 
 0.2&  0.6& 0.2\\ 
 0.4&  0.3& 0.3
\end{bmatrix} \begin{bmatrix}
 100\\ 
 0\\
 0
\end{bmatrix} = \begin{bmatrix}
40\\ 
20\\ 
40
\end{bmatrix}
$$

$$
x^{2} = Px^{1} = \begin{bmatrix}
 0.4&  0.1& 0.5\\ 
 0.2&  0.6& 0.2\\ 
 0.4&  0.3& 0.3
\end{bmatrix} \begin{bmatrix}
 40\\ 
 20\\
 40
\end{bmatrix} = \begin{bmatrix}
38\\ 
28\\ 
34
\end{bmatrix}
$$

$$
x^{3} = Px^{2} = \begin{bmatrix}
 0.4&  0.1& 0.5\\ 
 0.2&  0.6& 0.2\\ 
 0.4&  0.3& 0.3
\end{bmatrix} \begin{bmatrix}
 38\\ 
 28\\
 34
\end{bmatrix} = \begin{bmatrix}
35\\ 
31.2\\ 
33.8
\end{bmatrix}
$$

Thus, after 3 hours, the probability matrix is $$\begin{bmatrix}
35\\ 
31.2\\ 
33.8
\end{bmatrix}$$

**Fact**
If $|\lambda| < 1$, then $\lambda^{k} \to 0 \ as \ k \to \infty$

As the number of observations increases, the fixed vector can be determined by the method of **Diagonalization**.
The **eigenvalues** and **eigenvectors** explain the system's long-term behavior.

<img src="/assets/images/Solving_Markov_Chains/eigen_is_everything.jpg" alt="button" style="width:320px;height:240px;">

We shall now diagonalize **A**

$$det(\lambda I - A) = \left(\begin{array}{ccc}
\lambda -0.4000 & -0.1000 & -0.5000\\
-0.2000 & \lambda -0.6000 & -0.2000\\
-0.4000 & -0.3000 & \lambda -0.3000
\end{array}\right)$$

Characteristic polynomial: $\lambda^3 -1.3000\,\lambda^2 +0.2600\,\lambda +0.0400 = 0$

# Step 1

Algebraic multiplicity is the number of times the eigenvalue appears as a root repeatedly in the characteristic polynomial.
The eigenvalues are $ \lambda = -0.1, 0.4, 1 $ with algebraic multiplicity of 1 each.

**Corollary**: *If **A** is a square matrix of order n with n distinct eigenvalues, then **A** is diagonalizable*

**Theorem 2**: Let **A** be a square matrix of order n. **A** is diagonalizable if and only if there exists a basis $$ \{ u1,u2,...un \} $$ of $\mathbb{R}^{n}$ of eigenvectors of **A**.

**Theorem 3**: (Geometric multiplicty is no greater than algebraic multiplicity) $1 \leq dim(E_{\lambda}) \leq r \lambda (algebraic \ multiplicity)$

**Theorem 4**: Suppose **A** is a square matrix such that its characteristic polynomial can be written as a product of linear factors.

$$det(xI - A) = (x-\lambda_{1})^{r_{\lambda 1}} (x-\lambda_{2})^{r_{\lambda 2}} (x-\lambda_{3})^{r_{\lambda 3}} \cdots (x-\lambda_{k})^{r_{\lambda k}}$$

where $r_{\lambda i}$ is the algebraic multiplicity of $\lambda_{i}$, for $i=1,...k,$ and the eigenvalues are distinct, $\lambda_{i} \neq \lambda_{j}$ for all $i \neq j$. Then **A** is **diagonalizable** if and only if for each eigenvalue of **A**, its geometric multiplicity is equal to its algebraic multiplicity. 

$$dim(E_{\lambda i}) = r_{\lambda i}$$

If the number of parameters in the solution space of the linear system $(\lambda_{i}I - A)x = 0$ is not equal to the algebraic multiplicity, then A is **not** diagonlizable, meaning there exists an $i$ such that the $dim(E_{\lambda i}) < r_{\lambda i}$

# Step 2

Now find the eigenspace, meaning find a basis $S_{\lambda i}$ of the eigenspace $E_{\lambda i}$ for each eigenvalue $\lambda_{i}, i = 1,...,k$. Necessarily $ S \mid \lambda_{i} \mid  = r\lambda_{i}$ for all $i = 1,..., k$.

The eigenspace associated to an eigenvalue $\lambda$ of **A** is

$$ E_{\lambda} = \{ v \in \mathbb{R}^{n} | Av = \lambda v \} = Null(\lambda I - A)$$

The geometric multiplicity of an eigenvalue $\lambda$ is the dimension of its associated eigenspace,

$$ dim(E_{\lambda}) = nullity(\lambda I - A) $$

$\lambda = -0.1$

$$-0.1 I - A = \begin{pmatrix}
   -0.5000&   -0.1000&   -0.5000\\
   -0.2000&   -0.7000&   -0.2000\\
   -0.4000&   -0.3000&   -0.4000
\end{pmatrix} \overset{rref}{\rightarrow}

\begin{pmatrix}
 1&  0& 1\\ 
 0&  1& 0\\ 
 0&  0& 0
\end{pmatrix}. So \ \{ \begin{pmatrix}
-1\\ 
0\\ 
1
\end{pmatrix} \} \ is\ a\ basis\ for\ E_{-0.1}$$

$\lambda = 0.4$

$$0.4 I - A = \begin{pmatrix}
   0&   -0.1000&   -0.5000\\
   -0.2000&   -0.2000&   -0.2000\\
   -0.4000&   -0.3000&   0.1000
\end{pmatrix} \overset{rref}{\rightarrow}

\begin{pmatrix}
 1&  0& -4\\ 
 0&  1& 5\\ 
 0&  0& 0
\end{pmatrix}. So \ \{ \begin{pmatrix}
4\\ 
-5\\ 
1
\end{pmatrix} \} is\ a\ basis\ for\ E_{0.4}
$$

$\lambda = 1$

$$1 I - A = \begin{pmatrix}
   0.6&   -0.1000&   -0.5000\\
   -0.2000&   0.4000&   -0.2000\\
   -0.4000&   -0.3000&   0.7000
\end{pmatrix} \overset{rref}{\rightarrow}

\begin{pmatrix}
 1&  0& -1\\ 
 0&  1& -1\\ 
 0&  0& 0
\end{pmatrix}. So \ \{ \begin{pmatrix}
1\\ 
1\\ 
1
\end{pmatrix} \} is\ a\ basis\ for\ E_{1}
$$

+ The geometric multiplicity of $\lambda = -0.1$ is dim($E_{-0.1}$) = 1
+ The geometric multiplicity of $\lambda = 0.4$ is dim($E_{0.4}$) = 1
+ The geometric multiplicity of $\lambda = 1$ is dim($E_{1}$) = 1

The geometric multiplicity is equal to the algebraic multiplicity, hence A is diagonalizable.

# Step 3

Let 

P = ($u_{1}, u_{2}, ..., u_{n}$), and D = diag($\mu_{1}, \mu_{2}, ..., \mu_{n}$) = $$\begin{pmatrix}
 \mu_{1}&  0&  \cdots& 0\\ 
 0&  \mu_{2}&  & 0\\ 
 \vdots&  &  \ddots & \vdots\\ 
 0&  0&  \cdots& \mu_{n}
\end{pmatrix}$$

where $\mu_{i}$ is the eigenvalue associated to eigenvector $u_{i}$, $i=1,...,n,$ where $Au_{i} = \mu_{i}u_{i}$. Take the basis vectors and place them into matrix P where each eigenvector in P corresponds to the eigenvalue in the diagonal matrix.

$$A = P D P^{-1} = \begin{pmatrix}
 -1&  4& 1\\ 
 0&  -5& 1\\ 
 1&  1& 1
\end{pmatrix} \begin{pmatrix}
 -0.1&  0& 0\\ 
 0&  0.4& 0\\ 
 0&  0& 1
\end{pmatrix} \begin{pmatrix}
 -1&  4& 1\\ 
 0&  -5& 1\\ 
 1&  1& 1
\end{pmatrix}^{-1}$$

**Lemma**: Suppose $A^{m} = PD^{m}P^{-1}$. It's trivial to prove this lemma,

$$A^{3} = (PDP^{-1})(PDP^{-1})(PDP^{-1}) \\ 
A^{3} = PDDDP^{-1} \\ 
A^{3} = PD^{3}P^{-1} $$

Using the fact mentioned above,
$$
A^{k} = P D^{k} P^{-1} * \ initial\ state\ vector = \begin{pmatrix}
 -1&  4& 1\\ 
 0&  -5& 1\\ 
 1&  1& 1
\end{pmatrix} \begin{pmatrix}
 0&  0& 0\\ 
 0&  0& 0\\ 
 0&  0& 1
\end{pmatrix} \begin{pmatrix}
 -1&  4& 1\\ 
 0&  -5& 1\\ 
 1&  1& 1
\end{pmatrix}^{-1} \begin{pmatrix}
 100\\ 
 0\\ 
 0
\end{pmatrix}= \begin{pmatrix}
 33.3333\\ 
 33.3333\\ 
 33.3333
\end{pmatrix}
$$

```MATLAB
% Here is the code using MATLAB,
[P D] = eig(A);
D = [1 0 0; 0 0 0; 0 0 0]
P * D * inv(P) * [100; 0; 0]
```

Thus, in the long run, the population of ants will be equally divided into the 3 compartments a,b and c.

There are 2 obstructions to diagonalization
+ The characteristic polynomial does not factorize into real linear factors, and
+ the geometric multiplicity is strictly less than the algebraic multiplicity

the solutions are

- allow complex eigenvalues
- generalized eigenvectors

However, not all state vectors approach a fixed vector in a Markov Chain. Let's demonstrate using an example,

Let
$$ P = \begin{bmatrix}
 0& 1\\ 
 1& 0
\end{bmatrix} and \
x^{0} = \begin{bmatrix}
1\\ 
0
\end{bmatrix} $$

$$
P x^{0} = 
\begin{bmatrix}
 0& 1\\ 
 1& 0
\end{bmatrix}
\begin{bmatrix}
 1\\ 
 0
\end{bmatrix} = \begin{bmatrix}
 0\\ 
 1
\end{bmatrix}
$$

$$
P^{10} x^{0} = 
\begin{bmatrix}
 1& 0\\ 
 0& 1
\end{bmatrix}
\begin{bmatrix}
 1\\ 
 0
\end{bmatrix} = \begin{bmatrix}
 1\\ 
 0
\end{bmatrix}
$$

The system oscillates indefinitely between the two state vectors $$\begin{bmatrix}
 1\\ 
 0
\end{bmatrix}$$ and $$\begin{bmatrix}
 0\\ 
 1
\end{bmatrix}$$, so it does not approach any fixed vector. We need to impose a mild condition on the transition matrix, to show that a steady state vector can be approached. This condition is described the following definition.

**Definition**: A transition matrix is regular if some integer power of it has all positive non-zero entries
A Markov chain with a regular transition matrix is called a regular Markov chain.

**Lemma**: Every regular Markov chain has a state vector **q** such that $$P^{n}x^{0}$$ approaches **q** as n increases.

Let's state it as a **Theorem**,
If P is a regular transition matrix, then as n $\to \infty$

$$
P^{n} \to \begin{bmatrix}
 q1&  q1&  \cdots& q1\\ 
 q2&  q2&  \cdots& q2\\ 
 \vdots&  \vdots&  \ddots& \vdots\\ 
 qk&  qk&  \cdots& qk
\end{bmatrix}
$$
where the $q_{i}$ are positive non-zero numbers such that $q_{1} + q_{2} + ... + q_{k} = 1$ where it converges to **q** (fixed state vector).

**Theorem**

*If P is a regular transition matrix and x is any probability vector, then as* $n \to \infty$,
$$
P^{n}x \to \begin{bmatrix}
q1\\ 
q2\\ 
\vdots\\ 
qk
\end{bmatrix} = q
$$
where q is a fixed probability vector.

This implies that as $$n \to \infty, P^{n}x \to Qx = q$$. Thus, the system eventually approaches a fixed state vector **q** for a regular Markov chain.
The vector **q** is called the **steady-state** vector of the regular Markov chain.

One way of computing the steady state vector is to make use of the following theorem,

**Theorem**
*The steady state vector **q** of a regular transition matrix P is the unique probability vector that satisifies the equation P**q** = **q***. This can also be expressed as a homogenous linear system $(I - P)q = 0$ where **q** is a unique solution vector with nonnegative entries.

Example

$$
P = \begin{bmatrix}
 0.8& 0.3\\ 
 0.2& 0.7
\end{bmatrix}
$$

so the linear system $(I-P)q = 0$ is 
$$
\begin{bmatrix}
 0.2& -0.3\\ 
 -0.2& 0.3
\end{bmatrix}\begin{bmatrix}
q1\\ 
q2
\end{bmatrix} = \begin{bmatrix}
0\\ 
0
\end{bmatrix}
$$

The nullspace is spanned by $$s \begin{bmatrix}
1.5\\ 
1
\end{bmatrix} where \ s \in \mathbb{R}$$.

Set $$ s = \frac{1}{1.5+1}=0.4$$

$$q = \begin{bmatrix}
0.6\\ 
0.4
\end{bmatrix}$$
is the steady state vector of this regular Markov chain. 

This concludes on how to solve Markov chains using Linear Algebra, hope you had learnt something. It might be overwhelming especially if you are new to linear algebra, but try to dissect and understand the content piece by piece. A good resource to accompany this blog post are [lectures](https://ocw.mit.edu/courses/18-06-linear-algebra-spring-2010/resources/lecture-24-markov-matrices-fourier-series/) from Prof. Gilbert Strang.

Note:
All computations were performed in MATLAB, the source code is available [here](https://github.com/shankar-shiv/MA1508E_Notes/tree/main/Solving%20Markov%20Chains) 
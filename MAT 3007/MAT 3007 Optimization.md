# 1 Lecture 1: Introduction to Optimization

## 1.1 Components of an Optimization Problem

- **Decision**: Choosing the decision variables ($x$) to be determined.

- **Objective**: The objective function ($f(x)$) to minimize or maximize depending on the context.

- **Constraints**: The feasible set ($\Omega$) or restrictions that the decision variables must satisfy.

## 1.2 Mathematical Formulations

**Abstract Representation**:
$$\begin{aligned}
& \underset{x}{\text{minimize}} & & f(x) \\
& \text{subject to} & & x \in \Omega.
\end{aligned}$$
- Maximize $f(x)$ i.e. minimize $-f(x)$

**Restrictive/Explicit Algebraic Form**:
$$\begin{align*} 
\underset{\boldsymbol{x} \in \mathbb{R}^n} {\text{minimize }} & f(\boldsymbol{x}) \\ 
\text{subject to } & g_i(\boldsymbol{x}) \le 0, \quad \forall i = 1, \dots, m \\ 
& h_j(\boldsymbol{x}) = 0, \quad \forall j = 1, \dots, p 
\end{align*}$$
where $g(\boldsymbol{x}) \le 0$ represents inequality constraints and $h(\boldsymbol{x}) = 0$ represents equality constraints, where
$$
g(\boldsymbol{x}) = 
\begin{bmatrix}
g_1(x)\\
\vdots \\
g_m(x)
\end{bmatrix} \in \mathbb{R}^m
\quad \text{and} \quad
h(\boldsymbol{x}) = 
\begin{bmatrix}
h_1(x)\\
\vdots \\
h_p(x)
\end{bmatrix} \in \mathbb{R}^p.
$$
$$ \begin{aligned} \Omega &\coloneqq \left\{ x \in \mathbb{R}^n : g_i(x) \leq 0,\ i = 1, \dots, m,\ h_j(x) = 0,\ j = 1, \dots, p \right\} \\ &\space = \left\{ x \in \mathbb{R}^n : \mathbf{g}(x) \leq 0,\ \mathbf{h}(x) = 0 \right\}. \end{aligned} $$
## 1.3 Core Terminologies

- **Feasible Point**: A decision vector that satisfies all given constraints.

- **Feasible Region (Set $\Omega$)**: The complete set containing all feasible points.

- **Optimal Solution (Global Minimizer)**: A feasible decision variable that attains an objective value as good as any other point in $\Omega$.

- **Optimal Value**: The unique objective value achieved by an optimal solution.

- **Unconstrained vs. Constrained:** A problem is unconstrained if $\Omega = \mathbb{R}^n$ (i.e., $m = p = 0$); otherwise, it is a constrained optimization problem.

## 1.4 Formal Definitions of Minimizers

Open Ball ($B_\epsilon(y)$): Defined as $B_\epsilon(y) := \{x \in \mathbb{R}^n : \|x - y\| < \epsilon\}$ with center $y$ and radius $\epsilon > 0$.

**Local Minimizer**: $x^* \in \Omega$ is a local minimizer if there exists $\epsilon > 0$ such that $f(x) \ge f(x^*)$ for all $x \in \Omega \cap B_\epsilon(x^*)$.
> 对于任意局部极小值点的邻域，在局部最小值点取到极小值。

**Strict Local Minimizer**: $x^* \in \Omega$ where there is $\epsilon > 0$ such that $f(x) > f(x^*)$ for all $x \in (\Omega \cap B_\epsilon(x^*)) \setminus \{x^*\}$.
> Strict: Uniqueness.

**Global Minimizer**: $x^* \in \Omega$ where $f(x) \ge f(x^*)$ for all $x \in \Omega$.

**Strict Global Minimizer**: $x^* \in \Omega$ where $f(x) > f(x^*)$ for all $x \in \Omega \setminus \{x^*\}$.

**Remark: global minimizer ≡ global solution ≡ optimal solution.**

## 1.5 Properties and Possible States of Optimization Problems

**Key Facts**: Feasible sets can be completely empty , an optimal solution may not always exist , the optimal value can be unbounded , and optimal solutions are not necessarily unique.

**Four Fundamental States**:
1. Infeasible: The feasible set  is empty.

2. Feasible, with an optimal value that is finite but not attainable (可行，目标值有限但无法达到). $\mathcal{e.g.}$ $\underset{x} \min \space 1/x (x>0)$

3. Feasible, with an optimal value that is finite and attainable (可行，目标值有限且可达到).

4. Feasible, but the optimal value is unbounded (可行且无界).

## 1.6 Classification of Optimization Problems

- **Mathematical Program**: An optimization problem with a finite number of variables written using explicit algebraic expressions. "Programming" historically means "planning".

- ==**Linear Programming (LP)**==: Both the objective function and all constraints are strictly linear functions of the decision variables.

- **Nonlinear Programming (NLP)**: Occurs when either the objective function or at least one constraint is nonlinear.

- **Convex Optimization**: Occurs if the objective function $f(x)$ and all inequality constraints $g_i(x)$ are convex, while equality constraints $h_j(x)$ are linear.

- **Integer Programming (IP)**: A problem where some or all of the decision variables are restricted to be integers.

- **Continuous vs. Discrete**: Problems are assumed to be continuous by default unless explicitly specified as discrete. 

Linear programming is considered the easiest and most well-studied class, whereas NLP and IP can be significantly harder.

## 1.7 Modeling
For example: Shortest Path Problem

<div align="center"> 
<img src="Pasted image 20260604170510.png"  style="max-width: 80%; height: auto;"> 
</div>

Let $V$ denote the set of nodes, $E \subset V \times V$ represent the set of directed edges in a graph, and $w_{ij}$ represent the distance between node $i$ and node $j$. 
The problem aims to find the shortest path from a starting source node $s$ to a target sink node $t$. 

- **Decision Variables (决策变量)**: For each directed edge $(i, j) \in E$, define a binary indicator variable $x_{ij}$:$$x_{ij} = \begin{cases} 1, & \text{if edge } (i, j) \text{ is used} \\ 0, & \text{otherwise} \end{cases}$$
- **Objective Function (目标函数)**: Minimize the total cumulative distance of the chosen path across the network:$$\text{minimize}_{\{x_{ij}\}} \sum_{(i,j) \in E} w_{ij}x_{ij}$$
- **Constraints** The path selections must ensure binary integrity and maintain network flow balance across all vertices:
    
    1. **Integrity Constraint**:$$x_{ij} \in \{0, 1\}, \quad \forall (i, j) \in E$$
    2. **Source Node Balance**:$$\sum_{j:(s,j) \in E} x_{sj} - \sum_{j:(j,s) \in E} x_{js} = 1$$
        (The net flow leaving the source node $s$ must equal 1 )
    3. **Sink Node Balance**:$$\sum_{j:(j,t) \in E} x_{jt} - \sum_{j:(t,j) \in E} x_{tj} = 1$$
        (The net flow entering the target node $t$ must equal 1 )
    4. **Intermediate Node Balance**:$$\sum_{j:(i,j) \in E} x_{ij} = \sum_{j:(j,i) \in E} x_{ji}, \quad \forall i \neq s, t$$
        (For any intermediate node, the total inflow must strictly equal the total outflow )

- This model naturally defines a **constrained integer optimization problem** due to the discrete binary restrictions. It can be equivalently solved and transformed as a standard continuous **linear programming (LP)** problem.

# 2 Lecture 2: Modeling Techniques & Linear Programming

## 2.1 Foundations of Linear Programming

- **Definition**: An optimization class where the objective and all constraint functions are entirely linear in the decision variables. It is a vital subclass of continuous constrained optimization.
    
- **Introductory Formulation (Compact Form)**:$$\begin{align*} \underset{x \in \mathbb{R}^n} {\text{minimize }} & c^\top x \\ \text{subject to } & A_1 x \ge b \\ & A_2 x \le d \\ & A_3 x = e \\ & x_i \ge 0, \ \forall i \in n_1; \quad x_i \le 0, \ \forall i \in n_2; [cite_start]\quad x_i \text{ free}, \ \forall i \in n_3 \end{align*}$$
## 2.2 Standard Form of LPs

- **Definition**: An LP is in standard form if it is expressed as:
    $$\begin{align*} \underset{x \in \mathbb{R}^n} {\text{minimize }} & c^\top x \\ \text{subject to } & Ax = b \\ & x \ge 0 \end{align*}$$
    where $A$ is an $m \times n$ matrix with $m < n$, and $b \in \mathbb{R}^m$.

> Any LP can be written in the standard form.

## 2.3 Standard Form Transformation Techniques

1. **Maximization Goal**: Convert $\text{maximize } c^\top x$ into $\text{minimize } -c^\top x$.
    
2. **Inequality Constraints ($\le$ or $\ge$)**: Eliminate inequalities by introducing non-negative **slack variables** ($s \ge 0$，松弛变量) to form $Ax + s = b$ or $Ax - s = b$.
    
3. **Non-positive Variables ($x_i \le 0$)**: Replace $x_i$ with $-y_i$ and enforce $y_i \ge 0$.
    
4. **Free Variables ($x_i$ unconstrained)**: Replace $x_i$ with the difference of two non-negative variables ($x_i = x_i^+ - x_i^-$) where $x_i^+ \ge 0, x_i^- \ge 0$.

 **Example P1**

$$\begin{align*} \text{maximize}_{x_1, x_2} \quad & x_1 + 2x_2 \\ \text{subject to} \quad & x_1 \le 100 \\ & 2x_2 \le 200 \\ & x_1 + x_2 \le 150 \\ & x_1, x_2 \ge 0 \end{align*}$$
**Standard Form P2**

$$\begin{align*} \text{minimize}_{x_1, x_2, s_1, s_2, s_3} \quad & -x_1 - 2x_2 \\ \text{subject to} \quad & x_1 + s_1 = 100 \\ & 2x_2 + s_2 = 200 \\ & x_1 + x_2 + s_3 = 150 \\ & x_1, x_2, s_1, s_2, s_3 \ge 0 \end{align*}$$
Given $(x_1^*, x_2^*)$ of $P_1$, $(x_1^*,\space x_2^*, \space s_1^* = 100 - x_1^*, \space  s_2^* = 200 - 2x_2^*, \space s_3^* = 150 - x_1^* - x_2^*)$ is feasible for $P_2$. 

## 2.4 Minimax Problem Transformation

### 2.4.1 Example: Air Traffic Control

An air traffic controller needs to control the landing times of $n$ aircrafts: 

- Flights must land in the order $1, ..., n$.
- Flight $j$ must land in time interval $[aj ,\space bj ]$. 
- The objective is to maximize the minimum separation time, which is the interval between two landings.

**Decision variables:** Let $t_j$ be the landing time of the flight $j$. Then the seperation time in between: $t_j - t_{j-1}$.

**Optimization problem (P1):** 
$$\begin{align}
\underset{t \in \mathbb{R}^n} { \text{maximize}} & \min_{j = 1,\dots,n-1}\{ t_{j+1} - t_j\}  \\
\text{s.t.} & \quad a_j \leq t_j \leq b_j, && j = 1,\dots,n \\
& \quad t_j \leq t_{j+1}, && j = 1,\dots, n-1 \\
\end{align}
$$
### 2.4.2 **Solution:**

We can reformulate it into a **linear program** by introducing an auxiliary variable ΔΔ that acts as a lower bound for every separation time.

**The equivalent linear program (P₂)** is
$$
\begin{aligned}
\underset{t,\;\Delta}{\text{maximize}} &\quad \Delta \\
\text{s.t.} &\quad a_j \le t_j \le b_j, && j = 1,\dots,n \\
&\quad t_j \le t_{j+1}, && j = 1,\dots,n-1 \\
&\quad \Delta \le t_{j+1} - t_j, && j = 1,\dots,n-1.
\end{aligned}$$


We now prove that the optimal value $(z^*)$ of the original problem equals the optimal value $(\Delta^*)$ of this LP.  
The idea is a **two‑sided inequality** (squeeze method).

**Proof**

**Step 1: $z^* \le \Delta^*$**

Let $t^*$ be an optimal landing sequence for the original problem. Its objective value is  
$$
z^* = \min_{j=1,\dots,n-1} \bigl( t_{j+1}^* - t_j^* \bigr).
$$
Construct a candidate for the LP by taking the same times $t^*$ and setting  
$$
\Delta_0 = \min_{j} \bigl( t_{j+1}^* - t_j^* \bigr) = z^*.
$$
- The time‑window and ordering constraints are satisfied because $t^*$ is feasible.  
- By definition of the minimum, $\Delta_0 \le t_{j+1}^* - t_j^*, \forall j$.

Hence $(t^*, \Delta_0)$ is a **feasible solution** of the LP. Its objective value is $\Delta_0 = z^*$.  
Since $\Delta^*$ is the **maximum** achievable value over all feasible solutions of the LP, we must have  
$$
z^* = \Delta_0 \le \Delta^*.
$$

**Step 2: $\Delta^* \le z^*$**

Now let $(t^*, \Delta^*)$ be an optimal solution of the LP, so $\Delta^*$ is the optimal value.  
From the LP constraints,                                                                
$$
\Delta^* \le t_{j+1}^* - t_j^* \quad \forall\, j.
$$
Therefore $\Delta^*$ is a lower bound for every gap, and in particular  
$$
\Delta^* \le \min_{j} \bigl( t_{j+1}^* - t_j^* \bigr).
$$
The time vector $t^*$ satisfies the original constraints, so it is a feasible solution of the original problem.  
Its objective value in that problem is $m = \min_j (t_{j+1}^* - t_j^*)$.  
Because $z^*$ is the **maximum** possible value of this minimum gap, we have  
$$
m \le z^*.
$$
Chaining the inequalities yields  
$$
\Delta^* \le m \le z^*.
$$
---
**Conclusion**
From Step 1 and Step 2 we obtain  
$$
\begin{cases}
& z^* \le \Delta^* \\
& \Delta^* \le z^*
\end{cases}
\quad \Rightarrow \quad z^* = \Delta^*
$$
The two optimization problems yield identical optimal values, proving that the non-smooth maximin problem $(P_1)$ can be rigorously solved via the linear program $(P_2)$. $\blacksquare$


### 2.4.3 ==General Modeling Tool Lemma==

Consider the original optimization problem (2) and its reformulated counterpart (3):

$$\begin{align*} (2) \quad \text{minimize}_{x \in \Omega} \quad & \sum_{i=1}^m f_i(x) \\ (3) \quad \text{minimize}_{x \in \Omega, \ t \in \mathbb{R}^m} \quad & \sum_{i=1}^m t_i \\ \text{subject to} \quad & t_i \ge f_i(x), \quad i = 1, \dots, m \end{align*}$$
**Proof** 

Let $v^*$ and $w^*$ denote the optimal objective values of $(2)$ and $(3)$, respectively. We prove equivalence ($v^* = w^*$) via a two-sided inequality.

**Step 1: $v^* \ge w^*$**

- Let $x^*$ be an optimal solution to $(2)$, meaning its optimal objective value is $v^* = \sum_{i=1}^m f_i(x^*)$.
    
- We can construct a candidate solution for $(3)$ by choosing the same $x^*$ and setting $t_i^* = f_i(x^*)$ for all $i = 1, \dots, m$.
    
- This constructed point $(x^*, t^*)$ is clearly feasible for $(3)$ because it satisfies $t_i^* \ge f_i(x^*)$ with strict equality.
    
- The resulting objective value in $(3)$ is $\sum_{i=1}^m t_i^* = \sum_{i=1}^m f_i(x^*) = v^*$. Since $w^*$ is the global **minimum** value of $(3)$, any feasible solution's value must be greater than or equal to $w^*$:$$v^* \ge w^*$$

**Step 2: $v^* \le w^*$**

- Let $(x^*, t^*)$ be an optimal solution to $(3)$, yielding the optimal objective value $w^* = \sum_{i=1}^m t_i^*$.
    
- To minimize the sum $\sum_{i=1}^m t_i$ subject to $t_i \ge f_i(x)$, the optimization process forces each $t_i$ down to its lowest possible bound. Thus, at optimality, the constraints must bind exactly:$$t_i^* = f_i(x^*), \quad \forall i = 1, \dots, m$$
- This implies that the optimal value of $(3)$ can be written as $w^* = \sum_{i=1}^m f_i(x^*)$.
    
- Since $x^* \in \Omega$, it constitutes a valid feasible solution for the original problem $(2)$. Because $v^*$ is the global **minimum** value of $(2)$, it cannot exceed the value produced by this feasible point:$$v^* \le \sum_{i=1}^m f_i(x^*) = w^* \implies v^* \le w^*$$
**Geometric Interpretation: The Epigraph**

The fundamental reason this transformation works relies on the geometric concept of an **epigraph** (上镜图, the set of points lying on or above the graph of a function).

- The constraint $t_i \ge f_i(x)$ defines the epigraph of the sub-function $f_i(x)$, denoted as $\text{epi}(f_i) = \{(x, t_i) : t_i \ge f_i(x)\}$.
    
- **Linearization Condition**: Problem $(3)$ becomes a standard **Linear Program (LP)** if and only if the epigraph of every function $f_i(x)$ can be completely described by a finite set of linear inequalities.
    
- This requirement is perfectly satisfied when $f_i(x)$ is a convex piecewise-linear function (such as $|x|$ or $\max_k \{c_k^\top x + d_k\}$). In these cases, the epigraph forms a convex polyhedron, allowing non-smooth optimization problems to be solved seamlessly using standard LP algorithms.

> 核心结论：凸分段线性函数的上镜图 = 多面体。因为对于凸分段线性函数 $f(x)$，不等式 $t \geq f(x)$ 等价于**有限个线性不等式的组合**。

> 如果要最小化一堆凸分段线性函数的和，这个问题本质上就是一个线性规划问题。只需要用上镜图变换，引入辅助变量 $t_i$，把每个 $f_i(x) \leq t_i$ 拆成线性不等式，就能用任何线性规划求解器轻松解决。

### 2.4.4 Lemma Application: Minimax and Absolute Values

#### 2.4.4.1 General Minimax Objective

- **Connection to the Lemma**:
    
    The Minimax problem can be viewed as a single-term instance ($m=1$) of the lemma, where the sub-function is defined as the maximum of a collection of linear functions: $f(x) = \max_{i=1,\dots,p} \{c_i^\top x + d_i\}$.
    
- **Original Problem ($P_{\text{original}}$)**:
    $$\begin{align*} \text{minimize}_{x \in \mathbb{R}^n} \quad & \max_{i=1,\dots,p} \{c_i^\top x + d_i\} \\ \text{subject to} \quad & Ax = b, \quad x \ge 0 \end{align*}$$
    
- **Reformulated Linear Program ($P_{\text{LP}}$)**:
    $$\begin{align*} \text{minimize}_{x \in \mathbb{R}^n, \ y \in \mathbb{R}} \quad & y \\ \text{subject to} \quad & Ax = b, \quad x \ge 0 \\ & y \ge c_i^\top x + d_i, \quad \forall i = 1, \dots, p \end{align*}$$
    
#### 2.4.4.2 Absolute Value Minimization ($\ell_1$-Norm)

- **Connection to the Lemma**:
    
    When minimizing a sum of absolute values $\sum |c_i^\top x + d_i|$, each sub-function is  mapped to $f_i(x) = |c_i^\top x + d_i|$. According to the lemma, we assign an individual slack variable $t_i$ to each term such that $t_i \ge |c_i^\top x + d_i|$.
    
- **Original Problem ($P_{\text{original}}$)**:
    $$\begin{align*} \text{minimize}_{x \in \mathbb{R}^n} \quad & \sum_{i=1}^m |c_i^\top x + d_i| \\ \text{subject to} \quad & Ax = b, \quad x \ge 0 \end{align*}$$
    
- **Reformulated Linear Program ($P_{\text{LP}}$)**:
    $$\begin{align*} \text{minimize}_{x \in \mathbb{R}^n, \ t \in \mathbb{R}^m} \quad & \sum_{i=1}^m t_i \\ \text{subject to} \quad & Ax = b, \quad x \ge 0 \\ & c_i^\top x + d_i \le t_i, \quad \forall i = 1, \dots, m \\ & -(c_i^\top x + d_i) \le t_i, \quad \forall i = 1, \dots, m \end{align*}$$

#### 2.4.4.3 Non-Convex Counterexample: Maximizing Absolute Values

While _minimizing_ absolute values is perfectly linearizable, **maximizing** absolute values cannot be modeled as a Linear Program. This serves as a vital counterexample showing the limits of LP reformulation techniques.

- **The Core Mathematical Failure**:
    
    If we attempt to maximize $|x|$ by introducing a bounding variable $t$, we require $t$ to act as a _lower bound_ (a floor): $\text{maximize } t \quad \text{subject to } t \le |x|$
    The inequality $t \le |x|$ translates logically into:
    $$t \le x \quad \mathbf{OR} \quad t \le -x$$
- **Geometric Consequence**:
    
    In optimization, "AND" constraints represent the intersection of sets  (preserving convexity), whereas "OR" constraints represent the **union** of sets. The union of these two geometric regions creates a non-convex feasible set with a disjointed or "hollowed-out" center.

### 2.4.5 Remarks:

**Convex Set**

A set $S \subset \mathbb{R}^n$ is **convex** if and only if:

> For any two points $x, y \in S$ and any scalar $\lambda \in [0, 1]$, their **convex combination** $\lambda x + (1-\lambda) y$ also belongs to $S$.

**Intuition**: The entire line segment connecting any two points in the set lies entirely within the set. A convex set has no holes, no disconnected parts, and no indentations.

 **Closed Half-Space**
 
A **closed half-space** is the set of all points satisfying a single linear inequality:
$$
H = \left\{ x \in \mathbb{R}^n \mid a^T x \leq b \right\}
$$
where $a \in \mathbb{R}^n$ is a non-zero vector (called the **normal vector**) and $b \in \mathbb{R}$ is a scalar.

Since every half-space is a convex set, and the intersection of convex sets remains convex, **the intersection of any number of half-spaces is necessarily a convex set**.

> 在优化中，每一个线性不等式都对应一个半空间（half-space）。AND 操作 = 集合的交集，OR 操作 = 集合的并集。
> 交集（AND）保持凸性：任意多个凸集的交集仍然是凸集；并集（OR）破坏凸性：两个凸集的并集几乎总是非凸的。
# 3 Lecture 3: Linear Programming: Geometry

## 3.1 Graphical Solutions and Geometric Observations (图形法与几何观察)

- **Graphical Method**: Solved by mapping the feasible region and shifting level lines ($c^\top x = c$) to find the boundary point that optimizes the objective.
    
- **Key Visual Observations**: The feasible region forms a geometric polyhedron , the optimal solution tends to occur at a vertex or corner , and constraints can be categorized as active or inactive at the optimum.
    

## 3.2 Polyhedrons and Convexity Definitions (多面体与凸性定义)

- **Polyhedron**: A set that can be mathematically expressed in the form $\{x \in \mathbb{R}^n : Ax \ge b\}$. The standard form constraint set is proven to be a polyhedron.
    
- **Convex Set**: A set $S$ is convex if, for any two points $x, y \in S$, the entire connecting line segment lies within the set ($\lambda x + (1-\lambda)y \in S$ for all $\lambda \in [0,1]$).
    
- **Convex Combination**: A weighted sum $\sum_{i=1}^n \lambda_i x_i$ where all weights satisfy $\lambda_i \ge 0$ and $\sum_{i=1}^n \lambda_i = 1$.
    
- **Extreme Point (Vertex/Corner)**: A point $x$ within a polyhedron $P$ is an extreme point if it cannot be expressed as a convex combination of any two other distinct points in $P$.
    

## 3.3 Algebraic Representation: Basic Solutions (代数表示：基本解)

- **Standard Assumptions**: In the standard form LP, matrix $A$ is assumed to possess linearly independent rows (full row rank $m$). Failing this implies either redundant or inconsistent constraints.
    
- **Basic Solution (BS) Definition**: A vector $x$ is a basic solution if and only if $Ax = b$ and there exist $m$ indices $B(1), \dots, B(m)$ such that their corresponding columns in $A$ are linearly independent, and all other non-basic entries $x_i = 0$.
    
- **Procedure to Calculate a Basic Solution**:
    
    1. Choose $m$ linearly independent columns of $A$ to form the basis matrix $A_B$.
        
    2. Enforce all non-basic variables to equal 0 ($x_N = 0$).
        
    3. Solve the remaining linear system to find the unique basic variables: $x_B = A_B^{-1}b$.
        
- **Properties of Basic Solutions**: A basic solution is solely determined by the constraints and is independent of the objective function. The number of non-zero entries cannot exceed $m$. The maximum possible number of basic solutions is finite, bounded by the combination formula $C(n,m) = \frac{n!}{m!(n-m)!}$.
    

## 3.4 Basic Feasible Solutions (BFS) (基本可行解)

- **Definition**: A Basic Solution $x$ that additionally satisfies the non-negativity constraint $x \ge 0$.
    
- **The Equivalence Theorem**: For a standard form LP polyhedron, a point $x$ is a geometric **extreme point** if and only if it is an algebraic **basic feasible solution (BFS)**.
    

## 3.5 Fundamental Theorem of Linear Programming (线性规划基本定理)

- **Theorem Statements**:
    
    1. If the standard form LP feasible set is nonempty, there must exist at least one basic feasible solution.
        
    2. If the problem possesses an optimal solution, there must be at least one optimal solution that is also a basic feasible solution.
        
- **Crucial Implication**: To search for an optimal solution, algorithms only need to inspect the finite set of basic feasible solutions. Any LP with an optimal solution guarantees an optimal point containing at most $m$ positive entries.
    
- **Motivation for the Simplex Method**: Although the number of BFS is finite, checking them all via brute-force is computationally impossible for large dimensions because $C(n,m)$ grows exponentially (e.g., $C(1000, 100) \approx 10^{143}$). This creates the direct need for an intelligent search strategy: **The Simplex Method**.



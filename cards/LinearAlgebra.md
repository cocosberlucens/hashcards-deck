---
name = "Linear Algebra"
---

<!-- Bite 1: Vectors & Arrays — The ML Perspective — 2025-01-07 -->

Q: Why is "every row of data is a vector" a useful mental model for ML?
A: Because it makes ML geometric. A patient with [age=65, LOS=4, cost=12000]
is a point in 3D feature space. "Similar patients" means nearby points.
Distance-based algorithms (k-NN, K-Means, Linear Regression) all exploit
this geometry — similarity IS proximity in feature space.

Q: Why does vector magnitude become unreliable when features have different scales?
A: Because $\|\mathbf{v}\| = \sqrt{\sum v_i^2}$ sums squared components.
If cost is in thousands and age in decades, cost dominates the sum regardless
of clinical significance. A patient [50, 50000] has almost the same magnitude
as [0, 50000]. Feature scaling before distance computation is essential.

Q: Why does cosine similarity remove the effect of magnitude?
A: Because it divides by both magnitudes:
$\cos(\theta) = \frac{\mathbf{a} \cdot \mathbf{b}}{\|\mathbf{a}\|\|\mathbf{b}\|}$.
This normalizes both vectors to unit length, leaving only the angle between them.
Two patients with proportionally identical profiles (e.g., [10, 100] and [20, 200])
have cosine similarity 1 regardless of absolute values.

C: The [dot product] $\mathbf{a} \cdot \mathbf{b} = \sum a_i b_i = \|\mathbf{a}\|\|\mathbf{b}\|\cos(\theta)$ measures [alignment]: positive when vectors point similarly, zero when [perpendicular], negative when opposing.

C: A [unit vector] $\hat{\mathbf{v}} = \frac{\mathbf{v}}{\|\mathbf{v}\|}$ has magnitude [one] — it preserves [direction] but discards scale.

Q: Why is orthogonal vector decomposition the geometric foundation of regression?
A: It splits a vector into two components: the projection onto another vector
(the part "explained by" it) and the perpendicular residual. In regression,
predicted values are the projection of $\mathbf{y}$ onto the column space of $X$,
and residuals are the orthogonal remainder. Minimizing residuals IS finding the
closest point in the column space.

Q: When would you use Euclidean distance over cosine similarity, and vice versa?
A: Euclidean distance when absolute values matter — two patients are "close"
if they have similar age AND similar cost in actual units.
Cosine similarity when only the pattern/ratio matters — two patients are
"similar" if their feature profiles have the same shape, regardless of scale.
Example: [10, 100] and [20, 200] have cosine similarity 1 but Euclidean
distance ~112.

<!-- Bite 2: Matrices & DataFrames — 2025-01-08 -->

Q: Why is matrix multiplication not commutative ($AB \neq BA$)?
A: Because the two products have different semantic meanings. $AB$ computes
weighted combinations of B's rows using A's rows as weights; $BA$ does the
reverse. The shape rule alone shows this: $(m \times n)(n \times p) = m \times p$
but $(n \times p)(m \times n)$ may not even be valid. Even for square matrices
where both products exist, they represent different transformations applied
in different order.

Q: What are the four ways to understand matrix multiplication $C = AB$?
A: 1. Row $\times$ Column $\to$ Number: $c_{ij}$ is the dot product of row $i$
of A with column $j$ of B.
2. Matrix $\times$ Column $\to$ Column: each column of C is a linear combination
of A's columns, weighted by the corresponding column of B.
3. Row $\times$ Matrix $\to$ Row: each row of C is a linear combination of
B's rows, weighted by the corresponding row of A.
4. Column $\times$ Row $\to$ Matrix: C is the sum of rank-1 outer products
$\sum_k \mathbf{a}_{:,k}\, \mathbf{b}_{k,:}$.

Q: Why does $(AB)^T = B^T A^T$ — the "LIVE EVIL" rule?
A: Transpose swaps rows and columns. If $AB$ dot-products rows of A with columns
of B, then $(AB)^T$ must dot-product rows of $B^T$ with columns of $A^T$ — which
is $B^T A^T$. The reversal generalizes to any chain: $(ABC)^T = C^T B^T A^T$.
The mnemonic: reading "LIVE" backwards gives "EVIL".

Q: Why is $A^T A$ always symmetric, and why does this matter?
A: Because $(A^T A)^T = A^T (A^T)^T = A^T A$ by LIVE EVIL — the matrix equals
its own transpose. This is why covariance matrices ($X_c^T X_c / (n-1)$) are
always symmetric: they are necessarily of the form $M^T M$.

C: Matrix multiplication $(m \times [n]) \cdot ([n] \times p) = (m \times p)$ requires the [inner dimensions] to match; the result has the [outer dimensions].

C: The [Hadamard product] $A \odot B$ multiplies [corresponding elements], requiring matrices of [identical shape]. In NumPy: `A * B` (not `A @ B`).

C: A [symmetric] matrix can be created from any square matrix via $\frac{A + [A^T]}{2}$, which averages each element with its [transpose] counterpart.

Q: How do k-NN, K-Means, and Linear Regression form a "compression gradient"?
A: All three are distance-based, but differ in how much they compress training data:
k-NN — zero compression: stores all training points, compares every feature.
K-Means — moderate: reduces data to k centroids, still concrete points in
feature space.
Linear Regression — maximum: reduces data to learned weights, abstract parameters
unrelated to any single data point.
The gradient runs from memorization to abstraction.

<!-- Bite 3: Aggregation & Summary Statistics — 2025-01-18 -->

Q: Why is the Frobenius norm called "the Euclidean norm for matrices"?
A: Because it treats the matrix as a flattened vector and computes
$\|A\|_F = \sqrt{\sum_{i,j} a_{ij}^2}$ — exactly the Euclidean formula applied
to all elements. Just as the vector norm measures "how big" a vector is,
Frobenius norm measures "how big" a matrix is. This makes $\|A - B\|_F$
a natural element-wise distance between matrices.

Q: Why is $\|A - B\|_F \neq |\|A\|_F - \|B\|_F|$ — the classic norm trap?
A: Frobenius distance $\|A - B\|_F$ measures element-wise differences: every
cell contributes. The difference of norms $|\|A\|_F - \|B\|_F|$ only asks
whether the matrices have similar total "size". Two matrices with identical
Frobenius norms but completely different elements have norm-difference zero
but large Frobenius distance. The triangle inequality guarantees
$\|A - B\|_F \geq |\|A\|_F - \|B\|_F|$, with equality only in degenerate cases.

C: The [Frobenius norm] $\|A\|_F = \sqrt{\sum_{i,j} a_{ij}^2}$ can also be computed as $\sqrt{[{\text{trace}}](A^T A)}$, connecting matrix norms to the [diagonal sum] of the Gram matrix.

C: The [trace] of a square matrix is the sum of its [diagonal] elements. It has the [cyclic] property: $\text{trace}(AB) = \text{trace}(BA)$, even when $AB \neq BA$.

Q: What do the diagonals of $A^T A$ and $A A^T$ contain, and why does the trace of both equal $\|A\|_F^2$?
A: $A^T A$'s diagonal element $i$ is $\|\mathbf{a}_{:,i}\|^2$ (squared column norm).
$A A^T$'s diagonal element $i$ is $\|\mathbf{a}_{i,:}\|^2$ (squared row norm).
Both traces equal $\sum_{i,j} a_{ij}^2 = \|A\|_F^2$ because summing all squared
elements gives the same total whether you group by column or by row.

Q: Why does "axis = axis I want to collapse" work as a mental model for NumPy aggregations?
A: Because NumPy's axis parameter names the dimension that disappears from the
shape tuple. For a (5, 3) matrix: axis=0 removes dimension 0 (the 5 rows)
leaving shape (3,) — one value per column. axis=1 removes dimension 1
(the 3 columns) leaving shape (5,) — one value per row. The axis number
IS the index into the shape tuple that gets deleted.

<!-- Bite 0.3: NumPy Arrays — The Paradigm Shift — 2026-04-23 -->

Q: What does Pearson correlation measure that cosine similarity misses?
A: Pearson is cosine similarity computed on *mean-centered* vectors:
$\rho = \frac{\tilde{\mathbf{a}} \cdot \tilde{\mathbf{b}}}{\|\tilde{\mathbf{a}}\| \|\tilde{\mathbf{b}}\|}$
where $\tilde{\mathbf{v}} = \mathbf{v} - \bar{v}$. Cosine similarity sees the
absolute directions of $\mathbf{a}$ and $\mathbf{b}$ in feature space; Pearson
sees the directions of their *deviations from the mean*. So two vectors with very
different absolute values but identical fluctuation patterns have $\rho = 1$ but
lower cosine similarity. Pearson captures "co-fluctuation" — linear relationship
between variables — which is why it's the standard measure of statistical
correlation; cosine captures "shape similarity" without de-trending.

<!-- Bite 0.3 — Notation backfill: 2026-04-28 -->

Q: What does $\|\mathbf{v}\|$ denote, and why the double bars?
A: It's the Euclidean (L2) norm — the magnitude or length of vector $\mathbf{v}$,
computed as $\|\mathbf{v}\| = \sqrt{\sum_i v_i^2}$. This generalizes 2D Pythagorean
distance to $n$ dimensions. Double bars distinguish vector norm from absolute value
$|x|$ (a scalar operation). The subscript form $\|\mathbf{v}\|_2$ makes the "2"
in "L2" explicit, contrasting with $\|\mathbf{v}\|_1 = \sum_i |v_i|$ (Manhattan/L1)
and $\|\mathbf{v}\|_\infty = \max_i |v_i|$ (Chebyshev). When unsubscripted, $\|\cdot\|$
defaults to L2 by convention. The matrix analog $\|A\|_F$ (Frobenius) extends the
same idea to rectangular shapes.

Q: What does the hat in $\hat{\mathbf{v}}$ signify?
A: $\hat{\mathbf{v}}$ is the unit vector in the direction of $\mathbf{v}$ — same
direction, magnitude exactly 1. Defined as $\hat{\mathbf{v}} = \mathbf{v}/\|\mathbf{v}\|$,
the operation strips scale and preserves direction. The hat is the standard visual
marker for "normalized to length 1" — that's why standard basis vectors are written
$\hat{\mathbf{i}}, \hat{\mathbf{j}}, \hat{\mathbf{k}}$. Useful for cosine similarity
($\cos\theta = \hat{\mathbf{a}} \cdot \hat{\mathbf{b}}$) and any context where pattern
matters more than magnitude. Caveat: in statistics, $\hat{\theta}$ instead means
"estimated value of parameter $\theta$" — same hat, different convention.

Q: What does the centered dot in $\mathbf{a} \cdot \mathbf{b}$ between two bold letters denote?
A: The dot product (also called inner or scalar product) of two vectors —
algebraically $\sum_i a_i b_i$, geometrically $\|\mathbf{a}\|\|\mathbf{b}\|\cos\theta$.
The output is a SCALAR, despite combining two vectors. Distinguish from $2 \cdot 3$
(plain scalar multiplication — same dot symbol but operands aren't vectors) and from
$\mathbf{a} \times \mathbf{b}$ (cross product, which returns a vector in 3D). The dot
product measures alignment: positive when vectors point similarly, zero when
perpendicular, negative when opposing. Bold typesetting on the operands is the
visual cue: dot between bold letters = vector dot product.

Q: What does the tilde in $\tilde{\mathbf{v}}$ typically mean in vector/data contexts?
A: The tilde marks $\tilde{\mathbf{v}} = \mathbf{v} - \bar{v}$ — the mean-centered
version of $\mathbf{v}$, with each component shifted by the vector's mean so the
result has mean 0. This is the operation that turns cosine similarity into Pearson
correlation: $\rho(\mathbf{a},\mathbf{b}) = \cos(\theta_{\tilde{\mathbf{a}},\tilde{\mathbf{b}}})$.
Caveat: tilde is overloaded across math — it can also mean "approximately"
($a \approx b$ sometimes written $a \tilde{} b$), "distributed as" in probability
($X \sim \mathcal{N}$), or "equivalent to" in algebra. Context is mandatory; in
data/regression contexts, "centering" is the dominant reading.

Q: How do you read $\sum_{i=1}^{n} x_i$, and what do the index, lower limit, and upper limit mean?
A: "Sum over $i$ from 1 to $n$ of $x_i$." The $\sum$ symbol is uppercase Greek
sigma; the subscript $i=1$ initializes the dummy index, the superscript $n$ is the
(inclusive) terminal value, and $i$ takes each integer between. Equivalent to a
Python `for` loop accumulating into a total. Three conventions to know: (1) the
index name is arbitrary — $\sum_j, \sum_k$ are interchangeable; (2) when the range
is unambiguous from context, math papers omit the limits — just $\sum_i x_i$ or
$\sum x_i$; (3) the upper limit is INCLUSIVE (unlike Python's `range(n)`, which
stops at $n-1$). Building block of definitions like $\|\mathbf{v}\| = \sqrt{\sum_i v_i^2}$
and $\mathbf{a} \cdot \mathbf{b} = \sum_i a_i b_i$.

<!-- Bite 1: Vectors & Arrays — Notation backfill: 2026-04-28 -->

Q: Why are vectors typeset bold ($\mathbf{v}$) while their components ($v_i$) are not?
A: Convention: bold uppercase for matrices ($\mathbf{A}$), bold lowercase for
vectors ($\mathbf{v}$), italic non-bold for scalars and individual components
($v_i$, $\theta$, $n$). The visual distinction is critical when the same letter
denotes both: $v_i$ is the $i$-th scalar component of vector $\mathbf{v}$. So in
$\mathbf{a} \cdot \mathbf{b} = \sum_i a_i b_i$, boldness on the left signals
"whole vectors combining into a scalar"; non-bold on the right signals
"individual numbers being summed." Variants you'll see: arrow notation
$\vec{v}$ (more common in physics texts and on chalkboards where bold is hard);
blackboard bold for sets ($\mathbb{R}^n$, $\mathbb{Z}$); calligraphic for spaces
or sets ($\mathcal{X}$, $\mathcal{N}$). Some ML papers are sloppy and drop the
bold — usually no real ambiguity, but it's a paper-quality smell, not a
notation choice you should imitate.

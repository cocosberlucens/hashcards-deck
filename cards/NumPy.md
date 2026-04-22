---
name = "NumPy"
---

<!-- Detour: ps4ds Ch. 2 Discrete Variables — 2026-02-27 -->

Q: Why is np.unique(data, return_counts=True) the natural way to compute an empirical PMF?
A: It returns sorted unique values and their occurrence counts — dividing counts
by n gives relative frequencies, exactly $\hat{p}(a) = \text{count}(a)/n$. It's
natural because the PMF is defined over distinct values (the support), not over
a continuous range. Missing values correctly get no entry (probability zero).

Q: Why use scipy.stats.poisson.pmf(k, mu=lambda_hat) rather than the empirical PMF for prediction?
A: The Poisson PMF assigns probability to all non-negative integers, including
values not yet observed in the data. The empirical PMF assigns zero to unobserved
values — overconfident that they can't happen. With small samples, the parametric
model's structural assumption (independent events at constant rate) is more
informative than raw counts alone.

<!-- Bite 1: Vectors & Arrays — The ML Perspective — 2025-01-07 -->

Q: Why does `A @ B.T` compute all pairwise dot products between row vectors?
A: Each element $(i,j)$ of $AB^T$ is the dot product of row $i$ of A with row $j$
of B — because row $j$ of B becomes column $j$ of $B^T$. For patient vectors stored
as rows, this gives the full $(n \times n)$ pairwise similarity matrix in one
vectorized operation, no loops needed.

Q: How does the vectorized cosine similarity matrix work: `(A @ A.T) / (norms @ norms.T)`?
A: The numerator `A @ A.T` gives all pairwise dot products. The denominator uses
norms with shape (n, 1) from keepdims=True: the outer product `(n,1) @ (1,n) = (n,n)`
gives all pairwise products $\|\mathbf{a}_i\| \cdot \|\mathbf{a}_j\|$. Element-wise
division yields the full cosine similarity matrix. Use `np.fill_diagonal(sim, -np.inf)`
before `np.argmax` to exclude self-similarity when finding the most similar pair.

<!-- Bite 2: Matrices & DataFrames — 2025-01-08 -->

Q: Why does NumPy use `@` for matrix multiplication and `*` for Hadamard, and what goes wrong if you confuse them?
A: `A @ B` is matrix multiplication — dot products of rows with columns, requires
inner dimensions to match. `A * B` is Hadamard (element-wise) — multiplies
corresponding elements, requires identical shapes (or broadcastable). Confusing
them is silent: `*` on two (3,3) matrices happily returns a (3,3) result, but
it's element-wise products, not row-column dot products. The `@` operator
(Python 3.5+) exists specifically to make the distinction visible.

<!-- Bite 3: Aggregation & Summary Statistics — 2025-01-18 -->

Q: What are three ways to compute the Frobenius norm in NumPy, and why do they agree?
A: 1. `np.linalg.norm(A, 'fro')` — built-in.
2. `np.sqrt(np.sum(np.square(A)))` — direct from definition $\sqrt{\sum a_{ij}^2}$.
3. `np.sqrt(np.trace(A.T @ A))` — via trace, because $\text{trace}(A^T A)$
sums the squared column norms, which equals the sum of all squared elements.
All three compute the same quantity: the square root of the sum of all
squared entries.

Q: Why pass keepdims=True to aggregations like `np.mean(A, axis=0, keepdims=True)`?
A: Without it, the mean of a (5, 3) matrix along axis=0 gives shape (3,) — a
1D array. With keepdims=True, the result is (1, 3) — preserving the 2D structure.
This makes broadcasting explicit: `A - mean` subtracts column means from each row
because (5, 3) - (1, 3) broadcasts cleanly. Without keepdims, (5, 3) - (3,) also
broadcasts, but the intent is less clear and breaks with higher-dimensional arrays.

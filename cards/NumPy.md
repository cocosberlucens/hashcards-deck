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

<!-- Bite 0.3: NumPy Arrays — The Paradigm Shift — 2026-04-23 -->

Q: Why is NumPy 10-100x faster than Python lists for numerical operations?
A: Python lists are flexible heterogeneous containers — each element is a full
Python object with type info, so `x * 2` requires per-element type dispatch.
NumPy arrays are fixed-type: all elements share one dtype, stored in contiguous
memory. This enables vectorized CPU instructions (SIMD) and eliminates per-element
type checks. The tradeoff: you lose the ability to mix `int`, `str`, `dict` in one
array — which is exactly what makes it fast.

Q: Why does `np.array([1, 2, 3.0])` produce a float64 array when two of three inputs are integers?
A: NumPy arrays are fixed-type, so all elements must share one dtype. When the input
mixes types, NumPy promotes to the *least restrictive* type that can hold every value
without loss — int → float because 1 and 2 can be exactly represented as floats,
but not the reverse. Same rule applies in arithmetic: `int_array + float_array`
yields a float array. The logic mirrors Python's scalar upcasting (`1 + 2.0 == 3.0`)
but now decides the storage type of every element.

Q: What does `np.array([1.9, 2.9, 3.9], dtype=int)` return, and why?
A: `array([1, 2, 3])` — the decimals are *truncated* toward zero, not rounded.
When you force a dtype, NumPy performs a raw cast: float → int drops the fractional
part. To round, use `np.rint(arr).astype(int)` or `np.round(arr).astype(int)`
explicitly. This is the NumPy-level version of Python's `int(1.9) == 1`; the
"force" in `dtype=int` is mechanical, not semantic.

Q: When should you reach for `np.linspace(0, 1, 5)` rather than `np.arange(0, 1, 0.2)`?
A: `linspace` guarantees endpoint inclusion and specifies N evenly-spaced points;
`arange` specifies step size and may drift due to floating-point accumulation —
`np.arange(0, 1, 0.1)` famously returns 10 elements on some systems and 11 on others.
Use `linspace` for plotting grids, integration domains, or any float-valued range
where "exactly N points from a to b inclusive" matters. `arange` is for integer
sequences.

C: In NumPy, a 1D array of length n has shape [(n,)] — with a trailing comma — distinguishing it from the 2D [(n, 1)] column vector and [(1, n)] row vector. For broadcasting, `(n,)` is treated as a [row] (equivalent to `(1, n)`), so reshape is needed to turn a 1D array into a proper column.

C: Every NumPy array exposes four structural attributes: [.shape] (dimensions as a tuple), [.ndim] (number of axes), [.size] (total element count), and [.dtype] (element type). Together they fully describe the array's layout without touching any actual data.

Q: Why is `arr[i, j]` preferred over `arr[i][j]` for a 2D NumPy array?
A: `arr[i, j]` is a single multidimensional lookup — one memory access for element
(i, j). `arr[i][j]` is two operations: first extract row i (creating a 1D view),
then index into it. Both return the same value for single lookups, but the comma
syntax also enables slicing in both dimensions at once: `arr[:, 0]` gets all rows
of column 0, which `arr[:][0]` cannot express (it returns only the first row).
Python lists don't support comma indexing — this is NumPy-specific.

Q: Which NumPy operations return views (shared memory) versus copies (independent)?
A: Views: basic slicing (`arr[1:4]`), `reshape`, `transpose`, `ravel`.
Copies: `.copy()`, `flatten`, fancy indexing (`arr[[0, 2]]`), boolean indexing
(`arr[arr > 5]`). Mutations through a view touch the original; mutations through
a copy don't. Detect with `.base`: `view.base is arr` is True; copies have
`.base is None`. Views exist for efficiency — slicing a 1GB array shouldn't copy
it. Rule of thumb: if in doubt, `.copy()`.

Q: When would you use `arr.ravel()` instead of `arr.flatten()`?
A: `ravel()` returns a VIEW when possible (shared memory, free), `flatten()` always
returns a COPY (allocates new memory, independent). Use `ravel` for read-only
iteration or when you WANT mutations to touch the original. Use `flatten` when
you need an independent flat copy — passing to a function that might mutate, or
when memory isolation matters. The convention: `ravel` is cheaper, `flatten` is safer.

Q: What does the `-1` mean in `arr.reshape(3, -1)` or `arr.reshape(-1, 1)`?
A: "Infer this dimension so the total element count matches." NumPy computes the
missing dimension as `total_size / product_of_other_dims`. For a 12-element array:
`reshape(3, -1)` gives (3, 4); `reshape(-1, 1)` gives (12, 1) — a column vector;
`reshape(1, -1)` gives (1, 12) — a row vector. Exactly one dimension can be `-1`;
using two raises an error because NumPy needs one concrete value to pin the rest.

Q: Why do you write `(ages >= 50) & (ages <= 70)` instead of `ages >= 50 and ages <= 70` on NumPy arrays?
A: `and` and `or` are Python's short-circuit scalar operators — they ask "is this
one array truthy?" and raise ValueError because a multi-element array doesn't have
a single truth value. `&`, `|`, `~` are element-wise bitwise operators, which NumPy
hijacks to compute element-wise logical AND/OR/NOT on boolean arrays. Parentheses
are mandatory because `&` has higher precedence than `>=`: without them,
`ages >= 50 & ages <= 70` parses as `ages >= (50 & ages) <= 70` — nonsense that
may or may not raise.

Q: What does `np.where(cond, a, b)` return, and how does it relate to an if/else?
A: It's a vectorized ternary: returns an array where each element is taken from
`a` where `cond` is True and from `b` where False. Equivalent to
`[ai if ci else bi for ai, bi, ci in zip(a, b, cond)]` but runs in C. Broadcasts
`cond`, `a`, `b` to a common shape. Chain it for multi-way conditionals:
`np.where(ages < 40, 1, np.where(ages < 65, 2, 3))` implements if/elif/else without
any Python loop. The single-argument form `np.where(cond)` returns the INDICES
where cond is True — a different tool.

Q: When you call `np.sum(arr, axis=0)` on a 2D array of shape (5, 3), what's the result's shape and what's in position i?
A: Shape (3,) — the axis-0 dimension (the 5 rows) collapses, leaving the axis-1
dimension (the 3 columns). Position i contains the sum of column i across all 5
rows. Mnemonic: "axis = the dimension that disappears from the shape tuple."
axis=0 sums down columns (one value per column); axis=1 sums across rows (one value
per row); no axis collapses everything to a scalar.

Q: For arrays with shapes (3, 4) and (4,), does broadcasting succeed — and if so, what's the result shape?
A: Yes — result is (3, 4). Broadcasting rules: compare shapes right-to-left;
dimensions are compatible if equal OR one is 1 OR missing (missing is treated as
1 prepended on the left). So (4,) is treated as (1, 4), which broadcasts with
(3, 4) by stretching the leading 1 to 3. In contrast, (3, 4) with (3,) FAILS —
(3,) becomes (1, 3), and 3 ≠ 4. Rule of thumb: a 1D array of length m aligns with
the LAST dimension of the other array; to align with the first, reshape to (n, 1).

Q: Why does `data - data.mean(axis=0)` correctly center each column of a 2D array?
A: `data.mean(axis=0)` returns shape (n_cols,) — one mean per column. Broadcasting
treats this as (1, n_cols), which stretches across all rows of `data` (shape
(n_rows, n_cols)). Each element is subtracted from its column's mean, not the
global mean. The pattern extends to z-scoring:
`(data - data.mean(axis=0)) / data.std(axis=0)`. For clearer intent and safety in
higher-dimensional cases, add `keepdims=True` to the aggregations.

Q: When `X` has shape (n, m) and `v` has shape (m,), what does `X @ v` compute and why is it useful?
A: Each row of `X` gets dot-producted with `v`, producing an array of shape (n,) —
n dot products in one vectorized operation. This is the "compare one vector to many"
pattern: `X` stores n candidates (e.g., patients as rows of features), `v` is a
reference (e.g., ideal profile), and `X @ v` gives a similarity-like score per
candidate. For true cosine similarity, normalize rows first:
`(X / np.linalg.norm(X, axis=1, keepdims=True)) @ (v / np.linalg.norm(v))`.

Q: Why do `np.linalg.norm(v)`, `np.sqrt(np.sum(v**2))`, and `np.sqrt(v @ v)` all return the same value?
A: The Euclidean norm is defined as $\sqrt{\sum v_i^2}$ — the second form spells
this out literally. The dot product $v \cdot v = \sum v_i^2$ IS that sum, because
the dot product of a vector with itself squares each component and adds them. So
the third form computes the sum via matmul, then takes the root. `np.linalg.norm`
is the canonical named wrapper. All three are the same formula; reach for
`np.linalg.norm` for readability, the others when deriving from first principles.

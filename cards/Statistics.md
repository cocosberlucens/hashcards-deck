---
name = "Statistics"
---

<!-- Detour: ps4ds Ch. 2 Discrete Variables — 2026-02-27 -->

C: The [empirical PMF] estimates the true PMF as $\hat{p}_X(a) = \frac{\text{count of } a}{n}$, which is a [normalized histogram] of the observed data.

Q: Why is evaluating a model on its training data meaningless?
A: Because the empirical PMF achieves RMSD = 0 against its own training data — it
memorizes every fluctuation. This is the student-who-saw-the-answers problem from
ps4ds. Only held-out test data reveals whether the model captures the true underlying
distribution or just the noise in the training sample.

Q: Why do parametric models generalize better than nonparametric with small training sets?
A: With limited data the empirical PMF has large random fluctuations and assigns zero
probability to unobserved values. A parametric model (e.g., Poisson) smoothly allocates
probability across all plausible values using just the estimated parameter. The structural
assumption acts as a regularizer — reducing variance at the cost of potential bias.

Q: Why does the empirical PMF eventually catch up to parametric models as data grows?
A: By the law of large numbers, relative frequencies converge to true probabilities
as $n \to \infty$. With enough data every value in the support gets observed multiple
times, so the empirical PMF fills in all gaps. The parametric model's structural
advantage shrinks because the nonparametric model no longer needs it.

C: [RMSD] measures model quality as $\sqrt{\frac{1}{|S|}\sum_{a \in S}(\hat{p}(a) - p_{\text{test}}(a))^2}$, comparing the [model PMF] against the [test set's empirical PMF].

Q: What is the core idea of maximum-likelihood estimation (MLE)?
A: Choose the parameter value that makes the observed data most probable.
For a Poisson model, the MLE of $\lambda$ is simply the sample mean — the value
that maximizes $\prod_i P(X = x_i \mid \lambda)$. MLE is the standard bridge between
data and parametric models.

Q: When would a parametric model be worse than nonparametric, even with limited data?
A: When the parametric assumption is wrong — model misspecification. If the true
distribution is not Poisson but you fit a Poisson, the structural assumption introduces
systematic bias that no amount of data can fix. Nonparametric at least converges to
truth; parametric converges to the best approximation within its assumed family.

<!-- Bite 0.3 — Notation backfill: 2026-04-28 -->

Q: What does the symbol $\rho$ typically denote in statistics, and where does the convention come from?
A: Lowercase Greek rho is the standard symbol for the Pearson correlation coefficient
between two variables, ranging in $[-1, +1]$: $+1$ is perfect positive linear
relationship, $-1$ perfect negative, $0$ no linear relationship. Geometrically it's
cosine similarity of mean-centered vectors:
$\rho = \frac{\tilde{\mathbf{a}} \cdot \tilde{\mathbf{b}}}{\|\tilde{\mathbf{a}}\| \|\tilde{\mathbf{b}}\|}$.
Convention split: $\rho$ usually means the *population* (true) correlation; $r$ means
the *sample* estimate computed from observed data — the same distinction as $\mu$
vs $\bar{x}$ for means. Caveat: $\rho$ is overloaded across math — it also denotes
density (in physics/probability), spectral radius (linear algebra), and the rank
function (matroid theory). Statistical context disambiguates.

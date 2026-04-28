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

<!-- Bite 1: Vectors & Arrays — The ML Perspective — 2026-04-28 -->

Q: What separates continuous from discrete numeric data, and why does the distinction shape what's analyzable?
A: Continuous data can take any value within a range (with arbitrary precision in
principle): cost in dollars and cents, body temperature, time-to-event. Discrete
data takes only specific countable values, typically integers: number of
procedures, number of admissions, length of stay in whole days. The distinction
is operational: continuous variables admit means, standard deviations, and
density-based plots (PDFs, KDEs) without controversy; discrete variables are at
home with PMFs, counts, and stem plots. The line blurs in practice — "age in
years" is recorded discretely but treated continuously without harm. Honest test:
can the value lie strictly between two recorded values, and does that "between"
mean something? If yes, treat it as continuous; if not (admissions, ICD codes),
discrete is more faithful.

Q: How do binary, ordinal, and nominal categorical variables differ, and what operations does each admit?
A: Binary has exactly two categories — admitted/not, M/F, readmit Y/N. Numerically
encoded as 0/1; both order and counts are valid. Ordinal has ordered categories
where the order is meaningful but the SPACING is not — severity (Low/Med/High),
satisfaction (1–5 stars). You can sort and take medians, but "5 is twice 2.5" is
unwarranted because the gaps between adjacent levels need not be equal. Nominal
has unordered categories — diagnosis code, ZIP, blood type, insurance type. Only
counts and mode are valid; sort order is arbitrary. The hierarchy of permitted
operations runs nominal (mode, counts) → ordinal (+ sort, median, percentiles)
→ numeric (+ arithmetic, mean, std). Each level adds operations the simpler one
lacks, and ML pipelines that ignore this hierarchy silently produce nonsense.

Q: Why is `np.mean(zip_codes)` syntactically valid but semantically broken?
A: ZIP codes 10001 and 94102 are stored as integers because they're labels that
happen to be written with digits — the number is an identifier, not a measurement.
Computing 52051.5 as the "mean ZIP" produces no actual location: there is no
"between" relationship that would make the average a halfway point on any
meaningful scale. This generalizes to ANY nominal categorical variable encoded
numerically: diagnosis codes, customer IDs, gender as 0/1, severity stored
as 1/2/3/4 if you forget it's ordinal. NumPy will silently do arithmetic on any
numeric dtype — but the dtype tells you nothing about whether the operation is
meaningful. Whether arithmetic makes sense is a *domain* property (measurement
scale), not a Python type property. Always classify columns by measurement scale
before letting any aggregation run on them.

<!-- Bite 3: Aggregation & Summary Statistics — 2026-04-28 -->

Q: What does the gap between mean and median tell you about a distribution, and why?
A: Mean > median → right-skewed (long right tail pulls the mean up). Mean < median
→ left-skewed (long left tail pulls the mean down). Mean ≈ median → roughly
symmetric. Mechanism: every observation contributes its FULL VALUE to the mean
($\bar{x} = \frac{1}{n}\sum x_i$) but only its RANK to the median (the middle value
when sorted, regardless of magnitude). So an extreme outlier shifts the mean
dramatically while moving the median by at most one rank. In healthcare LOS data,
right-skewed by ICU patients staying weeks, the mean often exceeds the median by
several days — using mean alone misrepresents the typical patient's experience.
Practical rule: report median for skewed continuous data; mean only when symmetry
is plausible or when you genuinely want every observation weighted by magnitude.

Q: When is the trimmed mean preferable to either the plain mean or the median?
A: When you want SOME outlier resistance without throwing away all magnitude
information. The trimmed mean drops the top and bottom $\alpha\%$ of values then
averages the remaining middle — a 10% trim removes the lowest and highest 10%.
It sits between the mean (uses every value, sensitive to tails) and the median
(uses only the rank-middle, throws away magnitude entirely). Useful when: lab
measurements have occasional sensor errors you want survived but not dominant; LOS
data has a few extreme stays that distort the mean while the median throws away too
much information about typical-but-non-median patients. Trade-off: $\alpha$ is
discretional — report it explicitly. SciPy: `scipy.stats.trim_mean(data, 0.1)`.
Caveat: with very small samples, even small $\alpha$ can throw away meaningful
structure, so use sparingly below $n \approx 30$.

Q: For right-skewed data, why does standard deviation typically OVERSTATE the spread compared to IQR, and which is more honest?
A: Standard deviation squares each deviation from the mean
($s = \sqrt{\frac{1}{n-1}\sum (x_i - \bar{x})^2}$). Squaring amplifies large
deviations: a single observation ten times further from the mean contributes 100×
more to $s^2$ than one at typical distance. Outliers therefore dominate. IQR
($Q_3 - Q_1$, the gap between 75th and 25th percentiles) is robustly anchored to
ranks — immune to anything in the upper or lower 25% of values. For right-skewed
healthcare data (LOS, cost), $s$ inflates because of the long right tail while
IQR honestly describes the bulk-of-data spread. Empirical rule check: for true
normal data, ~68% of points fall within $\bar{x} \pm s$ — but for skewed data this
can climb to 89%+ (most data sits on one side of the mean), revealing $s$ no longer
means what the empirical rule promises. When the rule fails, IQR or percentile
pairs (5th/95th) tell the truer story.

Q: Why are five percentiles (5th, 25th, 50th, 75th, 95th) often more informative than mean and standard deviation?
A: Two summary numbers commit to a model — the mean assumes magnitudes matter
equally, the standard deviation assumes deviations should be squared. Five
percentiles don't commit: they describe shape directly. The 5th–95th gap shows
the typical range; the 25th–75th (IQR) shows the bulk; the median (50th) anchors
location; gaps between the 50th and tails reveal skewness (50th-to-95th much
bigger than 5th-to-50th means right-skewed). This is why box plots dominate
medical and clinical statistics — they show all five at a glance. Reporting
convention for clinical reference ranges: 5th–95th is the "normal range"; values
outside get flagged. Trade-off: each percentile is one rank position with no
aggregation across observations, so for $n < 30$ they can be noisy; mean/std with
their full-sample averaging are more stable for tiny samples.

Q: How do $\bar{x}$ vs $\mu$, $s^2$ vs $\sigma^2$, $s$ vs $\sigma$ differ — and why does the sample variance use $n-1$ instead of $n$?
A: Greek letters denote *population* parameters (true, usually unknown): $\mu$
population mean, $\sigma^2$ variance, $\sigma$ std, $\rho$ correlation. Latin
letters with bar or just lowercase denote *sample* estimates computed from data:
$\bar{x}$ sample mean, $s^2$ sample variance, $s$ sample std, $r$ sample
correlation. The convention is universal — confusion costs a layer of meaning
when reading any paper. The $n-1$ in $s^2 = \frac{1}{n-1}\sum_i (x_i - \bar{x})^2$
(Bessel's correction) compensates for $\bar{x}$ being itself estimated from the
same data: using $\bar{x}$ instead of the true $\mu$ artificially shrinks
deviations, so dividing by $n$ underestimates the population variance. Dividing
by $n-1$ — the "degrees of freedom" — corrects this bias. NumPy default
`np.var(x)` uses $n$ (`ddof=0`); pass `ddof=1` for sample variance. Pandas
`.var()` defaults to `ddof=1` — opposite of NumPy. This sign-flip default is one
of the most quietly destructive bugs in stats code; always check.

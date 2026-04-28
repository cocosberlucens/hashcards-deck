---
name = "Probability"
---

<!-- Detour: ps4ds Ch. 2 Discrete Variables — 2026-02-27 -->

Q: Why is a random variable defined as a function $\tilde{a}: \Omega \to \mathbb{R}$, rather than just "a number we don't know yet"?
A: Because the function definition makes explicit that randomness lives in the
outcome selection, not in the variable itself. Multiple random variables can be
defined on the same sample space — one hospital admission (one $\omega$) simultaneously
determines length of stay, procedures, and cost. All are deterministic functions
of the same random outcome.

C: A [random variable] is a [function] $\tilde{a}: \Omega \to \mathbb{R}$ that maps [outcomes] in a sample space to real numbers.

Q: Why does the PMF "completely characterize" a discrete random variable, making the underlying sample space unnecessary?
A: Because any probability $P(\tilde{a} \in S)$ can be computed as
$\sum_{a \in S} p_{\tilde{a}}(a)$ (Theorem 2.5). The PMF is the "user interface" —
once you have it, you never need the original sample space $\Omega$ or the mapping
function. This is encapsulation: the implementation (probability space) is hidden
behind the interface (PMF).

C: The [PMF] $p_{\tilde{a}}(a) = P(\tilde{a} = a)$ has three properties: values are [non-negative], they [sum to one], and $P(\tilde{a} \in S) = \sum_{a \in S} p_{\tilde{a}}(a)$.

Q: Why does deriving the PMF of $\tilde{b} = h(\tilde{a})$ require summing probabilities, not just substituting values?
A: Because $h$ can be many-to-one. Multiple values of $\tilde{a}$ may map to the same
value of $\tilde{b}$, so $p_{\tilde{b}}(b) = \sum_{a:\, h(a)=b} p_{\tilde{a}}(a)$
(Theorem 2.8). Substituting would miss probability mass from other pre-images.
Example: binning 0-2 procedures into "Light" tier sums three PMF entries.

C: The [joint PMF] $p_{\tilde{a},\tilde{b}}(a,b) = P(\tilde{a}=a, \tilde{b}=b)$ describes the [simultaneous] probability distribution of [two] discrete random variables.

C: The [Poisson] distribution $P(X=k) = \frac{\lambda^k e^{-\lambda}}{k!}$ models the number of [events] in a fixed interval when events occur [independently] at a constant average rate $\lambda$.

Q: Why does the Poisson distribution need only a single parameter $\lambda$?
A: Because it models independent events at a constant rate, which constrains the
distribution so that mean and variance are both equal to $\lambda$. The single parameter
simultaneously determines center, spread, and shape — an extreme economy that makes
it a powerful parametric model when the assumptions hold.

<!-- Bite 4: Distributions & Visualization — 2026-04-28 -->

Q: For a continuous random variable $X$, how do the PDF $f(x)$ and CDF $F(x)$ relate, and what do their values actually MEAN?
A: The PDF is the derivative of the CDF: $f(x) = F'(x)$, and conversely
$F(x) = \int_{-\infty}^x f(t)dt$. The CDF gives cumulative probability
$F(x) = P(X \le x)$ — directly readable as "probability $X$ is at most $x$." The
PDF's value, by contrast, is NOT a probability: for continuous $X$, $P(X = x) = 0$
for any specific $x$. PDF is a density; probabilities come from integrals,
$P(a < X \le b) = \int_a^b f(x)dx = F(b) - F(a)$. This is why PDFs can exceed 1
(a tall narrow bump over a small interval) without violating probability laws —
they're heights, not probabilities; the AREA under them is what's bounded by 1.
Discrete analog: PMF $p(x) = P(X = x)$ IS a probability directly; CDF is the
running sum $\sum_{k \le x} p(k)$.

Q: What characterizes the Normal distribution $\mathcal{N}(\mu, \sigma^2)$, and why is it everywhere in statistics?
A: The Normal distribution is fully determined by two parameters: mean $\mu$
(location) and variance $\sigma^2$ (spread). Its PDF
$f(x) = \frac{1}{\sigma\sqrt{2\pi}}e^{-\frac{(x-\mu)^2}{2\sigma^2}}$ is the iconic
bell curve — symmetric, unimodal, with rapidly decaying tails. Three reasons it
dominates: (1) the Central Limit Theorem — averages of many random variables tend
toward Normal regardless of original distribution, so anything that averages
multiple effects (heights, lab measurements, measurement error) tends Normal;
(2) maximum entropy under fixed mean and variance — the "least committal"
distribution given those constraints; (3) closed-form algebra makes it tractable
for inference — t-tests, linear regression, ANOVA all assume Normal residuals.
Catch: most healthcare data ISN'T Normal (LOS, cost, lab values with skew), and
those tools then give wrong answers. The QQ plot is the reality check.

Q: What does the exponential distribution model, and what is its memoryless property?
A: Exponential models *waiting time* until a random event when events occur at a
constant average rate — time between ER arrivals, time to next equipment failure,
survival time with constant hazard. Single parameter rate $\lambda$ (events per
unit time) or equivalently scale $1/\lambda$ (mean wait); PDF
$f(x) = \lambda e^{-\lambda x}$ for $x \ge 0$. The memoryless property:
$P(X > s + t \mid X > s) = P(X > t)$ — having already waited $s$ time units gives
NO information about how much longer you'll wait. The future is independent of
the past. This is the same "rate $\lambda$" structure that produces Poisson
counts: if events arrive at exponential inter-arrival times with rate $\lambda$,
the count in a fixed interval is Poisson($\lambda t$). The two distributions are
duals — same process viewed through "how many" (Poisson) vs "how long" (exponential).

Q: How do you read $X \sim \mathcal{N}(\mu, \sigma^2)$?
A: "$X$ is distributed as a Normal random variable with mean $\mu$ and variance
$\sigma^2$." Three pieces: the tilde $\sim$ here means "follows the distribution"
(NOT "approximately equal" — context-dependent overload); $\mathcal{N}$
(calligraphic N) is the conventional symbol for "Normal" / Gaussian; the
parameters in parentheses are convention-dependent — usually mean and *variance*
in statistics, sometimes mean and *standard deviation* in physics/engineering.
Always check which convention a paper uses. Standard normal is
$X \sim \mathcal{N}(0, 1)$. Same pattern for other families:
$X \sim \text{Poisson}(\lambda)$, $X \sim \text{Exp}(\lambda)$,
$X \sim \text{Uniform}(a, b)$, $X \sim \text{Bernoulli}(p)$. The tilde syntax is
how mathematicians declare a random variable's distribution without writing out
the full PDF every time.

Q: What do lowercase $f(x)$ and uppercase $F(x)$ conventionally denote in probability?
A: Lowercase $f(x)$ is the PDF (continuous) or PMF (discrete) — the density/mass
function. Uppercase $F(x)$ is the CDF — cumulative distribution function
$F(x) = P(X \le x)$. This case convention is universal in stats/probability
papers: lowercase for "local" density, uppercase for "cumulative." When multiple
random variables are in play, subscripts disambiguate: $f_X(x), F_X(x)$ for $X$;
$f_{X,Y}(x, y)$ for the joint distribution. Other related notation: $P(X \le x)$
explicitly spells out CDF semantics for clarity in proofs; $P(X = x)$ is
meaningful only for discrete $X$ (always 0 for continuous). The inverse CDF —
the percentile/quantile function — is sometimes written $F^{-1}(p)$ or $Q(p)$,
both meaning "the value $x$ such that $F(x) = p$."

<!-- Detour: ps4cs Ch. 0 Prerequisites — 2026-04-28 -->

Q: What does $|A|$ denote when $A$ is a set, and how does the same symbol overload across mathematics?
A: For a set $A$, $|A|$ is the cardinality — the number of elements:
$|\{1, 2, 3\}| = 3$, $|\emptyset| = 0$. The vertical-bar notation overloads:
$|x|$ for a real number is absolute value; $|z|$ for a complex number is modulus;
$|A|$ for a square matrix is the determinant; vector magnitude is sometimes
$|\mathbf{v}|$ but $\|\mathbf{v}\|$ with double bars is the linalg-standard form.
Context disambiguates: if the operand is a set, cardinality; a number, absolute
value; a matrix, determinant. Python: `len(A)` for sets and any sized container.
SQL: `COUNT(DISTINCT col)` is the database equivalent. For infinite sets the
symbol still works — $|\mathbb{N}| = \aleph_0$, "aleph-naught" — but cardinality
becomes a more delicate notion (countable vs uncountable infinity).

Q: What do $\cup$, $\cap$, $\setminus$, and $S^c$ each compute, and how are they written in Python?
A:
- $A \cup B$ is **union** — elements in $A$ OR $B$ (or both) → Python `A | B`.
- $A \cap B$ is **intersection** — elements in BOTH $A$ AND $B$ → Python `A & B`.
- $A \setminus B$ is **set difference** — in $A$ but NOT in $B$ → Python `A - B`.
T&F use the backslash $\setminus$ rather than the minus sign for clarity, since
$A - B$ for numbers means subtraction; not all texts agree.
- $S^c$ is the **complement** of $S$ relative to a universal set $\mathcal{U}$ —
elements in $\mathcal{U}$ but not in $S$ → Python `U - S`. Variant notation:
$\bar{S}$ (overline) means the same in some texts, especially probability.
Mnemonic: $\cup$ looks like a cup holding everything from both; $\cap$ caps off
the overlap. SQL parallels: `UNION`, `INTERSECT`, `EXCEPT`. Two empty-set
identities worth remembering: $A \cup \emptyset = A$, $A \cap \emptyset = \emptyset$.

Q: How do you read $\prod_{i=1}^n x_i$, and where does this notation show up beyond pure math?
A: "Product over $i$ from 1 to $n$ of $x_i$" — uppercase Greek pi, the multiplicative
analog of $\sum$. Subscript $i=1$ initializes the dummy index, superscript $n$ is
the (inclusive) upper limit, and $i$ takes each integer between. Equivalent to a
Python `for` loop accumulating into a running product with initial value 1. Three
killer applications: maximum likelihood estimation $L(\theta) = \prod_i f(x_i \mid \theta)$
— the joint likelihood under independence; probability of independent events
$P(\bigcap_i E_i) = \prod_i P(E_i)$; combinatorial counts like factorials
$n! = \prod_{i=1}^n i$. The $\log$ trick: in MLE,
$\log \prod_i f(x_i \mid \theta) = \sum_i \log f(x_i \mid \theta)$ converts a
product of small densities (which underflows fast in floating-point) into a
numerically stable sum. NumPy: `np.prod(arr)` evaluates the product directly.

Q: What do De Morgan's laws state for sets, and why are they critical in probability?
A: Two dual identities about complementation:
$(A \cup B)^c = A^c \cap B^c$ and $(A \cap B)^c = A^c \cup B^c$. In English: the
complement of a union is the intersection of complements, and vice versa.
Operationally: when you see "not ($A$ or $B$)" you can rewrite as
"(not $A$) and (not $B$)" — the only way an outcome avoids both $A$ and $B$ is to
avoid each individually. Why critical in probability: events are sets within a
sample space, so "the patient has neither cardiac nor respiratory diagnosis" is
$E_{\text{cardiac}}^c \cap E_{\text{resp}}^c = (E_{\text{cardiac}} \cup E_{\text{resp}})^c$.
The version on the right is often easier to compute because $P(A \cup B)$ has a
known formula (inclusion-exclusion) and $P(\text{complement}) = 1 - P(\cdot)$.
De Morgan generalizes to any finite or countable family:
$(\bigcup_i E_i)^c = \bigcap_i E_i^c$.

Q: Why does $|A \cup B| = |A| + |B| - |A \cap B|$ subtract the intersection, and how does this generalize?
A: Adding $|A| + |B|$ counts every element in the overlap TWICE — once for being
in $A$, once for being in $B$. Subtracting $|A \cap B|$ removes one of those
duplicate counts, leaving each element counted exactly once. SQL parallel:
`UNION` automatically de-duplicates while `UNION ALL` keeps duplicates — exactly
the difference between corrected and uncorrected counts. Three-set version:
$|A \cup B \cup C| = |A| + |B| + |C| - |A \cap B| - |A \cap C| - |B \cap C| + |A \cap B \cap C|$
— alternating signs by intersection size. The pattern generalizes to $n$ sets:
$|\bigcup_{i=1}^n A_i| = \sum_{k=1}^n (-1)^{k+1} \sum_{|S|=k} |\bigcap_{i \in S} A_i|$,
a signed sum over all non-empty intersections. In probability, the same formula
governs $P(A \cup B \cup C)$ with probabilities replacing cardinalities — the
algebraic skeleton is identical.

Q: Why is $\sum$ over an empty range defined to equal 0, and $\prod$ over an empty range defined to equal 1?
A: Each is the accumulator's initial value: a `for`-loop summing into `total = 0`
returns 0 if the loop runs zero times; a `for`-loop multiplying into
`product = 1` returns 1 if it runs zero times. The conventions ARE those
identity values — 0 is the additive identity ($x + 0 = x$), 1 is the
multiplicative identity ($x \cdot 1 = x$). Why this matters: many formulas have
natural cases where the range can shrink to empty without breaking. Expected
value $\mathbb{E}[X] = \sum_k k \cdot p(k)$ for a positive-only random variable —
terms for $k \le 0$ vanish without special casing because the empty sum returns
0. Likelihood $L(\theta) = \prod_i f(x_i \mid \theta)$ for an empty dataset gives
1, the "no information" likelihood that lets prior-times-likelihood reduce
cleanly to prior. NumPy preserves both conventions: `np.sum([])` returns 0.0,
`np.prod([])` returns 1.0. The conventions look pedantic until they make a
formula work in a corner case without an `if` clause.

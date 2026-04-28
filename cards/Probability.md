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

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

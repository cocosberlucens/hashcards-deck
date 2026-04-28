---
name = "Machine Learning"
---

<!-- Bite 1: Vectors & Arrays — The ML Perspective — 2026-04-28 -->

Q: What distinguishes supervised, unsupervised, and reinforcement learning at the data level?
A: The supervisory signal the algorithm sees during training. Supervised learning gets
labeled data $(X, y)$ — features paired with the answer to predict — and aims at
generalization: predict $y$ for new $X$. Unsupervised learning gets only $X$, no
labels — the goal is finding structure: clusters, lower-dimensional representations,
anomalies. Reinforcement learning gets neither static labels nor pure features but a
reward signal that depends on the algorithm's actions in an environment — the goal
is a policy maximizing cumulative reward. The labeling level (full / none / via
interaction) determines what's even askable, which is why classifying any ML problem
starts with "what's the supervisory signal, and where does it come from?"

Q: Both regression and classification are supervised — what's the substantive difference?
A: The output type. Regression predicts a continuous numeric value: episode cost,
length of stay, blood pressure. Classification predicts a discrete category:
readmit yes/no, diagnosis code, spam/not-spam. The distinction shapes downstream
choices: loss function (squared error vs cross-entropy), evaluation metric
(RMSE/MAE vs accuracy/precision/F1/AUC), and the algorithm itself — linear
regression for one, logistic regression for the other (despite the misleading
name; "logistic regression" is a classifier). Trap: a continuous target can be
binned into classes ("cost > \$10k → expensive"), but you usually lose information
that way and gain nothing the regression model couldn't already give you by
applying a threshold to its prediction.

Q: What signature distinguishes overfitting from underfitting in practice?
A: The gap between training and test error. Overfitting: training error low (often
near zero), test error much higher — the model memorized training noise and fails
on new data. Underfitting: BOTH training and test errors are high — the model is
too rigid to capture the actual signal. Common causes: overfitting comes from
too-flexible models (deep trees, high-degree polynomials, unregularized neural nets)
or too little training data relative to parameter count; underfitting comes from
overly simple models (linear when truth is nonlinear) or aggressive regularization.
Validation curves — error plotted against model complexity — make the trade-off
visible: training error monotonically falls while test error is U-shaped, with the
sweet spot at the U's bottom. The two failure modes are NOT symmetric in cost:
underfitting is often easier to spot and fix; overfitting can hide behind
optimistic training metrics until the model meets reality.

Q: How does the storage cost at deployment time betray whether an algorithm is instance-based or model-based?
A: Instance-based learners (k-NN being the canonical example) keep the entire
training dataset and consult it at prediction time — every query computes
distances to stored examples. Deployment storage scales with $n$, the number of
training points. Model-based learners (linear regression, neural nets, decision
trees) compress training data into a fixed set of parameters during fitting; at
prediction time they just apply the formula. Storage is independent of $n$. This
is why a linear regression on a million patients might be a 5-coefficient vector,
while k-NN on the same data is the full million rows. Trade-off: instance-based
methods adapt naturally to whatever's in the data (no model-form assumption);
model-based methods generalize cleaner and are cheap to deploy but commit to a
structural assumption that may be wrong (the misspecification problem). The
"compression gradient" k-NN → K-Means → Linear Regression spans this axis.

Q: Why must nominal categories be one-hot encoded before distance-based ML algorithms see them?
A: Because algorithms like k-NN, linear regression, and neural nets compute
distances or weighted sums on numeric inputs — operations that would silently
treat a numeric label as if it had ordinal meaning. If insurance type is encoded
Medicare=0, Medicaid=1, Commercial=2, k-NN reads "Medicare and Commercial are 2
units apart" but "Medicare and Medicaid are 1 unit apart" — a fiction with no
clinical basis. One-hot encoding turns one nominal column with $k$ values into $k$
binary columns, each indicating presence/absence. Distances now reflect pure
category mismatch (1 if different, 0 if same), faithful to the underlying nominal
structure. Cost: the feature dimension grows by $k-1$ per encoded column; for
high-cardinality nominals (ZIP codes, ICD-10 diagnoses with thousands of codes),
consider embeddings or target encoding instead — they keep the geometric
faithfulness without the dimension blow-up.

<!-- Bite 2: Matrices & DataFrames — 2026-04-28 -->

Q: Why is the feature matrix conventionally written $\mathbf{X}$ with shape $(n, p)$, and what does each axis carry?
A: $\mathbf{X}$ is the input data assembled for ML: $n$ observations as rows, $p$
features as columns — so $\mathbf{X}_{i,:}$ is the $i$-th observation's full
feature vector. The convention is so ingrained that sklearn's `fit(X, y)` API
silently enforces it: passing $(p, n)$ instead of $(n, p)$ is a common beginner
error that produces nonsensical models without crashing. Why this orientation and
not the transpose? Because we typically have many more observations than features
($n > p$), and CSV files / SQL queries naturally produce one row per record. Two
operational consequences: aggregations along axis=0 (down rows) compute per-feature
statistics — column means $\bar{\mathbf{x}}$, column stds — exactly what you want
for normalization; pairwise distances between observations are rows-vs-rows. The
$\mathbf{y}$ vector partner has shape $(n,)$ — one target per observation, aligning
with $\mathbf{X}$ by row index.

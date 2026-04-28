---
name = "Pandas"
---

<!-- Bite 0.4: Pandas Fundamentals — Labels Change Everything (+ 0.4b Views/Copies/SettingWithCopyWarning) — 2026-04-23 -->

Q: What mental model best captures a Pandas Series?
A: A Series is three things at once: a NumPy array (fast, vectorized, typed),
a typed ordered dictionary (keys-to-values, where keys can repeat), and a single
SQL column (labeled data with row identifiers attached). The labels live in the
Index object — itself set-like and immutable. Access values by label (`s['P001']`),
by position (`s.iloc[0]`), or via NumPy-style boolean masks (`s[s > 100]`). The
`.values` attribute exposes the underlying NumPy array directly.

Q: Why does `df['cost']` return a Series but `df[['cost', 'age']]` return a DataFrame?
A: A Series IS a single column — one labeled axis (the row Index) and one dtype.
Requesting one column gives you that column directly. Requesting a LIST of columns
returns a subset DataFrame because it has two labeled axes again (rows AND multiple
columns). The double-bracket syntax `df[['col']]` is a one-element list, so it
returns a one-column DataFrame rather than a Series. Useful when downstream code
expects a DataFrame API.

Q: When is `df['cost']` preferable to `df.cost` even though both access the column?
A: `df.cost` (attribute access) only works when the column name is a valid Python
identifier AND doesn't collide with a DataFrame method or attribute. `df.mean`
calls the `.mean()` method rather than returning a column called "mean";
`df['column with space']` is impossible via attribute. Also, assignment via attribute
is dangerous: `df.new_col = [1,2,3]` creates a regular Python attribute, NOT a
DataFrame column. Default to bracket access; keep attribute access for quick
interactive exploration only.

Q: What's the most accurate mental model of a DataFrame?
A: A DataFrame is a dict of Series that share the same row Index — plus a column
Index (the column labels). Each column is independently typed (mixed dtypes are
fine across columns, uniform within), and you can access data by row label, column
label, or both via `.loc[row, col]`. The SQL analogy holds: think of it as a table
with a primary-key-like Index — though Pandas allows duplicate index values,
unlike SQL primary keys.

Q: Why does `df.loc['P001':'P003']` include row P003, but `df.iloc[0:3]` excludes position 3?
A: `.loc` does LABEL slicing with INCLUSIVE endpoints (like SQL `BETWEEN`), while
`.iloc` does POSITION slicing with EXCLUSIVE stop (Python/NumPy convention). So
for rows P001-P005, `loc['P001':'P003']` spans three labels P001-P003 inclusive,
and `iloc[0:3]` spans positions 0, 1, 2 exclusive-stop. The two happen to return
three rows here, but the *rule* differs — miscounting is a classic Pandas off-by-one.
Mnemonic: labels are inclusive because a natural-language range "from A to C"
contains C.

C: In Pandas, [.loc] uses labels with [inclusive] endpoints while [.iloc] uses positions with exclusive endpoints. The bracket form `df['col']` selects columns; `df[mask]` with a boolean Series selects [rows].

Q: Why does `df[df['age'] > 65]` return rows where age exceeds 65, not columns?
A: `df['age'] > 65` produces a boolean Series aligned to df's row index. When
passed as the bracket argument, Pandas interprets a boolean Series as a ROW mask,
not a column selector — the `df[...]` syntax is overloaded based on the argument
type: a string selects a column, a list of strings selects multiple columns,
a boolean Series filters rows. This dispatch surprises learners expecting bracket
syntax to do one thing, but it's the reason one-liner filters read so cleanly.

Q: What's the correct way to filter rows AND select columns in a single Pandas expression?
A: Use `.loc` with a boolean mask and a column selector in ONE step:
`df.loc[(df['age'] > 65) & (df['dept'] == 'Cardiology'), ['cost', 'los']]`.
The `&`/`|`/`~` bitwise operators are element-wise (not `and`/`or`, which raise on
multi-element arrays), and each comparison needs parentheses because `&` has
higher precedence than `>`. The single-step `.loc[...]` form is also mandatory
for *writes*: `df.loc[mask, 'col'] = value` modifies df directly; chained
`df[mask]['col'] = value` triggers SettingWithCopyWarning and may silently fail.

Q: Why is the Pandas Index immutable, and why does it matter?
A: Immutability prevents silent data corruption when Index objects are shared.
Multiple DataFrames or Series can reference the same Index; if mutating it in one
place changed it everywhere, aligning operations (like `s1 + s2` matching by label)
would become unpredictable. The Index is also set-like: supports `.intersection()`,
`.union()`, `.difference()`, and — unlike dict keys or SQL primary keys — allows
duplicates. To "change" an index you rebuild it: `df.index = new_index` or
`df.reindex(new_labels)`, both of which construct a new Index rather than mutating.

Q: When you compute `s1 + s2` with partially overlapping indices, why does Pandas fill non-overlapping entries with NaN instead of raising?
A: Pandas treats operations between labeled objects as an IMPLICIT outer join on
the index. Labels present in both Series get their values combined; labels present
in only one produce NaN (no value to pair with). This magic makes "add 2023 costs
and 2024 costs, keyed by patient ID" work without any explicit merge — but it
surprises learners expecting positional alignment (as in NumPy). If you want
single-side labels to contribute their own value instead of NaN, use the explicit
method form: `s1.add(s2, fill_value=0)`.

Q: What does `fill_value` do in Pandas arithmetic methods like `.add()` or `.sub()`?
A: Normally, index alignment between labeled operands produces NaN wherever a
label is missing from one operand. `fill_value=0` tells Pandas: "before the
operation, pretend missing labels are 0." So `s1.add(s2, fill_value=0)` gives
`s1 + s2` where single-side labels contribute alone (`0 + value = value`) instead
of becoming NaN. Same idea for `.sub`, `.mul`, `.div`. Important: `fill_value`
only kicks in for the MISSING side — actual NaN values in the overlapping region
are NOT filled.

Q: Why does `pd.Series([1, 2, None, 4])` have dtype float64 instead of int64?
A: NumPy integer types have no special "missing" value — integers are dense.
NaN is a float64 concept (a specific bit pattern), so to represent missing data
alongside integers Pandas upcasts the whole column to float64 and replaces `None`
with `NaN`. Same rule in DataFrames: a single missing value anywhere in an int
column makes that column float. For genuine nullable integers, use Pandas' nullable
dtype: `pd.Series([1, None, 3], dtype='Int64')` (capital I) — this preserves integer
semantics and uses `pd.NA` instead of NaN.

Q: When should you choose `.dropna()` over `.fillna()` — and vice versa?
A: Drop rows/columns when missing values are sparse enough that losing them doesn't
hurt your analysis, or when missingness itself is evidence of bad data you don't
want to impute. Fill when dropping costs too much data, when missingness is
informative (e.g., 0 means "not applicable"), or when downstream code can't tolerate
NaN. Common fills: `fillna(0)` for counts, `fillna(df.mean())` for numeric means,
`fillna('Unknown')` for categoricals, `df.ffill()` for time series where "carry
last observation forward" is defensible. The right choice is driven by what the
missingness MEANS in your domain, not by convenience.

Q: Why does `df['cost'].mean()` return a sensible number even when cost contains NaN?
A: Pandas aggregation methods (mean, sum, std, min, max, median) skip NaN by
default — equivalent to SQL's `AVG` which ignores NULLs. A column with 3 real
values and 1 NaN returns the mean of the 3. To propagate NaN into the result
instead, pass `skipna=False`: `df['cost'].mean(skipna=False)` returns NaN if any
value is NaN. The "skip by default" convenience hides missingness — always check
`.isnull().sum()` separately when you want to know how many values actually
contributed to an aggregate.

Q: When should you use `df['new'] = expr` versus `df.assign(new=expr)` to add a column?
A: Direct assignment mutates `df` in place — fine for quick work and memory-efficient,
but breaks method chaining and makes it easy to accidentally modify a DataFrame
you meant to leave alone. `.assign(new=expr)` is functional: returns a NEW DataFrame
with the extra column, leaves the original untouched. Prefer `.assign` when chaining
(`df.assign(a=...).assign(b=...).pipe(process)`) or when you want referential safety.
Prefer direct assignment for one-off scripts where the extra allocation matters
and in-place modification is clearly intended.

Q: Why is the Pandas view-vs-copy rule less predictable than NumPy's?
A: NumPy's rule is clean: basic slicing returns a view (contiguous memory, same
dtype); fancy/boolean indexing returns a copy. Pandas adds two layers of
complexity — DataFrames can have mixed dtypes, so each column may live in a
separate memory block (a "row slice" can't necessarily be a view of contiguous
memory); and the Index adds indirection between labels and positions. Result:
single-column access (`df['col']`) usually returns a view, but `df[['col1', 'col2']]`,
boolean row filters, and row slices often return copies — "often" being the key
word. This unpredictability is why `.loc`-for-assignment and `.copy()`-for-independence
are mandatory disciplines.

Q: Why is `df[df['age'] > 65]['cost'] = 0` a bug, even when it raises no error?
A: That line is actually TWO operations: first `df[df['age'] > 65]` returns a
temporary intermediate, then `['cost'] = 0` assigns into that intermediate —
not into df. Under pre-3.0 Pandas this triggered SettingWithCopyWarning
(removed in Pandas 3.0). Under 3.0's default Copy-on-Write the assignment
silently mutates a short-lived copy and df is untouched: no warning, no error,
no change to your data. Either way the fix is the same: merge into one `.loc`
call: `df.loc[df['age'] > 65, 'cost'] = 0`, which Pandas guarantees operates on df.

Q: What are the three safe patterns for modifying a DataFrame without hitting view/copy ambiguity?
A: (1) Use `.loc[row_selector, col_selector] = value` for any assignment — both row
and column expressed in one step, which Pandas guarantees operates on the original.
(2) Call `.copy()` when you want an independent DataFrame to modify freely:
`subset = df[df['age'] > 65].copy(); subset['cost'] = 0` leaves df untouched.
(3) Use functional `.assign(new_col=expr)` to add columns without mutation —
returns a new DataFrame, ideal for chaining. Combined rule: read with any method,
write with `.loc`, isolate with `.copy()`, chain with `.assign()`.

Q: What does Pandas' Copy-on-Write (CoW) change about the view/copy story?
A: CoW makes every indexing operation behave like it returned a copy, but
delays the actual allocation until you mutate. Reads stay free (shared memory);
writes transparently trigger a copy before mutation, so derived objects never
affect the source. The view/copy ambiguity that motivated SettingWithCopyWarning
is gone — the warning itself was removed in Pandas 3.0 (January 2026), where
CoW became default. In 2.x it's opt-in via `pd.options.mode.copy_on_write = True`.
Tradeoff: in-place modifications must be explicit via `.loc`, making intent
clearer but code slightly more verbose.

<!-- Bite 3: Aggregation & Summary Statistics — 2026-04-28 -->

Q: What does `df.groupby('dept').mean()` actually do under the hood, and why is the GroupBy object lazy?
A: Three steps in order: SPLIT — partition rows into groups by unique values of
the key column (one group per department); APPLY — run the aggregation function
on each group's values independently; COMBINE — assemble the per-group results
back into a single DataFrame indexed by group key. The pattern is general enough
that custom functions slot in via `.agg(my_func)` or `.agg(named=('col', func))`.
Why lazy: `df.groupby('dept')` returns a DataFrameGroupBy object that has only
computed the partition mapping — it has NOT run any aggregations yet. Actual work
happens when you call an aggregation method or `.agg()`. Laziness lets you chain
multiple aggregations off the same partition (`.agg({'cost': 'sum', 'los': 'mean'})`
walks each group once, not twice) and explore the partition (`.groups`, `.size()`)
without paying full aggregation costs. SQL parallel: `GROUP BY` clauses are also
non-executed until paired with aggregates — the mental model carries.

Q: When would you use `.transform('mean')` instead of `.aggregate('mean')` on a Pandas GroupBy?
A: `.aggregate('mean')` REDUCES each group to one summary value — output has one
row per group (n_groups rows total), aligned by group key. `.transform('mean')`
BROADCASTS each group's summary back to one value per ORIGINAL row — output has
exactly the same shape as the source DataFrame, with each row receiving its own
group's mean. So `.transform` is what you reach for in patterns like "subtract
each row's department mean from its LOS":
`df['los'] - df.groupby('dept')['los'].transform('mean')` gives per-row
deviations. You couldn't do this with `.aggregate` directly — you'd need a merge
step. The signature distinction in one line: aggregate reduces (n_groups out),
transform broadcasts (n_rows out). SQL parallel: aggregate is `GROUP BY ...
SELECT AGG()`; transform is a window function `AVG(...) OVER (PARTITION BY ...)`.

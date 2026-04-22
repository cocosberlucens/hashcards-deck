---
name = "Python Patterns"
---

<!-- Bite 0.2: Iteration, Data Structures & OOP Basics — 2026-04-22 -->

Q: Why does `result = do_something(x)` sometimes give `result is None` even though the function clearly computed and printed a value?
A: A function without an explicit `return` statement implicitly returns `None`. Printing
is a side effect; it doesn't become the return value. This is unlike SQL, where a SELECT
always produces output. The fix is to end the function body with `return value` — or if
the function is meant to be pure side-effect (like logging), accept that `None` is the
correct return.

Q: What actually gets passed when you call `f(my_list)`?
A: Neither the list nor a copy — Python passes the *reference* (the binding between
name and object). Inside the function, the parameter name points at the SAME list
object the caller holds. Mutations via the parameter (`.append()`, `param[i] = x`) are
visible to the caller; rebinding the parameter (`param = new_list`) is not, because
that only changes what the local name points to. This "pass by object reference"
explains why functions can silently modify their arguments.

Q: Why is `def add(item, bucket=[])` almost always a bug?
A: Default arguments are evaluated ONCE at function definition time, not per call.
Every call that omits `bucket` reuses the same list object, so items appended during
one call persist into the next. Fix: `bucket=None` plus `if bucket is None: bucket = []`
inside — this creates a fresh list per call. Same trap with `{}` defaults.

C: Inside a function, if you [assign] to a name anywhere in its body, Python treats that name as [local] for the entire function. Reading it before the first local assignment raises [UnboundLocalError], even if a global with the same name exists.

C: Python's [in] operator tests membership — on a list it scans for an [element], on a string it tests for a [substring], and on a dictionary it tests whether a [key] is present (never a value).

Q: Why does `alias.append(4)` change `original` when you wrote `alias = original` just before?
A: `alias = original` doesn't copy — it binds a second name to the same list object.
Both names point to one object in memory, so mutations through either name are visible
through the other. This is the same mechanism behind NumPy views and Pandas chained
indexing: assignment binds names to objects, it doesn't duplicate them. To actually
copy: `original.copy()`, `original[:]`, or `list(original)`.

Q: Why do `list.append()`, `list.sort()`, and `list.reverse()` return `None` instead of the modified list?
A: Deliberate design — Guido's "command-query separation". Methods that *modify* the
list return `None` to signal the mutation was the point; functions that *compute a new
list* (like `sorted(x)` or `reversed(x)`) return the new list. The rule prevents the
ambiguity of `a = a.sort()`, which would silently assign `None` to `a`. When you see
`= .sort()` or `= .append()` in code, it's almost always a bug.

Q: When should you write `sorted(data)` versus `data.sort()`?
A: `sorted(data)` returns a new sorted list and leaves `data` untouched — use it when
you need both the original order and a sorted view, or when `data` isn't a list
(it accepts any iterable). `data.sort()` sorts in place and returns `None` — use it
when you want to modify `data` permanently and don't need the old order. Same pattern
applies to `reversed()` vs `.reverse()`.

Q: When should you use `==` and when `is`?
A: `==` compares *values* — the preferred default for "are these equivalent?". `is`
compares *identity* — "are these the same object in memory?". Use `is` only for
singletons: `x is None`, `x is True` / `x is False`, sentinel values. Avoid `x is 257`:
Python caches small integers (-5 to 256) so `256 is 256` is True but `257 is 257` is
implementation-dependent — a footgun that breaks silently.

C: Chaining [.get(key, {})] on nested dicts enables [defensive] access: if an intermediate key is missing, the default empty dict lets the chain [continue] instead of raising KeyError, and the final `.get` supplies a meaningful default.

Q: Why does `max(dept_costs, key=dept_costs.get)` return the department name with the highest cost, not the cost itself?
A: `max()` iterates over its first argument. For a dict, iteration yields keys. The
`key=` argument is a function `max` calls on each candidate to produce a comparison
value — so `max` compares `dept_costs.get('Cardiology')`, `dept_costs.get('Neurology')`,
etc., and returns the KEY that produced the largest comparison value. Passing
`dept_costs.values()` instead would give you the winning value but lose the name.
Same idiom works with `min`, `sorted`, etc.

Q: What makes `patient.get('demographics', {}).get('contact', {}).get('email', 'N/A')` safer than `patient['demographics']['contact']['email']`?
A: Bracket access raises `KeyError` the moment any key is missing; `.get(key, default)`
returns the default instead. Chaining `.get(..., {})` — empty dict as the intermediate
default — lets the chain continue past missing nodes because an empty dict still has
`.get`. Only the FINAL `.get` uses a meaningful default (`'N/A'`). Avoids try/except
for the common "optional nested field" case.

Q: Why is `groups.setdefault(key, []).append(item)` the classic "group into dict-of-lists" one-liner?
A: `.setdefault(key, [])` returns the existing list at `key` if it exists, OR inserts
a fresh `[]` and returns *that* if it doesn't. Either way you get back a list you can
immediately `.append(item)` to. Combines the "check, create if missing, then access"
steps into one atomic operation. Same result as `collections.defaultdict(list)` but
without importing, and more explicit about what's happening.

Q: Why is `{v: k for k, v in d.items()}` risky even though it cleanly "inverts" a dict?
A: Python dicts require unique keys. If two original values were equal, the inverted
dict keeps only one of them — later iterations silently overwrite earlier ones, and
you get a smaller dict than expected. Only safe when values are guaranteed unique
(often not the case with real data). For many-to-one mappings, invert into a
dict-of-lists: `result.setdefault(v, []).append(k)`.

C: In the list comprehension `[expr for item in iterable if condition]`, the [expr] is evaluated once per [item], and the optional [if] clause filters which items contribute to the output list.

Q: When should you reach for a list comprehension versus a vectorized NumPy expression?
A: Comprehension for non-numeric transforms (strings, dicts, objects), for building
sets/dicts, or when the operation can't be expressed as arithmetic on arrays.
NumPy (`arr * 2`, `arr[arr > k]`) when the operation is purely numerical — the implicit
loop runs in C and avoids the per-element Python overhead. Rule of thumb: if you'd
write it as `SELECT expr FROM col` on a column, a comprehension is idiomatic;
if you'd express it as matrix math, NumPy wins.

Q: What do `any(iterable)` and `all(iterable)` return, and why do they often beat an explicit loop?
A: `any` returns `True` if at least one item is truthy; `all` returns `True` if every
item is truthy (and `True` for an empty iterable, vacuously). Both short-circuit —
they stop iterating as soon as the answer is determined. So `any(x > 1000 for x in costs)`
stops at the first big cost; a hand-rolled loop with `if ... break` does the same thing
with more noise. Combined with a generator expression, they express "does at least one
/ do all" declaratively.

C: In a class, [__init__] runs once when an instance is [created] and sets its attributes via `self`, while [__str__] runs whenever the instance is converted to a string (e.g., by `print()` or `str()`) and returns the human-readable form.

Q: Why must instance methods take `self` as their first parameter?
A: When you call `obj.method(x)`, Python rewrites it to `Class.method(obj, x)` — the
instance is passed in as the first argument. `self` is just the conventional name for
that parameter; it's how the method accesses its instance's attributes (`self.age`)
and other methods (`self.is_elderly()`). Without it, the method has no way to know
WHICH instance it was called on. Static methods and class methods break this pattern
explicitly with `@staticmethod` and `@classmethod`.

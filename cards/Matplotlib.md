---
name = "Matplotlib"
---

<!-- Detour: ps4ds Ch. 2 Discrete Variables — 2026-02-27 -->

Q: Why use a stem plot rather than a bar chart or histogram to visualize a discrete PMF?
A: A stem plot shows probability mass at isolated points — each lollipop sits at
exactly one value with empty space between. This visually encodes the discrete
nature: $P(X = 3.5) = 0$. A histogram's connected bars suggest continuity;
wide bars imply ranges. The stem plot truthfully represents that probability
exists only at specific values.

<!-- Bite 0.5: Visualization Foundations — Seeing Your Data — 2026-04-23 -->

Q: Why prefer the object-oriented interface `fig, ax = plt.subplots()` over `plt.plot()` / `plt.xlabel()` style?
A: The pyplot form implicitly targets a "current axes" — whichever was last
touched. Fine for a single plot, but breaks for subplots, post-creation tweaks,
or anything that leaves global state ambiguous. The OO form gives you named
handles (`fig`, `ax`) and all drawing goes through explicit method calls
(`ax.plot(...)`, `ax.set_xlabel(...)`). Code reads the same whether you have one
subplot or ten; no mystery about where things land. The pyplot API is a
MATLAB-compatibility layer — fine for scratch work, avoid for real code.

Q: What's the difference between a Figure and an Axes, and why does it matter?
A: A Figure is the outermost container — the canvas or page. An Axes is a single
plotting area within the Figure — its own coordinate system, title, x/y axes, and
drawn artists. One Figure can hold many Axes (that's what subplots do). Method
calls differ by scope: `fig.savefig()`, `fig.tight_layout()`, `fig.suptitle()`
act on the whole canvas; `ax.plot()`, `ax.set_xlabel()`, `ax.legend()` act on
one plotting area. Getting this distinction right prevents bugs like "my title
appeared on the wrong subplot."

Q: What does the format string `'ro--'` in `ax.plot(x, y, 'ro--')` decode to?
A: It's a compact encoding of three styling choices: color-marker-linestyle.
`r` is red, `o` is circle marker, `--` is dashed line. Other common combinations:
`'b-'` (blue solid line, no markers), `'g^:'` (green triangles with dotted line),
`'k.'` (black dot markers, no line). For cases not expressible in format strings,
use named parameters: `color='#FF5733', marker='o', linestyle='--'`. Format
strings are useful for quick exploration; named params for production plots.

Q: When should you use `ax.scatter(x, y)` instead of `ax.plot(x, y, 'o')` even though both draw points?
A: `ax.plot(x, y, 'o')` gives every point identical size and color — fastest,
fine for homogeneous point clouds. `ax.scatter(x, y)` lets you VARY size via `s`
and color via `c` per-point, including color-mapping a third variable with `cmap`.
Use scatter when you want to encode extra dimensions visually
(`ax.scatter(x, y, c=ages, s=costs, cmap='viridis')` = 4D in a 2D plot).
Use plot when all points are stylistically uniform and rendering speed matters
for large N.

Q: How does `ax.scatter(x, y, c=ages, cmap='viridis')` encode three variables in a 2D plot?
A: x and y give the two spatial dimensions (one point per observation), then
`c=ages` passes a numeric array of per-point values which `cmap='viridis'` maps
to colors — so color becomes the third dimension. Call
`plt.colorbar(scatter, ax=ax)` (capturing the scatter return value) to give
viewers the color-to-value scale. Choose colormap by data type: sequential
(`viridis`, `plasma`) for ordered values, diverging (`RdBu`, `coolwarm`) for
data centered on zero, qualitative (`tab10`) for unordered categories — though
for categories, multiple scatter calls with distinct colors reads more cleanly.

Q: Why does choosing the number of histogram bins matter so much?
A: Bins control what structure you can see. Too few and the histogram oversmooths
— real modes, gaps, and skew disappear into blocky averages. Too many and each
bin holds only a handful of observations — noise dominates, artificial peaks
appear. Rules of thumb (Sturges' $k = \lceil \log_2 n + 1 \rceil$,
Freedman-Diaconis, $\sqrt{n}$) give starting points, but the honest answer is:
try 3 different values before deciding. If the distribution's story changes
dramatically between bin counts, your sample is too small for a confident
visualization.

Q: What does `density=True` in `ax.hist(data, density=True)` change about the histogram?
A: Without it, the y-axis is raw counts — the bars' heights (times bin width) sum
to the sample size. With it, the histogram is normalized so the total AREA under
the bars equals 1, making the y-axis a probability density. Two benefits:
(1) overlay a theoretical PDF on the same axes for direct comparison, and
(2) compare distributions across groups with different sample sizes fairly.
Without density, the larger group always "looks bigger" even if the shape is
identical.

Q: Why does `fig, axes = plt.subplots(2, 3)` give you a 2D array of Axes, and how do you index it?
A: When you request a grid of m×n subplots, Matplotlib returns an m×n NumPy array
of Axes objects — matching the grid's geometry. Index with 2D notation:
`axes[0, 0]` is top-left, `axes[1, 2]` is bottom-right, `axes[i, j]` is row i
column j. Corner cases: `subplots(1, 3)` returns a 1D array of length 3
(use `axes[i]`); `subplots(1, 1)` returns a single Axes (not an array). To always
get a 1D flat view regardless of shape, use `axes.flat` or `axes.flatten()` —
useful for `for ax, data in zip(axes.flat, datasets): ...` patterns.

Q: When do you need `fig.tight_layout()` and what does it do?
A: It auto-adjusts subplot spacing to prevent labels, titles, and tick marks from
overlapping or being clipped. Call it AFTER all subplots and their annotations
exist, typically as the last line before rendering. Without it, a 2×2 grid with
long axis labels often produces labels bleeding into neighboring subplots or
getting cut off at figure edges. The alternative is manual tuning with
`plt.subplots_adjust(wspace=..., hspace=...)`, but tight_layout nails the common
case. For complex layouts, consider `constrained_layout=True` at figure creation,
or explicit `GridSpec`.

Q: Why does `ax.quiver(0, 0, vx, vy, angles='xy', scale_units='xy', scale=1)` draw a vector that matches the axes correctly?
A: Quiver's defaults are tuned for vector FIELDS (many arrows), which silently
shrink single arrows. The three magic parameters override this: `angles='xy'`
computes arrow angles in data coordinates (not screen), and `scale_units='xy'`
+ `scale=1` means "one unit of vector = one unit of axis." With these, a vector
[3, 2] really points from origin to (3, 2) on the plot, matching the math you're
trying to show. Forgetting the trio is the #1 reason "my vector arrow looks
wrong" in tutorials.

Q: Why is `ax.set_aspect('equal')` critical when visualizing vectors geometrically?
A: By default Matplotlib stretches the plot to fit the figure, so one unit on
the x-axis may occupy twice as many pixels as one unit on the y-axis. For data
plots this is fine — nothing geometric depends on the aspect ratio. For vector
plots everything does: a right angle looks acute or obtuse, unit vectors look
unequal, the dot-product-as-projection intuition breaks. `set_aspect('equal')`
forces 1:1 data-to-pixel ratio on both axes, so angles and magnitudes render
truthfully. Pair with `ax.grid(True)` and tight axis limits to make the geometry
readable.

Q: What does each argument in `ax.annotate('outlier', xy=(x, y), xytext=(x+1, y+1), arrowprops=dict(arrowstyle='->'))` do?
A: Annotate places a text label with an optional arrow pointing FROM the text TO
a data point. `'outlier'` is the label; `xy=(x, y)` is the point being annotated
(the arrow tip); `xytext=(x+1, y+1)` is where the text sits (the arrow tail,
offset so the text doesn't overlap the point); `arrowprops=dict(arrowstyle='->')`
configures the arrow — omit it and you get text with no arrow. For text without
arrows use `ax.text(x, y, string)`. Use annotate for "highlight this specific
observation" and text for "place this note near this area."

Q: What's the canonical pattern for overlaying a median or threshold line on a plot?
A: `ax.axhline(y=value, color='gray', linestyle='--', label='median')` draws a
horizontal line at y=value spanning the full x-range; `ax.axvline(x=value, ...)`
does the same vertically. The line auto-adjusts if you change axis limits later
— no need to compute endpoints. Common uses: median/mean markers on histograms,
decision thresholds on scatter plots, reference levels on time series. Paired
with `ax.text(..., f'median={v:.1f}')` nearby, you get a self-explanatory
reference. For lines with specific endpoints, use `ax.plot([x1, x2], [y1, y2])`
instead.

Q: What do the arguments in `fig.savefig('plot.png', dpi=150, bbox_inches='tight')` control?
A: The filename's extension picks the format (PNG raster, PDF/SVG vector, and
more). `dpi=150` sets resolution — 100 for screen, 150 for presentations, 300+
for print. `bbox_inches='tight'` crops whitespace around the figure and prevents
labels from being clipped at edges — almost always what you want. Extras:
`facecolor='white'` forces a white background (handy when your rcParams default
is transparent), `transparent=True` saves with transparent background. Vector
formats (PDF, SVG) ignore `dpi` — they're resolution-independent.

Q: Why does a Marimo cell end with `plt.gca()` (or `fig`) instead of `plt.show()` to render a plot?
A: Marimo auto-renders the cell's last expression's value. `plt.gca()` returns
the current Axes object; Marimo recognizes it and displays the underlying figure.
`plt.show()` works in regular Python scripts and classic Jupyter by triggering
an imperative display — but in Marimo it's effectively a no-op with the default
backend, so nothing renders. Convention: build the plot, optionally call
`fig.tight_layout()`, then make the LAST expression `plt.gca()` or the fig/ax
variable itself. In OO code, `fig` as the last line is the cleanest form.

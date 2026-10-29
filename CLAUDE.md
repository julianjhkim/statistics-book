# CLAUDE.md

Quarto book, `statistics-book`. Teaches introductory statistics by simulating
every result before naming it. Python chunks execute at render time; there are
no pre-baked figures.

## Structure

- Title: **Statistics by Simulation**, by Junghyun Julian Kim.
- `_quarto.yml` is a `book` project with `output-dir: docs`. GitHub Pages
  serves `docs/` on the `main` branch, so **`docs/` is committed to the repo**.
- `index.qmd` is the preface and is `{.unnumbered}`. It is currently a heading
  and nothing else.
- `number-sections: true`, so chapters and sections carry numbers (chapter 1
  section 4 is `1.4`). Those numbers are how chapters get referred to in
  conversation; they shift if a `##` is added or removed.

### Part 1: Estimating from a Sample (written)

| # | File | Chapter title |
|---|---|---|
| 1 | `populations-and-samples.qmd` | Population, Sample, Mean, and Standard Deviation |
| 2 | `standard-error.qmd` | Standard Error |
| 3 | `normal-distribution-central-limit-theorem.qmd` | Normal Distribution and Central Limit Theorem |
| 4 | `confidence-interval.qmd` | Confidence Interval |

These four are the only written chapters. Read them before editing any of them,
because they share one running example and quote each other's numbers.

### Part 2: Comparing Means (not written)

Planned order: `why-compare-means`, `t-test`, `anova`, `post-hoc`,
`two-way-anova`, `ancova`, `one-model`.

Only `t-test.qmd`, `anova.qmd` and `ancova.qmd` exist. Each is a heading plus a
`Stub: outline only.` callout, and only these three are registered in
`_quarto.yml`. The other four are not yet files. Creating one means adding the
`.qmd` and registering it in `_quarto.yml`, in the order above.

## The running example

One year group of **10,000 university students** sitting the same statistics
exam, scored out of 100. Nothing else is ever used as data, apart from the
dice in chapter 2 and three invented year groups in chapter 3.

Every chapter rebuilds the identical population in its own `setup` chunk:

```python
rng = np.random.default_rng(68)
population = np.clip(np.round(rng.normal(70, 12, 10_000)), 0, 100)
our_sample = rng.choice(population, 20, replace=False)
```

`our_sample` is drawn from the **same** generator, immediately after the
population, so it depends on the draw order of that one stream. Inserting any
other `rng` call between those two lines silently changes `our_sample` and
therefore changes numbers quoted in all four chapters. Add new generators
instead of reusing `rng`.

In chapter 1 the two lines sit in separate chunks (`setup`, then `sample`),
which is the same stream order. In chapters 2, 3 and 4 they sit together.

Numbers this produces, verified by running it:

| Quantity | Value |
|---|---|
| Population mean `mu` | 69.99 |
| Population sd `sigma` | 11.94 |
| Population range | 22 to 100 |
| `our_sample` mean `xbar` | 70.80 |
| `our_sample` sd `s` | 14.87 |
| True standard error at n=20, `11.94/sqrt(20)` | 2.67 |
| Estimated standard error, `14.87/sqrt(20)` | 3.33 |

### Seeds

Simulations that have to agree across chapters share a seed. Do not change a
seed to make a sentence true; change the sentence.

| Seed | Variable | Where | What it is for |
|---|---|---|---|
| 68 | `rng` | all four chapters, `setup` | the 10,000 student population, then `our_sample` of 20 |
| 2024 | `rng_sim` | all four chapters | the 5,000 repeated samples of 20. Chapter 2 also uses it for the n=20/60/180 comparison, chapter 3 for the band coverage counts, chapter 4 for interval coverage |
| 58324 | `rng_demo` | chapter 1, `two-draws` | `draw_1` and `draw_2`, 10 students each: same mean (68.9 and 69.4), very different spread (4.46 and 17.95) |
| 7 | `rng_shape` | chapter 3, `fig-shape-matters` | the `crowded` set of 5,000 estimates, same standard error, different shape |
| 1859 | `rng_shapes` | chapter 3, `odd-populations` | the three non hump-shaped year groups: flat, too easy, and two clumps |

`N_SAMPLE, N_REPEATS = 20, 5_000` in chapter 1 is the canonical repeated
sampling setup. Chapter 1 section 1.3's `draw_1` and `draw_2` are genuinely 10
students; everything else in Part 1 draws 20.

## Writing conventions

- **Chapter titles take no leading article.** "Standard Error", not "The
  Standard Error". A question is fine as a section heading where it fits:
  "How far off is one estimate?".
- **Section headings** are sentence case with a kebab-case id:
  `## What the mean leaves out {#sec-standard-deviation}`. Chapter ids match
  the filename.
- **One paragraph per line. Never hard-wrap prose.** A paragraph is one long
  physical line. This keeps diffs readable per paragraph. Code blocks, callouts
  and `$$` math blocks are unaffected and wrap normally.
- **No em dashes anywhere.** Use a colon or a full stop.
- **Open with data or a question, never an introduction.** No "In this chapter
  we will". Chapter 2 opens by restating the two numbers chapter 1 left behind;
  chapter 3 opens with a question a colleague might ask.
- **Introduce a concept where it is first needed**, not in advance. Degrees of
  freedom arrives in chapter 4 at the point the `t` distribution needs it, not
  earlier.
- **Simulation first, formula second.** Show the result by brute force, let the
  reader see it, then give the formula that predicts it. Chapter 2 measures the
  `1/sqrt(n)` pattern across three sample sizes before deriving it from the
  variance rules.
- **American spelling throughout**: center, favorite, behavior, summarize,
  -ize and -ization endings.
- **All numbers as numerals, not words**: 20, 5,000, 10,000, 30. Exceptions
  that stay as words are counts of abstract things ("two numbers", "three more
  times"), fractions and idiom ("half a point", "two thirds", "four times the
  data"), and anything that would leave a sentence opening on a bare numeral.
  Score decades stay as words: "the sixties and seventies".

### Score and point

An exam result is a **score**. A distance or difference on that scale is in
**points**.

- Right: "all 10,000 scores", "Lowest score", "Those 20 scores are a sample",
  axis label `"Score"`.
- Right: "12 points away", "2.7 points", "half a point apart".
- Never **"marks"**, which is British.
- Never "points" for the result itself, because "data point" is already in
  active use in chapter 1.
- "full marks" has no American equivalent worth using. Write "the top of the
  scale" or "a perfect score".

### Numbers in prose

- **Never put a simulation number into prose unless the seed is fixed and the
  code has actually been run to check it.** Run the chunk, read the output,
  then write the sentence. This applies to ranges and approximations too: "the
  high fifties to the high seventies" is a claim about output.
- **When reusing a number from an earlier chapter, show where it came from**
  rather than naming it bare. Write `11.94 / \sqrt{20} = 2.67`, not 2.67. See
  `normal-distribution-central-limit-theorem.qmd` for both of these in use.
- Prose must agree with the chunk it describes. Prose, figure captions, axis
  labels and code comments all count as prose for this purpose, and the axis
  labels are the ones that get missed.

## Build and commit

Python runs in the **conda environment `book_stats`** (`jupyter: book_stats` in
`_quarto.yml`). `execute: freeze: auto` and `warning: false` are set, so cached
chunks do not re-execute and in-document warnings are suppressed. Delete
`_freeze/` to force a genuine from-scratch render.

`_freeze/` and `.quarto/` are gitignored. `docs/` is not, and must be committed.

**A background `quarto preview` must be stopped before rendering and
committing.** Preview rewrites pages into `docs/` on its own schedule, so a
live preview and a render race each other and commit half-stale output. Check
with `ps aux | grep quarto` and confirm the process `cwd` belongs to this repo
before killing it, since previews for other projects may also be running.

The sequence is:

1. Stop any `quarto preview` running in this repo.
2. `quarto render`
3. Stage and commit, including `docs/`.

Chapter pages carry a per-file "Last updated" date from `date-modified:
last-modified`, which reads **filesystem mtime, not git history**. Rendering in
CI would stamp every page with the CI checkout time, so **rendering happens
locally** and the built `docs/` is what ships.

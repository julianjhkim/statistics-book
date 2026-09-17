# statistics-book

**Statistics by Simulation**, an open statistics textbook written in Quarto and
published to GitHub Pages at
<https://julianjhkim.github.io/statistics-book/>.

## Language

**Everything is in English**: prose, code, comments, file names, commit
messages, and conversation with the user. Do not add `lang: ko` to
`_quarto.yml` or write any part of the book in Korean.

## Audience and voice

Readers know some math (algebra, basic calculus, summation notation) but are
**new to statistics**. The governing rule:

> Explain *why* something works before *how* to run it.

In practice that means: motivate the problem, build the intuition (usually a
picture), derive the formula rather than asserting it, and only then show the
code. A chapter that opens with `scipy.stats.ttest_ind()` has failed.

There is no fixed section template. Structure follows the chapter's argument.
`why-compare-means.qmd` is the worked reference for tone, pacing, and how far
to go on intuition before notation. Read it before writing a new chapter.

## Writing conventions

- **Chapter titles avoid leading articles.** "Standard Error", not "The
  Standard Error". Most are concept names, but a question or a statement is
  fine where it serves the chapter better, as in "Why Compare Means?".
- **One paragraph per line.** Prose in `.qmd` files is never hard-wrapped at a
  fixed column: each paragraph is one long line. Code blocks, callouts and math
  blocks are unaffected.
- **No em dashes.** Use a colon or a full stop.
- **Open with data or a question.** Never open with an introduction that
  explains what the chapter is about to do.
- **Introduce a concept where it is first needed**, not in advance.

## Structure

Two parts.

**Part 1: Estimating from a Sample**

| # | File | Title |
|---|------|-------|
| 1 | `populations-and-samples.qmd` | Population, Sample, Mean, and Standard Deviation |
| 2 | `standard-error.qmd` | Standard Error |
| 3 | *planned, not yet written* | The normal distribution |
| 4 | *planned, not yet written* | Confidence intervals |

The two planned chapters have no files yet. They sit after `standard-error.qmd`
when written.

**Part 2: Comparing Means**

| # | File | Topic |
|---|------|-------|
| 1 | `why-compare-means.qmd` | Why group means, and why variation makes it hard |
| 2 | `t-test.qmd` | Two groups |
| 3 | `anova.qmd` | Three or more groups |
| 4 | `post-hoc.qmd` | Which groups differ, and the multiplicity problem |
| 5 | `two-way-anova.qmd` | Two factors, and interaction |
| 6 | `ancova.qmd` | Adding a continuous covariate |
| 7 | `one-model.qmd` | It was one linear model all along |

`t-test.qmd` through `one-model.qmd` are currently stubs: an H1 and an outline
note, no prose or code.

The order is an argument, not a menu. Part 2 builds toward `one-model.qmd`,
where the whole flowchart collapses into a single regression. Write each
chapter so that payoff stays available: prefer framings that generalize (group
means as model predictions, residual variance as unexplained signal) over ones
that dead-end in a single test's notation.

## The running example

**One example carries the whole book: a year group of 10,000 university
students sitting a statistics exam marked out of 100.** Do not introduce a
fresh dataset per chapter. The reader should be answering harder questions
about the same students each chapter, which is what makes the methods feel like
one tool sharpening rather than a list of unrelated recipes.

Simulated (not real) data is a deliberate choice: the true parameters are
known, so the text can compare what a method *estimates* against what is
*true*. Use that.

Every chapter regenerates the same population, so the students are identical
from chapter to chapter:

```python
rng = np.random.default_rng(68)
population = np.clip(np.round(rng.normal(70, 12, 10_000)), 0, 100)
```

Simulations that must agree across chapters share a seed as well. The 5,000
repeated samples of 20 students use `np.random.default_rng(2024)` in both
`populations-and-samples.qmd` and `standard-error.qmd`, so the two chapters
describe one sampling distribution rather than two similar ones.

| Seed | Used for |
|---|---|
| `68` | the population of 10,000 students, in every chapter |
| `2024` | the 5,000 repeated samples of 20, in chapters 1 and 2 |

**Never hard-code a number produced by a simulation unless the seed is fixed
and the value has been verified by running the code.** `rng` is a single
sequential stream within a chapter, so inserting or removing a draw shifts
every number after it in that file. When editing a written chapter, re-read the
prose around the chunk: the narrative quotes specific values.

## Python environment

Code runs in the conda env **`book_stats`** (Python 3.14). It is pinned in
`_quarto.yml` as `jupyter: book_stats`, via a registered kernelspec, so renders
use the right interpreter regardless of the active shell environment.

```bash
conda activate book_stats                 # numpy, pandas, scipy,
                                          # statsmodels, matplotlib, seaborn
/opt/anaconda3/envs/book_stats/bin/python -m ipykernel install \
    --user --name book_stats --display-name "Python (book_stats)"
```

Install new packages into `book_stats` (`conda install -n book_stats ...`),
never into `base`. Note there is an unrelated, stale `stat_book` kernel
registered on this machine pointing at an env that no longer exists: don't use
it.

## Build

```bash
quarto render     # full book
quarto preview    # live reload while writing
```

Output goes to **`docs/`**, which is what GitHub Pages serves (Settings →
Pages → Source: `main` branch, `/docs` folder). `docs/` is committed, not
ignored. `.nojekyll` exists in both the repo root and `docs/` to stop GitHub
from running Jekyll over the output.

`execute: freeze: auto` means unchanged chapters are not re-executed. If a
chapter's output looks stale after changing a *dependency* rather than the
chapter itself, delete `_freeze/` for that chapter and re-render.

Each chapter page carries its own "Last updated" date, from
`date-modified: last-modified` in `_quarto.yml`. That reads **filesystem
mtime**, not git history, so render locally from the working copy. A CI build
checks every file out at the same moment and would stamp all chapters with the
same date.

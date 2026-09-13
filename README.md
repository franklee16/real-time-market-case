# real-time-market-case

> Generate a 5-slide "Today's Market Case" Beamer deck by connecting a recent financial-market event to the course topic being taught that week.

A class opener built in under 3 minutes: pick a teachable event from the past 48 hours, pull live market data, render a chart, and frame the event with the week's theory + Socratic discussion questions.

---

## What it does

When triggered, the skill:

1. **Searches** the past 48 hours of financial news (general + topic-specific + Asia/HK by default).
2. **Picks** the single most teachable event — one with a clear cause/effect that maps to a course concept.
3. **Pulls live data** via `yfinance` (Yahoo Finance v8 API fallback on 429 rate limits) for the entities named in the news — plus one broad benchmark.
4. **Renders** a matplotlib chart that tells the event's story in one frame.
5. **Composes** a 5-slide Beamer deck (`event → chart → historical parallel + theory link → discussion questions`).
6. **Reviews** the deck via a read-only PASS/BLOCK critic agent (max 3 revise rounds).

The output PDF is the class opener for the day.

---

## When to trigger

Use any of:

- **Natural language**: *"today's case"*, *"market case"*, *"class opener"*, *"market event for class"*, *"what happened in markets today"*
- **Slash form**: `/today-case`, `/market-case`
- **With an argument** to bias the topic:
  - `/today-case week5` → the 5th topic in your topic map
  - `/today-case derivatives` → the topic whose slug is `derivatives`
  - `/today-case rates` → the topic whose slug is `interest_rates` (alias resolution lives in `references/topic_data_map.md`)

If no argument is given, the skill auto-detects the best topic from current news.

---

## The 5-slide structure

| # | Frame | Contents |
|---|-------|----------|
| 1 | **Title** | "Today's Market Case" + topic name + date |
| 2 | **What Happened** | 2-3 sentence summary + booktabs table of 4-6 key metrics with 1W deltas |
| 3 | **Chart** | matplotlib PNG of the named protagonists (with one benchmark) + one-line caption |
| 4 | **Historical Parallel + Theory Link** | 1-2 sentences comparing to a past event from the topic data map + the week's formula in a `\[ … \]` display block |
| 5 | **Discussion Questions** | 2-3 Socratic questions tagged by Bloom's level (Apply / Analyse / Evaluate) — every question must require using the week's framework |

---

## Output

By default the deck lands in `My Slides/Today's Case/{YYYY-MM-DD}_{topic_slug}/`:

```
My Slides/Today's Case/{YYYY-MM-DD}_{topic_slug}/
├── {YYYY-MM-DD}_{topic_slug}.tex              # Beamer source
├── {YYYY-MM-DD}_{topic_slug}_chart.png        # matplotlib figure
├── {YYYY-MM-DD}_{topic_slug}_case_audit.md    # evidence base for the critic
├── {YYYY-MM-DD}_{topic_slug}.pdf              # compiled deck
└── {YYYY-MM-DD}_{topic_slug}_critic.md        # PASS/BLOCK review (overwritten each round)
```

The output path and folder-naming convention are configurable in the skill's workflow file — change them to fit your project layout.

If the folder exists, the skill appends `_v2`, `_v3`, … to avoid clobbering.

---

## How it works (workflow)

```
Parse argument → Search news → Pick event → Fetch data → Render chart
                                              ↓
                                       Compose Beamer
                                              ↓
                                       Compile (pdflatex)
                                              ↓
                          market-case-critic review (PASS/BLOCK)
                                              ↓
                                    Auto-revise loop (max 3)
                                              ↓
                                         Ship PDF
```

The whole run is sized for ~3 minutes total. The review gate adds another ~30-60 s.

---

## Review gate

The companion `market-case-critic` is a **read-only** agent (no Write/Edit/Bash tools). It checks:

| Check | What it catches |
|-------|-----------------|
| 1. Event veracity | Mischaracterized event vs the source articles |
| 2. Direction/magnitude | Sign errors, wrong magnitudes in the metrics table |
| 3. Dates | Fabricated or out-of-order dates |
| 4. Formula correctness | Math wrong in the Slide 4 theory block |
| 5. Internal consistency | Table ↔ chart ↔ audit numbers disagree |
| 6. Topic fit | Theory link + discussion questions actually use *this week's* concepts (not a different week's) |
| 7. Bloom's level | All questions are recall (every question must *use* the framework) |
| 8. Compilation | `pdflatex` exits clean |

**Max 3 revise rounds.** Strike-3 BLOCK ships the best version anyway and surfaces the residual findings to the professor for a human decision — a class opener should not be held indefinitely.

---

## Files in this folder

| File | Role |
|------|------|
| `SKILL.md` | The skill itself (loaded into context by Claude Code). Workflow, rules, failure handling. |
| `references/topic_data_map.md` | Topic-by-topic map: tickers, news keywords, chart types, historical parallels, course concepts. **The critic's ground truth for topic fit.** Replace this with your course's topic map to port the skill. |
| `references/beamer_template.tex` | Base Beamer source — copy + fill placeholders for each new case. Swap in your own theme. |
| `references/market_data_*.csv` | Cached data from past runs (for re-renders / audits). |
| `evals/evals.json` | Eval suite for skill-triggering accuracy and output quality. |

---

## Setup

**Python packages** (the skill installs nothing itself; rely on the host project's env):

```
yfinance
pandas
matplotlib
requests
python-pptx    # for slide-extraction in the audit
```

**LaTeX**: a TeX Live or MiKTeX distribution with `pdflatex` on PATH. The skill uses `pdflatex` (not `xelatex`/`lualatex`) for fast single-pass compilation. UTF-8 source files are handled by `\usepackage[utf8]{inputenc}` in the template.

**Data sources** (in priority order):

1. `yfinance` — preferred; one-liner for daily closes.
2. Yahoo v8 chart API — used on any `429 YFRateLimitError`. Drop-in helper in `SKILL.md` Step 2.
3. A cached-data fallback directory of your choice (FRED CSVs, your own snapshots, etc.) — the path is configurable in `SKILL.md`.

---

## Usage examples

### Auto-detect (most common)
> "today's case"

The skill scans the past 48 hours, picks the most teachable event, and ships the deck.

### Topic override
> "/today-case week7" or "/today-case fx_markets"

Forces the 7th topic (or the topic whose slug is `fx_markets`). The skill still scans news but biases toward that topic.

### Specific event override
> "today's case — NVDA's earnings"

The skill treats the news query as already-provided and skips the auto-search step.

### Critique-only mode
> "review my last case"

Runs the `market-case-critic` against the most recent deck without regenerating.

---

## Customizing for your course

The skill ships wired to a 10-topic finance course template. To port to a different course:

1. **Replace `references/topic_data_map.md`** with your own topic → tickers/keywords/charts/parallels map. Each row needs: slug, default tickers, news keywords, chart type, historical parallels, course concepts.
2. **Replace `references/beamer_template.tex`** with your own Beamer theme (university branding, color palette, footer).
3. **Update the output path and data-cache directory** in `SKILL.md` (Step 5 and the cached-data fallback in Step 2) to point at your project layout.
4. **Adjust the review gate** in `market-case-critic` to use your course's slide extraction paths.

Everything else (the 5-slide structure, yfinance/v8 fallback, chart-readability rules, revise loop, discussion-question rubric) is course-agnostic.

---

## Chart-readability rules (lessons from past runs)

These are the failure modes the skill has been bitten by — keep them in mind if you tweak the chart code:

1. **Y-axis must contain all peaks** — clip `set_ylim` at ≥1.10 × global max, not arbitrary round numbers.
2. **Annotations go in empty zones** — never on a data line. Use `ax.annotate(..., arrowprops=...)` to point at the event.
3. **Legend placement for 7+ series** — below the axes (`bbox_to_anchor=(0.5, -0.12)`, `ncol=4`), not the default upper-right.
4. **One chart, one story** — drop series that aren't central to the event. 4 lines that tell the story beats 8 lines that confuse it.
5. **Caption precision** — match the caption window to the chart window ("late-July forced-unwind sell-off", not just "drawdown").

---

## Limitations

- **Past-48-hour bias.** The skill looks for events that just happened. If the news cycle is quiet, it falls back to "This Week in Markets" with generic indices.
- **No live ticker / streaming.** The data is point-in-time at skill run. Don't re-render and expect the chart to update.
- **English-only sources.** The skill is configured for English-language financial press (with Hong Kong / China query terms as a default regional bias — change in `SKILL.md` Step 1 for your audience).
- **Topic map is a static file.** Adding a new topic requires editing `references/topic_data_map.md` and rebuilding the case.
- **The critic cannot run code.** It reads the audit file you emit and web-verifies load-bearing claims. If you skip writing the audit, the critic returns BLOCK on check #8.

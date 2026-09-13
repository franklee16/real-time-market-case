---
name: real-time-market-case
description: >
  Generate a "Today's Market Case" Beamer slide deck for EF5342 by searching
  recent financial news, pulling live market data, and connecting the event to
  the course topic being taught that week. Produces a 5-slide PDF with event
  context, a matplotlib chart, historical parallel, theory link, and discussion
  questions. TRIGGER when the user says "today's case", "market case",
  "class opener", "class opening", "market event for class", "/today-case",
  "/market-case", or wants to connect current market events to course material.
  Also trigger when the user mentions a news event and wants a teaching slide
  about it, or asks "what happened in markets today" in a teaching context.
argument-hint: "[topic-override or week-number, e.g. 'derivatives' or 'week5']"
allowed-tools: Read, Grep, Glob, Write, Edit, Bash, WebSearch, WebFetch, Task
---

# Today's Market Case — Workflow

You are generating a 5-slide Beamer deck that a finance professor opens class
with. The whole point is to make the abstract concepts concrete through
something happening in markets *right now*. Students should see that the
theories they're learning explain real events.

**Time budget**: aim for under 3 minutes total. Don't over-research — this is
a class opener, not a paper.

## User preferences (carried over from EF5342 workflow)

- **Ship after first critic pass if ≥80; don't pause for confirmation.**
  "Just do it" mode is the default after a clear instruction. The professor
  opens the PDF, not a chat.
- **No AI-tells in prose.** No "It's worth noting…", "delve into", "leverage",
  "robust", "comprehensive", "moreover/furthermore" openers, em-dash drama.
  Cut throat-clearing. Concrete claims over hedging.
- **Push back when a chart is wrong.** If the user says "the chart isn't showing
  properly", that's a signal the chart violated a basic readability rule
  (wrong series, mixed units, clipped peaks, annotation overlap). Diagnose
  the root cause; don't just relabel.
- **Bash cwd doesn't persist across tool calls.** Always use absolute paths
  or chain `cd "..." && command` in a single Bash invocation. Don't rely on
  the previous turn's directory.

## Step 0: Parse the argument

Check if the user specified a topic. Read `references/topic_data_map.md` for the
alias table. Examples:

- `/today-case week5` → Week 5: Stock Markets
- `/today-case derivatives` → Week 8: Derivatives and Hedging
- `/today-case` → auto-detect from news (proceed to Step 1)

If the argument doesn't match, list the 10 topics briefly and ask.

## Step 1: Search for a teachable market event

Use WebSearch (or `mcp__tavily__tavily-search`) to find recent financial news.
Search the past 48 hours. Run 2-3 queries:

1. **General**: `"financial markets news today" 2026` — broad sweep
2. **Topic-specific** (if user specified a topic): use the "News keywords"
   from `references/topic_data_map.md` for that topic
3. **Asia/HK** (always include): `"Hong Kong" OR "China" financial market news`

Pick the single most *teachable* event — the one that best connects to a
course concept. A good event:
- Has a clear cause and effect that maps to a theory or model
- Has quantifiable market impact (prices moved, spreads changed)
- Students can reason about using frameworks from the lecture

Briefly tell the user which event you picked and which topic it maps to. If
auto-detecting, also mention the topic. Move on quickly — don't wait for
confirmation unless the user seems unsure.

## Step 2: Fetch market data

Write and run a Python script to download daily data for the matched topic.
The script should:

1. Download data for 3-7 tickers (see "Which series to pull" below)
2. Compute key metrics: current level, % change over 1W / 1M / 3M
3. Save a CSV with the raw data (for the chart)
4. Print the key metrics (for the Beamer table)

### Which series to pull — depends on the event, not the topic defaults

The `topic_data_map.md` lists **default tickers per week** (e.g., Week 9 →
SPY/QQQ/ARKK). Those are *fallbacks for a generic "this week in markets" deck*.
For an actual event-driven case, the chart and Slide 2 table must reflect the
**entities named in the news**, not generic indices.

**Rule of thumb: chart series = the named protagonists, plus one benchmark.**

| Event type | Pull | Why |
|------------|------|-----|
| Fund collapse (hedge / family office) | The fund's *core long book* as named in the article (e.g., Aschenbrenner → CRWV, SK Hynix, MU, AMD, TSM, NVDA) + the relevant broad benchmark (SOX for AI/semis) | The chart should tell the story "these are the books that broke the fund." Generic SPY/QQQ are noise. |
| Single-stock event (M&A, earnings shock, fraud) | The stock + 2-3 sector peers + a broad index | Lets students see whether the move was stock-specific or sector-wide |
| Index/macro move (rate decision, CPI, jobs) | The named indices (S&P, NASDAQ, sector ETFs) + the rates or FX that moved | Topic_data_map defaults usually apply here |
| FX event | The currency pair(s) named + DXY + a regional comparator | Topic_data_map defaults apply |
| Bond/credit event | The specific bonds or spreads named + the broad Treasury curve | Topic_data_map defaults apply |

**Always include at least one broad benchmark** (S&P 500, SOX, DXY, or
equivalent) so students can see whether the event was idiosyncratic or
systemic. But it should be **one line, secondary**, not the headline.

**Fetching the data — yfinance first, Yahoo v8 API on 429.** Yahoo Finance
enforces per-IP rate limits and `yfinance` now returns `429 Too Many Requests`
reliably — often on *every* ticker in a run, not just HK ones. So:

- Try `yfinance.download(ticker, period="3mo", auto_adjust=True)` first; it's
  one line when it works.
- On **any** `429` / `YFRateLimitError` / empty frame, **stop retrying yfinance**
  and switch to the Yahoo v8 chart API helper below. Do not burn the time budget
  on yfinance retries — the v8 endpoint is the reliable path.

**v8 API helper (drop-in — returns a pandas Series of Close prices):**
```python
import time, requests, pandas as pd
from datetime import datetime, timedelta

def yf_close(ticker, days=95):
    """Yahoo Finance daily close via the v8 chart API (bypasses yfinance 429)."""
    hdr = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"}
    url = f"https://query1.finance.yahoo.com/v8/finance/chart/{ticker}"
    params = {"period1": int((datetime.now() - timedelta(days=days)).timestamp()),
              "period2": int(time.time()), "interval": "1d", "events": "history"}
    r = requests.get(url, params=params, headers=hdr, timeout=30)
    r.raise_for_status()
    res = r.json()["chart"]["result"][0]
    close = res["indicators"]["quote"][0]["close"]
    s = pd.Series(close, index=pd.to_datetime(res["timestamp"], unit="s")).dropna()
    s.name = ticker
    return s
```
Add `time.sleep(1)` between successive ticker calls to stay polite. The values
are the same Yahoo data `yfinance` would return — yields (^TNX, ^FVX, …) come
back in percent, index ticks (^GSPC) in points, single stocks in price.

**Notes:**
- Use `encoding='utf-8'` when writing CSVs.
- Some tickers are genuinely empty (delisted, wrong symbol) — handle with try/except and move on.
- For yield data (^TNX, ^FVX, etc.), the value is the yield in %.
- **Different tickers can have different start dates.** A new IPO or a Korean
  stock may have <90 days of history even when peers have the full window.
  Don't anchor all series to a single date; normalize each to its own first
  valid observation.

**Last-resort fallback if Yahoo itself is unreachable:**
Read cached FRED CSVs from `Data and Figures/data/slides_figures/`. Each CSV
has `Date` and `Value` columns. Use the most recent 90 days.

## Step 3: Generate the chart

Write a second Python script (or extend the first) that creates a matplotlib
PNG. Follow the chart type specified in `references/topic_data_map.md` for the
matched topic.

**Chart rules** (matching the project's content-standards):
- **No title inside the chart** — the Beamer frame title provides context
- `figsize=(10, 4.5)`, `dpi=200`
- Serif font: `plt.rcParams['font.family'] = 'serif'`
- Light grid: `ax.grid(True, alpha=0.3, linestyle='--')`
- Annotate the event date with a vertical dashed line and short label
- Direct-label lines where possible; legend only for 4+ series
- Colors: `#1f77b4` (primary blue), `#ff7f0e` (secondary orange), `#2ca02c`
  (green), `#d62728` (red for negative)
- Save as PNG to the output folder

**Chart-readability rules (lessons from past runs):**

1. **y-axis must contain all peaks.** Before picking `set_ylim`, compute the
   maximum value of every normalized series (`df[col].max()`) and set the
   upper bound to at least 1.10 × that maximum. Clipped peaks look like the
   chart is broken. A 250 ceiling is sometimes right; an automatic
   `ax.set_ylim(min*0.9, max*1.15)` is safer.
2. **Annotation boxes go in empty zones, never on a data line.** The crisis
   spike is usually where students need to look — putting a label box on top
   of it is the worst overlap. Prefer empty regions (top-left, top-center)
   and use `ax.annotate` with `arrowprops` to point to the actual event.
3. **Legend placement for 7+ series.** Move the legend below the axes
   (`bbox_to_anchor=(0.5, -0.12)`, `ncol=4`) so it doesn't cover data. The
   default upper-right legend becomes unusable above 5 series.
4. **Each series is a story; one chart, one story.** If you find yourself
   adding a series "for context" that isn't central to the event, drop it.
   A 4-line chart that tells the story beats an 8-line chart that confuses it.
5. **Caption precision.** If a 3-month chart shows positions finishing *up*
   (e.g., +30%), don't caption it as "led the drawdown." Specify the window:
   "led the **late-July** forced-unwind sell-off."

**Normalization discipline.** When series have different units (KRW vs USD
vs points) or very different magnitudes, normalize each to 100 at its own
first valid date:

```python
for col in df.columns:
    s = df[col].dropna()
    norm_s = s / s.iloc[0] * 100
    ax.plot(norm_s.index, norm_s.values, ...)
```

Do not use a single anchor date when series start on different dates — it
produces NaN lines that look broken.

**Chart types by topic:**
- **Line chart** (Weeks 2, 3, 4, 8): time series of rates/yields/spreads
- **Multi-line normalized** (Weeks 1, 7): indices/currencies normalized to 100
- **Bar chart** (Weeks 5, 9): top movers or performance comparison
- **Scatter or event study** (Week 6): sector returns vs VIX, or price reaction
- **Line + bar** (Week 10): bank stock index + metrics overlay

## Step 4: Compose the Beamer slide deck

Read `references/beamer_template.tex` for the structure. Create a new `.tex`
file in the output folder with all placeholders filled in. The deck has 5 slides:

### Slide 1 — Title
- Title: "Today's Market Case"
- Subtitle: "{Topic Name} — {Date}"
- Keep the standard EF5342 / CityU author line

### Slide 2 — Context + Key Numbers
- Frame title: "What Happened: {short headline}"
- 2-3 sentences summarizing the event (who, what, when, why markets care)
- A booktabs table with 4-6 rows of key metrics. Example for a yield event:

```
Metric              Value       Change (1W)
10Y Treasury        4.52%       +15 bps
2Y Treasury         4.78%       +8 bps
Yield Curve (2s10s) -26 bps    Flatter
Breakeven Inflation 2.34%       -3 bps
```

- Source line at the bottom

### Slide 3 — Chart
- Frame title describing the chart content
- `\includegraphics[width=0.85\textwidth]{chart.png}` — the matplotlib output
- One-line caption below

### Slide 4 — Historical Parallel + Theory Link
- **Historical parallel**: 1-2 sentences comparing to a past event from the
  "Historical parallels" field in the topic data map. Be specific with dates.
  Example: "The last time the 2s10s curve inverted this deeply was August 2019,
  preceding the 2020 recession — though COVID was the trigger, not the inversion."
- **Theory connection**: 1-2 sentences linking to the week's concept. If there's
  a key formula, include it in a `\[ ... \]` display math block. The formula
  should be the one students are learning that week, not a generic one.

  Example formulas by week:
  - Week 2 (Interest Rates): Fisher equation, loanable funds, yield curve
  - Week 3 (Bonds): Bond pricing formula, duration, YTM
  - Week 5 (Stocks): Gordon growth model, P/E ratio
  - Week 7 (FX): Interest rate parity, PPP
  - Week 8 (Derivatives): Put-call parity, Black-Scholes, payoff diagram

### Slide 5 — Discussion Questions
2-3 Socratic questions, each tagged with Bloom's level (in a LaTeX comment):

**Q1 (Apply):** Directly applies the week's concept. The student must use a
framework or formula to explain the event.
> "The 10Y yield rose 15 bps after the jobs report. Using the loanable funds
> framework, which curve shifted and in which direction?"

**Q2 (Analyse):** Compare or contrast — with the historical parallel, or between
sub-markets, or across time periods.
> "Compare today's yield curve shape to March 2020. What is similar about the
> driver, and what is different?"

**Q3 (Evaluate, optional):** Judgment-oriented. The student takes a position.
> "If you manage a bond portfolio, would you extend or shorten duration today?
> Justify using what we learned about interest rate risk."

Never ask factual recall ("What is the yield curve?"). Every question should
make a student *use* the concept on the real event.

## Step 5: Create the output folder and compile

**Output location**: `My Slides/Today's Case/`

Create a folder named `{YYYY-MM-DD}_{topic_slug}/` using the slug from the
topic data map (e.g., `2026-06-13_interest_rates/`). Save all files inside:
- `{YYYY-MM-DD}_{topic_slug}.tex`
- `{YYYY-MM-DD}_{topic_slug}_chart.png`
- `{YYYY-MM-DD}_{topic_slug}_case_audit.md`  ← **the critic's evidence base (see below)**
- (After compilation) `{YYYY-MM-DD}_{topic_slug}.pdf`

If the folder already exists, append `_v2`, `_v3`, etc.

### Emit the case audit (`{slug}_case_audit.md`)

The `market-case-critic` (Step 6) is read-only and cannot run your scripts or
open the chart, so it relies entirely on this audit. Write it to the output
folder as `{YYYY-MM-DD}_{topic_slug}_case_audit.md` with `encoding='utf-8'`.
Record:

- **Event chosen** — one line, plus 2-3 source URLs with their publish dates.
- **Week + slug** — e.g. "Week 8 — derivatives".
- **yfinance metrics** — a small table (ticker, name, current value, period %
  change, as-of date). These are the raw numbers behind the Slide 2 table and
  the chart.
- **Numbers placed in the Slide 2 table** — the exact values you wrote in, so
  the critic can compare them to the yfinance metrics above.
- **Slide 4 formula** — the LaTeX you used, and which course concept it states.
- **Historical parallel** — the past event you cited and its date.
- **Week's slide concepts** — extract the bullet text of that week's
  `My Slides/weekN_*.pptx` via `python-pptx` (title + body text per slide,
  condensed). This is the topic-fit ground truth for the critic. If
  `python-pptx` is unavailable, fall back to the "Course concepts" row for that
  week in `references/topic_data_map.md` and say so.
- **Compile status** — `pdflatex` success/fail and the attempt count.

Compile with `pdflatex`:
```bash
cd "My Slides/Today's Case/YYYY-MM-DD_slug/" && pdflatex -interaction=nonstopmode YYYY-MM-DD_slug.tex
```

Max 2 attempts. If compilation fails on the first try, read the error log,
fix the .tex (common issues: chart PNG path, special characters in text,
missing package), and retry. Record the final compile status in the audit.

Do **not** report to the user yet — proceed to Step 6 (review gate). The case
ships only after the critic signs off (or after the Step 7 revise loop).

## Step 6: Review gate (critic)

Dispatch the **market-case-critic** via the Task tool. Point it at:
- the generated `.tex`, and
- the `{slug}_case_audit.md` you just wrote.

It also reads `references/topic_data_map.md` itself. The critic returns a scored
**PASS/BLOCK** report as its message — it checks factual errors (event veracity,
direction/magnitude, dates, formula correctness, internal consistency) and topic
fit (does the theory link and do the discussion questions actually use this
week's slide concepts). It has no Write/Edit/Bash tools, so it cannot touch the
case — it only reports.

**Persist the report** to `My Slides/Today's Case/{slug}/{slug}_critic.md`
(overwrite in place each round; the report records the round number).

- **On PASS:** go to Step 7's terminal report (skip the revise loop).
- **On BLOCK:** go to Step 7's revise loop.

## Step 7: Auto-revise loop

For each **BLOCK** item in the critic report, apply the minimal fix in place:

| Critic finding | Fix |
|----------------|-----|
| Formula mathematically wrong (check #4) | Replace with the correct standard form in the `.tex` |
| Event mischaracterized / wrong direction (checks #1, #2) | Correct the summary text and/or the table headline; if a metric itself is wrong, re-run the Step 2 data script for that ticker and update table + chart |
| Fabricated / out-of-order date (check #3) | Correct or remove the date in the `.tex` |
| Table ↔ chart ↔ audit disagree (check #5) | Reconcile to the yfinance metrics (authoritative), update table and re-make the chart if needed |
| Theory link / questions use a different week's concept (check #6) | Rewrite the Slide 4 theory link and Slide 5 questions to use this week's concepts (from the audit's slide extraction) |
| All discussion questions are recall (check #7) | Rewrite at least the Apply and Analyse questions to require the week's framework |
| No compiled PDF (check #8) | Re-run `pdflatex` (max 2 attempts); if it still fails, follow the Failure-handling table below |
| **Chart y-axis clipping** | Compute each series' max, raise `ylim` to ≥1.10 × global max; re-render and recompile |
| **Chart annotation overlaps a data line** | Move the annotation to an empty zone (top-left/center); use `ax.annotate(..., arrowprops=dict(arrowstyle="->"))` to point at the event |
| **Chart mixes unrelated series on one axis** | Rebuild the chart around the *named protagonists* (Step 2 "Which series to pull"); drop generic indices unless they're a benchmark |
| **Caption claims window don't match the chart's window** | Specify the window in the caption (e.g., "late-July forced-unwind sell-off" not just "drawdown") |
| Missing audit / cannot verify | Re-emit a complete `{slug}_case_audit.md` |

Apply **all** blocking fixes in one pass, then:
1. Update the `{slug}_case_audit.md` to reflect the corrected numbers/formula/dates.
2. Recompile (`pdflatex`, max 2 attempts).
3. Re-dispatch the `market-case-critic` (round 2).

**Max 3 rounds.** After round 3:
- **PASS** at any round → ship.
- **Still BLOCK after round 3** → ship the best version anyway (a class opener
  should not be held indefinitely) and surface the residual BLOCK items to the
  user for a human decision.

### Terminal report to the user

Report:
- The PDF path (so they can open it)
- A one-sentence summary of the event
- Which course topic it maps to
- The critic's final score and PASS/BLOCK verdict
- (If shipped on strike-3) the residual BLOCK items, by check number

## Failure handling

| Problem | What to do |
|---------|-----------|
| yfinance rate-limited or empty | Retry yfinance at most once. On repeat 429 / empty, switch to the Yahoo **v8 API helper** in Step 2 (the reliable path). If v8 also fails, fall back to FRED CSVs in `Data and Figures/data/slides_figures/`. |
| FRED CSVs also missing or stale | Generate the slide without a chart. Replace the chart slide with a text-only "Key Market Moves" summary. Better than failing. |
| No relevant news found | Default to the topic matching the current teaching week. Pull generic market data (S&P, 10Y, VIX) and frame as "This Week in Markets." |
| pdflatex compilation fails twice | Save the .tex and .png. Report the error. The professor can compile manually or paste into Overleaf. |
| Web search times out | Skip the news search. Use the most recent data available. Frame as a generic market update. |
| Critic still BLOCK after 3 revise rounds | Ship the best version. Surface the residual BLOCK items (by check number) to the professor for a human decision — do not loop further. |
| Critic cannot run (Task dispatch fails) | Skip the gate, ship the case, and note in the terminal report that the review was skipped and the case is **unreviewed**. |

## Speed tips

- Run news search and data fetch in parallel when possible
- Use `period="3mo"` not `"1y"` — faster download, sufficient context
- Don't write elaborate Python — a 20-line script is enough
- Don't ask for confirmation after every step. Pick the event, mention it, and go.
- The review gate (Steps 6-7) adds ~30-60s. Keep the critic's web checks
  **targeted** — verify only the 2-3 load-bearing claims (event, direction,
  dates), not every clause — so the whole run still finishes near 3 minutes.

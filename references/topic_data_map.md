# Topic-to-Data Map — EF5342

Reference for the `real-time-market-case` skill. Each row maps a course topic to data sources, news keywords, chart types, and historical parallels.

---

## Week 1: Overview of the Financial System

| Field | Value |
|-------|-------|
| **Slug** | `financial_system` |
| **yfinance tickers** | `^GSPC` (S&P 500), `^IXIC` (NASDAQ), `^HSI` (Hang Seng), `000001.SS` (SSE Composite) |
| **FRED fallback** | `SP500`, `NASDAQCOM` (in `Data and Figures/data/slides_figures/`) |
| **News keywords** | financial system, systemic risk, market crash, Fed emergency, circuit breaker, flash crash, market structure, market regulation |
| **Chart type** | Multi-line: major indices over 3 months, normalized to 100 at start |
| **Historical parallels** | 2008 GFC (Lehman), 2020 COVID crash, 1987 Black Monday |
| **Course concepts** | Financial intermediation, direct vs indirect finance, adverse selection, moral hazard |

## Week 2: Determinants of Interest Rates

| Field | Value |
|-------|-------|
| **Slug** | `interest_rates` |
| **yfinance tickers** | `^TNX` (10Y yield), `^FVX` (5Y yield), `^IRX` (13W T-bill), `^TYX` (30Y yield) |
| **FRED fallback** | `FEDFUNDS`, `DGS10`, `TB3MS`, `T10Y3M`, `T10YIE`, `DFII10` |
| **News keywords** | interest rate, Federal Reserve, FOMC, rate decision, rate hike, rate cut, inflation, CPI, yield curve, monetary policy, tapering, quantitative easing |
| **Chart type** | Line chart: Treasury yields (2Y/5Y/10Y) over 3 months with event annotation; OR yield curve snapshot (x-axis: maturity, y-axis: yield) comparing today vs 1 month ago |
| **Historical parallels** | Volcker rate hikes (1980s), 2019 yield curve inversion, 2022 aggressive tightening cycle |
| **Course concepts** | Loanable funds framework, liquidity preference, Fisher effect, term structure, expectations hypothesis, risk premium |

## Week 3: Bond Markets and Bond Pricing

| Field | Value |
|-------|-------|
| **Slug** | `bond_markets` |
| **yfinance tickers** | `TLT` (long Treasury), `IEF` (7-10Y Treasury), `LQD` (investment-grade corp), `HYG` (high-yield corp), `BND` (total bond) |
| **FRED fallback** | `DGS10`, `AAA`, `BAA10Y`, `BAMLH0A0HYM2` |
| **News keywords** | bond market, Treasury auction, corporate bonds, credit spread, default, downgrade, credit rating, duration, convexity, TIPS, inflation-linked |
| **Chart type** | Dual-line: investment-grade spread vs high-yield spread over 3 months; OR bar chart of yield by rating tier (AAA, AA, A, BBB, HY) |
| **Historical parallels** | 2008 credit freeze, 2020 March liquidity crisis, Silicon Valley Bank (2023) duration losses |
| **Course concepts** | Bond pricing (PV of cash flows), yield to maturity, duration, convexity, credit risk, default risk premium, credit ratings |

## Week 4: Money Markets

| Field | Value |
|-------|-------|
| **Slug** | `money_markets` |
| **yfinance tickers** | `BIL` (1-3M T-bill), `SGOV` (0-3M T-bill) |
| **FRED fallback** | `SOFR30DAYAVG`, `DCPF1M`, `TGCRRATE`, `TEDRATE`, `TB3MS` |
| **News keywords** | money market, repo market, SOFR, commercial paper, liquidity, T-bill auction, cash crunch, LIBOR, overnight rate, reverse repo |
| **Chart type** | Line chart: SOFR vs Fed Funds vs 3M T-bill over 3 months; OR TED spread over 3 months |
| **Historical parallels** | 2008 repo freeze, 2019 repo spike, 2020 March money market stress |
| **Course concepts** | Money market instruments (T-bills, commercial paper, repos, NCDs), SOFR vs LIBOR, discount yield vs bond equivalent yield |

## Week 5: Stock Markets

| Field | Value |
|-------|-------|
| **Slug** | `stock_markets` |
| **yfinance tickers** | `^GSPC` (S&P 500), `^HSI` (Hang Seng), `^IXIC` (NASDAQ), plus top movers (e.g., `NVDA`, `AAPL`, `TSM`) |
| **FRED fallback** | `SP500`, `NASDAQCOM`, `VIXCLS` |
| **News keywords** | stock market, S&P 500, IPO, index rebalance, market cap, trading volume, tech stocks, earnings, rally, selloff, circuit breaker, market correction, bull market, bear market |
| **Chart type** | Line chart: index performance over 3 months with event annotation; OR bar chart of top 5 sector ETF returns past month |
| **Historical parallels** | 2000 dot-com bubble, 2020 COVID recovery, 2022 tech selloff |
| **Course concepts** | Stock exchanges, order types, market microstructure, short selling, market indices, P/E ratios, market cap |

## Week 6: Market Efficiency and Anomalies

| Field | Value |
|-------|-------|
| **Slug** | `market_efficiency` |
| **yfinance tickers** | `^GSPC`, `^VIX`, sector ETFs (`XLF`, `XLK`, `XLE`, `XLV`, `XLU`) |
| **FRED fallback** | `VIXCLS`, `SP500` |
| **News keywords** | market anomaly, momentum, value premium, size effect, earnings surprise, unusual volume, insider trading, market efficiency, behavioral finance, bubble, herding, meme stock, short squeeze |
| **Chart type** | Bar chart: sector returns past month; OR event-study line showing price reaction around earnings/news event |
| **Historical parallels** | 2021 meme stock frenzy (GME), 2000 dot-com bubble, LTCM (1998), January effect |
| **Course concepts** | EMH (weak, semi-strong, strong), anomalies (momentum, value, size, January effect), behavioral biases (overconfidence, herding, loss aversion) |

## Week 7: Foreign Exchange Markets

| Field | Value |
|-------|-------|
| **Slug** | `fx_markets` |
| **yfinance tickers** | `CNYUSD=X` or `CNY=X` (CNY/USD), `EURUSD=X` (EUR/USD), `JPYUSD=X` or `JPY=X` (JPY/USD), `GBPUSD=X` (GBP/USD), `DX-Y.NYB` (DXY) |
| **FRED fallback** | `DEXCHUS`, `DEXSIUS`, `TWEXBMTH` |
| **News keywords** | exchange rate, currency, yuan, dollar, euro, forex, central bank intervention, carry trade, currency crisis, peg, devaluation, capital controls, trade war, tariffs |
| **Chart type** | Multi-line: major currency pairs vs USD over 3 months (normalized to 100); OR DXY index line chart |
| **Historical parallels** | 1997 Asian Financial Crisis, 2015 CNY devaluation, 2022 strong dollar, Swiss franc unpeg (2015) |
| **Course concepts** | Exchange rate regimes, PPP, interest rate parity, carry trade, currency appreciation/depreciation, spot vs forward |

## Week 8: Derivatives and Hedging

| Field | Value |
|-------|-------|
| **Slug** | `derivatives` |
| **yfinance tickers** | `^VIX` (VIX), `SPY` (underlying for options context), `GC=F` (gold futures), `CL=F` (crude oil futures) |
| **FRED fallback** | `VIXCLS` |
| **News keywords** | derivatives, options, futures, hedging, volatility, VIX spike, margin call, contango, backwardation, options expiry, Greeks, swap, credit default swap, derivatives regulation |
| **Chart type** | Line chart: VIX over 3 months with event annotation; OR dual-axis: underlying price (left) vs implied volatility (right) |
| **Historical parallels** | 2008 CDS crisis, 2020 oil futures going negative, 2018 Volmageddon ( XIV implosion ) |
| **Course concepts** | Call/put options, futures contracts, payoff diagrams, hedging strategies, Black-Scholes intuition, implied volatility, Greeks, speculation vs hedging |

## Week 9: Asset Management Industry

| Field | Value |
|-------|-------|
| **Slug** | `asset_management` |
| **yfinance tickers** | `SPY` (S&P ETF), `QQQ` (NASDAQ ETF), `ARKK` (active/disruptive), `GLD` (gold ETF), `VTI` (total market) |
| **FRED fallback** | (none typically needed — yfinance covers ETFs well) |
| **News keywords** | ETF, mutual fund, hedge fund, passive investing, active management, fund flows, asset management, ESG fund, index fund, factor investing, smart beta, fund performance, fee war |
| **Chart type** | Bar chart: ETF/fund performance comparison over 3 months; OR pie chart of asset allocation; OR line chart comparing active vs passive AUM proxy |
| **Historical parallels** | ARK Innovation peak/decline (2021-2022), 2008 hedge fund redemptions, rise of passive investing (2010s) |
| **Course concepts** | Active vs passive management, ETFs vs mutual funds, NAV, expense ratios, factor investing, ESG, hedge fund strategies (L/S, global macro, event-driven) |

## Week 10: Banking Industry

| Field | Value |
|-------|-------|
| **Slug** | `banking` |
| **yfinance tickers** | `XLF` (financials ETF), `KBE` (bank ETF), `JPM`, `BAC`, `HSBA.L` (HSBC), `1398.HK` (ICBC) |
| **FRED fallback** | `USNIM`, `USROA`, `USROE`, `NPTLTL`, `EQTA`, `CORCCACBS` |
| **News keywords** | bank earnings, banking crisis, deposit flight, bank failure, NIM compression, credit loss, bank regulation, stress test, capital requirement, regional bank, SVB, credit crunch, bank run |
| **Chart type** | Line chart: bank stock index over 3 months; OR bar chart of key banking metrics (NIM, ROE, NPL ratio) |
| **Historical parallels** | 2023 SVB / regional banking crisis, 2008 global financial crisis, 1990s S&L crisis |
| **Course concepts** | Bank balance sheet, NIM, ROA/ROE, capital adequacy (Basel), credit risk, deposit insurance, bank regulation, too-big-to-fail |

---

## Argument Parsing Aliases

When the user passes a topic override, these aliases map to the correct week:

| Alias | Maps to |
|-------|---------|
| week1, overview, financial system, intro | Week 1 |
| week2, interest rate, interest rates, yield, fed, FOMC | Week 2 |
| week3, bond, bonds, bond market, credit, fixed income | Week 3 |
| week4, money market, money, SOFR, repo, cash | Week 4 |
| week5, stock, stocks, equity, equities, IPO, market | Week 5 |
| week6, efficiency, anomaly, EMH, behavioral, bubble | Week 6 |
| week7, FX, forex, currency, exchange rate, yuan, dollar | Week 7 |
| week8, derivative, derivatives, option, options, futures, hedging, VIX | Week 8 |
| week9, fund, ETF, mutual fund, hedge fund, asset management, ESG | Week 9 |
| week10, bank, banking, deposit, NIM, regulation | Week 10 |

If the argument doesn't match any alias, list the 10 topics and ask the user to pick.

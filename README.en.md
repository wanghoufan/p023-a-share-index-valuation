# A-Share Eleven Indexes · Ten-Year Valuation Percentiles

[简体中文](./README.md)

> A single-page HTML report plus its data notes, collecting the ten-year valuation percentiles of **11 core A-share indexes** (5 broad-market + 6 dividend) and labelling each as expensive / neutral / cheap so the current level is obvious at a glance.

🌐 **Live version**: [https://a-share-index-valuation-report.vercel.app/](https://a-share-index-valuation-report.vercel.app/)

## Preview

> Real screenshot of the deployed page (top of the homepage).

![Homepage top](screenshots/preview-hero.png)

## Index list

**Broad-market indexes (5)**

| Index | Cap tier | Construction note |
|---|---|---|
| CSI 300 | Large cap | The 300 largest, most liquid leaders across Shanghai and Shenzhen; about 60% of total A-share market cap |
| CSI 500 | Mid cap | Ranks 301–800 by market cap after excluding CSI 300 (average cap roughly ¥20–33bn) |
| CSI A50 | Large cap | The single largest leader in each level-1 industry ("core assets"), published 2024-01-02 |
| CSI A500 | Large cap, broad | Cross-industry large-cap broad index; about 77% of weight from CSI 300 and 18% from CSI 500, median cap about ¥50bn |
| Wind All-A (ex financials) | Whole market | All A shares excluding financials (banks, brokers, insurance), used as a whole-market proxy |

**Dividend indexes (6, in table order)**

| # | Index |
|---|---|
| 1 | Dividend Low Volatility |
| 2 | 300 Dividend Low Volatility |
| 3 | Dividend Low Volatility 100 |
| 4 | S&P China A-Share LargeCap Dividend Low Volatility 50 |
| 5 | Dividend Quality |
| 6 | Eastmoney Dividend Low Volatility |

## Dimensions and the expensive/cheap rule

All five percentile columns share one colour threshold:
- `<30%` low (green) · `30%–80%` neutral (amber) · `>80%` high (red)

Expensive/cheap judgement (consistent with "higher PE/PB = more expensive, higher dividend yield / risk premium = cheaper"):
- **PE / PB percentile**: higher → more expensive
- **Dividend yield / risk premium percentile**: higher → cheaper (more worth buying)

## Data sources and method

- **Current values (stable)**: Tencent `westock-data` Skill (`qt.gtimg.cn`), reconciled item by item against the public quote sheet "A股行情指标" (2026-08-06).
- **Percentiles (range reference)**: public figures from Wind / Lixinren / Xueqiu / Tonghuashun as disclosed in the media, cross-checked across sources. Different windows and dates, **not a stable channel** — reference ranges only.
- Risk premium (ten-year P/E) ≈ `100% − PE ten-year percentile` (an estimate derived from the PE percentile, standing in for the ten-year percentile of `1/PE − 10Y government bond`, which moves little).
- Risk premium (ten-year dividend) = the ten-year percentile of `dividend yield − 10Y government bond (≈1.73%)`.

## Important limitations

- All paid connectors (Wind / Eastmoney Miaoxiang / iFinD) are disconnected with no account, so an authoritative ten-year series cannot be obtained.
- Ten-year boundary: CSI A50 (2024-01), Eastmoney Dividend Low Volatility (2020-04) and 300 Dividend Low Volatility (2018-12) are younger than ten years and are labelled with the longest available window.
- For Wind All-A (ex financials) the free source only publishes Wind All-A, so that is used as a proxy; some S&P Dividend Low Volatility 50 percentiles are estimated or use a three-year window; Dividend Quality (931468) only has a five-year percentile, used to approximate the ten-year one.
- The data is **not investment advice**, and valuation percentiles move daily with the market.

## Data and presentation are separated (JSON-driven)

The report keeps data and presentation apart: every valuation number lives in `data.json`, and JavaScript loads and renders the table at page load. **Updating the numbers means replacing `data.json` only** — the page itself is untouched, so there is no risk of breaking the report.

## Change markers (highlight what moved)

Each percentile cell carries a "versus last period" comparison. When two periods differ by **more than 5pp**, the cell shows a `▲/▼ N.Npp` badge (▲ red = percentile up = more expensive, ▼ green = percentile down = cheaper, following the Chinese convention of red for up and green for down) and plays one **pulse highlight animation**, so indexes that moved materially are obvious at a glance; differences of 5pp or less stay unmarked.

## Automatic data-freshness check

A freshness indicator renders at the top of the page, graded by how many days old the data date is:
- **under 7 days**: green "fresh"
- **7–30 days**: yellow "note · getting old"
- **over 30 days**: red "expired"

So the first thing you see tells you whether the numbers can be trusted.

## Cross-index comparison · radar chart

At the bottom of the page sits a **pure SVG radar chart** (no dependencies, works offline, no CDN or external library), comparing six core indexes across four valuation dimensions:

- **Six lines (distinct colours)**: CSI 300, CSI 500, CSI A500, Wind All-A (ex financials), Dividend Low Volatility, Dividend Quality.
- **Four axes**: PE percentile, PB percentile, dividend-yield percentile, risk premium (ten-year dividend) percentile.
- **How to read it (important)**: the outer ring = higher percentile; **on the PE/PB axes further out means more expensive**, while **on the dividend/risk-premium axes further out means cheaper**. So a shape leaning toward the PE/PB axes is currently expensive, one leaning toward the dividend/risk-premium axes is relatively cheap — you can see which index is cheapest or dearest on which dimension.
- **Clickable legend**: clicking a legend item shows or hides that index, which makes pairwise comparison easy.
- Values come from `data.json` (the `v` of each index's `pe/pb/div/rp2`), so the radar redraws automatically when data is updated; a dimension marked `na` is treated as 0.

## Mini trend sparkline per percentile

Next to each index name sits a **mini line chart of its recent PE percentile** (inline SVG `<polyline>`, no dependencies):

- Values come from a new `history` field on each `data.json` row (a series of ten-year PE percentiles; the last point is the current one).
- Line colour follows the trend: **rising (percentile climbing = getting dearer) → red**, **falling (percentile sliding = cheaper) → green**, with a dot marking the latest value; hovering (aria-label) reveals the series.
- Why it helps: sliding down to 8% over years is a very different proposition from crashing to 8% this week.
- Maintenance: add or update the `history` array per row when refreshing `data.json`; if the field is missing or has fewer than two points the sparkline simply does not render and the table is unaffected.

> Note: the `history` currently in this repository is **sample shape data**, not real historical snapshots; it exists to demonstrate the chart. Real use requires maintaining each index's actual PE percentile series.

## Latest close and drawdown from the ten-year high (price dimension)

After the "index (price index)" column and before the PE/PB percentile columns sit two **price columns**, adding a price-position view that percentiles alone do not give:

- **Latest close**: the newest close for each index (or its proxy ETF) as of 2026-08-07, formatted with thousands separators.
- **Drawdown from the all-time high in the window**: `dd = (latest close − highest close since 2016) ÷ highest close × 100%`, where the high comes from `MAX(high)` of the `westock-data` daily candles (2016-01-01 to date). The cell shows the drawdown in large type (e.g. `-7.3%`) with "high XXXX.XX (YYYY-MM)" in small type.
- **Five colour bands** (deeper drawdown, further from the peak = greener = more margin of safety / relatively cheaper):
  - `0% to -10%` → red
  - `-10% to -20%` → yellow
  - `-20% to -30%` → light green
  - `-30% to -40%` → green
  - below `-40%` → dark green
- **Missing data**: Wind All-A (ex financials) has no free standalone code, so both columns read "no data source"; S&P Dividend Low Volatility 50 is proxied by its tracking ETF (515450).

The page is responsive: on phones it collapses into stacked cards with no horizontal scrolling.

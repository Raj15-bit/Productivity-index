# India Monthly Productivity Pulse

A self-contained, single-file dashboard tracking India's monthly productivity proxies using official MoSPI data plus high-frequency nowcast indicators.

**Live demo (after enabling GitHub Pages):** `https://<your-username>.github.io/<repo-name>/`

---

## What this is

`index.html` is a fully self-contained HTML dashboard (~133 KB, no build step, no server). Open it in any modern browser. It uses three CDN libraries — Chart.js, Grid.js, Mermaid — all loaded over HTTPS from jsdelivr.

Every numeric value displayed anywhere in the dashboard originates from one constant defined at the top of the script:

```js
const MASTER_DATA_AS_OF_2026_05_03 = { /* ... */ };
const MD = MASTER_DATA_AS_OF_2026_05_03;
```

Every section — Snapshot, Market Stress, Macro Pulse, India Map, IIP Heatmap, Core Industries, Mega Table, Markets charts, Manual Nowcast, Calendar, Consumption, Detailed Tables, Interpretation, Data Health — auto-derives from this object. Change one value in `MD` and it propagates everywhere.

---

## Sections

1. **Snapshot** — 24 verified prints as of 3 May 2026 (equity close 30 Apr because 1 May was Maharashtra Day)
2. **Market Stress** — daily/intraday metrics (Brent close vs stress high, INR with intraday low, 10Y, FX, FII/DII, peak power)
3. **Macro Pulse** — 6 headline KPI cards (CPI / WPI / IIP / GDP / Unemployment / Forex Reserves)
4. **India Inflation Map** — 37-state SVG choropleth, click any state for drill-down
5. **IIP Sector Heatmap** — 10 categories × 12 months
6. **Eight Core Industries** — heatmap with weights, leaders/laggards table
7. **Macro Mega Table** — 36+ indicators × 8 columns (Oct-25 → Apr-26 split into Official + Partial/Nowcast), with per-cell metadata tooltips and category filter
8. **Markets & Flows** — FII/DII bar chart, CPI vs WPI line, PMI line, weekly price moves bar
9. **Manual Nowcast** — 13-card grid: power, Brent close, Brent stress high, INR, 10Y, FII/DII, FPI NSDL, GST, UPI, bank credit, liquidity, FX
10. **Release Calendar** — Apr-May 2026 with auto-status (Released / Today / Upcoming / Delayed / Fetch failed)
11. **What changed since 18 April** — 12-row diff table
12. **Consumption (HCES MPCE)** — Structural Consumption Benchmark, NOT a monthly live tracker
13. **Detailed Tables** — sortable Grid.js tables for CPI / WPI / IIP / GDP / Unemployment / Forex
14. **Interpretation** — 7-category signal model: Productivity / Demand / Inflation/Cost / Labour / Liquidity-Market / Second-Order / What-changed
15. **Data Health** — 17 sources, each with **Fetch status** (✓ API success / ✗ failed / – Manual) AND **Data freshness** (separate fields — fetch can succeed yet serve stale data)

---

## Source priority hierarchy

1. **Official source** — MoSPI press, OEA/DPIIT, RBI release, CBIC
2. **Regulator/platform** — RBI MPC, NPCI, AMFI, NSE/NSDL
3. **Exchange** — NSE/BSE close, ICE Brent
4. **Credible media fallback** — Reuters / Bloomberg / Mint / BS / BL — only when (1) lags
5. **Manual fallback** — last verified value, chip-tagged "Manual"

Visible label tells the reader which level produced the displayed number. Curated values (`MASTER_DATA`) always come from Level 1.

---

## How to update

To refresh data for a new date:

1. **Run web searches first** (per A26 policy: up to 20 batched searches before patching). Verify each data point with ≥2 independent sources.
2. **Edit `MASTER_DATA_AS_OF_<DATE>` at the top of the script** in `index.html`. Update period, value, releaseDate, status, freshness, nextRelease per metric.
3. **Update `Decisions.md`** with a new A-numbered decision describing what changed and why.
4. **Update `TaskBoard.md`** with the new sprint row.
5. **Commit** and (if Pages enabled) push.

That's it. No build step, no transpile, no bundler.

---

## Files in this repo

| File | What it does |
|---|---|
| `index.html` | The dashboard. Self-contained. Open directly. |
| `Architecture.md` | Product goal, data layers, fetch sequence, dashboard sections, freshness/colour/revision logic, Source Priority Hierarchy, Web-Search Batch Policy, Data Health schema, MASTER_DATA schema |
| `Decisions.md` | Numbered decision log A1-A32+ with rationale and trade-offs for every architectural choice |
| `TaskBoard.md` | Sprint-by-sprint task tracker with v1-v13 history and forward-looking backlog |

---

## Architecture rules

- **Never mix data layers without labels** — Official monthly / High-freq nowcast / Market-liquidity / Manual snapshot / Interpretation are distinct, chip-tagged, never blended silently
- **Direction colour ≠ Economic-impact colour** — CPI ▲ shows a red border (rising inflation = bad), GDP ▲ shows green
- **Fetch status ≠ Data freshness** — an API call can succeed and still return stale data; both must be visible
- **Calendar status taxonomy** — Released / Today / Upcoming / Delayed / Fetch failed (no question marks)
- **Brent must be split** — `brentClose` (latest verified close) and `brentStressHigh` (recent intraday peak) are distinct fields, never confused
- **NSE FII/DII vs NSDL FPI never mixed** for absorption ratios (different scopes)

Read `Architecture.md` and `Decisions.md` before changing anything substantial.

---

## Setup as a GitHub repo

```
git init
git add .
git commit -m "Initial: India Monthly Productivity Pulse v13"
git branch -M main
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main
```

To enable GitHub Pages: Settings → Pages → Source = `main` branch, root folder. Your dashboard is live at `https://<your-username>.github.io/<repo-name>/`.

---

## Disclaimer

Not investment advice. All values represent the agent's best understanding as of 3 May 2026 (web-search verified). Re-verify against primary sources before making decisions.

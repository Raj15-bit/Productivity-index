# Architecture.md

## Product Goal
India Monthly Productivity Pulse tracks monthly productivity proxies using official MoSPI data plus high-frequency nowcast indicators.

## Core Principle
Never mix official monthly data, market data, manual snapshot data, and high-frequency data without labels.

## Data Layers
1. Official monthly data
2. High-frequency nowcast data
3. Market/liquidity stress data
4. Interpretation layer

## Source Priority
Official source > regulator > exchange/platform > credible media fallback > manual fallback.

## Fetch Sequence
1. Fetch MoSPI/eSankhyiki
2. Fetch MoSPI press releases if API fails
3. Fetch OEA/DPIIT for WPI and Core
4. Fetch RBI for forex, rates, liquidity, credit
5. Fetch GST/NPCI/AMFI/NSE/NSDL
6. Fetch high-frequency sources
7. Validate freshness
8. Render dashboard
9. Write fetch log

## Dashboard Sections
1. Data Health
2. Official Monthly Productivity Core
3. Labour and Productivity Pressure
4. Eight Core Industries
5. Inflation and Cost Pressure
6. Demand and Consumption Pulse
7. 10-Day / Half-Month Nowcast
8. Market and Liquidity Stress
9. Source-Level Detailed Tables
10. Interpretation and Second-Order Effects

## Freshness Logic
Fresh, lagged but latest, stale, manual fallback, estimate, flash, provisional, final.

## Colour Logic
Separate direction from economic impact.

## Revision Policy
When official data is revised, show previous value, revised value, and revision date.

## Build Sequence
- Phase 1: Fix labels and stale data.
- Phase 2: Add source/freshness panel.
- Phase 3: Add productivity proxy scoring.
- Phase 4: Add 10-day nowcast layer.
- Phase 5: Add interpretation engine.

---

## Compliance Checklist (every future build must pass)

Before shipping any update_artifact:

- [ ] Every cell carries a layer label (Official / Nowcast / Market / Interpretation)
- [ ] Every value carries a freshness tag (Fresh / Lagged / Stale / Manual / Estimate / Flash / Provisional / Final)
- [ ] Source pulled in priority order (no media-fallback used when an official source is available)
- [ ] Fetch log written after every refresh (date, source, status code, freshness)
- [ ] Colour separates direction (▲▼) from impact (good / bad / neutral) — rising unemployment is ▲ but bad
- [ ] Revisions show previous → revised + revision date
- [ ] Dashboard sections present in the order specified
- [ ] No silent data mixing across layers

---

## Current build status (v8 BRUTAL FIX — 2026-04-26)

- Phase 1 ✓ DONE (v6, hardened v8): labels uniform; v8 added: every label MATCHES the value shown.
- Phase 2 ✓ DONE (v7, expanded v8): Data Health now includes per-source next-release date + fallback values + Curated Headlines health card.
- Phase 3 ⊘ NOT STARTED: productivity proxy scoring (composite of IIP + power + freight + PMI weighted by sector).
- Phase 4 ✓ DONE (v7, refreshed v8): 10-day Nowcast updated with 25-30 Apr values (256.1 GW power · INR 95.33 intraday low · FPI -₹60,847 cr NSDL distinct from NSE FII -₹39,220 cr).
- Phase 5 ✓ DONE v0 (v7): Interpretation with hand-curated rows. Auto-derivation deferred.

**v8 NEW: Curated Override layer** — CURATED_HEADLINES const carries press-release values, applyCuratedOverrides() forcibly rewrites every KPI card with these values (overriding stale API). Source priority hierarchy formally adopted (A14).

All 15 TaskBoard items (T001–T015) shipped. v8 brutal fix corrected 15 misalignments where labels claimed one value but card displayed another. See Decisions.md A1–A17 for full rationale.

## Source Priority Hierarchy (adopted v8)
1. Official source (MoSPI press, OEA/DPIIT, RBI release, CBIC)
2. Regulator/platform (RBI MPC, NPCI, AMFI, NSE/NSDL)
3. Exchange (NSE/BSE close, ICE Brent)
4. Credible media (Reuters/Bloomberg/Mint/BS/BL) — only when (1) lags
5. Manual fallback (chip-tagged Manual)

Curated values always come from Level 1. API values fall to Level 2. Visible label tells user which level produced the displayed number.

## Web-Search Batch Policy (adopted v12 — A26)
**When user requests a date refresh or new prints:**
- Agent runs up to **20 batched web searches** before patching.
- Stop only when each requested data point has ≥2 independent verified sources, or 20-batch budget exhausted.
- **Batching strategy:**
  - 1-5: broad context + headline (CPI/WPI/IIP/GDP/Forex)
  - 6-10: secondary indicators (PMI/GST/UPI/FII/DII/NSDL FPI)
  - 11-15: high-frequency nowcast (Brent/INR intraday/10Y/power/liquidity)
  - 16-20: validation cross-checks + holiday/release-calendar
- **Source-priority hierarchy applies** — Level 1 result trumps Level 4.
- **Document every batch** in Decisions.md per-decision log.
- **NSE-vs-NSDL discipline (A28):** never compute absorption ratios across different data scopes.

## Data Health Schema (adopted v12 — A27)
Every source card has TWO separate fields:
1. **Fetch status:** ✓ API success / ✗ API failed / – Manual (no live hook)
2. **Data freshness:** chip-tagged period + status (e.g., "Dec 2025 · stale/lagged" vs "Mar 2026 · final")
A fetch can succeed and still serve stale data. Both states must be visible.

## MASTER_DATA Single Source of Truth (adopted v13 — A31)
**Mandatory pattern from v13 onward:** every numeric value displayed anywhere in the dashboard MUST originate from `MASTER_DATA_AS_OF_<DATE>` at the top of the script. No section may keep its own hardcoded numbers. Auto-derivation builders construct SNAP_KPIS / STRESS_KPIS / NOWCAST_CARDS / MEGA_MACRO Apr cells / RELEASES / CHANGES / CURATED_HEADLINES / PRICE_BARS from MD at script init.

**Why:** prevents drift. If Brent changes from $108.17 to $110.42, change MD once; every section updates.

**Field schema per metric:**
- `value` (formatted display string e.g. "+3.40%")
- `valueNum` (raw number for chart inputs)
- `period` (e.g. "Mar 2026", "wk-ended 24 Apr 2026")
- `src` (source attribution)
- `releaseDate` (when published)
- `freq` (Daily / Fortnightly / Monthly / Quarterly)
- `status` (provisional / final / quick estimate / flash / manual fallback)
- `freshness` (period + status combined)
- `confidence` (high / medium / low)
- `impact` (up=good / up=bad / down=good / down=bad / context)
- `nextRelease` (when next update expected)
- `delta` (formatted change vs prior period)
- `base` (where applicable: 2012, 2011-12, 2022-23)
- `spark` (array for sparkline chart)

**Brent special case (A31):** split into `markets.brentClose` (latest verified close) AND `markets.brentStressHigh` (recent intraday spike). Both visible everywhere as distinct fields. Never show one as the other.

**Apr-blank discipline:** for indicators not yet officially released (CPI Apr, WPI Apr, IIP Apr, Core Apr, SIP Apr, Equity MF Apr, FADA Apr, DGCA Apr), MD entries leave `value: null` or omit Apr field. Mega-table renders these as blank with tooltip showing "due [release date]".

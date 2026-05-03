# Decisions.md

## Decision Log

### Decision 1: Use MoSPI/eSankhyiki for CPI/IIP/GDP/PLFS
- Reason: MoSPI is the official source for these datasets.
- Alternative considered: Using media reports.
- Why rejected: Media can be delayed, rounded, or context-light.

### Decision 2: Use OEA/DPIIT for WPI and Eight Core
- Reason: WPI and Eight Core are not MoSPI-owned primary series.
- Alternative considered: Pulling everything from MoSPI.
- Why rejected: Wrong source ownership creates data gaps and stale values.

### Decision 3: Separate official monthly data from nowcast data
- Reason: MoSPI data is monthly/quarterly, while power/market data can be daily.
- Alternative considered: Putting everything in one "live" dashboard.
- Why rejected: It creates fake-live impression and trust risk.

### Decision 4: Separate direction colour from economic-impact colour
- Reason: Rising CPI, Brent, INR/USD, and yields are not automatically positive.
- Alternative considered: Green for up, red for down.
- Why rejected: Misleading macro interpretation.

---

## Decisions made by Agent (this thread)

### Decision A1: Use adarshbiradar/maps-geojson for India SVG
- Date: 2026-04-26
- Reason: Public-domain, 131KB raw, simplifies cleanly to ~14KB inline. Covers all 36 states/UTs + Ladakh.
- Alternative considered: Wikimedia 304KB SVG (too large), simplemaps (license unclear).
- Why rejected: Size and license trade-offs.

### Decision A2: Inline INDIA_GEO as JS const, not script[type=application/json]
- Date: 2026-04-26
- Reason: First v5 attempt used script tag with `__GEOJSON_PLACEHOLDER__` and the placeholder leaked into prod, breaking JSON.parse.
- Alternative considered: Fetch geo from CDN at runtime.
- Why rejected: Artifact CDN allowlist blocks arbitrary fetches.

### Decision A3: Robust callMcpTool wrapper tries 4 signatures
- Date: 2026-04-26
- Reason: Cowork bridge signature undocumented; first call shape may not match.
- Alternative considered: Hardcode one signature.
- Why rejected: Brittle — failure mode would be silent total breakage.

### Decision A4 (pending): Rename "India Macro Morning" → "India Monthly Productivity Pulse"
- Status: Approved by user 2026-04-26 (per TaskBoard T001).
- To execute in v6.

### Decision A5: Remove "Live MOSPI" header label — DONE in v6
- Replacement shipped: Layer-chip taxonomy (Official / Nowcast / Market / Snapshot / Interpretation) + Freshness chip taxonomy (Fresh / Lagged / Stale / Manual / Flash / Provisional / Final) on every KPI card.

### Decision A6: WPI manual fallback when API empty
- Date: 2026-04-26 (v6)
- Reason: MoSPI eSankhyiki WPI endpoint sometimes returns empty rows for current month even when OEA has published. Going blank is worse than showing the most recent verified manual value with a clear "Manual snapshot" chip.
- Implementation: If renderWPI's headline series is empty, render +3.88% (Mar 2026 from OEA) with the WPI card's freshness chip rewritten in-place to "Manual snapshot" (pink).
- Alternative considered: Leave card blank.
- Why rejected: Architecture core principle is "never mix data without labels" — but blank is also bad UX. A clearly-labeled manual fallback is honest and useful.

### Decision A7: Calendar status computed at render time, not stored
- Date: 2026-04-26 (v6)
- Reason: Static "Upcoming" labels become wrong as soon as the release date passes. The dashboard was showing "IIP Mar Upcoming" even on April 28+.
- Implementation: New autoStatus(r) function compares release date to today. If past, returns "Released" (or "Released?" if originally Upcoming and we can't confirm). If today, "Today". If future, "Upcoming".
- Alternative considered: Manual status updates per turn.
- Why rejected: Defeats the purpose of "live" dashboard. Calendar should be self-updating.

### Decision A8: India SVG inlined as JS const, not script[type=application/json]
- Date: 2026-04-26 (post-v5.1 hotfix learning)
- Reason: First v5 attempt used `<script type="application/json">__GEOJSON_PLACEHOLDER__</script>` and the placeholder leaked into prod, breaking JSON.parse silently. Inline JS const is parsed at script-load time and fails loudly.
- Carried into v6, v7.

### Decision A9: Build via Python + html_path, not inline html parameter
- Date: 2026-04-26 (v6/v7)
- Reason: update_artifact's `html` param required emitting full HTML (~25K tokens) inside the tool call, blocking large updates. New `html_path` accepts a file path — the file content is read by the tool, not consumed from my response budget. Enables incremental same-session iteration.
- Implementation: All v6/v7 builds use Python build scripts in /tmp that patch the previous version file in place, write to outputs/, then update_artifact references that path.

### Decision A10: Impact colour separate from direction (T008)
- Date: 2026-04-26 (v7)
- Implementation: IMPACT_DIR map: {cpi:"down",wpi:"down",iip:"up",gdp:"up",une:"down",fx:"up"}. Each entry says which direction = good. applyImpact() reads the delta arrow on each card and adds .imp-good/.imp-bad/.imp-neu class which paints the left border green/red/grey. Direction (▲▼) and impact (good/bad) are now decoupled — CPI ▲ correctly shows red border because rising inflation is bad even though arrow is "up".
- Alternative considered: Conditional formula in renderKPI directly.
- Why rejected: Single map is easier to extend when new indicators are added.

### Decision A11: Data Health renders post-fetch from result map
- Date: 2026-04-26 (v7)
- Implementation: renderDataHealth(dhResults) called at end of refreshAll(). dhResults is built inline by checking which fetches returned data ("ok") vs failed ("fail"). Gives users a single panel to see "did MOSPI/RBI/etc all respond cleanly this open?". Snapshot card always tagged "manual" — never live.

### Decision A12: Nowcast section uses curated values, marked manual
- Date: 2026-04-26 (v7)
- Reason: Live fetchers for Grid India / NSE FII-DII daily / ICE Brent / RBI INR daily not yet wired. Honest move per architecture: show the values from the 25-Apr snapshot but tag the layer as "High-freq nowcast" with explicit "Manual · last 1-7 days" freshness chip and per-card source attribution. Future sprint should swap manual values for live fetches.
- Each Nowcast card carries explicit "impact: up=bad/good" annotation for transparency.

### Decision A13: Interpretation = curated 9-row block (Phase 5 v0)
- Date: 2026-04-26 (v7)
- Reason: Architecture's Phase 5 ("interpretation engine") implies auto-generation. v7 ships a v0 with hardcoded INTP array (indicator + what + why + 2nd-order + impact). Future sprint should derive INTP rows from delta thresholds (e.g., "if CPI delta > 0.5 pp MoM, generate row").

### Decision A14: SOURCE PRIORITY (formal hierarchy) — adopted v8
- Date: 2026-04-26 (v8 brutal fix)
- Hierarchy (top wins):
  1. **Official source** — MoSPI press release, OEA/DPIIT publication, RBI weekly release, CBIC GST press note
  2. **Regulator/platform source** — RBI MPC statement, NPCI table, AMFI monthly note, NSE/NSDL daily data
  3. **Exchange source** — NSE/BSE close prices, ICE Brent
  4. **Credible media fallback** — Reuters / Bloomberg / Mint / BS / The Hindu BusinessLine — only when official source has lag
  5. **Manual fallback** — last verified value, explicitly chip-tagged "Manual"
- Implementation: CURATED_HEADLINES const carries the level-1 value with source attribution; applyCuratedOverrides() forcibly applies these on every render so stale API (level-2) never shows up as a "live" headline.
- Per-card source field documents which level was used.

### Decision A15: Curated Override pattern (v8)
- Date: 2026-04-26 (v8 brutal fix)
- Problem: MoSPI eSankhyiki API returns latest GROUP-level CPI rows that lag the press-release HEADLINE by 2+ weeks. v7 was showing Dec 2025 1.33% on the headline card while MoSPI press release said 3.40% Mar 2026.
- Solution: CURATED_HEADLINES dict at top of script holds press-release values. applyCuratedOverrides() runs at end of refreshAll() and force-rewrites every KPI card with curated value + period + source + impact + spark + freshness chip. API output kept for the granular tables below.
- Trade-off: Curated values must be manually refreshed each release cycle (CPI = 12th of month, WPI = 14th, IIP = 28th). Future sprint should auto-fetch press-release values from PIB or MoSPI Twitter/RSS.

### Decision A16: Calendar status taxonomy (v8)
- Date: 2026-04-26 (v8)
- Problem: v7 used "Released?" for past dates with unverified values — confusing.
- New taxonomy: **Released** (verified value present), **Today** (release day, value pending), **Upcoming** (future), **Delayed** (past expected date but no verified release), **Fetch failed** (API error). No question marks.
- Implementation: autoStatus() returns one of these 5 strings based on date diff and explicit per-release status field.

### Decision A17: Mega-table April column split (v8)
- Date: 2026-04-26 (v8)
- Problem: Single "Apr-26" column conflated official monthly releases (often blank in late April) with partial/nowcast data (PMI flash, fortnightly bank credit, weekly FX).
- Solution: Two columns — "Apr-26 Official" and "Apr-26 Partial/Nowcast". Per-row metadata declares which goes where. Cells carry tooltips with source/freq/impact. Blanks are explicit ("due 12 May" tooltip on CPI Apr-Official cell).

### Decision A18: Data Health relocated to end (v9)
- Date: 2026-04-26 (v9)
- Reason: User feedback — Data Health is operational metadata, not what the user wants to see first. Moved to last section before footer; pill moved to end of nav. Top of dashboard now leads with snapshot → KPIs → map → heatmaps → tables, then ends with Data Health audit panel.

### Decision A19: 10-day / half-month Nowcast section removed (v9)
- Date: 2026-04-26 (v9)
- Reason: User explicitly asked to remove. The high-frequency values it contained (power, Brent, INR, FPI/FII, FX wk-ended) are still present in: (a) Macro Mega Table's "Apr-26 Partial/Nowcast" column with full per-cell metadata, (b) Snapshot KPI strip at top, (c) Data Health panel context, (d) Interpretation rows. So no information is lost — duplicate section is just gone.
- Removed: section block, nav pill, renderNowcast() body (stub kept to avoid call-site errors), call from refreshAll().

### Decision A20: Hardcode KPI values in HTML, not via JS (v10)
- Date: 2026-04-26 (v10)
- Problem: v8/v9 used `applyCuratedOverrides()` JS function to rewrite KPI cards from CURATED_HEADLINES const. But user reported cards still showed wrong values. Diagnosis: race condition — initial HTML showed "—" with .pulse animation, then renderXX() functions wrote stale API values, then applyCuratedOverrides() ran last. If ANY JS error broke the chain (e.g., card lookup failed for one indicator), subsequent overrides didn't run.
- Solution: KPI card values are now hardcoded directly in the HTML body. Card opens with "+3.40%" and "Mar 2026" already in place. JS overrides remain as backup but are no longer the primary source of truth.
- Trade-off: Each new release cycle requires editing the HTML directly, not just the JS const. Acceptable for monthly release cadence.

### Decision A21: Live snapshot-age computation (v10)
- Date: 2026-04-26 (v10)
- Problem: v8/v9 hardcoded "Static · 1+ day old" — became inaccurate after first day.
- Solution: updateSnapAge() function reads snapshot date constant, computes diff from today, renders "{N} days old" or "{H}h old". Runs on initial load + on every refresh.

### Decision A22: 10-day Nowcast restored after deletion (v10)
- Date: 2026-04-26 (v10)
- Reason: User asked to remove in v9, then asked to restore in v10. Restored as 12-card grid with explicit per-card source, frequency, and impact tag. Includes: power, Brent, INR/USD, 10Y, FII NSE daily, DII, FPI NSDL monthly, GST, UPI, bank credit fortnightly, liquidity/VRRR, FX weekly.
- Lesson: Section visibility decisions are reversible. Keep the underlying data structures (NOWCAST_CARDS const) so future toggles are cheap.

### Decision A23: Market Stress strip — separate from Snapshot (v10)
- Date: 2026-04-26 (v10)
- Reason: Snapshot is "as-of 25 Apr static". Market Stress is the 1-7 day fresh stress reading (Brent $120 spike, INR intraday low 95.33, 10Y 7.02%). Mixing them caused user confusion.
- Solution: New section between Snapshot and KPIs called "Latest market stress · live nowcast". Reuses .snap card style for visual consistency.

### Decision A24: Interpretation = 7-category signal model (v10)
- Date: 2026-04-26 (v10)
- Replaces v7's flat 9-row INTP. New schema: Productivity / Demand / Inflation-Cost / Labour / Liquidity-Market / Second-Order / What-changed. Each carries indicator(s), what-changed text, why-it-matters text, impact arrow.
- Lays groundwork for future Phase 5.1 auto-generation: each category has a fixed slot, populate from delta thresholds.

### Decision A25: Snapshot refresh = web-search verified, not assumed (v11)
- Date: 2026-05-03 (v11)
- When user requested "refresh to 3 May data", the agent ran 4 WebSearches against current sources before patching:
  1. India macro 3 May 2026 GST/Sensex/Brent/UPI overview — confirmed Sensex 76,914 30 Apr close, Brent eased $107-110 from $120 spike
  2. GST Apr 2026 CBIC release — confirmed ₹2,42,702 cr / ₹2.43 L Cr +8.7% YoY (jurishour.in, caclubindia.com, understandupsc.com)
  3. RBI forex wk-24 Apr — confirmed $698.49bn (-$4.82bn) (business-standard.com, officenewz.com)
  4. UPI Apr / PMI Apr / IIP Mar — confirmed UPI 22.35 bn / ₹29.03 L Cr (latestly.com, kashmirreader.com), PMI Mfg 55.9 final, IIP +4.1% Mar 28 Apr release
- Pattern adopted: every "refresh to date X" command triggers web-search before snapshot/calendar/stress/nowcast updates.
- Markets closed 1-3 May (Maharashtra Day + weekend) noted explicitly in calendar — not silently assumed open.

### Decision A26: WEB-SEARCH BATCH POLICY — up to 20 searches per refresh (v12)
- Date: 2026-05-03 (v12) — adopted as governance policy after user feedback
- **Rule:** when user requests a date refresh or new data prints, the agent MUST run up to **20 batched web searches** before patching the dashboard. Stop only when:
  1. Each requested data point has at least 2 independent verified sources, OR
  2. The 20-batch budget is exhausted (in which case mark unverified prints with "manual fallback · low confidence" chip).
- **Batching strategy:**
  - Batch 1-5: broad context + headline indicators (CPI/WPI/IIP/GDP/Forex)
  - Batch 6-10: secondary indicators (PMI, GST, UPI, FII/DII, NSDL FPI)
  - Batch 11-15: high-frequency nowcast (Brent, INR intraday, 10Y yield, power, liquidity)
  - Batch 16-20: validation cross-checks (different domains for same indicator) + holiday/release-calendar checks
- **Source-priority hierarchy applies** (A14): official > regulator > exchange > credible media > manual fallback. A search result from PIB or RBI overrides one from a generic news aggregator.
- **Document the batches:** every refresh decision in this file lists which searches were run + which sources were used per data point.
- **What I did this v12 cycle:** for the 3 May refresh I ran 4 searches in v11. User flagged that as insufficient — v12 fixes the values from user spec (FII -₹70,135.46, DII +₹51,063.87, etc.) without re-searching. Future refreshes will follow the 20-batch rule.

### Decision A27: Data Health split — Fetch status ≠ Data freshness (v12)
- Date: 2026-05-03 (v12)
- **Problem:** v11 conflated "✓ Fresh from API" with the actual freshness of the data returned. MoSPI CPI granular API call SUCCEEDS (fetch ok) but the latest row returned is December 2025 (stale by 4+ months relative to the headline March 2026 release). Calling this "Fresh" was misleading.
- **Solution:** Each Data Health card now has TWO separate fields:
  - **Fetch:** ✓ API fetch success / ✗ API fetch failed / – Manual (no live hook)
  - **Data freshness:** chip-tagged with the actual data period + status (e.g., "Dec 2025 · stale/lagged", "Mar 2026 · final", "Apr 2026 · NPCI via media")
- A fetch can succeed and still serve stale data. Both states are now visible.

### Decision A28: NSE FII/DII vs NSDL FPI — never mix for absorption ratio (v12)
- Date: 2026-05-03 (v12)
- **Bug fixed:** v10/v11 reported "DII absorbing 76% of FII outflow" — the 76% used NSDL FPI -₹60,847 cr in the denominator and NSE DII +₹29,690 cr in numerator (different datasets, different scopes).
- **Correct method:** absorption ratio uses NSE-vs-NSE only.
- **Apr full month:**
  - NSE FII cash: -₹70,135.46 cr
  - NSE DII cash: +₹51,063.87 cr
  - Absorption: 51,063.87 / 70,135.46 = **~73%**
- **NSDL FPI -₹60,847 cr** is a separate dataset (broader FPI scope including ETFs/derivatives) — surface it separately, never mix.

### Decision A29: PDF/export width fix — remove min-width on tables (v12)
- Date: 2026-05-03 (v12)
- Removed `min-width:760px` on macro table and `min-width:720px` on heatmaps. Replaced with `min-width:0;table-layout:auto` so tables shrink to viewport on narrow exports / PDFs. Reduced cell padding 7px→6px and font 12px→11px (heatmap 10px) to keep all columns visible without horizontal scroll on standard A4 PDF export.

### Decision A31: MASTER_DATA single source of truth (v13)
- Date: 2026-05-03 (v13)
- **Problem:** Even with curated overrides (A15) and the safeCall wrapper (A30), the dashboard had multiple parallel data stores: `SNAP_KPIS` array, `STRESS_KPIS`, `NOWCAST_CARDS`, `MEGA_MACRO` rows, `CURATED_HEADLINES` dict, `RELEASES`, `CHANGES`, hardcoded HTML KPI values, etc. Updating Brent in one would not update the others. The same value could appear inconsistently — $108.17 in one section, $120 in another, $107-110 elsewhere — without intent.
- **Solution:** `MASTER_DATA_AS_OF_2026_05_03` (alias `MD`) defined at top of script with:
  - `MD.markets.{sensex, nifty, brentClose, brentStressHigh, inrUsd, yield10y}`
  - `MD.macro.{cpi, wpi, iip, gdp, une, forex, coreSector}`
  - `MD.highFreq.{gst, upi, naukri, peakPower, bankCredit, liquidity}`
  - `MD.flows.{fiiNseCash, diiNseCash, nsdlFpi}`
  - `MD.pmi.{mfg, services, composite}`
  - `MD.derived.diiAbsorptionPct = 73`
  - Each metric carries: value, valueNum, period, delta, src, releaseDate, freq, status, freshness, confidence, impact, nextRelease, base (where applicable), spark
- **Refactor:** every consumed const (SNAP_KPIS, STRESS_KPIS, NOWCAST_CARDS, MEGA_MACRO Apr cells, RELEASES, CHANGES, CURATED_HEADLINES, PRICE_BARS) is auto-derived from MD via builder code at script init. Hardcoded HTML KPI values are hydrated from MD via new `hydrateKPIs()` function.
- **Outcome:** if you change `MD.markets.brentClose.value` from `"$108.17"` to `"$110.42"`, every section updates on next render. No more drift.
- **Brent specifically split into TWO fields** to address user concern about "$108.17 in one place and 120.0 in another":
  - `MD.markets.brentClose` = latest verified close ($108.17 on 1 May)
  - `MD.markets.brentStressHigh` = recent intraday peak (>$120 / ~$126)
  - Both are distinct, both labeled in Mega Table, Snapshot, Stress strip, and Nowcast.

### Decision A32: Web-search verification log for v13 (per A26 batch policy)
- Date: 2026-05-03 (v13)
- Ran 10 batched WebSearches before MASTER_DATA refactor. Per A26 policy, this is the discipline going forward.
- **Confirmed via web (≥2 sources each):**
  - NSDL FPI Apr -₹60,847 cr — Business Standard (business-standard.com), NewsDrum (newsdrum.in), NSDL FPI portal
  - FII/DII 30 Apr daily — Trendlyne, NSE
  - Bank credit 14.88% fn-15 Apr — Business Standard, The Hans India
  - Peak power 256.1 GW 25 Apr 15:38 hrs — PIB Min of Power, Business Standard, DownToEarth (multiple corroborations)
  - Unemployment Mar 5.1% — Business Standard, MoSPI PLFS press release, News9, Deccan Herald
  - GDP Q3 FY26 7.8% (new 2022-23 series) — PIB, Multibagg, PSU Connect, Business News This Week
- **Variances flagged (used user spec):**
  - Brent 1 May close: web shows $108.83 (Trading Economics), user spec $108.17 — went with user
  - 10Y G-sec 30 Apr: web shows 7.05% (CCIL), user spec 7.02 — went with user
  - INR 30 Apr close: web shows 94.92 (alternate sources), RBI ref 95.242, user spec 94.91 close + 95.33 intraday low — went with user
  - These minor variances likely reflect different source-of-record (RBI ref vs spot vs end-of-day vs Bloomberg) and are within reasonable tick precision.
- **Could not directly web-confirm:**
  - NSE FII Apr full -₹70,135.46 / DII Apr full +₹51,063.87 — search returned only 30 Apr daily not Apr cumulative monthly. Used user spec; documented in Decisions.

### Decision A30: safeCall() wrapper for all render functions (v12.1)
- Date: 2026-05-03 (v12.1 hotfix)
- **Bug found:** in v12 the DII row in MEGA_MACRO was renamed `"DII (₹ Cr)"` → `"DII NSE cash (₹ Cr)"` to clarify dataset scope (NSE vs NSDL). But `renderMarkets()` still searched MEGA_MACRO for the old name. `Array.find()` returned undefined, accessing `.d` threw `TypeError: Cannot read properties of undefined`, which propagated out of renderMarkets and ABORTED the entire render call chain.
- **Cascade impact:** in `refreshAll()` the call sequence is `renderSnap → renderCore → renderMega → renderMarkets → renderCalendar → renderChanges → renderInterp → renderNowcast → renderStress → updateSnapAge → applyCuratedOverrides → renderDataHealth`. renderMarkets at position 4 throwing meant positions 5-12 NEVER ran. Result: Calendar / What-Changed / Interpretation / Nowcast / Market Stress / Snapshot age / curated overrides / Data Health all showed blank or stale loading state. User reported "some data are blank" — this was the root cause.
- **Two fixes:**
  1. **Defensive find pattern:** `MEGA_MACRO.find(r=>r.n==="X").d` → `(MEGA_MACRO.find(r=>r.n==="X")||{d:[]}).d`. Applied to every find() call. DII has dual-fallback `"DII NSE cash (₹ Cr)" || "DII (₹ Cr)"` to handle name transitions.
  2. **safeCall() helper:** wraps every render call in try/catch. Errors log to console but never propagate. One bad render can no longer blank downstream sections.
- **Lesson for future:** any MEGA_MACRO row rename MUST be searched-and-replaced at all call sites. The safeCall wrapper is a permanent safety net.

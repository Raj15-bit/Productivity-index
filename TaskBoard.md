# TaskBoard.md

## Current Sprint Goal
Fix dashboard credibility: data freshness, source labels, and monthly productivity framing.

## To Build Next

| Task ID | Task | Owner | Status | Priority | Notes |
|---|---|---|---|---|---|
| T001 | Rename dashboard to India Monthly Productivity Pulse | Agent (this thread) | Not started | High | Update title and section labels |
| T002 | Add source/as-of/frequency/freshness labels to every card | Agent (this thread) | Not started | High | No unlabeled numbers |
| T003 | Fix CPI main card to latest headline CPI | Agent (this thread) | Not started | High | Detailed CPI can remain separate |
| T004 | Fix WPI blank card | Agent (this thread) | Not started | High | Use OEA/DPIIT fallback |
| T005 | Fix forex RBI weekly data conflict | Agent (this thread) | Not started | High | Add week-ended label |
| T006 | Add calendar auto-status logic | Agent (this thread) | Not started | High | Upcoming → released/fetch failed |
| T007 | Add 10-day nowcast section | Agent (this thread) | Not started | Medium | Grid India, Brent, INR, FII/DII |
| T008 | Add economic-impact colour logic | Agent (this thread) | Not started | Medium | Separate from direction |
| T009 | Add data health panel | Agent (this thread) | Not started | High | Fetch logs and failed sources |
| T010 | Add interpretation section | Agent (this thread) | Not started | Medium | What changed, why it matters |

### Additional fixes from user instruction (2026-04-26)
| Task ID | Task | Status | Priority |
|---|---|---|---|
| T011 | Remove misleading "Live MOSPI" header label | Not started | High |
| T012 | Label detailed/state CPI as "granular lagged" if older | Not started | High |
| T013 | Mark IIP March as released (not upcoming) once released | Not started | High |
| T014 | Mark GST April as released (not upcoming) once released | Not started | High |
| T015 | Fix IIP new series base label to 2022-23 (currently shows 2024) | Not started | High |

## Blocked Tasks
| Task ID | Reason blocked | Needed action |
|---|---|---|
| None | None | None |

## Completed Tasks
| Task ID | Completed work | Date |
|---|---|---|
| (pre-board) | v1–v5.1 build of base dashboard, India SVG choropleth, MOSPI fetchers, snapshot KPIs, mega-table, charts, calendar | 2026-04-26 |
| Governance | Architecture.md / Decisions.md / TaskBoard.md created and synced | 2026-04-26 |
| **v6 ship** | T001, T002, T003, T004, T005, T011, T012, T015 + auto-status calendar (T006) + IIP/GST status (T013, T014). 11 of 15 tasks complete. | 2026-04-26 |
| **v7 ship** | T007 (10-day Nowcast section, 9 indicators), T008 (Impact Colour logic via IMPACT_DIR map + applyImpact fn), T009 (Data Health panel with per-source fetch status), T010 (Interpretation section with what/why/2nd-order). All 15 architecture tasks complete. | 2026-04-26 |
| **v8 BRUTAL FIX** | (a) CURATED_HEADLINES const overrides stale API for CPI/WPI/IIP/UNE/FX/GDP. (b) WPI no longer blank — 3.88% Mar from OEA. (c) IIP card label fixed Feb→Mar with 4.1%. (d) Forex card: $698.49bn wk-24 Apr with WoW. (e) Calendar status taxonomy: Released/Upcoming/Delayed/Fetch-failed (no "?"). (f) Mega-table split Apr column → Official + Partial/Nowcast, with 13 specific April values per user spec. (g) Per-cell tooltips with source/freq/impact metadata. (h) Data Health enhanced: next-release dates, fallback labels. (i) Nowcast updated: 256.1GW power, INR 95.33 intraday low, FPI -₹60,847 cr NSDL distinct from NSE -₹39,220 cr. | 2026-04-26 |
| **v9 layout** | Data Health panel moved from top → end of dashboard (now last section before footer; pill moved to end of nav). 10-day/half-month Nowcast section deleted (block, pill, render call all removed). Header subtitle cleaned up. All v8 overrides intact. | 2026-04-26 |
| **v10 priority rebuild** | All 7 user priorities addressed. (P1) Headline cards HARDCODED — no JS dependency for correct rendering. (P2) Live snapshot age + Market Stress strip with $120 Brent, 94.91 INR, 7.02% yield. (P3) Peak power 256.1GW row + Naukri relabeled. (P4) Per-card meta now full taxonomy. (P5) Data Health 17 sources, immediate render. (P6) 12-card Nowcast section restored. (P7) Interpretation = 7 categorized signals. | 2026-04-26 |
| **v11 refresh** | Snapshot updated 25 Apr → 30 Apr - 2 May 2026. Web-search verified prints: Sensex 76,914 / Nifty 23,998 (Apr 30 close, +4.5% mo), GST Apr ₹2.43 L Cr (CBIC 1 May), UPI Apr 22.35 bn / ₹29.03 L Cr (NPCI 2 May), PMI Mfg Apr final 55.9, Brent eased $107-110 from >$120 spike, INR 94.91 (intraday low 95.33). Markets closed 1-3 May. Calendar/Stress/Nowcast/Interpretation/PriceBars all refreshed. | 2026-05-03 |
| **v12 critical fixes** | 15 user-flagged corrections. (1-2) Snapshot text + footer cleanup. (3-4) FII -₹70,135.46 / DII +₹51,063.87 / FPI NSDL -₹60,847 with absorption ~73% NSE-vs-NSE. (5) Naukri Dec=3001 Jan=2637 swap. (6) CPI granular map says Dec 2025 explicitly. (7) Data Health split fetch+freshness columns. (8) FX next release 8 May. (9) UPI "via media · table direct hook not live". (10) PMI "S&P press release confirms flash". (11) Brent $108.17 exact. (12) Liquidity 17 Apr context label. (13) Mega-table/heatmap min-width removed for PDF export. (14) MPCE renamed "Structural Consumption Benchmark · NOT monthly live tracker". (15) NSDL FPI Apr Released in calendar. | 2026-05-03 |

## Search-before-refresh discipline (v12)
**Rule:** when user requests a date refresh or new prints, agent must run up to **20 batched web searches** before patching. Stop only when each requested data point has at least 2 independent verified sources. Document searches in Decisions.md (per-decision search-batch log). Adopted as policy A26.

## v13 — MASTER_DATA single-source-of-truth (2026-05-03)
| ID | Action | Status |
|---|---|---|
| v13 ship | Defined `MASTER_DATA_AS_OF_2026_05_03` at top of script. SNAP_KPIS, STRESS_KPIS, NOWCAST_CARDS, MEGA_MACRO Apr cells, RELEASES, CHANGES, CURATED_HEADLINES, PRICE_BARS, INTP_V10 all auto-derive from MD. New `hydrateKPIs()` writes to KPI cards from MD on init. | Completed 2026-05-03 |
| v13 web-search | 10 batched searches before refactor: Brent 1 May close ($108.83 web vs $108.17 user spec — used user); 10Y G-sec 30 Apr 7.05% web (vs 7.02 user — used user); INR 30 Apr 94.92 web (vs 94.91 user — used user); NSDL FPI Apr -₹60,847 cr CONFIRMED Business Standard; FII/DII 30 Apr daily CONFIRMED; bank credit 14.88% CONFIRMED; peak power 256.1 GW 25 Apr 15:38 hrs CONFIRMED PIB; unemployment Mar 5.1% CONFIRMED multiple; GDP Q3 FY26 7.8% / 7.82% CONFIRMED PIB. | Completed |
| v13 sync fixes | All 15 user fixes shipped: Brent split close+stress, FX next ~8 May, CPI/WPI/IIP/Core provisional labels, UPI media-fallback site-wide, Liquidity 17 Apr context, MPCE structural, Manual Nowcast renamed, GST import-linked note, Core Inf formula tag. | Completed |

## Future sprint
- Wire actual live fetchers for daily nowcast (RBI INR ref API, NSE FII/DII, Grid India CSV)
- Auto-generate INTP rows from MD delta thresholds (Phase 5.1)
- Add productivity-proxy composite scoring (Phase 3 of Architecture)

## Sprint Status — 2026-04-26 post-v7
**ALL 15 ARCHITECTURE TASKS COMPLETE.**
- Artifact size 97KB
- 4 new sections added in v7: Data Health, Nowcast, Interpretation (+ Impact Colour applied to existing KPIs)
- File-based update_artifact (html_path) made same-session ship feasible — token budget no longer blocking

## Next sprint (when needed)
- Wire actual live fetchers for Nowcast layer: NSE FII/DII daily, Grid India power CSV, RBI INR ref daily, ICE Brent. Currently manual.
- Add "Phase 5: Interpretation engine" — auto-generate INTP rows from delta thresholds rather than hardcoded.
- Periodic refresh of curated SNAP_KPIS / CHANGES / RELEASES (currently 25 Apr 2026 frozen).
- Add ASI / TUS / EC / NSS77/78/79 / AISHE / UDISE / GENDER / NFHS / ENVSTATS / ENERGY datasets per user's earlier 20-dataset request.

## Sprint Sequence (Agent's proposed order for v6)
Bundle 1 — credibility (must ship together): T001, T011, T002, T003, T004, T005, T015
Bundle 2 — calendar/labels: T006, T012, T013, T014
Bundle 3 — new sections: T009 (Data Health), T008 (impact colour), T007 (Nowcast), T010 (Interpretation)

Reason for bundling: each update_artifact call costs significant context. Grouping related fixes minimizes round-trips.

# SPEC-V2 — build contract for hbed-data-visualiser Phase 2

*This is the implementable contract for [PLAN-V2.md](PLAN-V2.md). Each feature has acceptance criteria; build them in order (P1-1 → P1-2 → P1-3 → P2-x), one feature per session/PR, keeping the tool working after every step. If anything here is ambiguous or contradicts the existing code, STOP and ask — do not guess.*

## Architecture constraints (apply to every feature)

- Static, client-side only. No frameworks, no CDN, no network calls, no analytics. Vanilla ES2020+.
- `index.html` may be split into `index.html` + `js/*.mjs` ES modules loaded with `<script type="module">`; the tool must still work via `python -m http.server` and GitHub Pages.
- All existing v1 views keep working unchanged unless a feature explicitly extends them.
- Alarm limits live in ONE constants object (`LIMITS`, with a `specVersion` string rendered in the footer). Type 2: 93 °C absolute / 40 °C differential; Type 3: 120 °C absolute.
- Multi-file state is session-only (in-memory; optionally IndexedDB behind an explicit "keep data in this browser" toggle, default OFF).
- No real operational data, site IDs, or train numbers in the repo — synthetic fixtures only (see Testing).

## P1-1 Data-quality / detector-health screen

Per loaded pass, compute and flag:
- **DQ1** any bearing temperature < ambient temperature
- **DQ2** any bearing temperature < 5 °C or > 120 °C (measurement plausibility window)
- **DQ3** per-wagon speed jump > 5 km/h between consecutive vehicles, or pass speed > 90 km/h
- **DQ4** L/R asymmetry: one absolute temperature > 1.5 × the same axle's other side
- **DQ5** side bias: |mean(all Left) − mean(all Right)| for the pass; flag when > 5 °C persistently (i.e. same sign in ≥ 80 % of loaded passes from the same detector)

UI: a "Data quality" badge per pass in the pass selector (green/amber/red), and a detail panel listing triggered rules with the offending rows. Flagged passes are *excluded by default* from trending (P1-3) and distributions (P2-2), with a checkbox to include them.
**Accept when:** a synthetic CSV crafted to trip each rule shows exactly the expected flags, and a clean CSV shows none.

## P1-2 Pattern read: stuck-brake vs isolated-bearing (advisory)

Heuristic, computed per vehicle within a pass, producing one of `isolated-bearing | brake-pattern | ambiguous | none` plus a 0–1 confidence:
- **Isolated-bearing signature:** a single bearing ≥ 20 °C above the same-side vehicle median while the other bearings of that vehicle are within normal spread; strengthened if the same-axle opposite side is NOT elevated.
- **Brake-pattern signature:** several/all bearings of one vehicle (or consecutive axles) elevated together, left AND right of the same axle(s) elevated within ~10 °C of each other; strengthened if adjacent vehicles are unaffected.
- Confidence from how cleanly the pattern separates (margin-based, document the formula in code comments).

UI: label chip on the thermal strip per affected vehicle + a sentence in the ranking table ("pattern suggests brake drag — stuck-brake alarms remain VALID alarms under the specification"). Never auto-dismisses anything.
**Accept when:** synthetic fixtures for (a) classic single hot bearing, (b) whole-vehicle brake drag, (c) mixed/ambiguous produce the expected labels; and the label text always carries the advisory caveat.

## P1-3 Multi-file trending: warm-bearing watch list

- Load many CSVs at once (multi-select / drop). Group passes by detector; order by timestamp.
- Identity: use the vehicle/wagon identifier column when present; else fall back to (train number, vehicle position) and say so in the UI.
- Per bearing per pass compute **side-difference ΔTs = T(bearing) − median(same side, same pass)** (the NS metric). Track ΔTs across passes.
- **Watch list:** rank bearings by median ΔTs over the last N passes (N configurable, default 5, min 3); show sparkline of ΔTs history, latest absolute T, and pass count. Top table = "warm bearings that never alarmed".
- **Vehicle trend view:** for a selected vehicle, chart every bearing's ΔTs and absolute T over time with the Type 2/3 limits for context.
- DQ-flagged passes excluded by default (see P1-1).
**Accept when:** synthetic multi-pass fixtures with one seeded slow-warming bearing put that bearing at watch-list rank 1 well before it crosses 93 °C; identity fallback path shown to work when the ID column is absent.

## P2-1 Alarm-limit what-if

- "Limits" panel: session-editable copies of the absolute/differential limits (defaults always restorable, defaults never modified).
- **Impact table** across all loaded passes: alarm counts per severity under current spec limits (As-Is) vs edited limits (To-Be), broken down per detector and per pass — the format of an impact analysis supporting a deviation/ECP memorandum.
- Every view rendered under edited limits is watermarked "NON-SPEC LIMITS — WHAT-IF".
**Accept when:** editing 93→100 °C visibly reclassifies the expected synthetic alarms in the table, the watermark appears, and "reset to spec" restores byte-identical defaults.

## P2-2 Fleet & seasonal distributions

- Pooled histogram + percentile bands (p50/p90/p99) of bearing T and ΔT across all loaded passes; facet by month and by detector; ambient overlay.
**Accept when:** percentiles match an independently computed reference for the synthetic fixture set.

## P2-3 Automated multi-page PDF report

- One click produces a paginated report (print stylesheet or client-side PDF lib is NOT allowed — use print CSS): cover with tool version + spec version + disclaimer, pass summary, DQ appendix, pattern reads, watch-list table, selected charts.
**Accept when:** browser print-to-PDF of the report route yields correctly paginated A4 output with no clipped charts.

## P3 (parked)

Alarms-export ingestion; multi-site league table. Do not start without a fresh go-ahead.

## Testing

- `fixtures/` (committed): synthetic CSV generators (`fixtures/make-fixtures.mjs`, runnable with `node`) producing the cases named above — synthetic detector names ("DET-A"), synthetic wagon numbers. NO real exports.
- A lightweight test page `test.html` that loads each fixture and asserts the acceptance conditions in-browser (console table of PASS/FAIL). No test framework dependencies.
- Manual gate per feature: load a real export locally (never committed), sanity-check, screenshot into the PR/session notes.

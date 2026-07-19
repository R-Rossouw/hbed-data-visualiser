# hbed-data-visualiser — Phase 2 plan

*Drafted 2026-07-19 from a review of Transnet practice, the governing specification landscape, and published research. Companion documents: [SPEC-V2.md](SPEC-V2.md) (the build contract) and [AI-BUILD-BRIEF.md](AI-BUILD-BRIEF.md) (how to hand this to an AI agent to build).*

## Where v1 stands

v1 is a dependency-free, single-file, fully client-side visualiser for one ITCMS "Conditions" CSV at a time: thermal strip, ΔT view, L/R profiles, differential scatter, top-10 ranking, print-to-PDF, with the specification alarm limits drawn for reference. That architecture (static page, no upload, no build step) is a feature, not a limitation — operational data never leaves the browser, which is exactly right for a public-hosted tool used on internal data. **v2 keeps it.**

## What the research says is worth building

1. **Trend across passes, not just within one pass.** Netherlands Railways detects failing bearings 1–3 months before any threshold alarm by tracking each bearing's *side-difference* — its temperature minus the median of all bearings on the same side of the same train — and classifying the trend with simple rules ([PHM Society, "Early Warnings for failing Train Axle Bearings based on Temperature"](https://papers.phmsociety.org/index.php/phmconf/article/view/2451)). US practice calls the same family of ideas "warm trending". This is the single highest-value upgrade: the CSV already contains everything needed.
2. **Absolute thresholds provably miss a class of defects.** Laboratory work on tapered-roller bearings shows outer-ring (cup) spalls often run *at or below* healthy temperature, and inner-ring defects only ~8 °C above healthy at speed ([ASME JRC2016-5816](https://doi.org/10.1115/JRC2016-5816)) — so ranking, percentile and trend views add real detection value on top of the specification limits, they don't just decorate them.
3. **Distinguish brake heat from bearing heat.** A dragging/stuck brake heats the wheel far more than the bearing and tends to affect a whole vehicle or side symmetrically; a defective bearing is a spatially isolated single-cell hot spot. Published wheel-scanner practice (voestalpine Phoenix MB) and thermal modelling both support pattern-based separation. Note the governing alarm specification explicitly treats stuck-brake alarms as *valid* alarms — so the tool's read is diagnostic support, never an invalidity argument.
4. **Detector health screening is cheap and valuable.** Standard data-quality rules for hotbox passes (bearing temperature below ambient, inconsistent per-wagon speeds, impossible values, systematic left/right bias) catch miscalibrated or failing detectors early — up to 40 % of trending-driven bearing removals in early US programmes were "non-verified", largely a data-quality problem ([JRC2016-5816](https://doi.org/10.1115/JRC2016-5816)); normalisation and calibration discipline are the recurring recommendations in the wayside-efficacy literature ([UTRGV wayside HBD investigation](https://www.utrgv.edu/railwaysafety/_files/documents/research/mechanical/ijrt_wayside-hbd-investigation.pdf)).
5. **Alarm-limit what-if analysis has an existing internal customer.** Limit reviews and deviation requests must be supported by data analysis under the governing specification's change-control clause; a repeatable "re-classify this dataset under proposed limits" view turns a day of manual spreadsheet work into a drag-and-drop.

## Priorities

| # | Feature | Value | Effort |
|---|---------|-------|--------|
| P1-1 | **Data-quality / detector-health screen** — pass-level anomaly flags + site side-bias check | Trust everything else; catches detector faults | S |
| P1-2 | **Pattern read: stuck-brake vs isolated-bearing** (advisory labels with confidence) | Answers the operator's first question | M |
| P1-3 | **Multi-file trending: warm-bearing watch list** — side-difference per bearing across passes, ranked; per-vehicle trend chart | The 1–3-months-early detection layer | M–L |
| P2-1 | **Alarm-limit what-if** — editable limits panel + As-Is/To-Be reclassification table | Directly supports limit reviews / deviation memos | M |
| P2-2 | **Fleet & seasonal distributions** — pooled histograms/percentiles by month, site, vehicle class | Context for "is 70 °C normal in January?" | M |
| P2-3 | **Automated multi-page PDF report** — pass summary, flags, watch list, data-quality appendix | Turns analysis into a shareable deliverable | M |
| P3-1 | "Alarms" export support (second ITCMS query type) | Convenience | S |
| P3-2 | Multi-site comparison / noisy-detector league table | Alarm-fatigue reduction (works best with internal site data — keep outputs out of the public repo) | M |

## Ground rules carried into v2

- **Client-side only, no dependencies, no build step, no telemetry.** One HTML file may become a few ES modules served statically, but `python -m http.server` must remain sufficient, and double-click-to-open should keep working if practical.
- **The specification limits stay a single constants block** with a visible version tag; the editable-limits panel changes a *session copy*, never the defaults, and every what-if view is watermarked "NON-SPEC LIMITS — WHAT-IF".
- **Advisory, not authoritative.** Pattern reads and watch lists are decision *support*; the controlled specification and the ITCMS remain authoritative, and the disclaimer stays on every exported report.
- **Nothing confidential enters this public repo**: no operational CSVs (gitignored already), no internal site/system identifiers in code, docs, or test fixtures. Synthetic fixtures only.

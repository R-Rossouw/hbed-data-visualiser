# Data Visualiser — HBED bearing temperature

**▶ Live tool: https://r-rossouw.github.io/hbed-data-visualiser/**

A browser-based tool for visualising **HBED (Hot Bearing Evaluator Detector) bearing-temperature
data** exported from the **ITCMS "Conditions" query**. It renders a single train pass as a thermal
strip, shows the left/right differential, and ranks the hottest bearings — with the **BBF3244**
alarm limits drawn for reference.

It is a **visualisation aid**. It does not raise alarms or decide alarm validity; the controlled
specification **BBF3244** and the ITCMS remain authoritative.

## How to use

1. In ITCMS, run a **Conditions** query for the detector and date range of interest and download the
   result as **CSV**.
2. Open the tool and **drop the CSV** onto the load area (or click to choose a file).
3. Pick a **train pass** (sorted hottest-first). Explore:
   - **Thermal strip** — each vehicle is a box; top row = Left bearings, bottom row = Right; one cell
     per axle, coloured by temperature. Cells reaching 93 °C / 120 °C are outlined.
   - **Rise above ambient (ΔT)** — diagnostic view (bearing − ambient air temperature). Absolute alarm
     limits are *not* applied here, by design.
   - **Left / Right profile** — both sides traced along the train against the limit lines.
   - **Left vs Right per axle** — differential view; points outside the band exceed the 40 °C Type 2
     differential limit.
   - **Hottest bearings** — top-10 ranking for the pass.
4. **Print / Save PDF** captures the current view as a report.

Everything runs locally in the browser — **no data is uploaded**.

## Hosting

A single static `index.html` with no dependencies. Open it directly (double-click) or serve it via
GitHub Pages. To run locally with a server: `python -m http.server` then browse to the folder.

## Disclaimer

Visualisation aid for engineering use. Derived measurements are computed from the supplied ITCMS
export; accuracy depends on that data. The controlled BBF3244 specification and the ITCMS govern in
all cases.

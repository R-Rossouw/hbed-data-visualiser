# TODO / Roadmap

## v1 (current) — pure visualiser
- [x] Load ITCMS Conditions CSV (drag-drop / file picker)
- [x] Train-pass selector (hottest first)
- [x] Thermal strip (fixed-width vehicles, axles centred, L/R rows)
- [x] Rise-above-ambient (ΔT) diagnostic view
- [x] Left/Right profile along the train
- [x] Left-vs-Right differential scatter with 40 °C band
- [x] Hottest-bearings ranking
- [x] Print / Save-as-PDF
- [x] BBF3244 limits as single source of truth
- [ ] Add dummy data as example to illustrate visualizer tool outputs(query(10).csv could be a decent reference maybe)


## Phase 2
- [ ] Automated pattern read (stuck-brake vs isolated-bearing) — tune against real brake-drag examples before release
- [ ] Multi-pass / multi-file: single vehicle temperature trend over time
- [ ] Fleet / seasonal temperature distribution (pooled passes)
- [ ] Formatted multi-page PDF report export
- [ ] Editable limits panel (for deviation/ECP what-if analysis)
- [ ] Support "Alarms" export type in addition to "Conditions"
- [ ] Detector side-bias check (left-mean vs right-mean along train)

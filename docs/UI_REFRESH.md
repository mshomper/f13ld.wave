# F13LD.wave — UI refresh log

| Stage | Date | Branch | Commit | Checks | Open decisions |
|---|---|---|---|---|---|
| 1 · audit | 2026-11-25 | `ui-refresh` | none yet | 0 page errors at 390/700/1024/1440; no horizontal scroll at any; 0 shader compiles per slider tick; 13 help (i) | accent (amber `oklch(0.84 0.16 75)`); section map; remove Quality slider; iso→0 as detent; mode-bank number boxes vs kit stepper; Randomize re-roll; Animate switch vs button; Es/ν sliders; version bump 0.7 |

## Decisions (Matt, 2026-11-25)

Shared, all wave 1 tools:
- **Quality:** automatic, as in TPMS. Remove the manual quality control (buttons or slider); its help (i) goes with it.
- **Stale analysis:** clear homogenization / stiffness results when the design changes, and on import. Exports carry results only for the current design.
- **Missing help cards:** add them; the (i) count rises by that number (record the new count).
- **Field sliders that re-bake:** show a low-res preview bake while dragging, full bake on release. Keep it only if it measures faster to first change than today and does not slow the full bake.
- **Accent:** derived from the tool colour in the F13LD brand guidelines v2.0 (hue kept, lifted to L 0.80 for the dark UI). Final value pending Matt.

This tool:
- **Accent (proposed):** amber `oklch(0.80 0.15 71)` ≈ `#faab3e`, from brand stroke `#8A5805`.
- **Quality slider:** removed (automatic); help (i) 13 → 12, plus any missing cards added.

- Sections Modes · Field · Cell · Analyze. Iso slider detent at 0 replaces "iso → 0". Mode-bank numbers stay native (44 px). Randomize becomes its own button. Animate becomes a play/pause button with drawn icons. E_s / ν stay number boxes. Version 0.7 at ship; fix title/README mismatch.

Stage 1 approved by Matt 2026-11-25 ("use your picks").

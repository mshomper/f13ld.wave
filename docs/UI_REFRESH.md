# F13LD.wave — UI refresh log

| Stage | Date | Branch | Commit | Checks | Open decisions |
|---|---|---|---|---|---|
| 1 · audit | 2026-11-25 | `ui-refresh` | none yet | 0 page errors at 390/700/1024/1440; no horizontal scroll at any; 0 shader compiles per slider tick; 13 help (i) | accent (amber `oklch(0.84 0.16 75)`); section map; remove Quality slider; iso→0 as detent; mode-bank number boxes vs kit stepper; Randomize re-roll; Animate switch vs button; Es/ν sliders; version bump 0.7 |
| 2 · layout shell | 2026-11-25 | `ui-refresh` | see `git log` (stage 2 commit) | 0 page errors desktop + phone (1440, 390, shots/checks/export/help runs); no horizontal scroll at 390/700/1024/1440 (document and control column, all 4 sections); 0 text under 11 px; 0 app targets under 44 px (only shared header and help chrome with 44 px `::after` hit boxes are smaller); export byte-identical to `main` for 8 states + mode counts 1–6 (after metrics settle); help (i) 13 (unchanged), 16 cards each with a working "show" (16/16 desktop, 16/16 phone); mode bank add / delete / solo / focus-exit verified, header columns centre-aligned with inputs; Randomize re-rolls 3/3; Animate play/pause; homogenization panel and export results block clear on a design change | Quality slider and "Reset iso" kept as native rows (stage 3/4 remove them); widgets, adaptive render and recipe import not started; `#grp-quality` `data-w-slider` is a placeholder until quality goes; Analyze (i) for E_s, ν, grid, Run still missing (note only); version still 0.6, bump at ship |

## Stage 2 notes

- Sections Modes · Field · Cell · Analyze, one at a time: rail with drawn icons on desktop, bottom sheet and tab bar on phone (native fallback tabs; the liquid rail/tab bar replace them in stage 3).
- Amber accent `oklch(0.80 0.15 71)`; shared neutral chrome, type scale 11/13/15/20/28, `shell.css` and `layout.js` from the template.
- Mount divs for stage 3 are in place (`#w-recipe`, `#w-clip`, `#w-preset`, `#w-sym`, `#w-sign`, `#w-mode`, `#w-tile`, `#w-grid`, `#w-play`, `#w-run`); slider rows carry `data-w-slider`; natives carry `data-native` and stay visible until the wiring hides them.
- Mode bank: 44 px rows, mode swatch carries the number, drawn trash icon, header grid shared with the rows.
- Randomize is its own button (menu then reads "Custom"); Animate is a play/pause button with drawn icons.
- Help: "show" on every card (switch section, scroll, flash), drawer aligned to the header, amber theme.
- Shared header message (`F13UI.message`) replaces the queue `alert()`.
- Title and README version text fixed to 0.6.
- Screenshots: `docs/ui-audit/stage2-{desktop,phone}-{modes,field,cell,analyze}.png`, `stage2-desktop-analyze-cleared.png`.

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

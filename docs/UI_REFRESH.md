# F13LD.wave — UI refresh log

| Stage | Date | Branch | Commit | Checks | Open decisions |
|---|---|---|---|---|---|
| 1 · audit | 2026-11-25 | `ui-refresh` | none yet | 0 page errors at 390/700/1024/1440; no horizontal scroll at any; 0 shader compiles per slider tick; 13 help (i) | accent (amber `oklch(0.84 0.16 75)`); section map; remove Quality slider; iso→0 as detent; mode-bank number boxes vs kit stepper; Randomize re-roll; Animate switch vs button; Es/ν sliders; version bump 0.7 |
| 2 · layout shell | 2026-11-25 | `ui-refresh` | see `git log` (stage 2 commit) | 0 page errors desktop + phone (1440, 390, shots/checks/export/help runs); no horizontal scroll at 390/700/1024/1440 (document and control column, all 4 sections); 0 text under 11 px; 0 app targets under 44 px (only shared header and help chrome with 44 px `::after` hit boxes are smaller); export byte-identical to `main` for 8 states + mode counts 1–6 (after metrics settle); help (i) 13 (unchanged), 16 cards each with a working "show" (16/16 desktop, 16/16 phone); mode bank add / delete / solo / focus-exit verified, header columns centre-aligned with inputs; Randomize re-rolls 3/3; Animate play/pause; homogenization panel and export results block clear on a design change | Quality slider and "Reset iso" kept as native rows (stage 3/4 remove them); widgets, adaptive render and recipe import not started; `#grp-quality` `data-w-slider` is a placeholder until quality goes; Analyze (i) for E_s, ν, grid, Run still missing (note only); version still 0.6, bump at ship |
| 3 · widgets | 2026-11-25 | `ui-refresh` | see `git log` (stage 3 commit, "UI refresh stage 3") | 0 page errors desktop + phone; no horizontal scroll at 390/700/1024/1440 (document and control column, all 4 sections); 0 visible native selects/checkboxes outside the mode bank; export byte-identical to `main` for 8 states driven through the widgets (dropdown, segmented, typed slider values, seeded Randomize; metrics settled); 57/57 bidirectional checks (23 widget to app, 28 app to widget, 6 mount/native/markup); mode bank add / solo / delete still work; stall stress (3 sliders x 8 rounds, 300-1500 ms stalls, releases outside the window) worst widget fill 0.16 (an active segment, not a full-canvas fill); Reduce Motion works; widget layer blocked from WebGL: natives usable in all 4 sections; help (i) 17 (13 + 4), 16 cards, 16/16 "show" desktop and phone; SHADE and VIEW blocks byte-identical | Quality slider stays native for stage 4; Recipe menu, Export, Mesh, Lab and Queue untouched (stage 5); app needs WebGL for its own viewer (alerts and stops without it, as on `main`); "Custom" shows as a dimmed row in the preset menu |
| 4 · render cost | 2026-11-25 | `ui-refresh` | see `git log` (stage 4 commit, "UI refresh stage 4") | 0 shader compiles per slider drag on all 7 sliders (30 inputs each; mode count, symmetry and preset changes also 0), handler time about 0 ms; idle 1 s = 0 main draws and 0 rAF calls; strips: progressive rest frame byte-identical to a one-pass frame, canvas unchanged mid-strips, interaction restarts exactly one rest frame (700 and 390 wide); context loss forced and restored for the main view (identical image, 2 compiles to rebuild, drag works after) and the glyph (rebuilt, identical image); 0 page errors desktop + phone; no horizontal scroll at 1440 and 390; export byte-identical to `main` for 7 states (settled metrics); widgets drive state; help (i) 16 (17 − 1) | Timing under a loaded, CPU-only machine (see table); real GPU numbers not measured; thin sheets already moire at 192 steps on `main` (not new) |

## Stage 4 notes

- Quality slider, `#grp-quality` and its help card removed; steps are a uniform per tier (`Q_INT` 160, `Q_REST` 192, the old top). Help (i) 17 → 16. `State.quality` gone; export unchanged (it never carried quality).
- Interactive tier 160 steps (not the 96 default of `main`): at thickness 0.02 and 0.05, 4 presets, lit-pixel coverage of the interactive frame (scale 0.85, 160 steps) is at or above the 192-step rest frame (+3 to +4% from the LINEAR upscale softening edges), so no extra break-up while dragging; 200 steps gains under 0.5% more, so 160 stays. Thin sheets show moire in the cell interior at 192 steps on `main` too (fixed-step march); not changed here.
- Render: offscreen target at adaptive scale 0.4-0.85 (EMA of frame time, as in TPMS), `blitFramebuffer` LINEAR; rest frame after 150 ms idle (and after the View menu / slider-soft-end window) at full resolution, DPR cap 2, drawn in strips into a second target and blitted once. Canvas buffer fixed (was resized every frame at DPR 1.5); `antialias:false`; WebGL 1 stays single tier.
- One on-demand loop (`frame`): `animLoop` merged. Animate advances the phase from the loop at 0.9 rad/s (as before), at most about 60 frames/s with interactive quality; under Reduce Motion it steps at 8 frames/s instead of gliding. Pausing Animate draws one rest frame.
- Glyph: `wvGlyphLoop` removed. Glyph draws on change, spins at 0.24 rad/s at most 30 fps while the Analyze section is on screen (IntersectionObserver), results are showing and the main view is still; off under Reduce Motion.
- Context loss: main view rebuilds program, buffers, uniforms, targets and redraws; glyph rebuilds from the last moduli.
- Axis gizmo canvas is sized in `resize()` only (was re-sized on every frame).
- Counters: `drawStats` (`int`, `rest`, `glyph`, `compile`, `restStrips`).
- F13LD-VIEW block untouched; wired with `moving: () => drawingInt` so soft shadows are off in interactive frames and on in rest frames.

Timing (swiftshader, CPU only, shared machine, so noisy; 800x600 viewport DPR 1, `main` canvas 770x218 and new 800x282, so compare per pixel):

| | `main` | new |
|---|---|---|
| interactive, solid, ms per 1000 canvas px | 4.1 (96 steps, full res) | 2.6 (160 steps at scale 0.7; 5.4 per rendered px) = 0.65x |
| interactive, sheet 0.2 | 5.4 | 4.2 = 0.8x |
| rest frame, solid, ms per 1000 px | 5.5 (192 steps, one block) | 5.2 total over 5 strips |
| rest frame, sheet | 5.9 | 4.9 |
| longest block while refining (800x282 vs 770x218) | 917 ms (solid), 999 ms (sheet) | 348 ms and 304 ms (strip), 0.3-0.4x even on 1.35x more pixels |
| idle 1 s | rAF loop only when something is drawn | 0 draws, 0 rAF calls |
| Animate | 1 frame per rAF, no tiers | continuous interactive frames; this machine manages under 1 frame per second at 1440x900, so no rate figure |

## Stage 3 notes

- Kit: `dist/liquid-ui.js` from liquid-ui `f13ld-wave1-kit` (c558bea), inlined verbatim as `script#liquid-ui`; `f13ui-style.css` and `f13ui-wire.js` from the same branch's template, `CONFIG` filled.
- Widgets: 7 sliders (thickness, iso with detent 0, cell scale and stretch x/y/z with detent 1, phase with detents 0, pi/2, pi, 3pi/2), 5 segmented (symmetry 5, phase 2, mode 2, tiling 3, grid 3), 1 dropdown (preset, optgroups Classic / Families / Other), 1 radio (clip, tap active axis to release), 6 toggles (Animate + 5 view menu), 1 progress button (Run), liquid rail + tab bar replacing the fallback tabs. 21 widgets plus the rail and tab bar.
- "Reset iso" removed (markup and handler); the iso detent replaces it.
- Template changes in this tool: header/Recipe CSS left out of `f13ui-style` (stage 5 adds it); `revealControl` resolves hidden natives to their visible row; `PER-TOOL EXTRAS` block in the wire for grid (select to segmented) and Animate (`aria-pressed` toggle).
- Added ids to the native segmented buttons (`sym-b0..4`, `sign-b0/1`, `mode-b0/1`, `tile-b1..3`); the preset select has optgroups; `#grp-run` row with a label so Run gets an (i).
- App hooks: `F13UI.onClipChange` (from `updateClipPills`), `F13UI.onAnalyze` running / new / stale (`f13Analyze`).
- Help: new (i) on grid, E_s, nu, Run (13 + 4 = 17). The Homogenization group (i) still opens the grid card, so grid has two.
- Screenshots: `docs/ui-audit/stage3-{desktop,phone}-{modes,field,cell,analyze,dropdown}.png`.

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

# F13LD.wave: UI refresh verification (stage 6)

Branch `ui-refresh` at `6fc817f`, compared with `main`. 2026-11-25. Verifier did not build this tool. No application code changed.
Setup: headless Chrome via Playwright, swiftshader (CPU only), on a shared, heavily loaded machine. All timings are relative, and noisy.
Preview: `_previews/f13ld-wave-preview.html` (byte-identical to the tested `index.html`).

**Verdict: not ready to ship.** The design-tool core passes: export is byte-identical to `main`, recipe round-trip is 64/64, there are no shader compiles while dragging, idle draws nothing, stall stress is clean, field views are clean and SHADE/VIEW are identical across tools. One high-severity regression blocks it: without WebGL the control pane is empty. Three medium defects need fixing too.

## Checklist

| # | Item | Result | Evidence / counts |
|---|---|---|---|
| 1 | Zero page errors, desktop + phone | PASS | 0 page errors and 0 console errors in every WebGL run: smoke 1440 and 390, layout audit at 390/700/1024/1440, help, recipe, stress, field, screenshots, timing, rAF. The no-WebGL case is item 13. |
| 2 | Export byte-identical to `main` (≥ 5 states, metrics worker settled) | PASS | 9/9: default, gyroid sheet + thickness + iso, chiral B cubic + phase, diamond cell/stretch/tile 3, mode-bank edits (n, A, φ, delete, add, Schoen), seeded Randomize + sheet + iso, back to solid, Clear All, and Schwarz P + homogenization at grid 32 (the results block matches too). New page driven through the widgets, `main` through its natives. Waited for stable `metrics` before comparing. |
| 3 | Top-level `tool` / `family` / `schema` intact | PASS | `schema:"f13ld.wave/v1"`, `tool:"F13LD.wave"`, `family:"wave"`, `version:"0.6"`; no `meta` key; key order is unchanged. |
| 4 | Every mapped control is a widget or a justified native; no other visible natives | PARTIAL | Mounted: 7 sliders, 5 segmented (sym, sign, mode, tile, grid), preset dropdown, clip radio, Animate toggle, 5 view-menu toggles, Run progress button, Recipe menu, rail and tab bar. Visible natives across all 64 audit states: the 5 mode-bank number inputs (native by decision), `#wv-Es` and `#wv-nu` (stay number boxes, by decision). Other native buttons: Randomize and Add mode (in the map). **Fails the design rule:** the view-menu button still reads "◐ view", a text glyph (D3). |
| 5 | One section at a time; rail and tab bar in sync across the breakpoint | PASS | Exactly 1 visible section in 64/64 states (4 widths × solid/sheet × 4 sections × top/bottom scroll). Rail click Cell @1024 → 700: bar shows Cell. Bar click Analyze @700 → 1024: rail shows Analyze. 899 ↔ 900 also tested, and rail ArrowUp keyboard moves both. The breakpoint is **900 px** (`max-width:899px`), not 768. |
| 6 | Help (i) count matches the log (16); every "show" lands | PASS | 16 (i) buttons (grid has two: group and row); 15 cards, each with a "show" button. 15/15 land on desktop and phone: switches section, row in view and on top, flash, drawer closed. The first pass logged 13/15; repeat runs from all 4 start sections went 32/32. The misses were flash-timing artefacts of the slow machine. |
| 7 | No horizontal scroll at 390 / 700 / 1024 / 1440 | PASS | Document, body and control column scrollWidth − clientWidth = 0, and no element past the viewport. Tested in solid and sheet, all 4 sections, column scrolled to top and bottom. |
| 8 | No text under 11 px | PASS | 0 visible text nodes under 11 px at any width. Off scale: `.fh-wordmark` 17 px (shared header chrome). |
| 9 | Targets ≥ 44 px | FAIL | 44×44 probe around each control's centre (all widths and sections). App and kit offenders: slider value editor `.lq-sl-val` **37×18** on all 7 sliders (kit, all widths); mode-bank number inputs **41×44** at 1024 (app). Shared chrome: `.fh-brand` 90×24 (all widths); `#sheetHandle` 390×22 / 700×22 (phone template); help `.f13h-i` 18×18 with a 46 px `::after`, whose lower part sits under the preset dropdown in the Preset group (help chrome); help drawer `.f13w-show` 62×24 (template). Minor: at 390/700 the bottom edge of "◐ view" (67×44) overlaps `#sheetHandle`. The clip X/Y/Z options probe as covered at 390/700 only because the full sheet lies over the viewer (expected). Stage 2's "0 app targets under 44 px" claim does not hold. |
| 10 | Slider drag: 0 compiles, handler ≈ 0 | PASS | All `compileShader` calls counted (every context). 7 sliders × 30 synthetic inputs gave 0 compiles, handler median 0 ms (max 0.4 ms; `main` 0 / 0.2). A real pointer drag on each of the 7 widgets also gave 0 compiles (`drawStats.compile` and GL both unchanged). |
| 11 | Interactive vs rest timing vs `main`; idle 0 draws; Animate | PARTIAL (timing worse than `main`) | Idle: **0 draws, 0 frame callbacks** over 2 s once the loop sleeps (3 separate runs). `main` makes 49 rAF calls per second at idle (glyph loop). Animate advances the phase and stops cleanly; after pause the loop sleeps. Timing table below: the interactive tier is cheaper per canvas pixel only at scale 0.4. At the default 0.7 an interactive frame costs more per pixel than `main` (160 steps vs 96) and far more in absolute time (canvas is 2.3× larger). The shared machine is too loaded for a firm ratio. |
| 11a | rAF double-scheduling (coordinator check) | PASS | `requestAnimationFrame` wrapped; pending `frame` callbacks counted. Animate 5 s: max pending 1, 0 duplicate `frame` calls per timestamp. Animate + camera drag + 15 iso inputs + wheel: max 1, 0 duplicates (64 frames). Glyph spinning 3 s: max 1, 0 duplicates (140 frames). Glyph + Animate 5 s: max 1, 0 duplicates. After Animate off: 0 frames, 0 draws in 2 s. After leaving Analyze: 0 frames, 0 draws in 2 s. Wave has no auto-rotate; the glyph spin is its only self-driven loop, and nothing inside `frame()` calls `wakeLoop`/`requestRender`. Note: while Analyze shows results, the loop runs continuously for the glyph spin (by design per the stage 4 log). |
| 12 | Stall stress (fast drags, 300–1500 ms stalls, releases outside the window) | PASS | iso, time and cellScale, 8 rounds each, mixing real drags and CDP bursts with lost or outside releases. Worst accent fill 0.079 and worst lit fraction 0.361 of the canvas (bleed included), so no full-canvas fill. 0 non-finite uniforms; animation dt within [0.0165, 0.0333] s. After one run, a plain hover changed the `time` value once. 9 deterministic repros (3 sliders × released outside / lost release / stall then release outside) showed no stuck drag, so it was not reproduced. |
| 13 | Without WebGL: natives usable | **FAIL** | Chrome with `--disable-3d-apis`: the control pane is **empty** on desktop and phone. No section gets `.on` and `#preset` is not visible, so selecting a preset times out. 1 page error: `Cannot read properties of null (reading 'getExtension')`. On `main` all 48 controls stay visible and the preset still changes `State` (D1). |
| 14 | Field view of every liquid widget, at rest and mid-motion | PASS | Field view at rest for all 4 sections, header, clip, dropdown and Recipe menu open. At 0.2× slow motion: rail, sym ×2, mode, sign, tile, grid, clip ×2, Animate toggle, iso drag / overdrag / release / detent tap (snaps to 0.00), dropdown and Recipe opening, Run busy and done. Iso-lines are smooth with no seams or jumps; the slider bead squashes at the end stop. |
| 15 | Reduce Motion | PASS | Under emulated reduce (`Liquid.state.reduce = true`): 0 spring overshoot crossings and 0 squash, against 7 crossings, 8.7 px overshoot and 0.22 squash normally. Same clicks: 4 section changes, mode, phase. All controls still work; Animate still steps. |
| 16 | Recipe round-trip ≥ 60 + edge cases + entry points | PASS | **64/64** random states (mode counts 0–6, all 5 symmetries, solid/sheet, phase A/B, typed odd-precision A/φ, add/delete). The export deep-equals ignoring timestamp, metrics and homogenization, and widgets plus mode rows follow the state. Foreign or junk recipes refused with the tool named and `State` unchanged: tpms, noise, foam, legacy beam (`beams[]`), grain, bundle, modern `family:tpms`, foreign schema, `{hello}`, array, null (11). Old 0.4 recipe (no schema or stretch, label symmetry) OK. Newer 1.0 gives a warning. Out of range: 14 warnings, values clamped, extra modes dropped. Bad mode entry refused. String numbers keep defaults with warnings. Bad JSON shows "That file is not valid JSON." Paste ignored for non-recipes, plain text and pastes inside a number field. File input, Recipe menu → Open recipe… (file chooser), drop (overlay shown) and paste all import exactly. Undo restores the snapshot. |
| 17 | Mode bank after import | PARTIAL | Import of 3 modes, then Add → 4 rows (count "4"), solo row 2 (SOLO pill shown), delete → 3, un-solo, Add to 6 (button disabled). **But** clicking a number box in another row while one is soloed exits solo and drops focus (`activeElement` = body), so typing is lost. This is pre-existing: audit finding 16, also on `main` (D7). |
| 18 | Homogenization cleared on design change and import | PARTIAL | A design change clears results, hides them, shows "Design changed. Run again." and removes `homogenization` from the export (within about 1 s under load). **But** pressing Run right after a change discards the fresh results (D2). |
| 19 | SHADE / VIEW byte-identical to the other F13LD tools | PASS | Block md5 for F13LD-SHADE `6fe646b976cb` and F13LD-VIEW `2c929e0f3fef`, identical in wave `ui-refresh` and `main`, tpms, noise, grain, beam, bundle, foam (branch and `main` of each). |
| 20 | Screenshots of each section, desktop + phone, menus open, read by the verifier | PASS (with findings) | 27 shots listed below, all read. Findings from them: D3, D4, D8, D10. |
| 21 | Design rules: OKLCH tokens, type scale, drawn icons, motion < 300 ms, terse wording | PARTIAL | Tokens are OKLCH (`--bg … --accent oklch(0.80 0.15 71)`). Remaining hex in page CSS: a mask `#000`, the Zener bar gradient, and inline mode-palette or brand colours (data, not chrome). Type: 11/13/15 plus the 17 px wordmark. Icons: "◐ view" glyph (D3); "×" in tile labels and "→" in import warnings are text, not icons. Motion: kit springs; the Zener marker and the surface-complexity bar use 0.4 s transitions (from `main`), and the flash is 1.4 s (a highlight). Wording is terse; Recipe and header text fit at 390. The Recipe button glow is strong next to "glow is subtle". |
| 22 | Thin sheet while dragging at the thinnest thickness | PASS | Gyroid sheet at thickness 0.02, dragging phase: interactive frame (scale 0.7, 160 steps) keeps the sheet whole, with blockier edges from the upscale; rest frame sharp. Both show dotted moiré patches at the same spots, also seen on `main` at 192 steps (fixed-step march, noted in stage 4). |

## Timing (swiftshader, CPU only, loaded shared machine; 1440×900, DPR 1)

`main` canvas 1070×352 (0.38 Mpx); new canvas 1008×852 (0.86 Mpx, 2.28× more pixels). Median of 3 synchronous draws, `readPixels`-synced. ms per 1000 canvas px in brackets.

| | `main` | new | new / main per canvas px |
|---|---|---|---|
| interactive, solid | 1027 ms (2.73), 96 steps, full res | scale 0.4: 1306 (1.52) · 0.7: 3794 (4.42) · 0.85: 4747 (5.53), 160 steps | 0.56× · 1.62× · 2.03× |
| interactive, sheet 0.2 | 1267 ms (3.36) | 0.4: 1172 (1.36) · 0.7: 3744 (4.36) · 0.85: 5538 (6.45) | 0.41× · 1.30× · 1.92× |
| rest frame, solid | 2192 ms (5.82), 192 steps, one block | one pass 9953 (11.6); strips: 5, longest 2759, total 8562 | longest block 1.26× `main`'s whole frame |
| rest frame, sheet | 1424 ms (3.78) | one pass 9355 (10.9); strips: 5, longest 2644, total 8865 | longest block 1.86× |
| live iso input at about 30 Hz for 3 s | 7 rAF frames | 2 interactive draws (adaptive scale held 0.7) | machine too loaded to read |
| Animate, 4 s | 0.9 rAF/s | 0.3 draws/s | machine too loaded to read |
| idle | 49 rAF/s (glyph loop) | **0 draws, 0 rAF** | |

A second run with the page order reversed (end of this file) gave the same picture, with `main` faster.

Reading: on this machine the new viewer is slower in absolute terms. It has 2.3× the pixels, and the interactive tier marches 160 steps against `main`'s 96. Per canvas pixel it beats `main` only when the adaptive scale reaches its 0.4 floor. Strips cut the longest blocking slice of a rest frame to about 1/3 of a one-pass rest frame, but each slice is still longer than `main`'s whole frame because the canvas is bigger. Stage 4's "0.65× per pixel" claim was not reproduced. A real GPU number is needed before calling render cost done.

## Defects (ranked)

| # | Sev | Defect | Repro |
|---|---|---|---|
| D1 | **High** (regression) | **No WebGL → empty control pane**; no control is usable. `glInit` throws on `gl.getExtension` with `gl` null, and the layout IIFE that adds `.sec.on` sits later in the same `<script>`, so it never runs. `main` keeps its controls usable. | Chrome `--disable-3d-apis --disable-webgl`, open the page: the header message shows, the pane shows only the fallback tabs, sections stay blank, `#preset` is hidden. Shots `verify-desktop-nowebgl.png`, `verify-phone-nowebgl.png`. |
| D2 | Medium | **Run right after a design change discards its own results.** `computeVF` debounces 60 ms then calls `invalidateHomog`; the Run solve starts at +10 ms and is synchronous, so the pending invalidate fires after the results land. New: results hidden, "Design changed. Run again." `main`: results stay visible, marked stale. Under load the window is long (seen 2 times in 4 runs during testing). | Analyze, grid 32; move iso, then press Run within 60 ms (script: set `#iso` + input, then `#wv-run.click()` in the same task). After the solve, results are hidden, stale is shown and the export has no `homogenization`. |
| D3 | Medium | **"◐ view" text glyph** in the viewer's view button (drawn icons only). TPMS swaps it for an SVG after mount (its index.html L5464); wave has the `.f13v>button .ic` CSS but no swap. | Any screenshot, top-right of the viewer. |
| D4 | Medium | **Preset dropdown panel narrower than its labels at 1440**: "Chiral Asymmetric (3,2,1)" and "Anisotropic Chladni (2,1,0)" are cut at the panel edge, and "Schwarz P Resonance" touches it. Phone (390) fits, just. | Desktop, Modes, open Preset: `verify-desktop-preset-open.png`. |
| D5 | Medium | **Targets under 44 px**: slider value editor 37×18 on all 7 sliders (kit); mode-bank number boxes 41 px wide at 1024 (app); sheet handle 22 px tall (phone template). Shared chrome: `fh-brand` 90×24, help (i) partly under the preset dropdown, help "show" 62×24. | Item 9. |
| D6 | Low | Slider aria-labels include the help button's "i": "Edit thicknessi", "Edit isoi", "Edit phase ωti", "Edit cell scalei", "Edit stretch xi". The wire reads the label text after the help drop-in appended (i). | Read `aria-label` on `.lq-sl-val`. |
| D7 | Low (pre-existing) | With a row soloed, clicking a number box in another row exits solo and loses focus, so the first typing is dropped (audit finding 16, not addressed). | Modes: solo row 2, click row 3's A box, type: nothing changes and focus is on body. |
| D8 | Low | Faint clipped shadow rectangles behind the preset dropdown and the symmetry segmented control (2–4 levels darker than the panel, with a straight cut edge). Visible on bright screens and in the brightened phone shot. | `verify-phone-modes.png`, brightened ×4. |
| D9 | Low | Motion over 300 ms: Zener marker and surface-complexity bar use 0.4 s transitions (inherited). The Recipe button carries a strong outer glow. | CSS L277, L286; header shots. |
| D10 | Low | The header message ("Imported … · Undo") overlaps the top of the control pane at 1200 px. Its note text runs straight on from "Undo" in `textContent` (layout looks fine). | `verify-desktop-import-undo.png`. |
| — | Info | Phone/desktop breakpoint is 900 px (brief says 768). After an import the symmetry widget updates one task later (MutationObserver), which is fine. The Analyze loop runs while results show (glyph spin, by design). | — |

## Claims in UI_REFRESH.md checked

- Stage 2 "0 app targets under 44 px": not true (D5).
- Stage 3 "widget layer blocked from WebGL: natives usable in all 4 sections": only the kit was blocked. With WebGL off for the whole page, the pane is empty (D1).
- Stage 4 "0.65× per px interactive": not reproduced (see timing).
- Kit re-vendor "help show 15/15, 16 (i)", export identical, round-trip, SHADE/VIEW untouched: confirmed.

## Screenshots (docs/ui-audit/)

| File | What |
|---|---|
| `verify-desktop-modes.png`, `-field.png`, `-cell.png`, `-analyze.png` | each section, 1440×900 (sheet mode) |
| `verify-desktop-analyze-results.png` | after Run (grid 32): glyph, tiles, bells |
| `verify-desktop-preset-open.png` | preset dropdown open (D4) |
| `verify-desktop-recipe-open.png`, `verify-desktop-recipe-menu.png` | Recipe menu open |
| `verify-desktop-view-open.png` | view menu open |
| `verify-desktop-help-open.png` | help drawer |
| `verify-desktop-import-undo.png` | header message after a paste import |
| `verify-desktop-modebank-after-import.png` | mode bank after import, add/solo/delete |
| `verify-desktop-thin-rest.png`, `verify-desktop-thin-middrag.png` | gyroid sheet at thickness 0.02, rest and mid-drag |
| `verify-desktop-nowebgl.png`, `verify-phone-nowebgl.png` | no WebGL: empty pane (D1) |
| `verify-phone-peek.png` | phone 390, sheet at peek |
| `verify-phone-modes.png`, `-field.png`, `-cell.png`, `-analyze.png` | phone, each section, full sheet |
| `verify-phone-analyze-results.png` | phone results |
| `verify-phone-preset-open.png`, `-recipe-open.png`, `-view-open.png`, `-help-open.png` | phone menus open |
| `verify-sync-1024-after-bar.png` | rail at 1024 after choosing Analyze on the tab bar at 700 |

Field-view shots (rest and 0.2× mid-motion, 38 files) were inspected in the scratchpad and not committed.

## Timing re-run (order reversed: new first, then `main`)

| | `main` | new | new / main per canvas px |
|---|---|---|---|
| interactive, solid | 627 ms (1.66) | 0.4: 1526 (1.78) · 0.7: 3628 (4.22) · 0.85: 5468 (6.37) | 1.07× · 2.54× · 3.83× |
| interactive, sheet | 938 ms (2.49) | 0.4: 1501 (1.75) · 0.7: 4595 (5.35) · 0.85: 4774 (5.56) | 0.70× · 2.15× · 2.23× |
| rest, solid | 1277 ms (3.39) | one pass 9436; strips 5, longest 2506, total 9063 | longest block 1.96× `main`'s frame |
| rest, sheet | 1398 ms (3.71) | one pass 10480; strips 5, longest 2173, total 8288 | longest block 1.55× |
| idle | — | 0 draws, 0 rAF | |

Both runs agree: per canvas pixel the interactive tier is at best level with `main` (scale 0.4) and 1.3–3.8× dearer at 0.7–0.85. With 2.3× the pixels, absolute interactive frames are 1.2–7× slower than `main` on swiftshader. Likely causes: 160 interactive steps against 96, and the larger canvas. This needs a real-GPU check, and possibly a lower `Q_INT` or a scale floor below 0.4, before render cost is signed off.

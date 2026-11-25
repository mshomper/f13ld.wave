# F13LD.wave — UI audit (stage 1)

Branch `ui-refresh`, 2026-11-25. Read-only. No code changed.
Screenshots: `docs/ui-audit/` (headless Chrome, swiftshader).

| File | What |
|---|---|
| `desktop-1440.png` | default state, 1440×900 |
| `desktop-1440-panel-mid.png` | panel scrolled: mode bank, view, homogenization |
| `desktop-1440-homog.png` | after Run homogenization (glyph + metric tiles) |
| `desktop-1440-viewmenu.png` | shared view menu open |
| `desktop-1440-help.png` | help drawer open |
| `tablet-1024.png` | 1024×900 |
| `phone-390.png` | 390×844 |
| `phone-390-modebank.png` | 390, mode bank rows |

## 1. Current state

- **Version:** three different values. `<title>` says v0.3 (L6), header says v0.6 (L495), export `version:"0.6"` (L2456), README says 0.3 (README L165).
- **Size:** `index.html`, 2932 lines, no `CLAUDE.md`, no `tests/`.
- **Blocks:**

| Lines | Block |
|---|---|
| 10–464 | page CSS (hex colours, no tokens beyond `--amber*`, `--text*`, `--border*`) |
| 466–738 | markup: header, `#cw` viewer, `#panel` with 7 `.group`s |
| 741–748 | `script#vs` |
| 751–1021 | `script#fs` (GLSL ES 1.00, `// @F13_SHADE_GLSL` insert) |
| 1023–2561 | app script: F13LD-SHADE (1025–1072), F13LD-VIEW (1073–1142), state/presets/mode bank (1149–1533), WebGL (1550–1795), metrics worker (1797–2115), homogenization + glyph (2117–2446), export + hand-offs (2448–2552), boot (2557–2560) |
| 2569–2681 | `style#f13h-style` (help drop-in) |
| 2683–2885 | `script#f13h-script` (help drop-in) |
| 2887–2930 | queue integration (loads `f13ld-queue.js` from mshomper.github.io; `alert()` fallback) |

- **Drop-ins:** help (yes), queue (yes, remote client), view (yes, F13LD-VIEW inline). F13LD-SHADE inline.
- **Layout:** one column, `max-width:1100px`, viewer on top (`flex:1`, min 220 px), panel below capped at `52vh` and scrolled. At 1440 the panel content is 1090 px tall in a 467 px box. No sections, no rail, no phone sheet.
- **Renderer:** WebGL2 (WebGL1 fallback), one full-screen quad, one program compiled once at boot (L1574–1579, `KHR_parallel_shader_compile` async link). Fixed-step march (`u_steps` = quality slider, 48–192, default 96; loop cap 200) with 6-step bisection, then F13LD-SHADE (soft shadow 48 steps, AO 5 taps). Every app value is a uniform: modes as 6× `vec4` + 6× `float`, symmetry int, tile, iso, stretch, thickness, sign, sheet. DPR capped at 1.5. Separate 2D canvas for axis gizmo and clip handle.
- **Second GL context:** `#wv-glyph` stiffness glyph (WebGL2), shader rebuilt per homogenization run with moduli baked in.
- **Export JSON** (`buildExportJSON`, L2451):

```
{ schema:"f13ld.wave/v1", tool:"F13LD.wave", family:"wave", version:"0.6", timestamp,
  field:{ symmetry, symmetryId, phase, signFlip, mode, thickness|null, iso, cellScale,
          stretch:[x,y,z], phaseTime, modes:[{n,m,p,A,phi}] },
  domain:{ unit, size, bounds, size_xyz }, coordinate:{...},
  metrics:{ modeCount, VF?, SSA?, SSA_unit?, field_RMS?, field_min?, field_max?, degenerate?, grid? },
  homogenization?:{ method:"MIL-HS", ... } }
```

- **`meta.tool`:** none. Wave has no `meta` block. Its identity is the top-level `tool:"F13LD.wave"`, `family:"wave"` and `schema:"f13ld.wave/v1"`. Queue client registers it as `tool:'f13ld.wave'` (L2911).
- **Routing check:**
  - Lab `60-add-design.js` `labRecipeInfo`: `family = json.family || json.tool` → `"wave"`; structural fallback `field.modes[]` → wave.
  - Mesh `50-recipe-router.js` `routeRecipe`: top-level `family:"wave"` → `families/fam-wave.js` `fromFamilyField` (subtype = `field.symmetry`); legacy detector order 50 on `field.modes`.
  - **Proposal:** no new tag. Keep `tool`, `family`, `schema` exactly as they are. Do not add `meta.tool`, and do not remove `family`: unlike TPMS, wave already depends on the top-level `family` key for Mesh routing. The brief's "no top-level `family`" rule applies to adding one, not to removing wave's existing key.

## 2. Findings

Severity: H = blocks the standard, M = visible defect / rule break, L = tidy-up.

| # | Issue | Where | Sev | Rule |
|---|---|---|---|---|
| 1 | No sections; one long scrolled panel under the viewer. Viewer is 350 px tall at 1440, 287 px at 390. | L36–42, L137–147 | H | Lesson 6; one focus per screen |
| 2 | Single render tier: full-res every frame, no interactive scale, no rest frame, no strips. Swiftshader: ~1.08 s per iso tick at 1440×900 (relative figure only). | L1743–1795 | H | Lesson 2 |
| 3 | `wvGlyphLoop` runs `requestAnimationFrame` forever from boot, even with the glyph hidden; draws every frame once results show. Idle ≠ zero frames. | L2357, L2446 | M | render-quality §4 |
| 4 | `animLoop` is its own rAF loop (time step fixed 0.015/frame, frame-rate dependent). | L1505–1512 | L | Lesson 3 (UI loops), §4 |
| 5 | Axis gizmo canvas reallocated (`cv.width=`) on every render. | L1668 | L | render cost |
| 6 | Quality slider exposes ray-march steps; becomes redundant with adaptive tiers. | L648–650, L1766 | M | fewer choices |
| 7 | Text under 11 px: 9 px and 10 px across group titles, hints, mode header, pills, metric labels, version. | L76, 170, 321–332, 375, 412, 426, 462, 566, 674, 680, 720–723 | M | type scale 11/13/15/20/28 |
| 8 | Hit targets under 44 px: segmented buttons ~24 px, sliders 12 px thumb, mode-row solo 22 px, delete ~18 px, clip pills ~20 px, number inputs ~22 px. | L160, L230–240, L290–345 | H | 44 px targets |
| 9 | Symbol glyphs as icons: ▶ ⏸ ⬡ ▦ ⊕ ▾ ◐ × → in buttons and labels. | L493–494, 624–625, 636, 1101, 1333, 1501, 2900 | M | drawn SVG only |
| 10 | Hex colours, no OKLCH tokens; warm-brown chrome (`--bg #0F0B07`, `--panel #15100A`) instead of shared neutral `--card`/`--panel`. | L11–24 | M | OKLCH; shared chrome neutral |
| 11 | Extra CSS glows: group-title dot, slider thumb, primary-button hover. | L184, L221, L262 | L | Lesson 8 |
| 12 | Low contrast: tool name and version `#5C4624` on `#0F0B07` = 2.2:1; clip pill text `#555` = 2.7:1; `--text-dim` on `--panel-2` = 4.3:1. | L490, L495, L160, L1667 | M | ink/dim/faint ≥ 4.5/4.5/3 |
| 13 | Mode-bank header has 7 grid columns, rows have 8: n/m/p/A/φ labels sit half a column left of their inputs (visible in `phone-390-modebank.png`). | L282 vs L424 | M | tidy output |
| 14 | `syncControls()` does not sync `#mode-toggle`, `#thickness`, `#thickness-row`, `#tile-toggle`, `#quality`. Any state write outside the click handlers leaves these stale. | L1518–1533 | M (import blocker) | lesson 4 |
| 15 | φ input `max="6.28"` but Randomize writes up to 2π (6.2832). A clamp-on-import using the input's max would warn on valid states. | L1290, L1332 | L | forgiving input |
| 16 | Focusing a number input in a non-soloed row while another row is soloed calls `rebuildModesUI()`, which replaces the input being focused. | L1349–1357 | M | widget re-mount risk |
| 17 | `alert()` for no-WebGL and queue offline. | L1557, L2918 | L | shared header message |
| 18 | "iso → 0" button duplicates what a detent at 0 on the iso slider gives. | L625, L1490 | L | fewer choices |
| 19 | Version strings disagree (v0.3 / v0.6 / README 0.3). | L6, L495, L2456 | L | housekeeping |
| 20 | Viewer stays blank for the first 2–3 s in swiftshader (async link). Captured once as a white canvas on the 390 run. | L1583–1606 | L | respond within 400 ms (first paint) |
| 21 | Homogenization `grid`, `Es`, `ν`, `Run` have help cards but no (i) of their own (only the group (i) opens `hgrid`). | help L2765–2781 | L | lesson 7 (note only) |
| 22 | No horizontal scroll at 390 / 700 / 1024 / 1440 (page and panel). Zero page errors, zero console errors at all four widths. | — | pass | no horizontal scroll |

## 3. Proposed section map

Four sections, one at a time (rail on desktop, tab bar on phone). Recipe actions move to the header Recipe menu.

| Section | Icon | Group | Controls |
|---|---|---|---|
| **Modes** | `surface` | Start | `#preset`, `#sym-toggle` (+ `#sym-hint`) |
| | | Mode bank (≤ 6 rows + add) | per row: solo, n, m, p, A, φ, delete; `#add-mode` |
| **Field** | `volume` | Level set | `#sign-toggle`, `#mode-toggle`, `#thickness` (sheet only), `#iso` (detent 0 replaces `#resetiso`) |
| | | Phase | `#time`, `#play` |
| **Cell** | `cell` | Cell | `#cellScale`, `#stretchX`, `#stretchY`, `#stretchZ`, `#tile-toggle` |
| **Analyze** | `analyze` | Homogenization | `#wv-grid`, `#wv-Es`, `#wv-nu`, `#wv-run` → results (glyph, 7 tiles, surface complexity, bells) |

- Header Recipe menu (`send` widget, icon `recipe`): Open recipe…, Export JSON (`#exp-json`), Queue, Choose queue…, Mesh, Lab.
- Viewer overlay: clip radio (X/Y/Z), view menu, VF/SSA/SOLO pills.
- `#quality` removed if adaptive render lands (decision below).
- Analyze gets the results badge (`F13UI.onAnalyze`) and the stale state (`#wv-homog-stale`) as an amber badge.

## 4. Control → widget map

Grep cross-check: `<select` 2, `type="range"` 8, `type="number"` 7 (2 static + 5 in the row template), checkbox template 1 (×5 view options), markup `<button>` 22, script-created buttons 8 kinds. All accounted for below.

### Static controls

| # | Control | Type | Widget | Notes |
|---|---|---|---|---|
| 1 | `#preset` | select (10 options) | `dropdown` | groups: Classic (P, Gyroid, Diamond), Families (Chiral, Chladni, Triad, Octave, Harmonic), Actions (Randomize, Clear). Re-picking the selected option fires no `change`, so Randomize twice needs a re-roll path (decision) |
| 2 | `#wv-grid` | select (32/48/64) | `segmented` sm | 3 short options read better than a dropdown |
| 3 | `#thickness` | range | `slider` | row shown only in Sheet mode (`#thickness-row`) |
| 4 | `#iso` | range | `slider`, detent 0 | replaces `#resetiso` |
| 5 | `#cellScale` | range | `slider`, detent 1 | |
| 6 | `#stretchX` | range | `slider`, detent 1 | |
| 7 | `#stretchY` | range | `slider`, detent 1 | |
| 8 | `#stretchZ` | range | `slider`, detent 1 | |
| 9 | `#time` | range | `slider`, format `v.toFixed(2)`, detents 0, π/2, π, 3π/2 | `animLoop` writes `.value` each frame → needs `mirrorValue` |
| 10 | `#quality` | range | remove (adaptive) or `slider` | decision |
| 11 | `#wv-Es` | number | `slider` 1–1000 GPa, typed entry | or keep native (decision); slider value editor covers typing |
| 12 | `#wv-nu` | number | `slider` 0–0.49 | |
| 13 | `#sym-toggle` (5 buttons) | segmented | `segmented` sm (5 options) | fits 390 at `sm`; sync on `class` |
| 14 | `#sign-toggle` (2) | segmented | `segmented` | labels "A · f > iso", "B · f < iso" |
| 15 | `#mode-toggle` (2) | segmented | `segmented` | |
| 16 | `#tile-toggle` (3) | segmented | `segmented` sm | |
| 17 | `#clipPillX/Y/Z` | toggle-radio | `radio` sm, `allowNone` | `CONFIG.clip = {get:()=>clipState.axis, set:onClipPill}`, same as TPMS |
| 18 | `#play` | action/toggle | `toggle` (label "Animate") | play/pause state; drawn icons if kept as button |
| 19 | `#resetiso` | action | remove (iso detent) | decision |
| 20 | `#add-mode` | action | native button, drawn `plus`, 44 px | disabled at 6 |
| 21 | `#wv-run` | action | `progressButton` (`CONFIG.run`) | needs progress hooks: run is synchronous today (`setTimeout` chain), report 0 → 1 |
| 22 | `#exp-json` | action | Recipe menu item "Export JSON" (`CONFIG.recipe.exportBtn`) | native stays hidden as the click target |
| 23 | header Mesh (`openInMesh`) | action | Recipe menu item | |
| 24 | header Lab (`openInLab`) | action | Recipe menu item | |

### Dynamic controls (created by script)

| # | Control | Created by | Count | Widget |
|---|---|---|---|---|
| 25 | row `.solo` | `rebuildModesUI` | 1/row | native round button, 44 px, mode swatch (it is a colour key, not a switch). Alternative: `radio` `allowNone` across rows — rejected, rows are rebuilt and reorder on delete |
| 26 | row `input[data-k=n]` | `rebuildModesUI` | 1/row | native number, 44 px, restyled; integer 0–8. Kit has no stepper widget (components/stepper.js is gallery only) |
| 27 | row `input[data-k=m]` | 〃 | 1/row | 〃 |
| 28 | row `input[data-k=p]` | 〃 | 1/row | 〃 |
| 29 | row `input[data-k=A]` | 〃 | 1/row | native number (−2…2, step 0.1). 6 rows × 2 sliders = 12 extra sliders: too much for one section |
| 30 | row `input[data-k=phi]` | 〃 | 1/row | native number (0…2π) |
| 31 | row `.del` | `rebuildModesUI` | 1/row | native icon button, drawn `trash`, 44 px |
| 32 | view menu: Shadows, Occlusion, Warm/cool, Cut faces, Lime edges | `f13ViewInit` | 5 checkboxes | `toggle` sm labelFirst (`CONFIG.viewMenu:true`) |
| 33 | view menu "◐ view" + "reset" | `f13ViewInit` | 2 | native, drawn icon; resync toggles after reset (TPMS pattern) |
| 34 | Queue main + caret | queue client | 2 | moved into Recipe menu (Queue, Choose queue…) |
| 35 | help `?`, close, 13× (i) | help drop-in | 15 | native, unchanged (count must stay 13) |

### Totals

| Type | Count |
|---|---|
| select | 2 |
| range | 8 |
| number | 2 static + 5 per mode row (15 at default 3 rows, 30 max) |
| checkbox | 5 (view menu) |
| segmented groups | 4 (sym 5, sign 2, mode 2, tile 3 = 12 buttons) + clip radio group (3) |
| app action buttons | 7 static (play, resetiso, add-mode, wv-run, exp-json, Mesh, Lab) + 2 per row (solo, del) |
| drop-in chrome buttons | 19 (help 15, view 2, queue 2) |
| **App controls, default state (3 rows)** | **2 + 8 + 17 + 5 + 5 groups + 7 + 6 = 50** |

→ liquid widgets: 1 dropdown, 5 segmented (incl. grid), 9–11 sliders, 1 radio, 6 toggles (Animate + 5 view), 1 progress button, 1 Recipe menu, 1 sections. Native by design: 30 max mode-row inputs and buttons, `#add-mode`, help chrome.

### `rebuildModesUI()` re-mount map

- **Recreates on every call:** `#modes-list` children — per row `.solo`, `.idx`, 5 number inputs, `.del`. Also rewrites `#mode-count` and `#add-mode.disabled`.
- **Callers:** `applyPreset` (L1302), solo click (L1340), focus in another row while soloed (L1352), delete (L1377), add (L1404). Not called by sliders or segmented controls.
- **Wiring:** wrap like TPMS `renderList` → `mountTermToggles`:
  `const r0 = rebuildModesUI; rebuildModesUI = function(){ r0.apply(this, arguments); mountModeRows(); };`
  The function is a top-level declaration in a classic script, so reassigning it reroutes the internal callers.
- **Key:** rows have no stable id, only `data-idx`; delete shifts indices. If any row widget is liquid (none in this proposal), key it by index and rebuild all on length change.
- **Hazard:** finding 16 — a focus-triggered rebuild destroys the focused control. Keep row inputs native, or move that rebuild to `change`.
- Restyling native row inputs to 44 px needs only CSS; nothing to re-mount.

## 5. Render cost

- **Shader build:** `script#fs` + `F13_SHADE_GLSL`, compiled once (L1574–1579). Structure never changes: modes, symmetry, sheet, sign, tile, steps are all uniforms.
- **Recompile call sites:** main view — none after boot. Glyph — `wvRebuildGlyph` once per Run homogenization (L2440), moduli baked in. Fine (one per click, not per tick).
- **Per slider tick:** zero compiles (measured: 20 iso ticks, `compileShader` count 0). Lesson 1 already satisfied. Each tick does `requestRender` + `computeVF` (60 ms debounce, worker; inline 32³ fallback) + `invalidateHomog`.
- **Quality tiers:** none. One tier, steps from `#quality` (48–192, default 96). F13LD-VIEW sets `uF13Interact` while dragging, which only switches off soft shadows.
- **Loop:** main view is on demand (`requestRender` coalesces into one rAF). Exceptions: `animLoop` while Animate is on; `wvGlyphLoop` always running (finding 3).
- **Buffer:** resized inside `render()` from client size × min(DPR, 1.5); no resize between tiers today (there are none).

render-quality.md changes for wave:
1. Uniforms: nothing to do. Add `drawStats.compile` counter for the stage-6 test.
2. Two tiers: Q_INT ≈ 96 steps (fixed-step marcher: steps drive cost linearly; TPMS's 160 is sphere-traced and not comparable), Q_REST ≈ 192 at full res; offscreen low FB at AQ scale + `blitFramebuffer`. Needs WebGL2 — keep the WebGL1 path single-tier. Raise DPR cap to 2 for rest frames only via the fixed buffer.
3. Rest frame in strips: `sceneDraw(fb,W,H,Q,[y0,y1])` with scissor; shader already reads `u_resolution`, so strips need only scissor + viewport.
4. On-demand: fold `animLoop` into `frame()` (time step from dt, bounded); gate `wvGlyphLoop` on visibility (Analyze section active + results shown), ≤ 30 fps, pause while the main view moves, respect Reduce Motion; move gizmo canvas sizing into `resize()`.
5. Remove `#quality` (or keep as a rest-tier cap: decision).

## 6. Recipe import

- **Recognise as wave:** `schema === "f13ld.wave/v1"` or `tool === "F13LD.wave"` or `family === "wave"` or `field.modes` is an array. Recognise others: `meta.tool` (`noise-scaffold-explorer`, `grain`, `beam`, `beam-builder`, `f13ld.foam`, `f13ld.tpms`), `surface.type`, `beams[]` → "That's a <tool> recipe" and abort.
- **Version keys:** `version` ("0.6"), `schema` ("f13ld.wave/v1"). Pre-0.6 exports lack `field.stretch` → default `[1,1,1]`. `field.symmetryId` preferred, fall back to `symLabels.indexOf(field.symmetry)`; `phase`/`signFlip` both present — prefer `signFlip`; `mode` "sheet"/"solid".
- **plan(recipe):** validate and clamp → `{modes ≤ 6 (n,m,p int 0–8; A −2…2; φ 0…2π), symmetry 0–4, signFlip 0/1, sheetMode 0/1, thickness 0.02–1.5 (null → keep current), iso −3…3, cellScale 0.5–3, stretch 0.5–2 ×3, time 0…2π}`. Extra modes beyond 6 → drop with a warning.
- **apply(plan):** order: `State.modes` → `State.soloMode=-1` → symmetry/sign/sheet/thickness/iso/cellScale/stretch/time → `rebuildModesUI()` → `updateSoloPill()` → `syncControls()` → sync mode-toggle, thickness row and value (finding 14) → `requestRender()` → `computeVF()` once.
- **Setters that exist:** `applyPreset(name)` (presets only), `syncControls()` (partial), `rebuildModesUI()`, `updateSoloPill()`, native slider `input` events via `bindSlider`, segmented button clicks. **Missing:** a state setter for modes/sheet/thickness. Write `State` directly then sync; or click `#mode-toggle` buttons for sheet so the thickness row follows.
- **Exclude from restore:** `metrics.*` (VF, SSA, field stats, grid — worker output), `homogenization.*` (MIL-HS results), `domain.*` and `coordinate.*` (derived), `timestamp`. Homog inputs (`grid`, `E_solid_GPa`, `poisson`) live only inside the results block: do not restore them either (Matt's rule).
- **Not in export, not restored:** `tile`, `quality`, solo, animate, clip, camera.
- **Round-trip caveat:** export writes `thickness:null` in Solid mode, so a solid state's hidden thickness is not carried. The export-equality test passes; the restored hidden thickness may differ. Leave the export as is (byte-identical rule).
- **Undo:** snapshot `buildExportJSON()`; `metrics`/`homogenization` in the snapshot are ignored on re-apply.

## 7. Help system

- **(i) count: 13.** Group (i): `#preset`, `#sym-toggle`, `#add-mode`, `#tile-toggle`, `#wv-run` (5). Row (i): `#sign-toggle`, `#mode-toggle`, `#thickness`, `#iso`, `#cellScale`, `#stretchX`, `#time`, `#quality` (8). 20 cards.
- **Anchors that must survive:** the ids above; `.group` with a `.group-title` child (group (i) goes into the title); `.row` with a `label` child (row (i) goes into the label); `.fh-right` (help trigger); card ids `f13h-card-<id>`. Classes `.f13h-*` belong to the drop-in.
- **Conflicts:**
  - Removing `#quality` drops the count to 12. Re-anchor the `quality` card elsewhere (e.g. view menu) or retire it — decision.
  - Folding `#resetiso` into the iso detent: no (i) involved.
  - `#add-mode` hosts the Mode Bank group (i): `#add-mode` must stay inside its `.group`.
  - Widgets hide native ranges: the `.row` + `label` must remain (TPMS keeps `.sc-group` rows; here it's `.row`).
- **"Show" buttons:** `helpTargets` → `{preset:'#preset', symmetry:'#sym-toggle', phase:'#sign-toggle', mode:'#mode-toggle', thickness:'#thickness', iso:'#iso', cellScale:'#cellScale', stretchX:'#stretchX', time:'#time', modebank:'#modes-list', tiling:'#tile-toggle', quality:'#quality', hgrid:'#wv-grid', hEs:'#wv-Es', hnu:'#wv-nu', hrun:'#wv-run'}`.

## 8. Accent colour — to confirm with Matt

| Role | Hex | OKLCH |
|---|---|---|
| `--amber` (active segments, primary buttons) | `#D97706` | 0.666 0.157 58.3 |
| `--amber-hi` (titles, slider thumbs) | `#F59E0B` | 0.769 0.165 70.1 |
| `--amber-glow` (header icon, dots) | `#FBBF24` | 0.837 0.164 84.4 |
| render surface tint (WAVE family, matches Mesh) | `#D97706` | 0.666 0.157 58.3 |
| sky (mode palette "aqua", Ex) | `#6BD9F2` | 0.830 0.106 214.7 |
| chrome `--bg` / `--panel` | `#0F0B07` / `#15100A` | 0.153 0.011 69 / 0.177 0.014 72 |

- **Recommended accent: amber, `oklch(0.84 0.16 75)`** (≈ `#FFB93A`), the kit's amber. Same lightness band as TPMS lime (0.88) and between the current `--amber-hi` and `--amber-glow`.
- Keep the render surface tint `#D97706` (family colour shared with Mesh). It is not UI chrome.
- Sky stays a data colour (mode palette, Ex). Not the accent.
- Chrome goes neutral (shared `--card`/`--panel`), dropping the brown tint.
- Clash: the brief also lists amber for noise, grain and bundle. If two tools both pick amber they look alike.

## 9. Risks

- **Mode-bank rebuilds** (`rebuildModesUI`, 5 callers, focus-triggered rebuild) — any liquid widget in a row is destroyed and must be re-mounted; keeping row inputs native avoids it.
- **`syncControls()` gaps** — recipe import and `mirrorValue` will show wrong sheet/thickness/tile state unless these are synced too.
- **Adaptive render on a fixed-step marcher** — cost is linear in steps; lowering steps at interactive scale can miss thin sheets (sheet mode, thickness 0.02). Interactive tier may need more steps than expected, or a scale-only reduction.
- **Glyph loop** — always-on rAF; easy to miss in the idle-draw test.
- **Existing `family:"wave"`** — the template says "don't add `family`"; a worker might "fix" it by deleting it and break Mesh routing.
- **Randomize preset** — a dropdown that keeps the native select as state won't re-fire for the same option; Randomize twice in a row stops working.
- **Help count** — removing quality / iso→0 changes (i) count unless planned.
- **Five-option symmetry segmented at 390** — 5 × 66 px; tight but fits at `sm`.

## Decisions for Matt

- Accent: amber `oklch(0.84 0.16 75)`? (noise/grain/bundle are also amber candidates.)
- Sections: Modes · Field · Cell · Analyze — OK, or keep Preset/Symmetry in its own section?
- Remove the Quality slider once the view adapts on its own? (Help (i) goes 13 → 12 unless the card moves to the view menu.)
- Replace "iso → 0" with a snap point at 0 on the iso slider?
- Mode-bank numbers (n, m, p, A, φ): keep as larger native number boxes, or add a stepper widget to the kit first?
- Randomize: keep as a preset entry (needs a re-roll fix) or make it its own button next to the preset menu?
- Animate: a switch, or a play/pause button with drawn icons?
- E<sub>s</sub> and ν: sliders with typed entry, or keep number boxes?
- Version for this refresh: bump to 0.7 and fix the title/README mismatch?

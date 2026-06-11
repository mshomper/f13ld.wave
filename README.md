# F13LD.wave

**Part of the [F13LD](https://f13ld.app) tool suite by [Not a Robot Engineering](https://notarobot-eng.com)**

**[Open the Tool](https://mshomper.github.io/f13ld.wave)**

<img width="1741" height="862" alt="f13ld wave github image" src="https://github.com/user-attachments/assets/241c3d5f-5bc8-4843-a7df-930286e15313" />

A browser-based standing-wave topology explorer for generating periodic metamaterial scaffolds from superposed cosine modes. It is the three-dimensional analog of the nodal figures sand traces on a vibrating Chladni plate: a bank of standing-wave terms is summed under a chosen symmetry operator and thresholded into a solid or thin-sheet surface. A single design pipeline carries the result through a WebGL 2.0 raymarched 3D preview, live volume-fraction and surface-area metrics, MIL-HS mechanical homogenization, and JSON export for downstream meshing and validation.

All geometry is fully implicit — no meshes, no STL intermediates, no skeleton graphs. Structure is defined as a scalar field evaluated on demand at any point in space.

---

## Overview

Periodic lattices come in two broad families: the smooth implicit surfaces of TPMS (gyroid, Schwarz P) and the strut-and-node graphs of beam lattices. Both are built from a fixed unit-cell motif. Standing-wave topology takes a different route — it composes the cell itself from interfering cosine modes, the same mathematics that governs resonance on plates, membranes, and acoustic cavities. By choosing wavenumbers, amplitudes, phases, and a symmetry operator, you sculpt the nodal set directly, and a wide range of recognizable architectures — including the gyroid and Schwarz P families — fall out as special cases of one continuous design space.

The five symmetry operators occupy distinct positions in that space:

| Operator | Construction | Natural topology | Notable cases |
|---|---|---|---|
| Pure | Direct mode product | Anisotropic cells, orthotropic by design | Simple cubic resonances |
| Anti (Chladni) | Sign-alternating axis permutations | Sharp nodal figures, true Chladni character | Requires distinct `(n,m,p)` |
| Cubic | All six axis permutations summed | Cubic-symmetric, bicontinuous | Schwarz P / gyroid family |
| Chiral | Three even cyclic permutations | Handed, chiral cubic | Asymmetric `(n,m,p)` preserves handedness |
| Schoen | Three-cycle, sine on first axis | Gyroid family | `(1,1,0)` → standard gyroid |

The field is periodic, not stochastic — every parameter set is fully deterministic and tiles seamlessly.

---

## Usage

Open `index.html` directly in a browser. No server, no build step, no dependencies.

**Recommended workflow:**

1. Start from a **preset**, or build a field from scratch
2. Choose a **symmetry operator** and add cosine **modes** to the mode bank
3. Tune **iso**, **topology**, and **phase** while watching the live **VF** / **SSA** pills
4. Sweep **phase ωt** (or hit **Animate**) to morph the standing-wave field
5. Run **homogenization** to get effective mechanical properties and the directional stiffness surface
6. **Save JSON** — carries all parameters, metrics, and homogenization results
7. Click **Open in F13LD.mesh**, or feed the JSON to [F13LD.mesh](https://mshomper.github.io/f13ld.mesh) for watertight 3MF export

---

## Symmetry Operators

A field is a sum of cosine modes, each transformed by the active symmetry operator before summation:

```
φ(x) = Σᵢ Aᵢ · 𝒮[nᵢ, mᵢ, pᵢ](q) · cos(φᵢ + ωt)        q = (π/5)·x
```

where `Aᵢ` is mode amplitude, `φᵢ` is mode phase, `ωt` is the global animation phase, and `q` maps one unit cell to the interval `[-π, π]`. Writing `cₐ(j) = cos(j·q_a)`, the operator `𝒮` is one of:

**Pure** — direct mode product. Anisotropic by construction; each axis carries its own wavenumber.
```
𝒮 = cₓ(n)·c_y(m)·c_z(p)
```

**Anti (Chladni)** — sign-alternating sum over the six axis permutations of `(n,m,p)` (even permutations positive, odd negative). This is the antisymmetric combination that produces true Chladni nodal figures.
> Requires `(n,m,p)` all distinct — any equal or zero pair cancels the term to zero.

**Cubic** — all six axis permutations summed with the same sign. Cubic-symmetric and bicontinuous; this is the Schwarz P / gyroid family.

**Chiral** — the three even cyclic permutations only. Produces handed, chiral cubic structures; handedness is preserved for asymmetric `(n,m,p)`.

**Schoen** — a three-cycle with a sine on the first axis: `sin(n·qₓ)·c_y(m)·c_z(p)` plus cyclic terms. Generates the gyroid family — mode `(1,1,0)` yields the standard gyroid.

---

## The Mode Bank

Up to **six** cosine modes superpose into a single field. Each mode is:

- **`n, m, p`** — integer wavenumbers along x, y, z. Higher integers pack more periods into the cell; mixed values break symmetry and create anisotropy.
- **Amplitude `A`** — relative weight of the mode in the sum.
- **Phase `φ`** — per-mode phase offset, shifts the mode's nodal set within the cell.

Each mode has its own color and a **solo** toggle that isolates it in the viewport for inspection — solo affects the preview only; presets, metrics, homogenization, and export always use the full mode bank.

---

## Presets

Eight starting points spanning the operator space:

| Preset | Character |
|---|---|
| Schwarz P Resonance | Classic cubic bicontinuous cell |
| Gyroid (Schoen, 1,1,0) | Standard gyroid via the Schoen operator |
| Chiral Asymmetric (3,2,1) | Handed chiral cell from distinct wavenumbers |
| Anisotropic Chladni (2,1,0) | Directionally biased nodal figure |
| Octave Doubling | Two modes an octave apart — hierarchical features |
| Cubic Triad (1,2,3) | Three-mode cubic mix |
| Diamond Modes | Diamond-network-like topology |
| Harmonic Series | Stacked harmonics for graded feature scale |

---

## Scaffold Geometry

**Topology** controls how the scalar field is converted to solid/void geometry. The SDF convention is **negative-inside** — values below zero are solid.

| Mode | SDF expression | Character |
|---|---|---|
| Solid | `iso − φ` | One solid domain bounded by the `φ = iso` level set |
| Sheet | `\|iso − φ\| − t` | A shell of half-thickness `t` straddling the level set |

- **iso** — shifts the isosurface threshold. In solid mode this directly controls volume fraction.
- **thickness `t`** — shell half-width in sheet mode (0.02–1.5).
- **Phase A / B** — sign flip; swaps which side of the level set is solid (the two complementary phases of the same field).
- **cell scale** — physical cell size (0.5–3). Scales the exported domain in millimetres; it does not change the field's frequency content.
- **phase ωt** — global standing-wave phase. Sweep it manually or press **Animate** to watch the nodal set evolve in real time.

---

## Render & Viewport

A single WebGL 2.0 raymarched view renders the field by sign-change detection with bisection refinement along each ray.

- **Quality** (48–192) sets the raymarch / sampling resolution. It affects preview sharpness only, not the underlying field or any computed metric.
- **Tiling** (1×1×1 / 2×2×2 / 3×3×3) repeats the cell in the shader for periodicity inspection at no additional compute cost.
- **Metric pills** (top-left) report **VF** (volume fraction) and **SSA** (specific surface area), recomputed live as you edit. A **solo** pill appears when a single mode is isolated.

Metrics are computed in a Web Worker on a 64³ grid (32³ synchronous fallback when Workers are unavailable), keeping the viewport responsive.

---

## Homogenization (MIL-HS)

Estimates effective elastic properties from the voxelized field using the Mean Intercept Length method coupled to the Hashin-Shtrikman upper bound. Button-triggered — too heavy to run on every slider move — and computed on the same solid mask the renderer uses, so the analyzed geometry matches the preview exactly.

**Controls:**
- **grid** (32 / 48 / 64) — analysis resolution
- **E_s (GPa)** — solid material stiffness (Ti6Al4V ≈ 114 GPa, PEEK ≈ 3.6 GPa, 316L stainless ≈ 193 GPa; default 100)
- **ν** — solid Poisson's ratio

**Outputs:**
- **Volume fraction ρ** — solid phase fraction
- **Ex, Ey, Ez** — directional Young's moduli (GPa)
- **Gxy** — shear modulus (GPa)
- **νeff** — effective Poisson's ratio
- **Zener A** — anisotropy ratio. A = 1 is perfectly isotropic.
- **Surface complexity** — normalized 0–1 surface-area measure, anchored to the F13LD.tpms scale so values are comparable across tools.
- **Directional stiffness surface** — rotating WebGL visualization of the directional modulus glyph, with per-axis spread bands.

> MIL-HS assumes statistical homogeneity and is a fast in-tool estimate. Results improve with grid resolution. [F13LD.lab](https://f13ld.app) refines the same recipe with full FFT-based elastic homogenization.

---

## Export JSON

The exported JSON carries the complete parameter set, the live metrics, and — if homogenization has been run — the mechanical results. This file is the handoff to [F13LD.mesh](https://mshomper.github.io/f13ld.mesh), and the in-app **Open in F13LD.mesh** button passes the same recipe directly via URL.

```json
{
  "schema": "f13ld.wave/v1",
  "tool": "F13LD.wave",
  "family": "wave",
  "version": "0.3",
  "timestamp": "...",
  "field": {
    "symmetry": "pure",
    "symmetryId": 0,
    "phase": "A",
    "signFlip": false,
    "mode": "solid",
    "thickness": null,
    "iso": 0.0,
    "cellScale": 1.0,
    "phaseTime": 0.0,
    "modes": [
      { "n": 1, "m": 0, "p": 0, "A": 1.0, "phi": 0.0 },
      { "n": 0, "m": 1, "p": 0, "A": 1.0, "phi": 0.0 },
      { "n": 0, "m": 0, "p": 1, "A": 1.0, "phi": 0.0 }
    ]
  },
  "domain": { "unit": "mm", "size": 10.0, "bounds": [-5.0, 5.0] },
  "coordinate": {
    "convention": "f13ld",
    "worldScale": 0.6283185307,
    "periodicityRadians": 6.2831853072,
    "signConvention": "negative-inside"
  },
  "metrics": {
    "modeCount": 3,
    "VF": 0.500000,
    "SSA": 1.830000,
    "SSA_unit": "1/mm",
    "grid": 64
  },
  "homogenization": {
    "method": "MIL-HS",
    "grid": 48,
    "E_solid_GPa": 100,
    "poisson": 0.3,
    "volume_fraction": 50.0,
    "Ex_GPa": 47.73,
    "Ey_GPa": 47.73,
    "Ez_GPa": 47.73,
    "Gxy_GPa": 19.09,
    "nu_eff": 0.15,
    "anisotropy": 1.0,
    "surface_complexity": 0.38
  }
}
```

**Coordinate convention.** The field is evaluated at `q = (π/5)·x`, so the world cube `[-5, 5]` maps to exactly one cell (`q ∈ [-π, π]`) and tiles seamlessly. `cellScale` sets the physical cell size in the `domain` block; it is not a frequency multiplier. The SDF is negative-inside; F13LD.mesh applies a single sign flip into the Manifold (positive-inside) convention on import.

---

## Part of the F13LD Suite

| Tool | Description |
|---|---|
| [TPMS Builder](https://mshomper.github.io/f13ld.tpms) | Periodic implicit surfaces — gyroid, Schwarz P, FRD, and more |
| [Noise Explorer](https://mshomper.github.io/f13ld.noise) | Isotropic stochastic scaffolds via simplex, cellular, FBM, and curl noise |
| [Grain](https://mshomper.github.io/f13ld.grain) | Anisotropic stochastic scaffolds — spinodoid, GRF, hyperuniform, reaction-diffusion |
| **Wave** | Standing-wave / Chladni topology — cosine modes under five symmetry operators |
| [F13LD.mesh](https://mshomper.github.io/f13ld.mesh) | Watertight 3MF export from any tool JSON — Draft / Standard / Fine resolution |

---

## Technical Notes

**Fully implicit** — all geometry is defined as a scalar field evaluated pointwise. No mesh generation or STL intermediates exist until Manifold levelSet is invoked in F13LD.mesh.

**3D view** — WebGL 2.0 raymarcher. The standing-wave field is traced by marching each ray until a sign change is detected, then refining the crossing by bisection for a clean isosurface hit. Surface normals are estimated from the analytic field gradient.

**Symmetry transforms** — applied per mode in the fragment shader and mirrored exactly in the CPU field evaluator used for metrics, homogenization, and export, so the preview, the reported numbers, and the meshed result all describe the same geometry.

**Metrics** — volume fraction and specific surface area run in a background Web Worker on a 64³ grid, with a synchronous 32³ fallback where Workers are unavailable.

**Homogenization** — runs on demand on the main thread; the directional stiffness surface renders in its own WebGL 2.0 context. Requires WebGL 2.0 for the glyph; metric outputs are reported regardless.

**Browser compatibility** — Chrome and Firefox. WebGL 2.0 required.

---

## References

- Chladni, E.F.F. (1787). *Entdeckungen über die Theorie des Klanges* (Discoveries in the Theory of Sound). Leipzig. — the original nodal figures.
- Schoen, A.H. (1970). Infinite periodic minimal surfaces without self-intersections. *NASA Technical Note D-5541*. — the gyroid.
- Hashin, Z. & Shtrikman, S. (1963). A variational approach to the theory of the elastic behaviour of multiphase materials. *J. Mech. Phys. Solids*, 11, 127–140.
- Whitehouse, W.J. (1974). The quantitative morphology of anisotropic trabecular bone. *J. Microscopy*, 101(2), 153–168. — mean intercept length.
- Wohlgemuth, M., Yufa, N., Hoffman, J. & Thomas, E.L. (2001). Triply periodic bicontinuous cubic microdomain morphologies by symmetries. *Macromolecules*, 34(17), 6083–6089.

---

## License

MIT — see LICENSE file.

*Not a Robot Engineering · notarobot-eng.com*

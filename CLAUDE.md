# CLAUDE.md — isopod-maze

Context for AI agents (and contributors) working in this repo.

## What this is

A single-file, browser-based parametric generator for 3D-printable isopod (pill bug / ダンゴムシ) mazes, aimed at **turn-alternation (TA / 交替性転向)** behavior experiments. Open `index.html` — no build, no install. Three.js is loaded from a CDN; everything else is vanilla HTML/CSS/JS in `index.html`.

## Architecture (all in `index.html`)

- **Params/UI**: left control panel; `getParams()` (random maze) and `getSTParams()` (serial T-maze).
- **Maze generation**:
  - `generateMaze()` — DFS recursive backtracker (random maze).
  - `generateTAMaze()` — TA mode: pre-carves an alternating staircase solution, makes every corner a T-junction, grows dead-end branches, fills the rest. Unique solution = the alternating staircase.
  - `generateSerialT()` + `rasterizeST()` — sparse "serial T-maze" experimental apparatus.
- **`solveMaze()`** — BFS, used to highlight the solution path.
- **2D**: Canvas 2D blueprint (`redraw2D`, `redraw2D_ST`), incl. R/L turn-label overlay.
- **3D**: Three.js preview (`init3D`, `rebuild3D`, `rebuild3D_ST`).
- **Export**: `buildExportGeometry` / `buildExportGeometryST` → `heightfieldTris()` → binary STL.

## Invariants — do NOT break these

1. **Single file, zero install.** Keep it all in `index.html`. Only external dep is Three.js via CDN.
2. **STL must be a single watertight manifold.** Geometry is emitted via `heightfieldTris()` (a 2-level height field), NOT overlapping boxes. Every edge must be shared by exactly 2 triangles — no coincident/internal faces, no self-intersection. (Overlapping separate solids previously caused the wall pattern to ghost into the base layer.)
3. **TA solution = real T-junctions.** In TA mode, every corner except the first must be: straight-ahead **walled**, both perpendiculars **open** (one continues the solution = the alternating turn, the other is a dead-end branch). The unique solution must require strict R/L alternation. The first turn is a forced reference turn (exception).
4. **Units are millimeters.** Corridor width should match the animal's body width (~4–7 mm for pill bugs).

## Run & verify

```bash
python3 -m http.server 8000   # then open http://localhost:8000/index.html
```

When changing geometry/export, verify in the browser console:

- **Watertight**: build the export tris, count undirected edges; every edge must appear exactly twice (0 boundary edges, 0 edges shared by >2 faces).
- **TA correctness**: the solution's turn sequence (cross-product sign of consecutive segments) must strictly alternate; every non-first corner must satisfy straight-blocked + both-perpendiculars-open.

Always look at the actual rendered result (2D blueprint + 3D) before claiming a visual change works.

## Conventions

- Vanilla JS; match the surrounding style. No frameworks, no bundler.
- Develop directly in this repo (branches / `git worktree` / PRs). Commit and push here — this repo is the single source of truth.

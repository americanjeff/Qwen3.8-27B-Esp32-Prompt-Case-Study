# ESP32-C3 LED Circuit Dashboard — Model Comparison

Four local models were given the same prompt to build a single-page interactive SVG dashboard of an ESP32-C3-DevKitM-1 wired to a breadboard with an LED and pushbutton. This directory holds the outputs, session transcripts, and configuration for each run.

**Original prompt:** [andrisgauracs/gist](https://gist.github.com/andrisgauracs/90613bfae85b3b976c189b491ced06b2) (also in [`prompt.txt`](prompt.txt))

## Results Summary

| | qwen3.8-27b-default | qwen3.8-27b-nothink | qwen3.6-27b-default | qwen3.6-27b-nothink | qwen3.6-27b-nothink-fixed |
|---|---|---|---|---|---|
| **Model** | qwen3.8-27b-mtp | qwen3.8-27b-mtp-nothink | qwen3.6-27b-mtp | qwen3.6-27b-mtp-nothink | qwen3.6-27b-mtp-nothink |
| **Thinking** | medium | off | medium | off | off |
| **Generation time** | 18m 19s | 6m 34s | 28m 27s | 10m 56s | 16m 37s |
| **Tool calls** | 16 | 34 | 23 | 20 | 34 |
| **Tokens (in/out)** | 71,540 / 63,767 | 25,473 / 26,970 | 71,077 / 47,927 | 59,790 / 44,814 | 44,587 / 65,939 |
| **Cache read** | 67,264 | 1,313,427 | 1,226,135 | 858,584 | 2,577,639 |
| **Output file** | `index.html` (28 KB) | `index.html` (24 KB) | `dashboard.html` (42 KB) | `dashboard.html` (51 KB) | `index.html` (54 KB) |
| **JS lines** | 63 | 120 | 70 | 44 | 31 |
| **ARIA attrs** | 3 | 1 | 0 | 0 | 0 |
| **Semantic HTML** | 4 | 0 | 0 | 0 | 0 |
| **requestAnimationFrame** | no | no | yes | yes | no |
| **Color bands (resistor)** | no | yes | yes | yes | yes |
| **Hover tooltips** | yes | no | yes | yes | yes |
| **Click interaction** | yes | yes | yes | yes | yes |

## Per-Directory Contents

Each subdirectory contains:

- **`<output>.html`** — the generated dashboard
- **`preset.ini`** — llama-cpp configuration used for the run
- **`session-excerpt.md`** — GitHub-compatible transcript with timestamps, collapsible thinking/tool calls/tool results
- **`session-excerpt.html`** — styled HTML transcript (self-contained except for shared images)
- **`session-excerpt.jsonl`** — verbatim session lines from prompt through final turn
- **`images/`** — decoded PNG screenshots from tool results (referenced by both md and html)
- **`preview.png`** — screenshot of the dashboard (only in `qwen3.6-27b-nothink-fixed/`)

## Quality Notes

**qwen3.8-27b-default** — The most polished output. Clean dark-themed UI with a well-organized status panel, `requestAnimationFrame`-free CSS transitions, proper ARIA labels, and semantic HTML (`<header>`, `<main>`, `<footer>`). Missing resistor color bands and the SVG is JS-created with a helper factory. Took the longest of the 3.8 runs (18m) with moderate tool usage (16 calls) and high token count (135K total). The model verified its work via screenshot inspection.

**qwen3.8-27b-nothink** — Fastest run (6m 34s) with the lowest token count (52K). The output is compact and clean with a dark theme and good CSS variable usage. However, it lacks hover tooltips entirely and has no semantic HTML or meaningful ARIA attributes. The SVG is JS-created with 120 JS lines — the most verbose JS of any run. Notably achieved good results with zero thinking and heavy cache reads (1.3M).

**qwen3.6-27b-default** — Largest output (42 KB) with the most SVG features (patterns, radial gradients, filters, `requestAnimationFrame` for current animation). Uses `requestAnimationFrame` for smooth current-flow animation along wire paths. Has hover tooltips and click interaction. No ARIA or semantic HTML. Took the longest overall (28m 27s) with high tool usage (23 calls).

**qwen3.6-27b-nothink** — Original run without `presence-penalty`. Largest file (51 KB) with the most SVG primitives (polylines/polygons, linear gradients, patterns). Uses `requestAnimationFrame` for current animation. Has hover tooltips and click interaction. No ARIA or semantic HTML. Moderate generation time (10m 56s) with 20 tool calls. The most feature-complete SVG of any run but with no accessibility considerations.

**qwen3.6-27b-nothink-fixed** — Re-run with `presence-penalty = 1.5` added to the preset (matching the provider-recommended configuration). The output is slightly larger (54 KB) and uses radial gradients instead of linear gradients/patterns. No `requestAnimationFrame` — current animation is CSS-driven. Has hover tooltips and click interaction. No ARIA or semantic HTML. Took longer than the original (16m 37s vs 10m 56s) with more tool calls (34 vs 20) and higher cache reads (2.5M vs 858K). The presence-penalty made the model more verbose and less efficient but produced a cleaner, more compact JS implementation (31 lines vs 44).

## Observations

- **Thinking vs. no-thinking:** The thinking models (default) generally produced more structured, accessible output with fewer tool calls but longer generation times. The nothink models were faster but produced more verbose JS and less accessible markup.
- **3.8 vs. 3.6:** The 3.8 models produced smaller files with cleaner code. The 3.6 models produced larger files with more SVG features but also more verbose implementations.
- **All models** correctly implemented the core requirements: GPIO4/9 wiring, 220Ω resistor, pushbutton with pull-up, 3V3/GND rails, breadboard layout, and interactive toggle. None left debug code or console logs.
- **Presence-penalty effect:** The `qwen3.6-27b-nothink` → `qwen3.6-27b-nothink-fixed` comparison shows that adding `presence-penalty = 1.5` increased generation time by 52%, tool calls by 70%, and cache reads by 200%. However, it reduced JS verbosity (31 lines vs 44) and switched from `requestAnimationFrame` to CSS-driven animation. The output quality is comparable but the efficiency cost is significant.

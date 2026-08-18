# Local-model comparison — ESP32-C3 LED circuit dashboard

Comparison of four local llama.cpp Qwen-27B configurations (one re-run, see below) on
the same build task: an
interactive, electrically-accurate SVG dashboard of an ESP32-C3-DevKitM-1 wired to a
breadboard, pushbutton, and LED. The prompt is
[prompt.txt](prompt.txt), from
[andrisgauracs's gist](https://gist.github.com/andrisgauracs/90613bfae85b3b976c189b491ced06b2).

Each subdirectory contains:

- the model's output (`index.html` or `dashboard.html`, single self-contained file)
- the llama.cpp preset used (`preset.ini`)
- for the nothink-fixed run: a `preview.png` screenshot of the rendered page
- the full agent-session excerpt that produced it, in three formats:
  [`session-excerpt.md`](qwen3.8-27b-default/session-excerpt.md),
  [`session-excerpt.html`](qwen3.8-27b-default/session-excerpt.html), and a verbatim
  [`session-excerpt.jsonl`](qwen3.8-27b-default/session-excerpt.jsonl) (raw session lines,
  initial prompt through the final turn, with timestamps per message; thinking, tool calls,
  and tool results are collapsible in the rendered versions).
- decoded screenshots the agent took while verifying its own work, in `images/`
  (only the 3.8 runs captured screenshots)

**Note on the two nothink-3.6 runs:** `qwen3.6-27b-nothink` was re-run as
[qwen3.6-27b-nothink-fixed](qwen3.6-27b-nothink-fixed/) because the original preset was
missing a provider-recommended setting. The diff is a single line —
`presence-penalty = 1.5` (see the two `preset.ini` files). The original is kept for
comparison; the metrics table and the "fresh-eyes" review below use the fixed run.

## Run summary

| Config | Output | File size | Time to done | Turns | Tokens in / out (cumulative) | Cache read (cumulative) |
|---|---|---:|---:|---:|---:|---:|
| [qwen3.8-27b-default](qwen3.8-27b-default/) | `index.html` | 28.1 KB | 18m 19s | 14 | 71.5 K / 63.8 K | 672.6 K |
| [qwen3.8-27b-nothink](qwen3.8-27b-nothink/) | `index.html` | 23.9 KB | 6m 34s | 35 | 25.5 K / 27.0 K | 1.31 M |
| [qwen3.6-27b-default](qwen3.6-27b-default/) | `dashboard.html` | 41.7 KB | 28m 27s | 22 | 71.1 K / 47.9 K | 1.23 M |
| [qwen3.6-27b-nothink](qwen3.6-27b-nothink/) *(original, no `presence-penalty`)* | `dashboard.html` | 49.7 KB | 10m 56s | 18 | 59.8 K / 44.8 K | 858.6 K |
| [qwen3.6-27b-nothink-fixed](qwen3.6-27b-nothink-fixed/) | `index.html` | 53.1 KB | 16m 37s | 33 | 44.6 K / 65.9 K | 2.58 M |

Notes on the numbers:

- "Tokens in/out" are summed across all assistant turns of the session (the context is
  re-sent every turn, so these greatly exceed any single turn); the llama.cpp provider did
  not report reasoning tokens, but the thinking blocks in the MD/HTML excerpts give a sense
  of scale (~137 KB of thinking for 3.8-default, ~359 KB for 3.6-default, none for the
  nothink runs).
- All four runs researched the pinout via web search against Espressif's ESP-IDF docs
  before drawing anything, and all claim to have done the prompt's §5 self-check pass.

## General impressions

All four outputs are working, self-contained single-file dashboards with the right
topology: GPIO4 → 220 Ω (red-red-brown-gold bands) → LED anode → cathode → GND rail;
GPIO9 → pushbutton → GND with pull-up; color-coded wires; a status panel showing both
GPIO levels, circuit state, and a plausible LED current (≈5.9–6.8 mA, consistent with
(3.3 − V_f) / 220 Ω); hover tooltips on pins, wires, and components; and animated current
flow (dashed-path SMIL/CSS/`requestAnimationFrame`, each done differently). Differences
emerge in rigor, spec-fidelity, and code style:

### qwen3.8-27b-default — most rigorous, best engineering

The standout on accuracy. It caught two problems in the prompt itself and deliberately
overrode the spec, noting both in the page footer:

- the real DevKitM-1 has a **Micro-USB** port, not USB-C, so it drew Micro-B;
- a real **830-tie-point** breadboard has **63 columns**, not 30 (30 columns is a
  400-point board) — it built the true 63-column layout, while the other three runs all
  followed the spec's 30 columns (which cannot actually be 830 points).

It generated the board from data arrays and programmatically diffed the rendered pin
labels against the official J1/J3 tables, verified every wire endpoint against the
breadboard node grid, and even teaches a subtle point: with the specified wiring the LED
loop is powered *through* GPIO4, so the 3V3→rail wire correctly shows **0 mA** (unloaded
supply) instead of faking flow. Animation is gated to the actual closed loop (plus a µA
input loop while the button is held). Cleanest, most compact code (28 KB, ~145 lines of
JS, data-driven SVG, zero inline handlers). It also verified its own screenshots and
re-checked the DOM when a screenshot artifact looked wrong.

### qwen3.8-27b-nothink — fastest, solid, one math slip

Fastest and cheapest run (6m 34s, ~27 K output tokens). Correct 30-column board, nice
details (strapping-pin notes in tooltips, plunger-animated pushbutton, SMIL flow path
built programmatically, V_R/V_LED split in the status panel), and it caught and fixed a
real bug in `render()` during self-testing. However, it kept the spec's 30 columns while
claiming "830 tie-points = 630 main (30 cols × 10 rows)" — internally inconsistent math
(30 × 10 = 300, and 630 is the 63-column number); it also followed the spec's USB-C
rather than the real board's Micro-USB. Good compact code (24 KB, ~400 lines incl. the
`el()` helper, 19 functions, no inline handlers).

### qwen3.6-27b-default — thorough but slow

Full spec coverage with the nicest "textbook" component rendering (dome-shaped LED with
flat cathode side, zigzag resistor, pushbutton straddling the center gap, 8-particle flow
animation, consistent 5-color wire convention, per-wire explanatory tooltips including
why the red rail carries no current). It followed the spec's 30 columns without flagging
the 830-point contradiction. It was the slowest run (28m 27s) with the largest thinking
volume (~360 KB) and heaviest code (43 KB, ~335 lines of JS) for comparable fidelity to
the 3.8 runs.

### qwen3.6-27b-nothink (original) — kept for the config-diff comparison

Original run without `presence-penalty`: visually the most elaborate board (CP2102
bridge, LDO, module outline, PCB antenna meander), largest file of the 3.6 runs (51 KB)
in 10m 56s. The LED was rendered as a schematic diode symbol (triangle + bar) rather than
a physical LED, and interaction was two independent toggles. Kept side-by-side with the
[fixed re-run](#qwen36-27b-nothink-fixed--re-review-fresh-eyes) to show the effect of one
missing sampling parameter.

### qwen3.6-27b-nothink-fixed — re-review (fresh eyes)

With `presence-penalty = 1.5` restored, the same model takes longer (16m 37s, 33 turns,
~66 K output tokens vs 44.8 K) and produces the largest file of the whole comparison
(54 KB). Judging `index.html` and its `preview.png` on their own merits:

**What it does well.** The page has the best overall aesthetic of the four: a clean
dark-navy theme, a proper title/subtitle, and a well-labeled green DevKit (module
outline, UART bridge, LDO, Micro-USB, BOOT/RST, RGB/PWR LEDs — all hoverable). The LED
is finally a physical dome with glow and emission arrows (the original run's diode-symbol
regression is gone), the resistor has color bands, and the status panel is a tidy
card with color-coded HIGH/LOW/ON badges and a live mA readout (5.9 mA). Tooltips cover
every pin, wire, and component, and the JS is straightforward; note this run did its
self-check via bash only — it never rendered the page in a browser, unlike the 3.8
runs (hence the `preview.png` was captured after the fact).

**Problems a fresh look reveals.**

- **The two signal wires are collinear and the same color.** Both the GPIO4 and GPIO9
  wires run horizontally along the same row (row a) in identical orange, so in the
  render the GPIO9 connection is invisible — it reads as one wire from the board. The
  prompt asked for a *distinct* color per signal line; visually there is only one.
- **Layout is unbalanced.** Board and breadboard occupy the top-left half of a large
  canvas; the status card floats centered below a big empty region. It looks like a
  1920×1080 screenshot of a page that only uses the top 40%.
- **Breadboard labeling is wrong.** Captioned "Full-Size Breadboard — 830 Tie Points" and
  commented as such, but it has 30 columns × 10 rows plus four 30-hole rails = 420
  points (a real 830 board has 63 columns and 50-hole rails). It followed the prompt's
  "columns numbered 1–30" while also demanding 830 points, and silently kept the
  contradiction in the label instead of flagging it the way
  [3.8-default](qwen3.8-27b-default/) did.
- **Interaction is two independent toggles** (click pushbutton for GPIO9, click the LED
  itself for GPIO4) rather than firmware where the LED follows the button; clicking an
  LED to change its own driving GPIO is not physically meaningful.
- **The animated loop closes through the breadboard, not the GND pin.** The dots'
  `animateMotion` path runs 3V3 → GPIO4 → resistor → LED → bottom (−) rail, then jumps
  vertically at x=360 straight up through the breadboard to the top rail and back to the
  3V3 pin — visually shorting the two power rails — instead of the real return path
  (− rail → GND pin → chip). There's also a small zigzag glitch in the mid-section of
  the path.

**Net:** a more polished surface than the original 3.6-nothink output, but the visual
polish actually exposes the underlying inaccuracies (wire collision, 830 label) more
blatantly. Same model family behavior as its sibling run, at ~1.5× the cost.

### Takeaways

- The 3.8 default (thinking on) run is the clear quality winner: it's the only one that
  challenged the prompt's own inconsistencies and verified against data rather than
  self-report. Its extra rigor cost ~12 min and ~37 K more output tokens than the fastest
  run — a fair price given the task explicitly prioritized accuracy.
- Both nothink runs are competitive on feature coverage and much faster; 3.8-nothink is
  the best cost/performance pick. The 3.6 nothink outputs (both the original and the
  presence-penalty-fixed re-run) land last on accuracy for their cost.
- The 3.6 default (thinking on) run spent the most time and thinking tokens but mostly
  reproduced what 3.8-default delivered with less, without catching the breadboard
  contradiction.
- The original-vs-fixed 3.6 nothink pair is a useful data point on sampling
  sensitivity: one missing line (`presence-penalty = 1.5`) cost ~6 min, ~21 K output
  tokens, a larger file, and a different (worse in key details, better in styling)
  page — same model, same prompt, different run.

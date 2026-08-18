# ESP32-C3 LED Circuit Dashboard — Model Comparison

Four agent sessions were given the same prompt — build a single self-contained interactive
HTML page that teaches how current flows through an ESP32-C3-DevKitM-1 wired to a breadboard,
a pushbutton, and an LED. The prompt originated from
[andrisgauracs/circuit-dashboard gist](https://gist.github.com/andrisgauracs/90613bfae85b3b976c189b491ced06b2)
(see [`prompt.txt`](prompt.txt)).

The qwen3.6-27b no-think run was later **re-run** into [`qwen3.6-27b-nothink-fixed/`](qwen3.6-27b-nothink-fixed/):
the first run had dropped one provider-recommended preset line (`presence-penalty = 1.5`). The
original `qwen3.6-27b-nothink/` directory is kept as-is for comparison — the only config
difference between the two is that single line (see both `preset.ini` files).

Each subdirectory holds one session's working output plus a full transcript excerpt:

| File | Purpose |
|---|---|
| `index.html` / `dashboard.html` | The generated dashboard (the deliverable). |
| `preset.ini` | The llama-cpp configuration the agent ran under. |
| `session-excerpt.md` | GitHub-readable transcript: timestamps, collapsible thinking / tool calls / tool results, inline screenshots. |
| `session-excerpt.html` | Rendered transcript (same content, styled, collapsible sections). |
| `session-excerpt.jsonl` | Byte-exact verbatim session lines (initial prompt → final completion turn). |
| `images/` | Decoded screenshots the agent took while verifying its output (referenced by both `.md` and `.html`). |
| `preview.png` | A rendered snapshot of the dashboard (only in `qwen3.6-27b-nothink-fixed/`). |

Each excerpt spans the initial prompt through the final (task-complete) assistant turn;
post-completion chatter is excluded.

## Results

| Variant | Gen time | Output tok | Input tok | Cache read | Thinking (chars) | Output file | Size |
|---|---:|---:|---:|---:|---:|---|---:|
| qwen3.8-27b default (thinking on) | 18m 19s | 63,767 | 71,540 | 672,644 | 135,027 | `index.html` | 28 KB |
| qwen3.8-27b no-think | 6m 34s | 26,970 | 25,473 | 1,313,427 | 0 | `index.html` | 24 KB |
| qwen3.6-27b default (thinking on) | 28m 27s | 47,927 | 71,077 | 1,226,135 | 358,940 | `dashboard.html` | 42 KB |
| qwen3.6-27b no-think | 10m 56s | 44,814 | 59,790 | 858,584 | 0 | `dashboard.html` | 51 KB |
| qwen3.6-27b no-think (fixed) | 16m 37s | 65,939 | 44,587 | 2,577,639 | 0 | `index.html` | 54 KB |

The "fixed" row is the re-run with `presence-penalty = 1.5` restored — same model, same
no-think mode, only that one preset line different. It took longer and emitted more output
tokens than the original no-think run, and produced a cleaner result (see below).

Token counts are summed across all assistant turns in the excerpt; "thinking" is the total
character count of reasoning blocks (pi records these as content, not in the usage reasoning
field, so they're reported separately). Cache-read dominance reflects the long in-context
research-then-build workflow.

## Impressions

### qwen3.8-27b default (thinking on) — `index.html`
**Code:** Clean, well-structured static SVG built up with templated HTML + a small JS state
machine. Good separation of wire palette, pin tables (correct 15-pin J1/J3 silkscreen with
per-pin function tooltips), and CSS-keyframe current animation along the real closed loop.
Interactivity is the most flexible of the set: a demo mode auto-couples the button to GPIO4,
plus a manual GPIO4 toggle and press-and-hold via pointer *and* spacebar. Highlights "used"
pins distinctly. **Visuals:** Polished dark theme, legend, status chips (HIGH/LOW, OPEN/CLOSED,
current in mA + pull-up µA). Electrically sound: 220 Ω resistor with correct red-red-brown-gold
bands, ≈6.4 mA (Vf≈1.9 V). The most usable interactive design.

### qwen3.8-27b no-think — `index.html`
**Code:** Compact, JS-generated SVG (smallest source). Breadboard draws individual holes with
column-number/row-letter labels and translucent node highlights showing the 5-hole-per-column
grouping — a nice teaching touch. Single-button toggle (press = LED off) plus spacebar.
**Visuals:** Clean and readable, tooltips on every pin and component, CSS dash-flow animation.
**Accuracy snag:** the resistor is labeled "220 Ω" but the color bands are red-red-**black**-gold
(= 22 Ω) — a real electrical mismatch, even though the stated current (5.9 mA) uses 220 Ω.
Otherwise the tightest, most economical implementation.

### qwen3.6-27b default (thinking on) — `dashboard.html`
**Code:** The most elaborate. Static inline SVG plus JS-generated breadboard holes, and it
animates current *two* ways — CSS keyframe flow **and** `requestAnimationFrame` particle dots
rolling along the path. In-SVG status panel *and* separate HTML controls, with the LED toggle
and pushbutton as independent controls (decoupled, so you can explore states the auto-coupled
variants can't). Extensive tooltips (77 annotated elements). **Visuals:** Cyan-accented dark
theme, busy but informative. Correct 220 Ω bands (red-red-brown-gold), ≈6.8 mA (Vf≈1.8 V).
Highest effort and feature count; the trade-off is the longest generation time by far.

### qwen3.6-27b no-think (original run) — `dashboard.html`
> **Superseded** by the `qwen3.6-27b-nothink-fixed/` re-run; kept here as evidence of the
> effect of the missing `presence-penalty = 1.5` line.

**Code:** Largest output file. Independent LED + switch controls, current animation, a detailed
in-page "how it works" explainer, and correct 220 Ω bands / 5.9 mA. Per-element tooltips and a
clear status panel. **Rough edges:** signs of iterative patching left in — a duplicate/overridden
GPIO4 wire path with "Hide the old GPIO4 wire" / "Override" comments, and a mislabeled
band-comment (the colors themselves are correct). The duplicate `id="wire-gpio4"` is invalid
HTML (two elements share an id). **Visuals:** Functional and legible, slightly less refined than
the 3.8 variants; the cruft doesn't affect runtime behavior.

### qwen3.6-27b no-think (fixed run) — `qwen3.6-27b-nothink-fixed/index.html`
Reviewed fresh, ignoring the original run above.

**Code:** Static inline SVG (752 lines) with a compact JS state machine. All 30 breadboard
columns × a–e / f–j holes are drawn explicitly (verbose but unambiguous), with red `+` / blue
`−` rails, column numbers, and the center gap. Correct 15-pin J1/J3 tables with per-pin
tooltips (`data-name` / `data-info`); the two used pins (IO4, IO9) are highlighted gold.
Components are real symbols: 220 Ω resistor with correct red-red-brown-gold bands, LED with
anode/cathode + glow, pushbutton with a moving actuator. Current flow is animated with SMIL
`<animateMotion>` dots staggered along a `current-path` (several dots, different `begin`
offsets) — a different technique than the CSS-keyframe / `requestAnimationFrame` approaches
elsewhere. Interactivity is decoupled: separate LED toggle (click the LED or its button) and
pushbutton (click or hold spacebar), so you can set states the auto-coupled variants can't.
Sidebar carries a live status panel (GPIO4/GPIO9/LED/button + 5.9 mA), a circuit description,
wire legend, and an active-pin reference. **Cleanliness:** no duplicate ids, no override/dead-wire
cruft — the `presence-penalty = 1.5` addition clearly helped the model stop re-patching its own
earlier decisions. **Visuals:** Dark navy theme, six sidebar panels, `preview.png` (1400×900)
captures the rendered board + breadboard + sidebar. **One accuracy call to flag:** the board
connector is labeled **"Micro-USB"** — this matches the real ESP32-C3-DevKitM-1 v1 hardware
(and the 3.8-default run's researched conclusion), but diverges from the prompt's stated
"USB-C". Defensible on hardware-accuracy grounds, but it's a deliberate departure from the
literal prompt rather than an oversight.

## Takeaways

- **Thinking vs no-think:** thinking mode roughly doubles wall-clock time and produces noticeably
  more reasoning text but doesn't clearly win on final output quality here — the no-think
  variants are competitive (the 3.8 no-think resistor-band slip aside). The biggest quality
  jump is between *models* (3.6 builds the richer dashboards) more than between modes.
- **Small config changes matter:** the qwen3.6-27b no-think re-run differs from the original by
  exactly one preset line (`presence-penalty = 1.5`). That single change turned a messy output
  (duplicate `id="wire-gpio4"`, "Override"/"Hide the old wire" dead code) into clean, valid
  HTML with no cruft — at the cost of ~50% more wall-clock and output tokens. A vivid reminder
  that provider-recommended sampler settings aren't optional decoration.
- **3.8 vs 3.6:** 3.8 favors compact, tidy, single-control designs; 3.6 favors feature density
  (dual animations, decoupled controls, embedded explainers) at the cost of larger, slightly
  rougher code.
- **Common strengths:** all five render the DevKitM-1 with the correct 15-pin J1/J3 tables,
  the ESP32-C3-MINI-1 module outline, BOOT/RESET buttons, onboard RGB LED, a real 830-point
  breadboard layout with labeled rails/columns/rows, distinct component symbols, color-coded
  wires, a live status panel, animated current on the closed path, and hover tooltips.
- **USB connector divergence:** the prompt says "USB-C", but three runs (3.8-default,
  3.6-nothink original, 3.6-nothink fixed) ship **Micro-USB** — which is what the real
  ESP32-C3-DevKitM-1 v1 actually has (3.8-default states this explicitly after research);
  only 3.8-nothink and 3.6-default follow the prompt's "USB-C" wording. So this is a genuine
  prompt-vs-hardware tension rather than a uniform error.
- **Common watch-item:** the resistor color bands are the classic accuracy trap — one variant
  shipped the wrong multiplier band while labeling the value correctly.
# Magic Clock — 3×3 puzzle

A single-file, browser-only demonstrator of a Rubik's-Clock-style puzzle. Open `magic-clock.html` in any modern browser — no build step, no dependencies.

## Design

### Aesthetic direction
Clean, minimal, neutral. The puzzle is the only thing on the page; the UI fades back so the clocks read clearly. Light off-white background by default, with full dark-mode support via `prefers-color-scheme`. Subtle hairline borders, generous whitespace, system sans-serif. Accent colors carry meaning: blue = pin is UP (and that clock is coupled), amber = a clock just moved.

### Color palette

| Token | Light | Dark | Role |
|---|---|---|---|
| `--bg-primary` | `#ffffff` | `#1f1e1c` | Page background |
| `--bg-secondary` | `#f5f4ef` | `#262522` | Clock face, legend card |
| `--bg-tertiary` | `#ebe9e1` | `#2f2e2a` | Pin (down state) |
| `--bg-info` | `#e1ecf7` | `#0c447c` | Clock face when engaged |
| `--bg-warning` | `#faeeda` | `#412402` | Clock face flash on move |
| `--text-primary` | `#1f1e1c` | `#ede9df` | Hand, ticks, body text |
| `--text-info` | `#185fa5` | `#85b7eb` | Pin (up state) |
| `--text-tertiary` | `#888780` | `#888780` | Labels, minor ticks |

### Typography
System sans-serif throughout (`-apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif`) at 13–16px. Two weights: 400 regular, 500 medium. Sentence case everywhere, no all-caps, no decorative serifs — the visual interest sits in the puzzle, not the chrome.

### Layout
A simple vertical stack centered at max 520px:

1. Title + one-line subtitle.
2. The puzzle: a top row of two wheels, the SVG board, a bottom row of two wheels.
3. An actions strip: move counter, Scramble, Reset, status text.
4. A legend card explaining the rules.

Wheels use horizontal `[label] [+1] [−1]` (or mirrored on the right side) and float to the outer edges of the puzzle so they sit visually at each corner of the board. The SVG board is 300×300 in user units and scales fluidly up to 320px.

### Motion
- **Clock hands** tween with a cubic ease-out (~340ms per move), driven by `requestAnimationFrame` updating the SVG `transform` attribute directly. Displayed rotation is tracked separately from the canonical 0–11 hour value, so a hand can wind continuously across many moves rather than snapping back.
- **Scramble** tweens each hand along the shortest arc to the new state over ~680ms.
- **Pin toggle** is a 200ms color transition.
- **Move feedback** flashes the affected clocks amber for ~320ms.

## Game functionality

### Components
- **9 clocks**, each with an integer hour value 0–11. Solved value is 0 (pointing to 12).
- **4 pins** at the interior grid intersections, each UP or DOWN.
- **4 wheels** at the outer corners, each with `+1` and `−1` controls.

### Rules
A wheel always rotates its own corner clock by ±1 hour. If the pin in that wheel's quadrant is UP, the other three clocks in the quadrant rotate along with it. Pins are the clutch; wheels are the motor.

| Pin | Coupled clocks (when UP) |
|---|---|
| TL | 0, 1, 3, 4 |
| TR | 1, 2, 4, 5 |
| BL | 3, 4, 6, 7 |
| BR | 4, 5, 7, 8 |

A clock face is tinted blue whenever it's covered by an UP pin, so you can see at a glance which clocks are currently coupled. The puzzle is solved when every clock reads 0.

### Controls
- **Mouse:** click a pin to toggle; click `+1` or `−1` on a wheel to turn it.
- **Keyboard:** <kbd>1</kbd> <kbd>2</kbd> <kbd>3</kbd> <kbd>4</kbd> toggle pins TL · TR · BL · BR. Pins are also focusable via Tab and toggle on Enter or Space.
- **Buttons:**
  - `Scramble` — applies ~28 random pin-flips and wheel-turns, animates into the scrambled state. Guarantees the puzzle isn't already solved.
  - `Reset` — returns all clocks to 12, all pins DOWN, move count to 0.

### State model

```js
state = {
  clocks: [c0, c1, c2, c3, c4, c5, c6, c7, c8],  // each 0–11
  pins:   { tl: bool, tr: bool, bl: bool, br: bool }
}
displayed = [d0, ..., d8]   // accumulated rotation in degrees
moveCount = integer
```

The split between `state.clocks` (canonical 0–11) and `displayed` (raw degrees) is what lets the hands rotate naturally over many moves rather than snapping back to the same arc each time.

### Module layout (single-file)

```
magic-clock.html
├── <style>     CSS variables (light + dark via prefers-color-scheme), layout, components
├── <body>      header, puzzle (wheels + SVG board + actions), legend
└── <script>    IIFE
    ├── constants:  PIN_CLOCKS, WHEEL_CORNER, CLOCK_POS, PIN_POS
    ├── buildBoard():   generates SVG nodes for clocks and pins
    ├── turn(wheel, dir):    apply a move + animate + bookkeep
    ├── togglePin(name):     flip a pin's state
    ├── scramble() / reset()
    ├── tweenHand(idx, from, to, duration):  rAF-based rotation tween
    ├── updateUI():  syncs pin classes, engaged highlights, status, count
    └── event listeners (clicks, keydown)
```

### Possible extensions
- **Double-sided mode.** In the real Rubik's Clock, both faces share the same physical clocks (mirrored), and pins look opposite from each side. A "flip" button could swap to a back view.
- **Undo / redo.** Easy given the move log — keep a stack of `(wheel, dir, pin-snapshot)` entries.
- **Solver.** With 8 generators in (ℤ/12ℤ)⁹, a meet-in-the-middle search or linear solver could give optimal solutions.
- **Move count target.** Score scrambles by how close the user gets to optimal.

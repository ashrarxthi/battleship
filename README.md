# Battleship — AI vs. Player

A fully playable Battleship game built as a single self-contained HTML file, deployable to GitHub Pages with no build step, no dependencies, and no server required.

**[▶ Play the game](https://ashrarxthi.github.io/battleship/)**
**[GitHub Repo](https://github.com/ashrarxthi/battleship)**

---

## How It Was Built

Built using [Devin](https://cognition.ai), Cognition's autonomous AI software engineer.

Before prompting Devin, I wrote a detailed PRD specifying exactly how the game should work — the AI opponent's logic, edge cases to handle, and a manual QA checklist Devin was expected to pass before the task was considered complete. The quality of the output depends on the quality of the input: vague prompts produce vague code, a precise spec produces precise behavior.

---

## Bugs Found and Fixed

After Devin generated the game, I ran **Devin Review** — Devin's built-in static analysis layer. It identified three bugs that didn't surface during manual gameplay testing. All three were fixed before submission.

---

### Bug 1 — Stale 600ms AI turn timer fires after reset

**Severity:** Medium
**Found by:** Devin Review
**Fixed in:** PR #3

**What it was:**
Each AI turn schedules a 600ms delay before firing. If the player won immediately after an AI turn and clicked "Play Again", the timer was still running. It would fire after the reset, placing a phantom AI shot on the freshly reset player board.

**How it was fixed:**
Stored the timer reference in `aiTurnTimer`. Added `clearTimeout(aiTurnTimer)` at the top of `resetGame()` so any pending AI shot is cancelled on reset.

**Verified:** Playwright confirmed no hit or miss markers on the player board after reset, with `aiTurnTimer` value of 51 pending at time of reset. ✓

---

### Bug 2 — Stale 800ms AI status timer overwrites placement message after reset

**Severity:** Medium
**Found by:** Devin Review
**Fixed in:** PR #3

**What it was:**
A separate 800ms timer updates the status bar to "Your turn — click a cell to fire!" after the AI fires. If the player won and clicked "Play Again" before this timer expired, it would overwrite the new game's status message ("Place your ships to begin.") with the wrong message — misleading the player at the start of a fresh game.

**How it was fixed:**
Stored this timer reference in `aiStatusTimer`. Added `clearTimeout(aiStatusTimer)` inside `resetGame()` alongside the aiTurnTimer clear, ensuring neither stale timer can bleed into a new session.

**Verified:** Playwright confirmed status reads "Place your ships to begin." immediately after reset and remains unchanged after 1 second. ✓

---

### Bug 3 — Missing `setInitialTabindex()` in `resetGame()` breaks keyboard navigation

**Severity:** Low–Medium
**Found by:** Devin Review
**Fixed in:** PR #3

**What it was:**
The initial game setup called `setInitialTabindex()` after `createBoard()`, setting the first cell of each board to `tabindex="0"` and making the grids keyboard-focusable. However, `resetGame()` called `createBoard()` (which resets all cells to `tabindex="-1"`) but never called `setInitialTabindex()` afterward. After clicking "Play Again", no cell had `tabindex="0"`, so keyboard-only users couldn't navigate the grids for the rest of the session.

**How it was fixed:**
Added `setInitialTabindex()` to the `resetGame()` sequence immediately after `attachBoardListeners()`, mirroring the original initialization order.

**Verified:** Playwright confirmed `tabindex="0"` on cell A1 of both boards after reset, with arrow key navigation working correctly. ✓

---

### Why these bugs weren't caught during manual testing

All three are timing and reset-path bugs — they only appear under specific sequences that don't occur in normal gameplay. The timer bugs require winning within milliseconds of an AI turn and immediately clicking Play Again. The tabindex bug requires completing a full game and switching to keyboard navigation in the second game. None appear in a standard playthrough. Static analysis caught what manual testing couldn't.

---

## Manual QA Checklist

| Check | Result |
|---|---|
| All 5 ships placeable and repositionable | ✓ |
| Randomise generates valid non-overlapping placement | ✓ |
| Already-fired cells are unclickable | ✓ |
| AI uses hunt/target logic, not random fire | ✓ |
| Sunk ships highlight distinctly from hits | ✓ |
| Win overlay appears with shot count and accuracy | ✓ |
| Play Again fully resets board and placement | ✓ |
| Mobile layout stacks boards vertically | ✓ |

---

## Game Rules

**Fleet**
| Ship | Size |
|---|---|
| Carrier | 5 |
| Battleship | 4 |
| Cruiser | 3 |
| Submarine | 3 |
| Destroyer | 2 |

**Placement:** Horizontal or vertical only. Ships may not overlap or touch adjacent cells.

**Gameplay:** Player fires first. Click any cell on the AI grid. Turns alternate until one fleet is fully sunk. AI fires after a 600ms delay.

**Win condition:** First to sink all five opponent ships wins.

---

## Tech Stack

- HTML5 / CSS3 / Vanilla JavaScript (ES6+)
- No frameworks, no dependencies, no build step
- Single `index.html` file — runs locally or deploys to GitHub Pages as-is

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

After Devin generated the game, I ran **Devin Review** — Devin's built-in static analysis layer. It identified two bugs that didn't surface during manual gameplay testing. Both were fixed before submission.

---

### Bug 1 — Stale `setTimeout` overwrites status message after quick reset

**Severity:** Medium
**Found by:** Devin Review
**Fixed in:** PR #3

**What it was:**
Each AI turn schedules a 600ms delayed status update guarded by `if (!gameOver)`. If the player won quickly after an AI turn and immediately clicked "Play Again", `resetGame()` set `gameOver = false`. The stale timer then passed the check and overwrote the new game's placement status ("Place your ships to begin.") with "Your turn — click a cell to fire!" — a misleading message at the start of a fresh game.

**How it was fixed:**
Stored the `setTimeout` reference in a variable called `aiTurnTimer`. Added `clearTimeout(aiTurnTimer)` at the top of `resetGame()` so any in-flight timer is cancelled before the game resets.

**Verified:** Playwright test confirmed status reads "Place your ships to begin." immediately after reset and remains unchanged after 1 second. ✓

---

### Bug 2 — Missing `setInitialTabindex()` in `resetGame()` breaks keyboard navigation

**Severity:** Low–Medium
**Found by:** Devin Review
**Fixed in:** PR #3

**What it was:**
The initial game setup called `setInitialTabindex()` after `createBoard()`, setting the first cell of each board to `tabindex="0"` and making the grids keyboard-focusable. However, `resetGame()` called `createBoard()` (which resets all cells to `tabindex="-1"`) but never called `setInitialTabindex()` afterward. After clicking "Play Again", no cell had `tabindex="0"`, so keyboard-only users couldn't navigate the grids for the rest of the session.

**How it was fixed:**
Added `setInitialTabindex()` to the `resetGame()` sequence immediately after `attachBoardListeners()`, mirroring the original initialization order.

**Verified:** Playwright test confirmed `tabindex="0"` on cell A1 of both boards after reset, with arrow key navigation working correctly. ✓

---

### Why these bugs weren't caught during manual testing

Both are timing and reset-path bugs — they only appear under specific sequences that don't happen in normal gameplay. The timer bug requires winning within 800ms of an AI turn and immediately clicking Play Again. The tabindex bug requires completing a full game and using keyboard navigation in the second game. Neither appears in a standard playthrough. Static analysis caught what manual testing couldn't.

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

# Battleship — AI vs. Player

A fully playable Battleship game built as a single self-contained HTML file, deployable to GitHub Pages with no build step, no dependencies, and no server required.

**[▶ Play the game](https://ashrarxthi.github.io/battleship/)**

---

## How It Was Built

This game was built using [Devin](https://cognition.ai), Cognition's autonomous AI software engineer.

Rather than prompting Devin iteratively and debugging the output, I front-loaded the work into a detailed product requirements document before writing a single line of code. The PRD specified not just *what* to build, but the exact edge cases the implementation had to handle, the AI opponent's decision logic, and a manual QA checklist Devin was expected to pass before considering the task complete.

After generation, I ran Devin Review — Devin's built-in static analysis layer — which identified two issues that hadn't surfaced during manual gameplay testing. Both were fixed before final submission. Details in the Bug Log below.

---

## Engineering Decisions

**Single-file architecture**
The entire game — HTML, CSS, and JavaScript — lives in one `index.html` file. This was a deliberate constraint, not a shortcut. It ensures zero deployment friction (GitHub Pages, any static host, or just opening the file locally), no dependency drift, and nothing to install. For a game of this scope, a build pipeline would have added complexity without value.

**AI opponent: hunt/target logic**
The most consequential engineering decision in the game was the AI's firing strategy. A naive implementation fires randomly every turn — technically functional, but immediately obvious to any player. Instead, the AI uses a two-phase hunt/target algorithm:

- **Hunt phase:** The AI fires at random un-fired cells until it scores a hit.
- **Target phase:** Once a hit is registered, the AI queues and systematically checks all four adjacent cells (up, down, left, right), eliminating directions that go out of bounds or were already fired. It continues until the ship is fully sunk.
- **Resume:** After sinking a ship, the AI returns to hunt phase.

This creates a noticeably more competent opponent without requiring any machine learning — just state management and directional logic.

**Preventing invalid game states via spec, not runtime checks**
Common Battleship bugs — AI firing the same cell twice, the player clicking already-fired cells, ships overlapping, game logic continuing after a winner is declared — were addressed in the requirements document rather than discovered during QA. Each edge case was explicitly enumerated as a constraint Devin had to satisfy, which meant the implementation was built with those guard rails from the start rather than patched in after the fact.

This approach reflects how I think about AI-assisted development: the quality of the output is largely determined by the quality of the input. Vague prompts produce vague code. A precise spec produces precise behavior.

**Accessibility as a first-class requirement**
Grid cells carry `aria-label` attributes (e.g. "B7 - miss") and the board is keyboard navigable via arrow keys and Enter. Color is never the sole indicator of state — hits show a red ✕ symbol, misses a grey dot — so the game is playable regardless of color perception. This was specified upfront, not retrofitted.

---

## QA Process

QA ran in two layers: manual gameplay verification against the original requirements checklist, followed by Devin Review — Devin's static analysis pass that inspects the generated code for logic errors, race conditions, and edge cases that don't surface during normal play.

**Manual QA checklist**

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

Manual testing passed. Devin Review then identified two issues in the underlying code — both fixed before submission.

---

## Bug Log

### Bug 1 — Stale `setTimeout` overwrites status message after rapid reset
**Severity:** Medium
**Found by:** Devin Review (static analysis)
**File:** `index.html`, lines 807–809 and 857

**What it was:** Each AI turn schedules a delayed status update (600ms) guarded by an `if (!gameOver)` check. If the player fired the winning shot quickly after an AI turn and then immediately clicked "Play Again", `resetGame()` set `gameOver = false` at line 857. The stale `setTimeout` callback then passed the `!gameOver` check — which now evaluated true again — and overwrote the ship placement status ("Place your ships to begin.") with "Your turn — click a cell on the enemy board to fire!" This produced a misleading UI state at the start of a new game. The timeout reference was never stored or cleared on reset.

**Fix:** Stored the `setTimeout` reference in a variable (`aiTurnTimer`) and called `clearTimeout(aiTurnTimer)` at the top of `resetGame()`, ensuring no in-flight callbacks could bleed into a new game session.

---

### Bug 2 — Missing `setInitialTabindex()` call in `resetGame()` breaks keyboard navigation after first game
**Severity:** Low–Medium
**Found by:** Devin Review (static analysis)
**File:** `index.html`, lines 880–883 and 1022–1027

**What it was:** The initial setup sequence correctly called `setInitialTabindex()` after `createBoard()`, which set the first cell of each board to `tabindex="0"` and made the grids keyboard-focusable. However, `resetGame()` called `createBoard()` (which resets all cells to `tabindex="-1"`), `attachBoardListeners()`, and `initPlacement()` — but never called `setInitialTabindex()`. After clicking "Play Again", no cell on either board had `tabindex="0"`, making the grids unreachable via keyboard navigation for the remainder of the session.

**Fix:** Added `setInitialTabindex()` to the `resetGame()` call sequence, immediately after `attachBoardListeners()`, mirroring the original initialization order.

---

### Why these bugs weren't caught during manual testing

Both issues are state-timing and reset-path bugs — they only manifest under specific interaction sequences that don't occur in normal linear gameplay. The stale timeout requires firing a winning shot within 800ms of an AI turn and immediately clicking "Play Again." The tabindex bug requires completing a full game and starting a second one while using keyboard navigation. Neither would appear in a standard playthrough, which is precisely why static analysis is valuable alongside manual QA. Devin Review caught what gameplay testing couldn't.

---

## Game Rules

Standard Battleship ruleset:

**Fleet**
| Ship | Size |
|---|---|
| Carrier | 5 |
| Battleship | 4 |
| Cruiser | 3 |
| Submarine | 3 |
| Destroyer | 2 |

**Placement**
Ships may be placed horizontally or vertically. Ships may not overlap or occupy adjacent cells.

**Gameplay**
Player fires first. Click any cell on the AI's grid to fire. Turns alternate until one side's fleet is fully sunk. The AI fires 600ms after the player's turn to maintain game flow and give the player time to register the result.

**Win condition**
First player to sink all five of the opponent's ships wins.

---

## Tech Stack

- HTML5 / CSS3 / Vanilla JavaScript (ES6+)
- No frameworks, no dependencies, no build step
- Deployable to GitHub Pages as-is

---

## Repository Structure

```
/
└── index.html    # Complete game — HTML, CSS, and JS in one file
└── README.md     # This document
```

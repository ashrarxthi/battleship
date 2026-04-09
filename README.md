# Battleship — AI vs. Player

A fully playable Battleship game built as a single self-contained HTML file, deployable to GitHub Pages with no build step, no dependencies, and no server required.

**[▶ Play the game](YOUR_GITHUB_PAGES_LINK_HERE)**

---

## How It Was Built

This game was built using [Devin](https://cognition.ai), Cognition's autonomous AI software engineer.

Rather than prompting Devin iteratively and debugging the output, I front-loaded the work into a detailed product requirements document before writing a single line of code. The PRD specified not just *what* to build, but the exact edge cases the implementation had to handle, the AI opponent's decision logic, and a manual QA checklist Devin was expected to pass before considering the task complete.

The result: the game passed all QA checks on the first build with no post-generation debugging required.

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

Before submitting, I manually verified the following against the original requirements:

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

All checks passed on the first build. No bugs were found during QA.

The absence of bugs wasn't luck — it was a consequence of writing requirements that were specific enough to eliminate ambiguity before Devin began generating code. Edge cases that typically surface during testing (double-firing, invalid ship placement, premature game-over triggers) were treated as specification constraints rather than implementation afterthoughts.

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

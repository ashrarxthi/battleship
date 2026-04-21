# Battleship — AI vs. Player

A fully playable Battleship game built as a single self-contained HTML file, deployable to GitHub Pages with no build step, no dependencies, and no server required.

**[Play the game](https://ashrarxthi.github.io/battleship/)**

---

## How It Was Built

Built using [Devin](https://cognition.ai), Cognition's autonomous AI software engineer.

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

**End game:** Click the End Game button at any time to resign the current game.

---

## Tech Stack

- HTML5 / CSS3 / Vanilla JavaScript (ES6+)
- No frameworks, no dependencies, no build step
- Single `index.html` file, runs locally or deploys to GitHub Pages as-is

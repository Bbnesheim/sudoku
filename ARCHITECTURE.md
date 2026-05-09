# The Garden: Architecture

## Overview

The Garden is a progressive web app (PWA) puzzle hub built with vanilla HTML/CSS/JavaScript. It runs on any modern browser and can be installed on phones like a native app.

## File Structure

```
sudoku/
├── index.html              # Home page (puzzle hub)
├── sudoku.html             # Sudoku game
├── wordle.html             # Wordle game (coming soon)
├── words.txt               # Wordle word list
├── README.md               # User-facing documentation
├── AGENT_SPEC.md           # Complete development specification
├── ARCHITECTURE.md         # This file
├── CONTRIBUTING.md         # Code contribution guidelines
├── .claude.md              # Oz configuration
├── assets/
│   ├── icons/
│   │   └── sprite.svg      # SVG symbol sprite for all UI icons
│   ├── icon-play.svg       # Play button icon (source)
│   └── icon-pause.svg      # Pause button icon (source)
├── .github/
│   └── workflows/          # GitHub Actions (to be created)
└── .git/                   # Git repository
```

## Home Page (index.html)

The hub page that displays all available games.

### Structure
- **Header** — Branding, leaderboard button, settings button, theme toggle
- **Tagline** — "Pick a puzzle, take a break 🌿"
- **Games section** — Card-based layout with Sudoku and Wordle entries
- **Stats display** — Best times and win percentages from localStorage

### Key Elements
- Game cards are styled as interactive links with icons, descriptions, and personal stats
- Theme toggle switches between light and dark modes
- Settings and leaderboard buttons are placeholders for future modals
- Play icons are sourced from the SVG sprite

### Scripts
- Theme persistence (localStorage)
- Service worker registration for offline support
- Statistics display (reads from localStorage)
- Manifest generation for PWA installation

## Sudoku Game (sudoku.html)

A full-featured Sudoku solver with scoring, hints, and multiple difficulty levels.

### Game State (Global `G` object)
- `board` — 81-cell array representing the 9×9 grid
- `solution` — Solution board (generated on new game)
- `selected` — Currently selected cell index
- `won` — Boolean, true when puzzle is complete
- `paused` — Boolean, true when game is paused
- `timerRunning` — Boolean, game timer state
- `time` — Elapsed seconds
- `errors` — Count of invalid entries
- `hints` — Number of hints used
- `reveals` — Number of cells revealed
- `combo` — Current combo multiplier (0–10)
- `comboBonusPool` — Accumulated combo points
- `completionBonusPool` — Points from row/col/box/number completion
- `difficulty` — Current difficulty name (Easy, Medium, Hard, etc.)
- `diffIdx` — Difficulty index (0–5)

### Difficulty Levels
| Index | Name | Clues | Par (sec) |
|-------|------|-------|-----------|
| 0 | Easy | 46 | 240 |
| 1 | Medium | 38 | 420 |
| 2 | Hard | 32 | 660 |
| 3 | Expert | 28 | 1080 |
| 4 | Extreme | 24 | 1800 |
| 5 | Nightmare | 20 | 2700 |

### Scoring System
- Base score depends on difficulty
- Time bonus: points awarded for finishing below the par time
- Penalties: deducted for hints, reveals, and errors
- Combos: multiplier that increases with consecutive correct inputs (10s window)
- Completion bonuses: awarded for completing rows, columns, boxes, and number sets

See AGENT_SPEC.md Phase 1 for exact formulas.

### UI Components
- **Board** — 9×9 grid with cell selection and highlighting
  - Blue tint: selected cell's row, column, and 3×3 box
  - Amber/gold: other cells with the same number as selected cell
  - Red text: numbers that conflict with others in their house
- **Numpad** — 1–9 buttons for number entry
- **Control buttons** — New, Undo, Erase, Notes, Hint, Reveal
- **Timer** — Displays elapsed time, paused during modals
- **Status bar** — Shows errors, current combo, and other feedback
- **Modals** — New Game, Settings, How to Play, Help

### Features
- **Undo** — Unlimited undo steps
- **Hint** — Fills a cell with the correct number, or a random empty cell
- **Reveal** — Shows a cell without deducting from hints
- **Notes mode (Pencil)** — Enter candidate numbers (small digits)
- **Auto-note removal** — Removes notes from a cell when that number is placed elsewhere
- **Conflict highlighting** — Shows invalid entries in red
- **Dark mode** — Full dark theme support
- **Keyboard shortcuts** — 1–9 (number), 0/Delete/Backspace (erase), Arrow keys (move), P (notes), H (hint), Ctrl+Z (undo)

## Wordle Game (wordle.html)

A Wordle clone (coming soon). Structure mirrors sudoku.html but with different game logic.

## Icons (assets/icons/sprite.svg)

An SVG symbol sprite containing all UI icons. Each icon is defined as a `<symbol>` with a unique ID and 24×24 viewBox.

### Available Icons
- `icon-home` — House outline
- `icon-refresh` — Clockwise circular arrow (new game)
- `icon-help` — Circle with question mark
- `icon-chart` — Three ascending bars (statistics)
- `icon-sun` — Circle with 8 rays (light mode)
- `icon-moon` — Crescent moon (dark mode)
- `icon-settings` — Gear with 6 teeth
- `icon-pause` — Two vertical bars
- `icon-play` — Right-pointing triangle
- `icon-undo` — Arrow curving left
- `icon-erase` — Rectangle with notch (backspace)
- `icon-pencil` — Diagonal pencil
- `icon-lightbulb` — Lightbulb outline (hint)
- `icon-eye` — Eye outline (reveal)
- `icon-trophy` — Trophy cup (leaderboard)
- `icon-user` — Person silhouette (profile)
- `icon-close` — × mark
- `icon-back` — Left chevron

Icons are styled with `fill="currentColor"` or `stroke="currentColor"` to inherit button text color and respond to dark mode.

## Data Storage (localStorage)

All user data is stored locally in the browser. Keys use the `kg_` prefix (Karen's Garden).

### Current Keys
- `kg_theme` — "light" or "dark"
- `kg_sudoku_best` — Best scores per difficulty (being migrated)
- `kg_sudoku_stats` — Game history (being migrated)

### Future Schema (Phase 5+)
- `kg_user` — User identity (username, UUID, createdAt)
- `kg_runs` — Array of completed game runs (append-only, capped at 500)
- `kg_best` — Derived best scores per difficulty (for fast lookup)

See AGENT_SPEC.md Phase 4 for complete schema.

## PWA & Service Worker

index.html includes an inline service worker that:
- Caches all game files on first visit
- Serves cached files when offline
- Clears old caches when a new version is deployed
- Cache version: `kg-v4`

Files cached:
- index.html
- sudoku.html
- wordle.html
- words.txt

## Dark Mode

Dark mode is controlled by the `data-theme` attribute on `<html>`. CSS variables switch based on theme:

```css
:root {
  --bg: #f5f5f5;
  --surface: #ffffff;
  --text: #1a1a1a;
  --text-muted: #888;
  --border: #e0e0e8;
  --accent: #e94560;
  --btn-bg: #eeeef6;
  --btn-hover: #e2e2ee;
}

[data-theme="dark"] {
  --bg: #1a1a2e;
  --surface: #1e1e40;
  --text: #e8e8f2;
  --text-muted: #9090b0;
  --border: #50508a;
  --btn-bg: #252550;
  --btn-hover: #383878;
}
```

## Responsive Design

- **Mobile-first:** 320px–480px primary target
- **Tablet support:** Graceful scaling to 768px+ (max-width guards prevent excessive stretching)
- **Units:** Use `dvh` (dynamic viewport height) for mobile, relative units for text
- **Grid system:** CSS Grid for the 9×9 Sudoku board; Flexbox for UI layouts

## Future: Capacitor Wrapper (Phase 7+)

Once core game is stable:
1. Add `package.json` and Capacitor configuration
2. Build Android app with Google Play integration
3. Build iOS app with App Store integration
4. Connect to AdMob for monetization (banner ads at bottom of game)

Capacitor packages to be added:
- `@capacitor/core`, `@capacitor/cli`, `@capacitor/android`, `@capacitor/ios`
- `@capacitor/splash-screen`, `@capacitor/status-bar`, `@capacitor/app`
- `@capacitor-community/admob` (ads)

## Future: Supabase Backend (Phase 6+)

For leaderboard sync:
1. Create Supabase project (EU region)
2. Create `users` and `runs` tables
3. Add sync function to post-game completion
4. Display global leaderboard on home page

See AGENT_SPEC.md Phase 6 for SQL and setup details.

## Development Workflow

1. Make changes on the `production` branch
2. Test in browser (local or via `python3 -m http.server 8080`)
3. Commit with `git commit -m "message"`
4. Push to origin: `git push origin production`
5. GitHub Pages will auto-deploy to the live site
6. When ready for production launch, merge `production` → `main`

See CONTRIBUTING.md for more details.

# The Garden: Sudoku & Puzzles — Overview

> **Version:** 1.2  
> **Last updated:** 2026-05-09  
> **Status:** Development (on production branch)

---

## App Identity

| Field | Value |
|---|---|
| App name | The Garden: Sudoku & Puzzles |
| Short name | The Garden |
| Primary game | Sudoku |
| Platform targets | Android (primary), iOS (secondary) |
| Tech stack | Vanilla HTML/CSS/JS → Capacitor wrapper |
| Backend | Supabase (planned Phase 6) |
| Ad strategy | Banner slot reserved at bottom of game screen; no interstitials mid-game |

---

## Overview

**The Garden** is a clean, focused puzzle game experience with:
- **6 difficulty levels** — Easy through Nightmare
- **Scoring system** — Points, combos, completion bonuses
- **Local leaderboard** — Track personal bests
- **Dark mode** — Full theme support
- **PWA support** — Installable on phones like a native app
- **Offline play** — Works without internet via service worker

The game is built as a **single-file HTML application** with no build step, making it easy to deploy and maintain.

---

## Project Structure

```
sudoku/
├── index.html              # Home hub (game picker)
├── sudoku.html             # Sudoku game
├── wordle.html             # Wordle (coming soon)
├── words.txt               # Wordle word list
├── README.md               # User documentation
├── AGENT_SPEC.md           # This file (overview)
├── ARCHITECTURE.md         # Technical architecture reference
├── CONTRIBUTING.md         # Development guidelines
├── .claude.md              # Oz agent configuration
├── assets/
│   ├── icons/
│   │   └── sprite.svg      # 18+ SVG icons for UI
│   ├── icon-play.svg       # Play icon (source)
│   └── icon-pause.svg      # Pause icon (source)
└── docs/
    └── PHASES.md           # Complete phase specifications (1-10)
```

---

## Key Documentation

### For Development

- **docs/PHASES.md** — Complete specification for all 10 development phases. Start here when implementing a feature.
- **CONTRIBUTING.md** — How to work on the project, code style, testing, deployment.
- **ARCHITECTURE.md** — Technical details: game state, scoring, UI components, localStorage schema, PWA setup.
- **.claude.md** — Oz agent configuration and project conventions.

### For Users

- **README.md** — How to play, keyboard shortcuts, installation, troubleshooting.

---

## Development Phases (1–10)

All phases are documented in `docs/PHASES.md`. Quick reference:

| Phase | Focus | Status |
|-------|-------|--------|
| **1** | Difficulty & scoring (base formulas, combos, bonuses) | Not started |
| **2** | Icon system (SVG sprite) | ✅ In progress |
| **3** | UI & navigation (home page, modals, game layout) | ✅ In progress |
| **4** | Statistics & leaderboard (local UI) | Not started |
| **5** | Settings & first-run (username, clear history) | Not started |
| **6** | Supabase backend (setup, sync) | Not started |
| **7** | Capacitor wrapper (Android/iOS) | Not started |
| **8** | GitHub Actions (CI/CD) | Not started |
| **9** | Store submission (assets, checklists) | Not started |
| **10** | QA before launch | Not started |

**Execution order:** 1 → 5 → 3 → 2 → 4 → 6 → 7 → 8 → 9 → 10

See `docs/PHASES.md` for detailed specs, code examples, and requirements for each phase.

---

## Branch Strategy

- **production** — Active development branch. All work happens here until launch.
- **main** — Stable release branch. Merged to only when ready for production launch.

Always commit to `production` unless explicitly told otherwise.

---

## Quick Start for Developers

1. **Read** `.claude.md` for project conventions and task management
2. **Check** `docs/PHASES.md` for the phase you're implementing
3. **Review** `ARCHITECTURE.md` for technical context (game state, scoring, UI structure)
4. **Follow** `CONTRIBUTING.md` for code style and deployment
5. **Code** on the `production` branch
6. **Test** locally before pushing

---

## Key Decisions & Rationale

### Why No Build Step?

Vanilla HTML/CSS/JS with inline service worker. This keeps the codebase simple, fast to load, and easy to deploy. No npm dependencies means no supply chain risk.

### Why Custom SVG Icons?

Lucide and Phosphor don't fit the garden/nature aesthetic. Custom SVG sprite gives us full control and matches the design vision.

### Why Vanilla JavaScript?

No React, Vue, or other frameworks needed. The game state is simple enough that vanilla JS with a global `G` object is clear and maintainable.

### Scoring Design

The scoring system rewards speed (time bonus), penalizes mistakes (errors, hints, reveals), and rewards play quality (combos, row/col/box/number completion bonuses). This encourages skillful, engaged play.

### Dark Mode

Full support from day one. Uses CSS variables (`--bg`, `--text`, etc.) that switch based on `data-theme` attribute. Persisted in localStorage.

---

## Future Roadmap

### Post-Launch

1. **Global leaderboard** — Supabase-powered global rankings (Phase 6)
2. **More games** — Wordle, other puzzles (placeholder in UI)
3. **Achievements** — Badges for milestones
4. **Themes** — Alternative color schemes
5. **Multiplayer** — Compete with friends (optional)

---

## Questions?

- **How do I implement Phase X?** → See `docs/PHASES.md`
- **What's the code structure?** → See `ARCHITECTURE.md`
- **What are the code style rules?** → See `CONTRIBUTING.md`
- **How do I deploy?** → See `CONTRIBUTING.md` → "Deploying Changes"
- **How do I set up locally?** → See `CONTRIBUTING.md` → "Local Development"

---

## Status & Contact

- **Current branch:** production
- **Live site:** https://bbnesheim.github.io/sudoku/
- **Repository:** https://github.com/Bbnesheim/sudoku

Last updated: 2026-05-09

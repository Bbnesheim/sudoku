# WARP Agent Initialization

**READ THIS FIRST** — This file initializes the Oz agent at conversation start.

## Immediate Actions

1. **Read `.claude.md`** for workflow, branch strategy, and conventions
2. **Read `AGENT_SPEC.md`** for project overview, phases, and current status
3. **Read `docs/PHASES.md`** (if working on a specific phase)
4. **Review `ARCHITECTURE.md`** for technical context when implementing features
5. **Check `CONTRIBUTING.md`** for code style and deployment guidelines

## Quick Context

- **Project:** The Garden: Sudoku & Puzzles
- **Current branch:** production (all work here)
- **Tech:** Vanilla HTML/CSS/JS, PWA, localStorage, Capacitor (Phase 7+), Supabase (Phase 6+)
- **Status:** Phases 1–5 in development (Difficulty/Scoring/UI/Settings)
- **Execution order:** 1 → 5 → 3 → 2 → 4 → 6 → 7 → 8 → 9 → 10

## File References

- `.claude.md` — Oz configuration, branch strategy, task management
- `AGENT_SPEC.md` — App identity, structure, phase overview
- `docs/PHASES.md` — Detailed specs for all 10 development phases
- `ARCHITECTURE.md` — Game state, scoring, UI structure, localStorage schema
- `CONTRIBUTING.md` — Code style, testing, deployment
- `README.md` — User-facing documentation
- `PROGRESS.md` — Development progress log

## Key Files

- `index.html` — Home page hub
- `sudoku.html` — Main game file
- `wordle.html` — Wordle game (coming soon)
- `assets/icons/sprite.svg` — Icon sprite
- `words.txt` — Wordle word list

## Branch Strategy

- **production** — Active development (commit here by default)
- **main** — Stable release (merge only when ready for launch)

## When Starting Work

1. Identify the task/phase
2. Check `docs/PHASES.md` for detailed requirements
3. Review `ARCHITECTURE.md` for relevant technical context
4. Follow `CONTRIBUTING.md` code style guidelines
5. Commit to `production` with co-author line: `Co-Authored-By: Oz <oz-agent@warp.dev>`

---

**This file should be the first thing read at conversation start.**

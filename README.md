# Sudoku PWA

A fully-featured Sudoku game that runs in any browser and can be installed on your phone like a native app. Single `index.html` file, no build step, no app store needed.

## Play it now

**https://bbnesheim.github.io/sudoku/**

> If the link shows a 404, the repo owner needs to enable GitHub Pages once:  
> GitHub repo → **Settings** → **Pages** → Source: **Deploy from branch** → Branch: **gh-pages** → **Save**

---

## Install on your phone (PWA)

### iPhone / iPad (Safari)
1. Open the link above in **Safari** (must be Safari, not Chrome)
2. Tap the **Share** button (box with arrow at the bottom)
3. Scroll down and tap **"Add to Home Screen"**
4. Tap **Add** — the game now appears as an app icon on your home screen

### Android (Chrome)
1. Open the link above in **Chrome**
2. Tap the **⋮ menu** (top right)
3. Tap **"Add to Home screen"** or **"Install app"**
4. Tap **Install** — done

Once installed it opens full-screen with no browser chrome, just like a native app.

---

## How to play

| Action | How |
|--------|-----|
| Select a cell | Tap it |
| Enter a number | Select a cell, then tap a number on the pad |
| Erase a cell | Select it, tap **Erase** or the ⌫ button |
| Pencil / notes | Tap **Notes** to toggle — numbers go in as small candidates |
| Undo | Tap **Undo** (unlimited steps) |
| Hint | Tap **Hint** — fills the selected cell (or a random empty one) |
| New game | Tap **New**, pick a difficulty, tap **Start** |
| Light/dark mode | Tap 🌙 / ☀️ in the top right |
| Settings | Tap ⚙️ — toggle conflict highlighting, same-number glow, auto-note removal |

### Highlighting explained
- **Blue tint** — the row, column, and 3×3 box of the selected cell
- **Amber/gold** — all other cells that contain the same number as the selected cell
- **Red text** — a number that conflicts with another in its row, column, or box

### Difficulty levels
| Level | Approx. clues | Notes |
|-------|--------------|-------|
| Baby | ~50 | Extra cells pre-filled from solution |
| Easy | ~36 | Standard easy generation |
| Medium | ~32 | Default starting difficulty |
| Hard | ~27 | Standard hard generation |
| Expert | ~24 | Hard + 3 extra cells removed |
| Extreme | ~21 | Hard + 6 extra cells removed |

---

## Keyboard shortcuts (desktop)

| Key | Action |
|-----|--------|
| `1`–`9` | Enter number in selected cell |
| `0` / `Delete` / `Backspace` | Erase selected cell |
| Arrow keys | Move selection |
| `P` | Toggle pencil/notes mode |
| `H` | Hint |
| `Ctrl+Z` | Undo |

---

## Run locally

No server needed for basic play — just open the file:

```bash
# Clone the repo
git clone https://github.com/Bbnesheim/sudoku.git
cd sudoku

# Option A: open directly (works in most browsers)
open index.html

# Option B: serve locally (required for PWA install prompt)
python3 -m http.server 8080
# then open http://localhost:8080
```

To access from a phone on the same Wi-Fi:
```bash
# Find your machine's local IP
hostname -I   # Linux
ipconfig      # Windows

# Then open on your phone:
# http://<your-ip>:8080
```

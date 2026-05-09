# The Garden: Development Phases (1–10)

This document contains the complete specification for all development phases. Each phase builds on previous ones and follows the execution order below.

## Execution Order

Run phases in this order to avoid dependency issues:

1. **Phase 1** (scoring, combos, difficulty) — pure JS logic, no dependencies
2. **Phase 5** (username, settings) — adds localStorage schema used by all other phases
3. **Phase 3** (UI/navigation, icon placeholders) — use emoji placeholders until Phase 2 done
4. **Phase 2** (icon SVG sprite) — drop in and replace placeholders
5. **Phase 4** (stats/leaderboard UI) — depends on new localStorage schema from Phase 5
6. **Phase 6** (Supabase setup) — manual step, then add `syncRunToSupabase` stub
7. **Phase 7** (Capacitor) — wraps completed web app
8. **Phase 8** (GitHub Actions) — set up after Capacitor builds locally
9. **Phase 9** (store assets & submission) — final step
10. **Phase 10** (QA) — before each store submission

---

## Phase 1 — Difficulty & Scoring Overhaul

### 1.1 Difficulty Levels

Remove **Baby** level. Rename levels and adjust clue counts:

| Index | Name | Clues | Par (sec) |
|---|---|---|---|
| 0 | Easy | 46 | 240 |
| 1 | Medium | 38 | 420 |
| 2 | Hard | 32 | 660 |
| 3 | Expert | 28 | 1080 |
| 4 | Extreme | 24 | 1800 |
| 5 | Nightmare | 20 | 2700 |

**Rationale for Nightmare:** replaces the Baby slot in the grid, gives experienced players a meaningful ceiling. At 20 clues the puzzle is at the edge of human-solvable without guessing — test generation stability before shipping. If generation takes >5 seconds on mobile, cap at 21 clues.

**Default difficulty on new install:** Hard (index 2).

### 1.2 Scoring Formula

```javascript
// Constants per difficulty (index matches table above)
const SCORE_CONFIG = [
  { base: 10000,  par: 240,  min: 500,   reveal: 1500, hint: 750,  error: 300  }, // Easy
  { base: 20000,  par: 420,  min: 1000,  reveal: 2500, hint: 1250, error: 500  }, // Medium
  { base: 35000,  par: 660,  min: 2000,  reveal: 4000, hint: 2000, error: 700  }, // Hard
  { base: 60000,  par: 1080, min: 5000,  reveal: 6000, hint: 3000, error: 1000 }, // Expert
  { base: 100000, par: 1800, min: 10000, reveal: 9000, hint: 4500, error: 1500 }, // Extreme
  { base: 160000, par: 2700, min: 15000, reveal: 12000,hint: 6000, error: 2000 }, // Nightmare
]

function calcScore(diffIdx, seconds, errors, hints, reveals, comboBonusPool, completionBonusPool) {
  const cfg = SCORE_CONFIG[diffIdx]
  const timeBonus = Math.max(0, (cfg.par - seconds) / cfg.par)
  const base = Math.round(cfg.base * (1 + timeBonus))
  const penalties = (reveals * cfg.reveal) + (hints * cfg.hint) + (errors * cfg.error)
  const raw = base + comboBonusPool + completionBonusPool - penalties
  return Math.max(cfg.min, raw)
}
```

**Key guarantee:** A run with 0 errors will always outscore an identical-time run with errors. A slower clean run will outscore a faster run with errors up to the crossover point defined by the penalty constants — verify this with unit tests before shipping.

### 1.3 Combo System

```javascript
// State additions
G.combo = 0
G.lastInputTime = 0
G.comboBonusPool = 0

// Call on every CORRECT user input (not reveals/hints)
function registerCombo() {
  const now = Date.now()
  const COMBO_WINDOW_MS = 10000
  if (now - G.lastInputTime < COMBO_WINDOW_MS) {
    G.combo = Math.min(G.combo + 1, 10)
  } else {
    G.combo = 1
  }
  G.lastInputTime = now
  const bonus = G.combo * 10  // 10 pts per combo level per input
  G.comboBonusPool += bonus
  if (G.combo >= 3) showToast(`Combo ×${G.combo}! +${bonus}`)
}

// Reset combo on error input
function breakCombo() {
  G.combo = 0
}
```

### 1.4 Completion Bonuses

```javascript
// State addition
G.completionBonusPool = 0

// After each correct input, check:
function checkCompletions(cellIndex, value) {
  const row = (cellIndex / 9) | 0
  const col = cellIndex % 9
  const boxIdx = box(cellIndex)

  // Row complete?
  if (isRowComplete(row)) {
    G.completionBonusPool += 500
    showToast('Row complete! +500')
  }
  // Column complete?
  if (isColComplete(col)) {
    G.completionBonusPool += 500
    showToast('Column complete! +500')
  }
  // Box complete?
  if (isBoxComplete(boxIdx)) {
    G.completionBonusPool += 750
    showToast('Box complete! +750')
  }
  // Number complete (all 9 of this digit placed)?
  if (isNumberComplete(value)) {
    G.completionBonusPool += 1000
    showToast(`All ${value}s placed! +1000`)
  }
}

function isRowComplete(row) {
  return Array.from({length: 9}, (_, i) => G.board[row * 9 + i]).every(c => c.value > 0 && !hasConflict(...))
}
// isColComplete, isBoxComplete, isNumberComplete follow same pattern
```

**Important:** Only award each completion bonus once. Track with `G.completedRows = new Set()`, etc. Check the Set before awarding and adding to it after.

---

## Phase 2 — Icon System

### 2.1 Custom SVG Sprite

Custom SVG set built specifically for this app (no Lucide/Phosphor — they don't fit the garden aesthetic).

**Implementation:**
1. Create `assets/icons/sprite.svg` as an SVG symbol sprite
2. Reference icons via `<use href="assets/icons/sprite.svg#icon-name">`
3. Style via CSS `fill: currentColor` so icons inherit button text color and respond to theme

**Icons needed:**

| ID | Usage | Design |
|---|---|---|
| `icon-home` | Home button | Simple house outline |
| `icon-refresh` | New game | Circular arrow, clockwise |
| `icon-help` | How to play | Circle with ? inside |
| `icon-chart` | Statistics | 3 ascending bars |
| `icon-sun` | Light mode | Circle + 8 rays |
| `icon-moon` | Dark mode | Crescent moon |
| `icon-settings` | Settings | Gear / cog, 6 teeth |
| `icon-pause` | Pause game | Two vertical bars |
| `icon-play` | Resume game | Right-pointing triangle |
| `icon-undo` | Undo | Arrow curving left |
| `icon-erase` | Erase cell | Rectangle with left-pointing notch |
| `icon-pencil` | Notes mode | Diagonal pencil |
| `icon-lightbulb` | Hint | Lightbulb outline |
| `icon-eye` | Reveal cell | Eye outline |
| `icon-trophy` | Leaderboard | Trophy cup |
| `icon-user` | Username/profile | Person silhouette |
| `icon-close` | Close modal | × mark |
| `icon-back` | Navigate back | Left chevron |

**App icon (store asset):** Separate task. Create 1024×1024 SVG with pink/rose background `#e94560`, rounded square shape, white stylized flower/leaf centered. Export to PNG at: 1024, 512, 192, 144, 96, 72, 48px. Android adaptive: provide foreground layer (icon on transparent) + background layer (solid `#e94560`).

---

## Phase 3 — UI & Navigation

### 3.1 Home Page (index.html) Redesign

**Layout (top to bottom):**
1. App name header + theme toggle + leaderboard button
2. Hero Sudoku card — large square, tappable, navigates to sudoku.html
3. Horizontal carousel (overflow-x scroll, CSS scroll snap) — future games
4. Wordle card in carousel: grayed out, `pointer-events: none`, "Coming Soon" badge, `opacity: 0.45`
5. Reserved ad slot `<div id="ad-slot-bottom">` — 50px height, full width, transparent background

### 3.2 Leaderboard Button on Home Page

Add to header alongside theme toggle:

```html
<button class="icon-btn" id="leaderboardBtn" aria-label="Leaderboard">
  <svg>...<use href="assets/icons/sprite.svg#icon-trophy"/></svg>
</button>
```

### 3.3 Sudoku Game Page Layout

Add `<div id="ad-slot-bottom">` as last child of `<body>`, outside all game UI. CSS:

```css
#ad-slot-bottom {
  width: 100%;
  max-width: 480px;
  height: 50px;
  flex-shrink: 0;
  background: transparent;
}
```

The existing `height: 100dvh; overflow: hidden` layout must account for this 50px. Adjust the tile size calc:

```css
/* Was: 100dvh - 262px */
/* Now: 100dvh - 312px  (extra 50px for ad slot) */
--ts: min(56px, calc((100dvh - 312px) / 6), calc((100vw - 48px) / 5));
```

### 3.4 Hint Controls — Remove Dropdown

Replace the hint dropdown with two flat control buttons in the controls row:

```html
<button class="ctrl-btn" id="hintBtn">
  <svg>...<use href="assets/icons/sprite.svg#icon-lightbulb"/></svg>
  Hint
</button>
<button class="ctrl-btn" id="revealBtn">
  <svg>...<use href="assets/icons/sprite.svg#icon-eye"/></svg>
  Reveal
</button>
```

Remove: `hintMenu`, `hintRandom`, `hintReveal`, `hintCheck`, all dropdown JS, `closeHintMenu()`.

Controls row now has 6 buttons: New · Undo · Erase · Notes · Hint · Reveal

### 3.5 Timer Pause on Modal Open

```javascript
let _modalPauseActive = false

function pauseForModal() {
  if (!G.won && !G_paused && G.solution) {
    stopTimer()
    _modalPauseActive = true
  }
}

function resumeAfterModal() {
  if (_modalPauseActive && !G_paused) {
    startTimer()
    _modalPauseActive = false
  }
}
```

Wrap all modal opens with `pauseForModal()` and all modal closes/dismissals with `resumeAfterModal()`. Applies to: stats, leaderboard, settings, help, new game modal.

---

## Phase 4 — Statistics & Leaderboard

### 4.1 Local Storage Schema

```javascript
// User identity
localStorage.kg_user = JSON.stringify({
  username: "Karen",        // editable
  id: "uuid-v4",           // generated once on first run, never changes
  createdAt: 1234567890
})

// Per-run history (append-only array, cap at 500 entries)
localStorage.kg_runs = JSON.stringify([
  {
    id: "uuid-v4",
    diff: "Medium",         // difficulty name
    diffIdx: 1,
    score: 2387,
    time: 486,              // seconds
    errors: 0,
    hints: 0,
    reveals: 0,
    comboBonusPool: 450,
    completionBonusPool: 2000,
    ts: 1234567890          // unix timestamp
  }
])

// Best scores (derived, but kept separate for fast lookup)
localStorage.kg_best = JSON.stringify({
  Medium: { score: 2387, time: 225, runId: "uuid" },
  Hard:   { score: 5100, time: 441, runId: "uuid" }
})
```

**Migration note:** On first load after update, migrate existing `kg_sudoku_stats` and `kg_sudoku_best` to the new schema. Write a `migrateStats()` function that runs once.

### 4.2 Statistics Modal — Main View

Redesign the stats modal to match the screenshot aesthetic (difficulty tiles in a 2×3 grid):

Each tile shows:
- Difficulty name (bold)
- ★ Best score (gold, bold)
- ⏱ Best time
- W/P · Win%

Tapping a tile opens the **drill-down view**:

```
[Close button top-right only — no Back button]
[Difficulty name] — Top Runs
Sort by: [Score] [Time]   ← toggle, actually re-sorts array

#   TIME    SCORE     ERRORS
🥇  03:45   2214 pts  0
🥈  04:45   2075 pts  0
...
```

**Fix sort toggle:**

```javascript
let sortMode = 'score' // or 'time'
function renderTopRuns(diffName) {
  const runs = getRunsForDiff(diffName)
  if (sortMode === 'score') runs.sort((a, b) => b.score - a.score)
  if (sortMode === 'time')  runs.sort((a, b) => a.time - b.time)
  // then render table
}
```

### 4.3 Home Page Leaderboard (Trophy button)

Opens a full-screen modal or navigates to a leaderboard view showing:

- Toggle: **Top Score** | **Best Time**
- Table: one row per difficulty, showing user's personal best
- Subheader: "Your Rankings" (local only for now)
- Placeholder section: "Global Leaderboard — Coming Soon" (greyed out)

This placeholder section is where the Supabase-powered global board will appear. It signals the feature to users from day one.

---

## Phase 5 — Settings & First-Run Flow

### 5.1 First-Run Username Modal

Check on app load:

```javascript
const user = JSON.parse(localStorage.getItem('kg_user') || 'null')
if (!user) showUsernameModal()
```

Modal:
- Title: "Welcome to The Garden 🌿"
- Subtitle: "What should we call you?"
- Text input, placeholder "Your name", maxlength 20
- Button: "Let's Play"
- If input is empty on submit → store `"You"`
- Generate and store UUID as `user.id`

### 5.2 Settings Modal — Updated

Add to existing settings:

```html
<!-- Username section -->
<div class="s-row">
  <span class="s-label">Username</span>
  <div style="display:flex;gap:8px;align-items:center">
    <span id="settingsUsername" style="font-size:0.85rem;font-weight:700"></span>
    <button class="btn-ghost" id="editUsernameBtn" style="padding:5px 12px;font-size:0.78rem">Edit</button>
  </div>
</div>

<!-- Existing toggles: Show Conflicts, Highlight Same Numbers, Auto-Remove Notes -->

<!-- Danger zone -->
<div class="s-row" style="margin-top:16px;border-top:1px solid var(--border);padding-top:16px">
  <span class="s-label" style="color:var(--error-color)">Clear All History</span>
  <button class="btn-danger" id="clearHistoryBtn">Clear</button>
</div>
```

**Clear History flow:**

```
[Clear] tapped →
Confirmation modal:
  "Are you sure?"
  "This will permanently delete all your game history and best scores. This cannot be undone."
  [Cancel]  [Confirm Deletion]  ← red button
```

On confirm: clear `kg_runs`, `kg_best`, `kg_sudoku_stats`, `kg_sudoku_best`. Keep `kg_user` (username and ID survive).

**Edit Username flow:**

Inline: replace the username span with an `<input>` + Save button. On save, update `kg_user.username`.

### 5.3 btn-danger CSS

```css
.btn-danger {
  background: transparent;
  border: 1.5px solid var(--error-color);
  color: var(--error-color);
  padding: 6px 14px;
  border-radius: 8px;
  font-size: 0.78rem;
  font-weight: 700;
  cursor: pointer;
}
.btn-danger:hover { background: rgba(220,38,38,0.08); }
```

---

## Phase 6 — Supabase Setup (one-time, manual step)

1. Create project at supabase.com
2. Create tables:

```sql
-- Users table
create table users (
  id uuid primary key,
  username text not null,
  created_at timestamptz default now()
);

-- Runs table
create table runs (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references users(id),
  difficulty text not null,
  score integer not null,
  time_seconds integer not null,
  errors integer default 0,
  hints integer default 0,
  reveals integer default 0,
  combo_bonus integer default 0,
  completion_bonus integer default 0,
  created_at timestamptz default now()
);

-- Indexes for leaderboard queries
create index runs_score_idx on runs(difficulty, score desc);
create index runs_time_idx on runs(difficulty, time_seconds asc);
```

3. Enable Row Level Security. Runs are readable by all, writable only by owner:

```sql
alter table runs enable row level security;
create policy "runs_insert_own" on runs for insert with check (auth.uid() = user_id);
create policy "runs_select_all" on runs for select using (true);
```

4. Store the Supabase URL and anon key in `capacitor.config.json` environment or as JS constants in a `config.js` file (gitignored if contains secrets, but anon key is safe to expose).

5. **Post-win sync function (add now, wire up later):**

```javascript
async function syncRunToSupabase(run) {
  const SUPABASE_URL = 'https://YOUR_PROJECT.supabase.co'
  const SUPABASE_KEY = 'YOUR_ANON_KEY'
  try {
    await fetch(`${SUPABASE_URL}/rest/v1/runs`, {
      method: 'POST',
      headers: {
        'apikey': SUPABASE_KEY,
        'Content-Type': 'application/json',
        'Prefer': 'return=minimal'
      },
      body: JSON.stringify({
        id: run.id,
        user_id: getUser().id,
        difficulty: run.diff,
        score: run.score,
        time_seconds: run.time,
        errors: run.errors,
        hints: run.hints,
        reveals: run.reveals,
        combo_bonus: run.comboBonusPool,
        completion_bonus: run.completionBonusPool
      })
    })
  } catch (e) {
    // Fail silently — local score already saved
    console.warn('Supabase sync failed:', e)
  }
}
```

---

## Phase 7 — Capacitor / App Wrapping

### 7.1 Setup

```bash
# In repo root
npm init -y
npm install @capacitor/core @capacitor/cli @capacitor/android @capacitor/ios
npx cap init "The Garden" "com.thegarden.sudoku" --web-dir "."
npx cap add android
npx cap add ios  # only if Mac available
```

### 7.2 capacitor.config.json

```json
{
  "appId": "com.thegarden.sudoku",
  "appName": "The Garden",
  "webDir": ".",
  "server": {
    "androidScheme": "https"
  },
  "plugins": {
    "SplashScreen": {
      "launchShowDuration": 1500,
      "backgroundColor": "#e94560",
      "showSpinner": false
    }
  }
}
```

### 7.3 Required Capacitor Plugins

```bash
npm install @capacitor/splash-screen @capacitor/status-bar @capacitor/app
```

### 7.4 AdMob Preparation

```bash
npm install @capacitor-community/admob
```

Do not wire up ads yet. The plugin just needs to be installed so the build includes it. Ad unit IDs will be configured once Play Console / App Store Connect accounts have the app registered.

Reserve ad slot in HTML (already defined in Phase 3). When AdMob is activated:

```javascript
// In sudoku.html, after game loads:
import { AdMob, BannerAdSize, BannerAdPosition } from '@capacitor-community/admob'

async function initAds() {
  await AdMob.initialize()
  await AdMob.showBanner({
    adId: 'ca-app-pub-XXXXXXXX/XXXXXXXX',  // real ID from AdMob console
    adSize: BannerAdSize.BANNER,
    position: BannerAdPosition.BOTTOM_CENTER,
    margin: 0
  })
}
```

---

## Phase 8 — GitHub Actions

### 8.1 deploy-web.yml (GitHub Pages)

```yaml
name: Deploy Web
on:
  push:
    branches: [production]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: .
          exclude_assets: 'node_modules,android,ios,.github,docs'
```

### 8.2 build-android.yml

```yaml
name: Build Android
on:
  push:
    tags: ['v*']
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npx cap sync android
      - uses: actions/setup-java@v4
        with: { distribution: temurin, java-version: '17' }
      - name: Build release AAB
        working-directory: android
        run: ./gradlew bundleRelease
      - name: Sign AAB
        uses: r0adkll/sign-android-release@v1
        with:
          releaseDirectory: android/app/build/outputs/bundle/release
          signingKeyBase64: ${{ secrets.SIGNING_KEY_BASE64 }}
          alias: ${{ secrets.KEY_ALIAS }}
          keyStorePassword: ${{ secrets.KEY_STORE_PASSWORD }}
          keyPassword: ${{ secrets.KEY_PASSWORD }}
      - uses: actions/upload-artifact@v4
        with:
          name: release-aab
          path: android/app/build/outputs/bundle/release/*.aab
```

**GitHub Secrets required:** `SIGNING_KEY_BASE64`, `KEY_ALIAS`, `KEY_STORE_PASSWORD`, `KEY_PASSWORD`

Generate keystore locally once:
```bash
keytool -genkey -v -keystore garden-release.keystore -alias garden -keyalg RSA -keysize 2048 -validity 10000
base64 -i garden-release.keystore | pbcopy  # paste as SIGNING_KEY_BASE64 secret
```
**Store the keystore file securely. If lost, you cannot update the app.**

---

## Phase 9 — Store Submission

### 9.1 Privacy Policy

Create `privacy-policy.html` in repo root. Must state:
- What data is collected: username (user-provided), game scores, device type
- Where it is stored: locally + Supabase (EU region recommended)
- No selling of data
- How to request deletion: email address
- GDPR/CCPA compliance statement

Host it on GitHub Pages. URL format: `https://[username].github.io/[repo]/privacy-policy.html`

### 9.2 Android (Google Play) — Checklist

- [ ] Google Play Developer account ($25 one-time) at play.google.com/console
- [ ] App bundle (.aab) built and signed via GitHub Actions
- [ ] App icon: 512×512 PNG (no alpha)
- [ ] Feature graphic: 1024×500 PNG
- [ ] Screenshots: min 2, portrait, phone (1080×1920 or similar)
- [ ] Short description (80 chars max)
- [ ] Full description (4000 chars max)
- [ ] Privacy policy URL
- [ ] Content rating questionnaire (result: Everyone / PEGI 3)
- [ ] Target API level: 34+ (required by Play as of Aug 2024)
- [ ] In `android/app/build.gradle`: `compileSdkVersion 34`, `targetSdkVersion 34`

**App store copy (draft):**

> **Short:** The Garden — Sudoku & Puzzles. Six difficulty levels, clean UI, no mid-game interruptions.
>
> **Full:** The Garden is a focused puzzle experience designed for people who just want to play. Solve Sudoku across six difficulty levels — from Easy warm-ups to Nightmare-tier brain-breakers. Earn points, build combos, and track your personal bests. More games coming soon.

### 9.3 iOS (App Store) — Checklist

- [ ] Apple Developer account ($99/year) at developer.apple.com
- [ ] Mac required for Xcode build (or use Codemagic CI)
- [ ] App icon: 1024×1024 PNG (no alpha, no rounded corners)
- [ ] Screenshots: min 3 per device size (6.7", 6.5", 5.5")
- [ ] Launch screen configured via Capacitor splash plugin
- [ ] Info.plist: `NSUserTrackingUsageDescription` if using AdMob (ATT prompt required)
- [ ] Minimum iOS version: 16.0
- [ ] Submit via Xcode Organizer or Transporter

**If no Mac is available:** Use [Codemagic](https://codemagic.io) — free tier supports Capacitor builds and connects to GitHub. Add `codemagic.yaml` to repo.

---

## Phase 10 — QA Checklist Before Submission

### Scoring verification (run these manually):
- [ ] Easy, 0 errors, below par → score > base
- [ ] Easy, 0 errors, above par → score between min and base
- [ ] Easy, 1 error vs 0 errors at same time → 0 errors wins
- [ ] 1 reveal vs 1 hint at same time → hint scores higher
- [ ] Nightmare completes without crash or generation timeout
- [ ] Combo counter resets after 10s gap
- [ ] Row/col/box/number completion bonuses fire exactly once each

### UX verification:
- [ ] Timer pauses when stats/leaderboard/settings/help/new game modal opens
- [ ] Timer resumes on modal close (unless manually paused)
- [ ] Back button (Android hardware) closes modals, doesn't exit app
- [ ] Wordle card is unclickable with "Coming Soon" badge
- [ ] Username persists across sessions
- [ ] Clear history confirmation modal works; history is wiped; username survives
- [ ] Ad slot is visually correct and doesn't overlap game board
- [ ] Dark mode persists across sessions and across pages

### Device testing:
- [ ] Small phone (360×640): tiles and numpad still fit
- [ ] Large phone (412×915): tiles don't exceed 56px max
- [ ] Tablet (768×1024): layout doesn't stretch absurdly — add `max-width` guards

---

## Open Questions (to resolve before execution)

1. **Nightmare difficulty:** Generation at 20 clues may be slow on low-end Android. Test generation time on a mid-range device. If >3 seconds, add a loading state or increase to 22 clues.

2. **Scoring constants:** The multiplied-by-10/100 range in the spec gives scores in the thousands-to-hundreds-of-thousands range. Confirm this feels good after playtesting Phase 1 implementation. Constants live in `SCORE_CONFIG` array and are easy to adjust without logic changes.

3. **Supabase project region:** Create in EU (Frankfurt) for GDPR simplicity. Note this in project setup.

4. **AdMob account:** Needs to be created at admob.google.com before Android build. Ad unit IDs must be added to build before release. Test ads are available during development.

5. **iOS build path:** Confirm whether a Mac is available or Codemagic should be the default iOS CI.

6. **App icon final design:** Needs sign-off before store submission. SVG source generated by Claude Code, final PNG exports reviewed by human.

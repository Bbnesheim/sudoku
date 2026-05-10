# Contributing to The Garden

Thank you for contributing! This guide explains how to work on this project.

## Branch Strategy

- **production** — Active development branch. All work happens here.
- **main** — Stable release branch. Only merged to when ready for production launch.
- Work directly on `production`. Feature branches are not used.

**Always commit and push to `production`** unless explicitly told otherwise.

## Before You Start

1. Read **AGENT_SPEC.md** for the complete specification and current phase of development
2. Review **ARCHITECTURE.md** to understand the codebase structure
3. Check `.claude.md` for Oz conventions and project guidelines

## Making Changes

### Local Development

```bash
# Clone the repo
git clone https://github.com/Bbnesheim/sudoku.git
cd sudoku

# Start a local server (required for PWA features)
python3 -m http.server 8080

# Open http://localhost:8080 in your browser
```

### Code Style

- **HTML** — Semantic markup, single-line attribute layout where possible
- **CSS** — Use CSS variables (`--bg`, `--text`, etc.) for theming. Mobile-first responsive design.
- **JavaScript** — Vanilla JS only, no frameworks. Use `const` and `let`. Avoid `var`.
- **Icons** — Reference from the SVG sprite: `<use href="assets/icons/sprite.svg#icon-name"/>`
- **Dark mode** — Always test CSS in dark mode. Use `[data-theme="dark"]` selector for theme-specific styles.

### Commit Messages

Format commits with a clear, concise message:

```
git commit -m "Brief description of change"
```

Examples:
- `"Add combo system to sudoku game"`
- `"Fix dark mode contrast in settings modal"`
- `"Update icon-play to use solid fill"`

## Testing

### Manual Testing Checklist

Before pushing, test:

1. **Light and dark modes** — Click theme toggle, refresh page, verify persistence
2. **Mobile viewport** — Use browser DevTools (F12) to test at 360×640px and 768×1024px
3. **Offline support** — Open DevTools → Network → set to "Offline", refresh, verify app still works
4. **Touch interactions** — On mobile/tablet, tap all buttons and cards
5. **Keyboard shortcuts** — Test any new keyboard handlers (1–9, arrow keys, etc.)
6. **Performance** — Check for console errors (DevTools → Console)

### Automated Testing (Future)

Once GitHub Actions is set up, tests will run automatically on push. For now, manual testing is the standard.

## Deploying Changes

### Push to production

```bash
# Make sure you're on production branch
git checkout production

# Stage and commit your changes
git add .
git commit -m "Your message here"

# Push to GitHub
git push origin production
```

The live site at https://bbnesheim.github.io/sudoku/ will update automatically once GitHub Pages deploys.

### Merging to main (Release)

When the project is ready for production launch:

```bash
# Ensure production is up-to-date
git checkout production
git pull origin production

# Switch to main
git checkout main

# Merge production
git merge production

# Push to main
git push origin main
```

This will trigger the deploy workflow (when configured) and update the stable release branch.

## Code Review

If working with other contributors:
1. Commit and push to `production`
2. Request review via GitHub
3. Address feedback and push updates
4. Once approved, changes stay on `production` (no PR merge needed, since `production` is the main branch)

## Undoing Changes

If you made a mistake:

```bash
# Undo last commit (keep changes in working directory)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Undo unpushed commits
git reset --hard origin/production
```

## Debugging Tips

1. **Service worker issues** — Hard-refresh (Cmd+Shift+R on Mac, Ctrl+Shift+R on Windows) or open DevTools → Application → Cache Storage, delete old caches
2. **localStorage issues** — Open DevTools → Application → Local Storage → check keys starting with `kg_`
3. **SVG icons not showing** — Verify sprite.svg path is correct and icon ID exists in sprite
4. **Dark mode not persisting** — Check localStorage for `kg_theme` key
5. **Game logic bugs** — Add `console.log()` statements in sudoku.html to trace game state

## Requesting Help

If you're stuck:
1. Check AGENT_SPEC.md for the relevant phase specification
2. Review ARCHITECTURE.md for code structure
3. Search the code for similar implementations
4. Ask a question in the commit message or PR description

## Updating Documentation

When adding new features:
1. Update AGENT_SPEC.md if the feature affects specification
2. Update ARCHITECTURE.md if the code structure changes
3. Update README.md if it affects user-facing features
4. Keep .claude.md current with new conventions

## Continuous Deployment

The `production` branch auto-deploys to GitHub Pages. There is no staging environment — changes are live immediately after push.

**Test thoroughly before pushing!**

---

Happy coding! 🌿

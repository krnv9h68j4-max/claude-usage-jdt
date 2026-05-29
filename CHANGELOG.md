# Changelog

## v1.2.1 — 2026-05-29

### Extension

- Bundle the Python sources (`cli.py`, `scanner.py`, `dashboard.py`) inside the `.vsix` so the extension works standalone — the only end-user dependency is Python 3.8+ on `PATH`. The `vscode:prepublish` script copies the files from the repo root at package time, so each extension version embeds the matching Python snapshot.
- Auto-start the dashboard when the user clicks the sidebar icon (no Command Palette step needed). `DashboardSidebar` accepts an `onShow` callback wired to `openDashboard()`; in-flight startup coalescing on the host side keeps clicks idempotent.
- Discover `cli.py` in any open VS Code workspace folder — covers the "cloned the repo into c:\github\claude-usage and opened it in VS Code" case that the original monorepo-sibling fallback couldn't reach for installed extensions.
- Platform-aware error messages: missing-Python guidance now leads with the right install command (python.org installer on Windows with the "Add to PATH" reminder; `brew install python` on macOS; distro package manager on Linux). The Homebrew suggestion is hidden on Windows.
- Add a dedicated [vscode-extension/README.md](vscode-extension/README.md) for the marketplace listing.
- Real gauge icon shipped (was a placeholder).
- 4 new install-mode tests for bundled discovery + ordering; 4 sidebar tests for the auto-start callback. Total extension test count: 74.

## v1.2.0 — 2026-05-29

### Distribution

- Add a VS Code extension under [`vscode-extension/`](vscode-extension/). Spawns the existing Python dashboard server in the background and embeds it via a webview iframe — no UI rewrite, all existing charts and filters reused. Activity-bar sidebar entry, four commands (`Open Dashboard`, `Rescan`, `Restart Server`, `Show Logs`), and two settings (`pythonPath`, `cliPath`, `port`). 63 tests covering Python discovery, install-mode resolution, port allocation, server lifecycle, and webview HTML/CSP. Built as `.vsix`, installable via [scripts/install.sh / install.ps1](vscode-extension/scripts/).
- Auto-discovery: extension finds `claude-usage` on PATH (Homebrew users) or a sibling `cli.py` in the monorepo (dev). Setting overrides for both Python interpreter and CLI path.

### CI

- Add `.github/workflows/extension-ci.yml`: clean Ubuntu, `npm ci && npx tsc && npm test && npm run package`, uploads the `.vsix` as an artifact on every push to DEV/main that touches `vscode-extension/`.

## v1.1.2 — 2026-05-29

### Project / docs

- Add auto-tag-from-CHANGELOG GitHub Actions workflow ([.github/workflows/tag-on-merge.yml](.github/workflows/tag-on-merge.yml)). Every push to `main` that adds a new `## vX.Y.Z` heading to `CHANGELOG.md` now creates a matching lightweight git tag automatically. CHANGELOG is the source of truth; tags are a deterministic projection of it.
- Revise versioning convention in AGENTS.md: lightweight tags driven by the workflow are the new baseline (supersedes the "annotated, by hand" rule introduced in v1.1.1). No formal GitHub Releases at any cadence.

## v1.1.1 — 2026-05-28

### Packaging

- Add Homebrew formula at `Formula/claude-usage.rb` and install instructions; `claude-usage` is now installable on macOS/Linux without `git clone` (#46, #71, thanks @HaydenHaines)

### Project / docs

- Adopt versioning convention: SemVer with tags on every release, formal GitHub Releases only for major versions. Documented in AGENTS.md.
- Tighten `/triage` routine to leave the CHANGELOG date as `TBD` and let the maintainer fill it in at tag time.

## v1.1.0 — 2026-05-28

### Dashboard

- Fix `ReferenceError: cutoff is not defined` in the hourly filter that blanked the entire dashboard (#73, thanks @thomasleveil)
- Fix hourly chart ignoring the range upper bound for `week` / `month` / `prev-month` ranges
- Fix 404 on dashboard URLs containing query strings (`?range=...&models=...`) so reloads and bookmarks work, with regression tests (#81, thanks @jakduch)
- Fix blank dashboard for users whose data only contains non-billable / unknown models, or rows with empty model names: `COALESCE(NULLIF(model, ''), 'unknown')` normalises empty-string models (in both SELECT and GROUP BY so mixed NULL + '' rows collapse to a single "unknown" group), and the default selection falls back to all models when no billable models exist (#109, thanks @HaydenHaines)
- Use `ThreadingHTTPServer` so a slow `/api/data` no longer blocks other dashboard requests (#79, thanks @jakduch)
- Add a `Today` range button (#112, thanks @Fruhji)

### Scanner

- Fix incremental scan not updating `first_timestamp` when a newly discovered session's records arrive out of order (#111, thanks @Fruhji)

### Project / docs

- Adopt `AGENTS.md` as the canonical agent guide (shared with Codex); `CLAUDE.md` is now a thin `@AGENTS.md` import
- Codify the rule that future PR merges must preserve original-author commits (`merge --no-ff` for full merges, `cherry-pick` for partial, `Co-Authored-By` trailers when applying hunks manually)
- Drop unused `.claude/launch.json`

## v1.0.0 — 2026-04-09

- Fix token counts inflated ~2x by deduplicating streaming events that share the same message ID
- Fix session cost totals that were inflated when sessions spanned multiple JSONL files
- Fix pricing to match current Anthropic API rates (Opus $5/$25, Sonnet $3/$15, Haiku $1/$5)
- Add CI test suite (84 tests) and GitHub Actions workflow running on every PR
- Add sortable columns to Sessions, Cost by Model, and new Cost by Project tables
- Add CSV export for Sessions and Projects (all filtered data, not just top 20)
- Add Rescan button to dashboard for full database rebuild
- Add Xcode project directory support and `--projects-dir` CLI option
- Non-Anthropic models (gemma, glm, etc.) no longer incorrectly charged at Sonnet rates
- CLI and dashboard now both compute costs per-turn for consistent results

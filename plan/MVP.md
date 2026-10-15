# MVP scope (first milestone per PLAN §12)

## Goal
Prove product feel + portability boundary end-to-end before feature build-out.

## In scope
Vertical slice: Tauri window → ConPTY → xterm.js → global hotkey toggle → Acrylic backdrop → `rakuten toggle` over named pipe, with every OS call routed via `rakuten-platform`.

Plus two slices that can land alongside (PLAN §12):
- `rakuten open` minimal: Start Menu discovery → fuzzy match → launch.
- `k` alias minimal: shim dir + session env inject + multicall dispatch (`k toggle`).

## Phases tracked for MVP
0 Foundation (all) · 1 Working terminal (all) · 2 Quake behavior (all) · 3 Glassmorphism (core) · 4 CLI/IPC (core + open slice + alias slice).

## Out of scope (post-MVP)
- Phase 5 polish/dist: command palette "Open app..." mode, scrollback search, shell integration (OSC 7/133), tray pinned apps, per-shell alias init, WSL alias, autostart, single-instance, installer, auto-update, signing, crash reporting.
- `open` extras beyond Start Menu slice: UWP, App Paths, `--admin`, `--with`, args quoting hardening, exit-code/json/list/reindex (core exit codes only if trivial).
- Alias extras: `fallback=open`, shadowing check, per-shell init, WSL support.

## Success criteria
1. App builds/runs on Windows 10 1809+ / 11.
2. Quake window toggles via global shortcut; position/size persist and re-validate on show.
3. ConPTY shell streams to xterm with resize/copy/paste/scrollback/links.
4. Backdrop shows Acrylic or safe fallback; no acrylic lag while dragging.
5. `rakuten toggle` round-trips over named pipe with auto-launch.
6. `rakuten open <start-menu-app>` launches.
7. `k toggle` works inside a Rakuten-spawned shell; guard rejects outside (exit 9009-style).
8. `cargo check` passes for mac/linux stubs; cfg grep check passes (no `#[cfg(windows)]` outside platform crate).

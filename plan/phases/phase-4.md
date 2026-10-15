# Phase 4 — CLI, IPC, open, alias (PLAN §9 Phase 4)

Goal: named-pipe IPC + CLI commands (incl. doctor), `open` MVP slice, terminal-scoped alias core.

Exit criteria:
- App IPC server + CLI client with auto-launch; `rakuten toggle` round-trips.
- Core commands (position/size/tab/run/config/theme/doctor/completions) work with `--json`.
- `rakuten open <start-menu-app>` discovers + fuzzy-matches + launches.
- `k toggle` works inside Rakuten-spawned shell; guard rejects outside.

Task IDs: P4-01..P4-08, P4-10 (deferred: P4-09, P4-11, P4-12).
Key refs: PLAN §6, §7, §8, §4 (IPC), §12.
Risks: R-08, R-11, R-13, R-14, R-15, R-16.
Depends on: P0 (traits/schema), P1 (pty), P2 (window), P3 (theme for theme cmd).

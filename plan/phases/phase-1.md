# Phase 1 — Working terminal (PLAN §9 Phase 1)

Goal: frameless transparent window over ConPTY streamed to xterm.js.

Exit criteria:
- Tauri window (frameless/transparent) shows live shell, resize, copy/paste, scrollback, links, tabs.
- Session env injection + alias shim creation hook wired (shim itself Phase 4).
- Cross-target cargo check passes.

Task IDs: P1-01..P1-09.
Key refs: PLAN §1, §3, §4, §9 Phase 1, §12.
Risks: R-01, R-06.
Provides: base for Phase 2 toggles/backdrop.

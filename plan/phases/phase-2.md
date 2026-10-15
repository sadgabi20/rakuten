# Phase 2 — Quake behavior (PLAN §9 Phase 2)

Goal: shortcut toggle + persistent/revalidated placement + pin levels + tray/taskbar + widget.

Exit criteria:
- Global shortcut toggles visibility with focus handling; hide-on-blur optional.
- Position/size stored physical px + monitor id; re-validated on show.
- Pin levels, tray icon, taskbar hiding work; widget spike documented (fallback = always-on-top).

Task IDs: P2-01..P2-07.
Key refs: PLAN §4, §9 Phase 2, §10 (risks), §12.
Risks: R-03, R-04, R-05.
Depends on: P1-04 (window).

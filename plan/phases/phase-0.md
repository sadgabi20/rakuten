# Phase 0 — Foundation (PLAN §9 Phase 0)

Goal: scaffolding, shared contracts, portability guards before features.

Exit criteria:
- Workspaces build locally; CI green (Windows test/build, mac/linux cargo check).
- Traits + config schema compile; rakuten-platform contains all cfg-specific code.
- IPC message types defined/tested; portability grep gate passes.

Task IDs: P0-01..P0-12 (see tasks.md).

Key refs: PLAN §1, §2 (traits/cfg), §3 (structure), §5 (config), §7.5 (apps skeleton), §8.4 (alias schema), §9 Phase 0.
Risks: none specific.
Related: P0-06 defines AppLauncher used by P4-08.

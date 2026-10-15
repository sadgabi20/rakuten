# Tasks (single ledger; MVP-critical path)

## Phase 0 — Foundation (PLAN §9 Phase 0)
- [ ] P0-01 Cargo workspace root + crate skeletons (core, platform, pty, ipc, window, apps, cli; desktop src-tauri placeholder) | ref: PLAN §3 | files: Cargo.toml, crates/
- [ ] P0-02 pnpm workspace + turborepo config | ref: PLAN §1,§3 | files: pnpm-workspace.yaml, turbo.json, package.json
- [ ] P0-03 Lint/format tooling (rustfmt, clippy, eslint/prettier) | ref: PLAN §9 Phase0 | files: rustfmt.toml, .eslintrc*, prettier*
- [ ] P0-04 CI workflows (windows test/build; mac/linux cargo check; cfg grep) | ref: PLAN §9 Phase0,§2 | files: .github/workflows/*
- [ ] P0-05 rakuten-core config schema incl [apps],[cli.alias] + types/errors/events | ref: PLAN §5,§6 | files: crates/rakuten-core
- [ ] P0-06 rakuten-platform traits (WindowEffects,Monitors,Autostart,ShellDiscovery,AppLauncher,AliasShims,IpcTransport) + cfg impls + current() | ref: PLAN §2 | files: crates/rakuten-platform
- [ ] P0-07 rakuten-apps crate skeleton: AppEntry, index/cache, fuzzy matcher, aliases, target classification | ref: PLAN §7.5 | files: crates/rakuten-apps
- [ ] P0-08 rakuten-pty session manager skeleton + session env builder fn | ref: PLAN §8.4 | files: crates/rakuten-pty
- [ ] P0-09 rakuten-ipc JSON-RPC protocol types + server/client stubs | ref: PLAN §6,§4 | files: crates/rakuten-ipc
- [ ] P0-10 rakuten-window pure placement/snapping/monitor validation fns | ref: PLAN §2 (principle 3) | files: crates/rakuten-window
- [ ] P0-11 Config JSON Schema codegen script stub | ref: PLAN §5 | files: scripts/, packages/config-schema
- [ ] P0-12 cfg grep gate + mac/linux cargo check wired into CI | ref: PLAN §2.5,§9 Phase0 | files: scripts/, .github/workflows

## Phase 1 — Working terminal (PLAN §9 Phase 1)
- [ ] P1-01 apps/desktop Tauri shell (thin) + React UI Vite scaffold | ref: PLAN §3 | files: apps/desktop
- [ ] P1-02 packages/terminal xterm.js wrapper + PTY bridge hooks | ref: PLAN §1,§3 | files: packages/terminal
- [ ] P1-03 packages/ui, theme, tsconfig, eslint-config | ref: PLAN §3 | files: packages/
- [ ] P1-04 Frameless transparent Tauri window | ref: PLAN §4,§12 | files: apps/desktop/src-tauri
- [ ] P1-05 ConPTY session spawn + stream to xterm | ref: PLAN §1,§12 | files: crates/rakuten-pty, packages/terminal
- [ ] P1-06 Resize, copy/paste, scrollback, clickable links | ref: PLAN §9 Phase1 | files: packages/terminal, apps/desktop
- [ ] P1-07 Shell profiles detection + management | ref: PLAN §4,§5 | files: crates/rakuten-platform, crates/rakuten-core
- [ ] P1-08 Tabs | ref: PLAN §1,§6 | files: apps/desktop
- [ ] P1-09 Session env injection + alias shim creation hook | ref: PLAN §8.2 | files: crates/rakuten-pty

## Phase 2 — Quake behavior (PLAN §9 Phase 2)
- [ ] P2-01 Global shortcut toggle + focus handling | ref: PLAN §4,§12 | files: apps/desktop/src-tauri, crates/rakuten-platform
- [ ] P2-02 Persistent position/size (phys px + monitor id) + re-validate on show | ref: PLAN §9 Phase2 | files: crates/rakuten-window, apps/desktop
- [ ] P2-03 Snap to edge | ref: PLAN §9 Phase2 | files: crates/rakuten-window
- [ ] P2-04 Pin levels (Normal/AlwaysOnTop/Desktop) | ref: PLAN §2,§9 Phase2 | files: crates/rakuten-platform
- [ ] P2-05 Tray icon + taskbar hiding | ref: PLAN §4,§9 Phase2 | files: apps/desktop/src-tauri
- [ ] P2-06 Hide-on-blur | ref: PLAN §9 Phase2 | files: apps/desktop
- [ ] P2-07 Widget mode spike (desktop-level) with always-on-top fallback | ref: PLAN §4,§10 | files: crates/rakuten-platform, crates/rakuten-window

## Phase 3 — Glassmorphism (PLAN §9 Phase 3)
- [ ] P3-01 Acrylic/Mica backdrop via platform trait + fallback chain | ref: PLAN §4,§12 | files: crates/rakuten-platform, apps/desktop
- [ ] P3-02 Rounded corners + border (DWM / CSS) | ref: PLAN §4 | files: crates/rakuten-platform, apps/desktop
- [ ] P3-03 Reduce-effects-while-moving toggle | ref: PLAN §9 Phase3 | files: apps/desktop, crates/rakuten-core
- [ ] P3-04 Theme token system + loader (glass presets) | ref: PLAN §5,§9 Phase3 | files: packages/theme, themes/
- [ ] P3-05 Settings panel with live preview | ref: PLAN §9 Phase3 | files: apps/desktop
- [ ] P3-06 Theme import/export | ref: PLAN §5,§9 Phase3 | files: apps/desktop, packages/theme

## Phase 4 — CLI, IPC, open, alias (PLAN §9 Phase 4)
- [ ] P4-01 IPC named pipe server in app + CLI client + auto-launch | ref: PLAN §4 IPC,§6,§12 | files: crates/rakuten-ipc, crates/rakuten-cli, apps/desktop
- [ ] P4-02 CLI commands: toggle/show/hide/pin/unpin | ref: PLAN §6 | files: crates/rakuten-cli
- [ ] P4-03 position/size/tab new|list|close/run | ref: PLAN §6 | files: crates/rakuten-cli
- [ ] P4-04 config get/set/edit/path | ref: PLAN §6 | files: crates/rakuten-cli
- [ ] P4-05 theme list/set/install | ref: PLAN §6 | files: crates/rakuten-cli
- [ ] P4-06 doctor (shortcut, backdrop, WebView2, ConPTY, shells, index, WSL) | ref: PLAN §6,§7.7,§8.5 | files: crates/rakuten-cli
- [ ] P4-07 completions (incl. open/app names from cached index) | ref: PLAN §6,§7.7 | files: crates/rakuten-cli
- [ ] P4-08 open core: Start Menu discovery + fuzzy match + launch | ref: PLAN §7,§7.6,§7.8 | files: crates/rakuten-apps, crates/rakuten-cli, crates/rakuten-platform
- [ ] P4-10 alias core: shim dir + env injection + multicall dispatch + session guard | ref: PLAN §8,§8.6 | files: crates/rakuten-cli, crates/rakuten-pty, crates/rakuten-platform

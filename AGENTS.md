# AGENTS.md — Rakuten

Quake-style terminal app (Tauri 2 + Rust + React). Target: Windows 10 (1809+)/11, port-ready architecture.

## Read first
- `PLAN.md` — product/tech spec (source of truth; cite anchors, don't duplicate).
- `plan/README.md` — MVP execution index + agent protocol.
- `plan/MVP.md` — in/out of scope; success criteria.
- `plan/tasks.md` — single task ledger; status lives here only.

## Architecture (non-negotiable)
- Thin shell: `apps/desktop/src-tauri` only wires; logic in `crates/`.
- One OS boundary: ALL `#[cfg(windows)]` confined to `crates/rakuten-platform`. CI greps for violations.
- Pure placement logic in `rakuten-window` (unit-testable without display).
- Portable data formats: shortcuts as strings (`Ctrl+Backquote`), shell profiles, `directories` crate for paths, OS settings under `[platform.windows]`.
- Cross-targets must `cargo check` for macOS/Linux from day one (stubs return `Unsupported`).
- `rakuten` CLI stands alone where possible; scoping via env injection, never profile/registry/global PATH.

## Key paths
- `crates/` — Rust workspace members (core, platform, pty, ipc, window, apps, cli).
- `apps/desktop/` — React UI + thin Tauri shell.
- `apps/docs/` — docs/landing.
- `packages/` — terminal, theme, ui, config-schema, tsconfig, eslint-config.
- `scripts/` — release, codegen, icon gen.
- `themes/` — built-in themes (JSON).

## Conventions
- `snake_case` Rust, no comments unless asked.
- Never commit secrets/keys; never log them.
- Edit existing files over creating new ones.
- Keep product decisions in `PLAN.md`; update `plan/` for execution only.
- Update task status in `plan/tasks.md` only (`[ ]`/`[~]`/`[x]`/`[!]`); one `[~]` at a time; mark `[x]` only after verification.
- Run lint/typecheck before marking done (commands below, finalized in Phase 0).
- Never commit/push/PR unless explicitly asked.

## Build & verify (finalize in Phase 0; update here)
- Rust: `cargo fmt --check`; `cargo clippy --workspace -- -D warnings`; `cargo test --workspace`; `cargo check --workspace --target x86_64-pc-windows-msvc`; cross: `cargo check --workspace --target x86_64-apple-darwin`, `cargo check --workspace --target x86_64-unknown-linux-gnu`.
- TS: `pnpm install`; `pnpm -r typecheck`; `pnpm -r lint`; `pnpm -r test`.
- Portability gate: grep `#[cfg(windows)]` must be confined to `crates/rakuten-platform`.

## Token discipline
- Read smallest relevant file first; avoid loading all phases.
- Quote `PLAN.md` by anchor (`PLAN §12`), never paste sections.
- One line per task in `plan/tasks.md`.

## Working directory
`F:\github\rakuten` on `win32`. Use dedicated tools for file ops (Read/Edit/Write/Glob/Grep); Bash for git/npm/cargo etc.

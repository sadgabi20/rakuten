# Verification

Commands are finalized in Phase 0 (P0-03/P0-05 area); fill placeholders below once known.

Language / build:
- Rust: `cargo fmt --check`; `cargo clippy --workspace -- -D warnings`; `cargo test --workspace`; `cargo check --workspace --target x86_64-pc-windows-msvc`.
- Cross-check: `cargo check --workspace --target x86_64-apple-darwin`; `cargo check --workspace --target x86_64-unknown-linux-gnu`.
- TS: `pnpm install`; `pnpm -r typecheck`; `pnpm -r lint`; `pnpm -r test`.
- Config: regenerate + check JSON Schema from `rakuten-core` (script under `scripts/`).
- Portability: grep `#[cfg(windows)]` must be confined to `crates/rakuten-platform`.

Acceptance rules:
- Mark task `[x]` only after commands for touched code pass.
- Prefer targeted crate tests while iterating; full workspace before marking done.
- Unknown commands found during Phase 0 must replace placeholders above.

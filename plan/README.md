# Rakuten — Agent Plan (MVP)

Token-first execution index. Product source of truth is `PLAN.md` (repo root); this folder tracks MVP only. Cite PLAN anchors, never duplicate prose.

## Files
- `MVP.md` — MVP scope, success criteria, out-of-scope.
- `tasks.md` — single task ledger; status lives here only.
- `phases/phase-N.md` — goal, exit criteria, task IDs, refs (no duplicate task text).
- `risks.md` — condensed risk register (from PLAN §10).
- `verification.md` — commands + acceptance.
- `trace.md` — PLAN section ↔ task IDs reverse index.

## Status legend
- `[ ]` pending · `[~]` in_progress · `[x]` done · `[!]` blocked
- Exactly one task may be `[~]` at a time.

## Agent protocol
1. Read `README.md` + `MVP.md` first; then only the needed phase file.
2. Update status in `tasks.md` only. Phase files never carry status.
3. Mark `[x]` only after verification commands pass for touched code.
4. Append real file paths to the task line (`files:`) only if they differ from plan.
5. Never commit without explicit user request.
6. Blocked → `[!]` with `blocked: reason`, keep `[~]` cleared.
7. Do not edit `PLAN.md`; it is the stable reference.

## Token budget
- Max one line per task in `tasks.md`.
- Fields only: `id`, `title`, `ref`, `files`, `deps`, `blocked`.
- Quote PLAN by anchor (e.g. `PLAN §12`), never paste sections.
- Read the smallest relevant file; avoid loading all phases.

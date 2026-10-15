# Phase 3 — Glassmorphism (PLAN §9 Phase 3)

Goal: OS-level Acrylic/Mica backdrop with fallback + theme tokens + settings + reduce-effects.

Exit criteria:
- Backdrop applies with runtime build detection + fallback chain (Mica→Acrylic→solid).
- Rounded corners/border correct on Win10/11; reduce-effects toggle stops drag lag.
- Theme tokens load from JSON; settings panel live preview; import/export.

Task IDs: P3-01..P3-06.
Key refs: PLAN §4, §5, §9 Phase 3, §12.
Risks: R-02, R-06.
Depends on: P1 (window), P2 (placement).

# Phase 1 — Mechanics Quiz — take2 summary

One-glance recap of the completed 12-question mechanics quiz. Full verbatim transcripts: `Q01.md … Q12.md`.

## Result: all 12 resolved at 3-star

| Q | Topic | Outcome |
|---|-------|---------|
| Q1 | Story → data-model change (long-form) | 3★ first try — TEXT, cited `InitialMigration.ts:21` + entity |
| Q2 | Story → validation change | 3★ — `@IsNotEmpty` on title+body, repo's own pipe, inline-vs-`@UsePipes` |
| Q3 | Entity change ≠ DB change | 3★ — only executed `ALTER` DDL changes the column |
| Q4 | Migration registration gotcha | 3★ — `migrationsList` + `disableDynamicFileAccess`, bundle-safe reason |
| Q5 | Review the generated migration (ship/fix/reject) | **1★→3★** — over-engineered a backup-table rollback; re-derived to "ship + document the lossy `down()`; recovery is the operator's runbook + Aurora backups, not the migration" |
| Q6 | Opt-in `@UsePipes` (controller) | 3★ — both create+update, repo's pipe, 400→422 plan decision; correctly delegated endpoint enumeration to Cline |
| Q7 | Auth middleware wiring | 3★ — verified against source; `configure(consumer).forRoutes(...)` |
| Q8 | AWS stateless constraint | 3★ core — per-instance array → Aurora unique constraint. Spun off the exception-handling thread (see below) |
| Q9 | Frontend `Result` vs throw | 3★ — user-fixability axis; `.match` forces both branches |
| Q10 (ran as Q9 bonus) | BE↔FE error-shape trace | Confirmed a **real shipped bug**: `buildError` emits strings, `genericErrorsDecoder` wants `string[]` → `verify` throws → AC3 fails; shared with the user endpoints |
| Q11 | Frontend decoder: required field backend omits | 3★ — `verify` throws on missing required; fix `optional(string)` + DoD debt; learned `""` is a valid string |
| Q12 | AI Usage: weak→strong Cline prompt | 3★ — reference files ("model it on X") for DTO + controller, pipe gotcha, grep-for-consumers, right fields |

## Candidate profile (for the build coach)

- Strong senior full-stack; new to NestJS / MikroORM / monads+decoders. Pushes hard on design and is
  usually right (drove the Q5 re-derivation and several package fixes).
- **Preferences (enforced):** be tight, no over-explaining; **never ghostwrite** (re-anchor on frame
  slips instead of validate-then-explain); restate specs inline (no "recall Q2"); reads visually.

## Package changes made this session (files on disk are current)

- `02-VOICE.md`: takeN banking rule; "Reward delegating recall"; "Highest-signal exercise (trace
  BE↔FE under uncertainty)" + "set the ground / disambiguate same-named artifacts"; ghostwrite
  "re-anchor, don't validate-then-explain"; "no anchoring to a prior question / state inputs".
- `01-COACH.md`: §7 item 6 (DB-constraint→field MVP gap: options A/B/C with verdicts + illustrative
  code, verified by spike — explicit `@Unique({ name })` mandatory + tested ESLint rule); §2.7
  "HTTP lives at the boundary, not in services"; §2.5 confirmed BE↔FE error-shape bug.
- `03-TUTORIAL.md`: new **Q10** (BE↔FE error-shape trace); count 11→12; Q11 (subtitle) reworded to
  set the ground.
- `04-PROMPT.md`: count → 12.
- Spike branch in the code repo: `spike/mikroorm-unique-constraint-naming` (committed; survives).

## Next

Resume at **Phase 2** (explain the agentic build loop once), then **Phase 3** (coach the build).
Entry prompt: `coach-ws-eng-conduit-ai-assessment/05-PROMPT-PHASE-2.md`.

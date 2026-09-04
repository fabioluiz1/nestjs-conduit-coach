# Build Loop — live quick-reference

Four beats per build step. You hold the context; Cline does the typing. The worked example
throughout is the **data-model change**: moving `article.body` and `article.description` from
`varchar(255)` to the MySQL `TEXT` type.

In the table, the **Good example** column is the actual prompt or command you would send; the
**Bad example** column is the failure mode that loses the point.

| Beat | What you do | Good example | Bad example |
|---|---|---|---|
| **1 · Gather** | Direct Cline to read the relevant files and report the current facts, changing nothing. The acceptance criterion is already in your head — there is no separate "state the task" beat. | "Inspect `@backend/src/article/article.entity.ts` and `@backend/src/migrations/InitialMigration.ts`. Report the column types declared for `body` and `description`, and whether either has any validators. **Do not change anything — read and report only.**" | "Which files do I need to change to support long articles?" — Cline must guess your intent, so it over-reaches or under-reaches. |
| **2 · Plan** | State each decision with the alternative you rejected and the reason; Cline writes them into `plan.md`; you fix any rationale it invents. | "Decision: `body` and `description` → `TEXT`. Alternative considered: keep `varchar(255)` and truncate past 255 chars — rejected, silent data loss. Write this into the Decisions section of `@plan.md`." Then read it and fix any guessed line. | Paste Cline's report in unchanged; let its wording stand unread; or gather a fact and never record the decision (gathered-but-unrecorded scores nothing). |
| **3 · Build** · change the entity | Cline edits the entity; you point at the existing style instead of describing the syntax. | "In `@backend/src/article/article.entity.ts` change `body` and `description` to `TEXT`, matching the existing `@Property` style. Change nothing else." | Spelling out `@Property({ type: 'text', fieldName: 'body' })` in the prompt — redundant once you've said "match the existing style." |
| **3 · Build** · generate the migration | The command-line tool writes the migration by diffing your changed entity against the live database. | From `backend/`: `npx mikro-orm migration:create --name AlterArticleBodyDescriptionToText` | Hand-writing the DDL, or "this is complex, I'll just edit it myself" — a by-hand change scores **zero** on AI Usage. |
| **3 · Build** · review the generated migration | You read what the tool emitted before it ships, and decide ship / fix / reject. | Confirm `ALTER TABLE ... MODIFY` (not `CREATE`), `text not null` on both columns, and a `down()` reverting to `varchar(255)`. | Accepting it blind / "looks good" — caps AI Usage at 2 stars. This is the Q5 skill. |
| **3 · Build** · register the migration | Cline adds the generated class to `migrationsList`. This is the load-bearing gotcha. | "Add the generated migration to `migrationsList` in `@backend/mikro-orm.config.ts`, matching how `InitialMigration` is listed." | Skipping it — the migration silently never runs, because `disableDynamicFileAccess: true` turns off folder discovery. |
| **3 · Build** · restart the backend | Restart the server; migrations run automatically on boot. | Restart — `AppModule.onModuleInit()` calls `migrator.up()`, which applies the new migration. | — |
| **4 · Verify** | Give Cline the exact command and the exact expected output, and have Cline run it. Screenshot the passing criterion into `submission/`. | "Restart the backend, POST a 600-character body, and re-GET it after the restart. Confirm the body comes back intact, not truncated." Then screenshot the pass. | Running it in your own terminal; or "verify it works" with no command and no expected output. |

---

## Two notes that keep this from inflating

- You articulate the acceptance criterion **once**, at Gather. It then lives in `plan.md` (Beat 2),
  and your Build prompts point Cline at `@plan.md`. You do not restate it three times.
- `SHOW CREATE TABLE article` is **not** a sixth row. It is a five-second probe to confirm the
  migration applied — reach for it only if the round-trip in Beat 4 looks wrong. The round-trip is
  the real acceptance criterion and the thing you screenshot.

# INTERVIEW_SETUP — Everything to prepare BEFORE you start

> **This is the definitive pre-flight guide.** Every step here happens *before* the
> graded clock starts. Work through it top to bottom. Verbatim source material lives
> in [`../instructions/`](./instructions) — this file is the actionable distillation.

---

## 0. The one rule that governs everything

**The timer starts the moment you open the user-story link — not when you open the
repo or the Codespace.** (Confirmed in the walkthrough video, see
[`../instructions/05-video-notes.md`](./instructions/05-video-notes.md).)

Everything in this document is *free time*. You can set up the environment, read the
code, run the app, configure the AI tool, and rehearse — all without spending a second
of the graded budget. **Do not click the link until this whole checklist is green and
you have 3 uninterrupted hours.**

The link itself (kept here only as a record — **do not open yet**):
```
https://rscyxa4qtij2k5wjbrkjkf442q0eshgb.lambda-url.us-east-1.on.aws/?asrId=a0Bfv00000CDgs9EAD&type=design-and-implement
```

---

## 1. What this assessment actually is (the mental model)

You will extend a working full-stack web application by adding one **user story**. You do
the work by **directing an AI coding assistant (Cline)** — you're the senior engineer, the
AI is a capable junior with zero context on this project.

- **Stack:** NestJS + MikroORM backend, React + Redux Toolkit + Vite frontend, MySQL. The
  opinionated bits to know going in are in `../01-COACH.md` (Unit-of-Work persistence, custom
  auth middleware, per-endpoint validation, no-thunk Redux, runtime-decoded responses).
- **Time budget:** ~3 hours. Speed is itself graded (the "Velocity" criterion).
- **You are graded on 5 things** (full table in
  [`../instructions/01-instructions.md`](./instructions/01-instructions.md)):
  Plan Soundness, Code Quality, Correctness, **AI Usage** (how well you direct the AI),
  and Velocity.

The single most important idea: **AI Usage is scored on the quality of your direction,
not the volume of AI output.** Sloppy prompts produce sloppy code and score low even if
the AI wrote everything. This is the same muscle as the Phase-1 architecture warm-up:
own every decision, give the AI context, reject bad output.

---

## 2. Pre-flight checklist (do all of this before the link)

- [ ] **GitHub account** ready (free) — https://github.com/signup
- [ ] **Open the Codespace:** use the link in
      [`../instructions/00-setup.md`](./instructions/00-setup.md). It auto-installs
      dependencies, seeds the database, and starts both servers. First build ≈ 2–3 min.
- [ ] **Confirm you are on the correct branch:** `rwa/design-and-implementation-v2`.
      Other branches exist in the repo and using the wrong one is a scored failure.
- [ ] **Open the app:** http://localhost:3001 — log in with
      `jcosten0@purevolume.com` / `password`. (In Codespaces the URL is auto-forwarded,
      so it will look different — that's normal.)
- [ ] **Open the API docs (Swagger):** http://localhost:3000/docs — the live endpoint list.
- [ ] **Configure the AI tool (Cline)** — see section 3 below.
- [ ] **Explore the code** (free time!) — read the backend and frontend, follow how one
      existing feature works end to end. Use the `CLAUDE.md` repo map in your coach workspace
      (written during onboarding; it's a reference, not part of the repo) and your `../01-COACH.md`
      as your guide.
- [ ] **Connect the database browser** (optional but useful): the MySQL VS Code
      extension is pre-installed but **not** configured — point it at the credentials in
      `backend/mikro-orm.config.ts` to inspect tables.

---

## 3. Configure Cline (the AI assistant) — exact values

**Cline** is a VS Code extension that lets an AI read your code and propose edits. You
must use **Cline only** — no other AI tool is allowed.

Open Cline settings (Cline landing page → *Use your own key*, or press `F1` →
*Cline: Focus on View* → settings wheel → *API Configuration* tab) and set:

| Field | Value |
|-------|-------|
| Provider | **OpenAI Compatible** |
| Base URL | `https://wnogqpmdu74ndach7m36xntowe0ecgzb.lambda-url.us-east-1.on.aws/v1/` |
| Key | `a0Bfv00000CDgs9EAD` |
| Model ID | `gpt-5`, `gpt-5.2`, or `gpt-5.4` (your choice) |

In the Codespace, Cline is **pre-installed and version-pinned** by the devcontainer to
`saoudrizwan.claude-dev@3.34.0` — don't upgrade it; practice on the same version you'll grade on.

How to drive it well: treat Cline like a junior dev with **no project context**. Always
give it context and **@-mention the specific files** it should look at before asking for
a change. It asks permission before editing any file you add to the chat — that's a
safety gate, acknowledge to proceed.

---

## 4. Codespace gotchas (so they don't cost you time mid-clock)

- **502 Bad Gateway** can appear even when the servers are running. Fix: open the
  **Ports** view, right-click the port, toggle visibility **Public ↔ Private**.
- **The seeder is idempotent** — it only seeds an empty DB and no-ops once users exist
  (`database.seeder.ts`: `if ((await em.count(User)) > 0) return;`). So a restart **won't
  duplicate or reset** your test data; it also won't "refresh" the DB if you want a clean slate.
- **Inactivity:** the Codespace stops after 30 min idle; restart from
  github.com/codespaces — your changes persist. Stop it manually between prep sessions
  to preserve your free monthly quota. Use the **4-CPU** machine.

---

## 5. The 0-star traps (memorize these — each one zeros your score)

From the README's "Mandatory Rules" — see
[`../instructions/01-instructions.md`](./instructions/01-instructions.md):

1. **Do not fork.** Work on the provided repo on its original remote, or the submission
   script can't see your changes.
2. **Stay on branch `rwa/design-and-implementation-v2`.**
3. **Use Cline exclusively** for all AI.
4. **Include screenshots of the acceptance tests passing.** No screenshots = 0 stars.
5. **Submit a plan** (`plan.md`), written *before* you code. No plan = 0 stars.
6. **Do not clear your Cline chat history.** No history = 0 stars.

---

## 6. Strategy notes (decided before the clock, applied during)

- **Plan first, always.** The plan is graded on its own. Fill `plan.md` with: the
  step-by-step implementation, every new library/component/pattern you introduce, and
  the 2–3 core decisions (alternatives + rationale). If you add infrastructure, explain
  how it deploys in AWS in the Notes section.
- **Go straight for the advanced variant.** The basic variant caps at 1 star; 2–3 stars
  live in the advanced version. You don't have to build basic first.
- **The optional "Homework" story is upside-only** — skipping it can still earn full
  points. Don't let it steal time from the graded story (where Velocity counts).
- **Respect the production topology.** ECS Fargate, 1–10 auto-scaled instances behind an
  ALB, shared Aurora MySQL. Stateless rule: **no in-memory state, no local-disk
  assumptions, no in-process timers/loops** — shared/durable state goes in the DB (or a
  justified shared service). Constraint details and fixes are in `../01-COACH.md §6`.

---

## 7. Submission (the very end, on the clock)

You submit by running the repo's submission script — **`npm run submit`** (which runs
`tsx submit.ts` from the repo root; the walkthrough video calls it `submit.sh`, but the repo
ships `submit.ts`). It detects your changes on the original remote and returns a **Submission ID**.
That ID is the one required field on the submission form (full form in
[`../instructions/00-form.md`](./instructions/00-form.md)). The page does **not** save
drafts and submission is **final** — capture the ID and screenshots before closing
anything.

---

## 8. Where everything lives

| Path | What |
|------|------|
| [`../instructions/00-setup.md`](./instructions/00-setup.md) | Cline config, Codespace links, hard rules (verbatim) |
| [`../instructions/00-form.md`](./instructions/00-form.md) | Assessment submission form fields (verbatim) |
| [`../instructions/01-instructions.md`](./instructions/01-instructions.md) | Full README (verbatim) |
| [`../instructions/02-plan.md`](./instructions/02-plan.md) | The required plan template |
| [`../instructions/03-architecture-diagram.png`](./instructions/03-architecture-diagram.png) | AWS production architecture |
| [`../instructions/05-video-notes.md`](./instructions/05-video-notes.md) | Walkthrough video, net-new tips |
| `../01-COACH.md` | Your didactic onboarding into the tech (NestJS, AWS, the codebase) |
| `../03-TUTORIAL.md` | Learn-by-doing practice feature, directing Cline end-to-end |
| `01-TUTORIAL_SETUP.md` | Run the app locally + Cline setup (practice without Codespace credits) |
| `CLAUDE.md` | Factual repo map / tribal knowledge (kept here, not in the repo) |

# TUTORIAL_SETUP — Run Conduit in a local Dev Container + wire up Cline (no Codespace credits)

## 🧭 Operating model — read this first

**Host = Coach · Dev Container = Coachee.** Run the **coach** (Claude Code) in a plain host
**Terminal**; do the hands-on build with **Cline inside a local Dev Container — _not_ Codespaces**
(keeps your free GitHub credits). The repo is bind-mounted, so the coach on the host sees Cline's
edits live.

```mermaid
flowchart LR
  subgraph HOST["🖥️ HOST — your Mac"]
    direction TB
    T["<b>Terminal:</b> Claude Code = COACH<br/>sees coach docs + the repo<br/>probes · corrects · never writes code"]
    R[("repo folder")]
    T --- R
  end
  subgraph DC["📦 Dev Container"]
    CL["<b>Cline</b> in VS Code / Cursor<br/>= graded agent + app :3000 / :3001<br/>sees /workspace = the repo ONLY<br/><b>NOT Codespaces → 0 credits</b>"]
  end
  R <-->|"bind mount · same files"| CL
```

---

> Goal: bring up the app through the repo's **own Dev Container harness** — unedited — so you
> practice `../03-TUTORIAL.md` in an exact replica of the exam environment, on your own Docker
> (**no GitHub Codespace credits**). This is a throwaway study clone; it is never submitted.

---

## 0. Two things that are safe to know first

- **Using the API key / making Cline calls does NOT start the assessment timer.** The clock
  starts *only* when you open the user-story link (see `02-INTERVIEW_SETUP.md`). So you can
  freely configure and test Cline against the real endpoint beforehand.
- **You change nothing in the repo.** The harness (`.devcontainer/` + `docker-compose.yml` +
  `.vscode/tasks.json`) is the contract — it supplies MySQL, Node 22, Cline, and the running
  servers. If a step ever tells you to edit `mikro-orm.config.ts`'s `host`, add a `ports:`
  mapping, or run a standalone `docker run mysql`, that's a sign you've **left** the harness —
  don't. The real Codespace runs this same harness untouched, and so should you.

---

## 1. Prerequisites

- **Docker Desktop** — installed and **running**.
- **VS Code** + the **Dev Containers** extension (`ms-vscode-remote.remote-containers`).
- **No Node/npm on your Mac.** The container ships Node 22 (`node:22.11.0`) and npm — that's
  the version the exam uses, so you don't manage it yourself.

---

## 2. Open the repo in the Dev Container

In VS Code, open the clone
(the Conduit checkout), then run
**“Dev Containers: Reopen in Container”** from the Command Palette (`F1`).

That one action does everything (first build ≈ 2–3 min):

- builds the **app** image (`node:22.11.0`) and starts the **db** service (MySQL 8.1, on a
  persistent named volume);
- **bind-mounts** the repo into the container, so you edit files on your Mac and they run inside;
- **installs Cline** at the pinned version `saoudrizwan.claude-dev@3.34.0` (from
  `devcontainer.json`);
- **forwards ports 3000 / 3001 / 3306** to your Mac;
- runs the `runOn: folderOpen` tasks in `.vscode/tasks.json` → **installs deps, starts the
  backend + frontend, and seeds the DB** — all automatically. (Migrations also run on backend
  boot via `AppModule.onModuleInit`.)

> **Why no `host` edit?** The `app` container shares the `db` service's network namespace
> (`network_mode: service:db`), so the shipped `host: 'db'` resolves as-is. The host edit only
> exists if you run Node *outside* the container — which we don't. Harness untouched = `db` works.

**Codespaces alternative.** “Reopen in Container” and GitHub Codespaces run the *same* harness;
Codespaces just runs it in the cloud and **burns your free CPU-hours**. Use the local Dev
Container for practice; the graded run happens in a fresh Codespace.

---

## 3. Verify it's up

The Dev Container forwards ports **3000 / 3001 / 3306 to your Mac** (§2), so every check below is
reachable from the host shell — **the coach runs these itself and does not ask you to report
them** (see `../02-VOICE.md` §"Verify it yourself"). Watch the tasks finish (backend prints `Nest
application successfully started`, frontend prints the Vite `Local:` line, the seed task
completes), then:

**Coach-verifiable (agentic — the coach curls these from the host):**

- **API smoke test:** `curl http://localhost:3000/api/api` → `Hello World!`
  (the path is doubled on purpose: `main.ts` sets the global prefix `api` **and** the root
  `AppController` is `@Controller('api')`, so the root route is `/api/api`. Feature routes are
  single-prefixed, e.g. `GET /api/articles`.)
- **Swagger (endpoint explorer):** `curl -sI http://localhost:3000/docs` → `200 OK`
  (browse it at http://localhost:3000/docs).
- **Frontend up:** `curl -sI http://localhost:3001` → `200 OK`
  (open http://localhost:3001 — already seeded; log in with `jcosten0@purevolume.com` /
  `password`).

If any check fails, the env isn't up: open the repo in the Dev Container (§2 — "Dev Containers:
Reopen in Container"), wait for the start tasks, then the coach **re-runs the checks itself**.
Don't hand these back to the user to eyeball.

**User-confirmed (manual — the coach can't see VS Code):**

- **Cline replies:** Cline is configured per §4 with an exam-legal GPT-5.x model and a test prompt
  (`say hello`) returned a reply. Only you can confirm this — the coach can't drive the VS Code UI.

Notes:
- **Seeding is automatic** (the `Run Seed` task) and **idempotent** — it only seeds an empty DB
  and no-ops once users exist, so a restart won't duplicate or reset your data.
- **Driving the backend without the UI:** there's **no Postman collection** — use **Swagger at
  `/docs`** (try any endpoint interactively, including auth) or `curl`. The frontend, by
  contrast, *does* need the backend (no mocks; it calls `/api/`).

---

## 4. Configure Cline's model (Cline is already installed)

Cline came up with the container at version **3.34.0**. Do not upgrade it — you grade on this
version, so practice on it.

For practice you drive Cline with **your own GPT model** — your own OpenAI-compatible Base URL and
API key. This gives realistic training on GPT-class models on your own budget. **Do not use the
interviewer's assessment key for practice.** (The exam itself uses a separate, assessment-provided
endpoint; you configure that at exam time per `02-INTERVIEW_SETUP.md` / `../instructions/00-setup.md`,
not here.)

### 4a. Prepare your own OpenAI API access (do this first)

1. Go to https://platform.openai.com and sign in (or create an account).
2. Add billing so API calls succeed: **Settings → Billing** → add a payment method or prepaid
   credits.
3. Create a key: **Dashboard → API keys → Create new secret key** → copy the `sk-...` value
   immediately (it is shown only once) and store it safely.
4. Your **Base URL** for the official OpenAI API is:
   ```
   https://api.openai.com/v1
   ```
5. Use **`gpt-5.2`** as your Model ID. The exam lets you pick any of `gpt-5`, `gpt-5.2`, or
   `gpt-5.4` — all three are exam-legal, so practice on one of them (do **not** use `*-mini`
   variants; they aren't offered in the exam and build a different muscle).

   Once your account tier has settled (see the propagation note below), all three carry the
   **same rate limits** (≈ **500 requests/min + 500k tokens/min** — plenty for Cline), so the
   choice comes down to **capability vs cost**, not limits:

   | Model | Capability | Cost/token | Exam-legal | Best for |
   |-------|-----------|-----------|-----------|----------|
   | `gpt-5`   | base     | cheapest | ✅ | — (weaker than exam needs) |
   | `gpt-5.2` | strong   | middle   | ✅ | **drilling the loop (recommended)** |
   | `gpt-5.4` | strongest| priciest | ✅ | one final dress-rehearsal |

   **Drill on `gpt-5.2`, then do one end-to-end rehearsal on the exact model you'll pick in the
   graded exam.** `gpt-5.2` is flagship-class `gpt-5.x` (same reasoning/tool-calling muscle as the
   exam) but cheaper than `gpt-5.4` across the *many* repeated practice runs — save `gpt-5.4`'s
   premium for a single calibration run, not every rep. (Verify exact prices at
   <https://platform.openai.com/pricing>; the API does not expose pricing.)

   > **⏳ First-time credit/rate-limit propagation (~15 min).** When you add credits to a fresh
   > account, the payment and the resulting **usage-tier upgrade are NOT instant** — allow up to
   > **~15 minutes** for the higher rate limits to take effect. Rate limits track your **usage
   > tier** (cumulative spend + time), **not** your current balance, so before propagation a fresh
   > account sits at tiny caps (e.g. **50 req/day + 10k tokens/min** on the newest models). Cline
   > sends large file context per turn, so during that window you'll see transient `429`s like
   > *"Request too large … TPM limit 10000"* or *"requests per day (RPD): Limit 50"* — and each
   > attempt still burns a daily slot even when it fails. These clear once the tier settles. **If
   > you top up right before the graded run, wait ~15 min before relying on it**, or your first
   > Cline turns may throw these same `429`s.
   To confirm your account has it, list your models (replace the key):
   ```bash
   curl https://api.openai.com/v1/models \
     -H "Authorization: Bearer sk-REPLACE_WITH_YOUR_KEY" \
     | python3 -m json.tool | grep '"id"' | grep -i gpt
   ```
   (No terminal handy? Browse https://platform.openai.com/docs/models instead.)

> **Using a different GPT provider** (Azure OpenAI, OpenRouter, a company gateway)? The setup is
> identical — substitute that provider's own **Base URL**, **API key**, and **Model ID** in the
> steps below. It just has to be OpenAI-compatible.

### 4b. Enter your credentials in Cline

1. Open the Cline panel in VS Code (the Cline icon in the Activity Bar, left edge).
2. On the **"Hi, I'm Cline"** landing screen, click **"Use your own API key"** to reveal the
   configuration fields. (To reopen this screen later: press `F1`, run **"Cline: Focus on View"**,
   then click the **settings gear** → **API Configuration**.)
3. Set **API Provider** to **OpenAI Compatible**.
4. In **Base URL**, paste your Base URL from §4a step 4 (`https://api.openai.com/v1`).
5. In **OpenAI Compatible API Key**, paste your `sk-...` key from §4a step 3.
6. In **Model ID**, type `gpt-5.2` (the exam-legal practice model — see §4a step 5).
7. Leave **Custom Headers**, **Set Azure API version**, and **Model Configuration** at their
   defaults (untouched).
8. Ignore the red "works best with Claude models" note — it is a generic Cline warning, not an
   error.
9. Click **Let's go!**.
10. In the chat box, send `say hello` and confirm you get a reply. A reply proves the Base URL,
    key, and model all resolve.
11. If step 10 errors: re-check the **Base URL** and **API Key** for a leading/trailing space,
    confirm the **Model ID** exactly matches one from your §4a list, and confirm billing is active.
    Then resend.

> **Aider?** Skip it. The exam's "Single AI Tool" rule means only Cline is graded, and
> "AI Usage" scores *how you drive that specific tool*. Practicing in Aider builds a different
> muscle. Use Cline.

> **How to drive Cline well** (this is the graded skill): give it context up front and
> **@-mention the exact files** it should read before asking for a change. Treat it as a
> sharp junior with zero project knowledge. It asks permission before editing files you've
> added to the chat — that's the review gate; read every diff before accepting.

---

## 5. Gate — confirm setup before the tutorial

**⛔ DO NOT PROCEED TO `../03-TUTORIAL.md` UNTIL THE COACHEE EXPLICITLY SAYS THE SETUP IS COMPLETE
AND THEY ARE READY TO START.**

This binds the coach too: do **not** check the setup state and then move into Step 1 on your own
initiative. Wait for the coachee's explicit "setup is done / ready to start" before teaching any
tutorial step. If setup is not confirmed, stay on setup — help finish and verify it, nothing else.

Setup is complete only when all of these are true:

1. *(coach-verifiable)* App reachable at http://localhost:3001 and login works
   (`jcosten0@purevolume.com` / `password`).
2. *(coach-verifiable)* Swagger reachable at http://localhost:3000/docs.
3. *(coach-verifiable)* `curl http://localhost:3000/api/api` returns `Hello World!`.
4. *(user-confirmed — the coach can't see VS Code)* Cline is configured per §4 (OpenAI Compatible
   + **your own** Base URL/key + an exam-legal model id — `gpt-5.2` recommended) and a test prompt
   returned a reply.

Items 1–3 are mechanical: **the coach runs them itself** (the host reaches the forwarded ports)
and never asks you to report them. Only item 4 needs your word.

When every item above is true **and you have said you're ready**, go to **`../03-TUTORIAL.md`** and
build the practice feature end-to-end by directing Cline — the same loop you'll run in the exam.

When done practicing, stop the container (Docker Desktop → stop, or **Dev Containers: Reopen
Folder Locally**). The MySQL named volume persists, so your data survives to the next session.

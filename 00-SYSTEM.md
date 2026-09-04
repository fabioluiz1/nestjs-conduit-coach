<!-- ===========================================================================
AGENT CONTRACT — GOVERNANCE LAYER (00-SYSTEM.md)
ROLE: The rules for maintaining the coaching system. Governs HOW the other files are
  edited. It sits ABOVE the runtime files; it is not part of a coaching session.
AUDIENCE: agents EDITING the system — NOT the runtime coach. This file is NOT in the
  orchestrator's "read before you start" list.
SOURCE OF TRUTH FOR: the Invariants · the routing procedure · the file map.
EDIT POLICY: change an Invariant here and nowhere else; headers cite Invariants by number.
  Changing an Invariant's number/wording is itself an Invariant-5 cross-file sweep.
============================================================================ -->

# 00-SYSTEM — How the coaching system is maintained

> Read this **before editing any of the coaching files**. It defines where content belongs,
> the rules every edit obeys, and how the files relate. It is maintainer-facing — the runtime
> coach never reads it.

---

## The two layers

```
MAINTENANCE LAYER   00-SYSTEM.md  ── governs HOW everything below is edited
                          │           (audience: agents EDITING the system)
   ─────────────────────────────────────────────────────────────────────
RUNTIME TRIAD       01-COACH.md · 02-VOICE.md · 03-TUTORIAL.md
                          └── orchestrated by → 04-PROMPT.md
                                      (audience: the COACH RUNNING a session)
```

---

## 1. The file map

| File | Layer | Role (one line) |
|------|-------|-----------------|
| **00-SYSTEM.md** | maintenance | The Invariants + routing + this map. Governs how the rest is edited. |
| **01-COACH.md** | runtime — knowledge | Every technical fact / concept / file path / gotcha the coach reasons from. |
| **02-VOICE.md** | runtime — communication | How the coach talks, grades, and guides. Behaviour, not content. |
| **03-TUTORIAL.md** | runtime — session | This session's stories, ACs, questions, clues, scoring, debrief. |
| **04-PROMPT.md** | runtime — orchestrator | Sequences the session. Owns no facts; references the three above. |

Each runtime file carries its own `AGENT CONTRACT` header (role · put-here · do-not-put-here
redirects · file-specific edit policy). Those headers are the per-file detail; this file holds
only the cross-cutting rules they cite.

---

## 2. The routing procedure (run before writing ANY line)

Run in order, stop at the first match:

```
Q1. Is it a TECHNICAL fact — how the code/stack/repo works, a file path, a gotcha?
      → 01-COACH.md   (if NEW: add here FIRST, then others may reference it)

Q2. Is it a rule about HOW THE COACH BEHAVES — tone, one-question-at-a-time,
    never reveal answers, never self-justify, how to redirect a stuck user,
    the grading PHRASING pattern, the per-step coaching loop?
      → 02-VOICE.md

Q3. Is it SESSION CONTENT — a specific question, clue, tip, acceptance criterion,
    user story, the time budget, a scoring rule / star definition, the debrief?
      → 03-TUTORIAL.md   (a fact echoed here must already exist in 01-COACH.md)

Q4. Is it only ORDERING / a POINTER — "run X, then Y", "deliver the opening",
    "grade per VOICE"?
      → 04-PROMPT.md
```

If two answers seem true, the **earlier** file wins for the *fact*, and the later file holds a
*pointer*. (A scoring rule that cites a technical fact: the fact is in 01-COACH.md, the rule is in
03-TUTORIAL.md and references it.)

---

## 3. The Invariants

The numbered, canonical list. Headers and edits cite these by number ("[Inv.4]"); they are never
restated in full elsewhere.

1. **Knowledge originates in 01-COACH.md.** A *new* technical fact MUST be added to `01-COACH.md` first.
   No other file may host a net-new technical fact.

2. **The orchestrator owns no facts.** Every fact in `04-PROMPT.md` must already exist in
   one of the three runtime source files; duplication is allowed only as a verbatim echo. If it
   isn't in a source file, it cannot be in the prompt — replace it with a pointer.

3. **Duplication is tolerated; drift is not.** Two copies of a fact must be identical, or one must
   become a pointer.

4. **Place by locality, not by convenience.** A new fact / question / rule goes where its topic and
   architectural layer put it — not wherever is easiest to paste. (Example: an API fact belongs
   *after* the frontend section and near the *top* backend layers — far from the lower-layer
   DB/migration facts. A new quiz question goes into its concept group.) The end of the file is the
   right place **only when locality lands there** (e.g. a new section that genuinely belongs last);
   it is never the default dumping ground. If nothing fits, that signals a **missing section** —
   create it in the right place.

5. **Number / count / ordering changes propagate — verified by repeated sweeps.** Any edit that
   changes a count, label, sequence, or numbered reference (e.g. "11 questions", "Concept 7", step
   numbers, line refs like `:21`) is **not done** until every one of the runtime files is swept for
   stale references. Delegate the mechanical sweep to **Sonnet or Haiku**, and **re-run the sweep
   2–4×** until a pass finds zero stale references. One pass is never enough.

6. **Vendor-agnostic content.** Coach-authored materials never name the specific assessment platform
   or interviewer by name. Use generic terms — "the assessment", "when your timer
   starts", "the submission form". The package is valid senior-interview prep regardless of who
   administers it. **Carve-out:** files under `instructions/` are a *verbatim* record of the real
   assessment and are left exactly as received — their purpose is fidelity, not neutrality; the
   codebase guide `setup/CLAUDE.md` likewise describes the repo as-is.

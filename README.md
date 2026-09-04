# NestJS Conduit Coach

A knowledge base and teaching package for the [RealWorld](https://github.com/gothinkster/realworld)
Conduit reference application on NestJS, structured as a set of agent contract layers rather
than as a flat tutorial.

Roughly 42,000 words. Written to bring a senior engineer up to working fluency on an
unfamiliar NestJS + MikroORM codebase quickly, and to be driven by a coding agent.

## Layout

| Path | What it is |
| --- | --- |
| `00-SYSTEM.md` | Governance layer: what the agent may and may not do |
| `01-COACH.md` | The knowledge base. NestJS request lifecycle, pipes and validation at the boundary, MikroORM entities and migrations, exception filters, the service-to-HTTP boundary |
| `02-VOICE.md` | Communication rules for the coach persona |
| `03-TUTORIAL.md` | Session content, delivered in order |
| `04-PROMPT.md` | Runnable orchestrator prompt |
| `05-PROMPT-PHASE-2.md` | Phase-two resume orchestrator |
| `05-THE-BUILD-LOOP.md` | Live quick reference for the build loop |
| `setup/01-TUTORIAL_SETUP.md` | Running Conduit in a local dev container |
| `setup/02-INTERVIEW_SETUP.md` | Environment checklist |
| `setup/CLAUDE.md` | Codebase guide for the Conduit monorepo |
| `setup/00-COACH-zod-vs-decoders-vs-pydantic.md` | Comparative note on runtime validation ergonomics |
| `quiz/take1`, `quiz/take2` | Twelve mechanics questions, two passes, used as a fixation exercise |

## Why the contract layers

Splitting governance, knowledge, voice and content into separate files means each can be
revised without disturbing the others, and the orchestrator can load only what a given
session needs. The knowledge base is the durable artifact; the prompt is disposable.

## Provenance

Written while preparing for a technical assessment built on Conduit. The material is about
NestJS, MikroORM and the RealWorld specification, all of which are public. No assessment
instructions or submission content are included.

Conduit and the RealWorld spec are MIT licensed and belong to their respective authors.

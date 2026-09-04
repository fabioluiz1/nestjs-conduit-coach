# `decoders` vs `zod` — same job, different ergonomics

Both are TypeScript runtime-validation libraries. They exist because TS types are erased at runtime, so you validate `unknown` data at I/O boundaries and get back a statically-typed value. This repo (Conduit) uses `decoders`; `zod` is the more popular equivalent. If you know one, you understand the other.

---

## Same core idea

- Runtime validation libraries for TS; exist because TS types vanish at runtime.
- Schemas built compositionally from primitives (`string`, `number`, `object`, `array`, `optional`, …).
- Take `unknown` input → return a statically-typed value, or structured errors.
- Belong at I/O boundaries (API responses, forms).
- Both can infer the static type from the schema.

---

## API style — side by side

**zod** is fluent/builder — chains methods off a base schema.
**decoders** is functional-combinator — composes standalone functions.

| | zod | decoders |
|---|---|---|
| **Style** | Fluent / builder | Functional combinator |
| **Optional string** | `z.string().min(1).optional()` | `optional(string)` |
| **Object schema** | see below | see below |

```ts
// zod
import { z } from "zod";

const articleSchema = z.object({
  slug: z.string(),
  title: z.string(),
});

const optionalField = z.string().min(1).optional();

type Article = z.infer<typeof articleSchema>;
```

```ts
// decoders
import { object, string, optional, Decoder } from "decoders";

const articleDecoder: Decoder<Article> = object({
  slug: string,
  title: string,
});

const optionalField = optional(string);
```

---

## Key differences

| Aspect | zod | decoders |
|---|---|---|
| **Type direction** | Schema-first: `type T = z.infer<typeof schema>` | Often type-first: annotate `Decoder<Article>`, though inference is also possible |
| **Error surface** | `.parse()` (throws) / `.safeParse()` → `{ success, data \| error }` | `.verify()` (throws) / `.decode()` (Result-like); this repo pairs it with the `Result` monad from `@hqoss/monads` |
| **Ecosystem** | De-facto standard — integrated with tRPC, react-hook-form, and more | Older, smaller, more FP-flavored |

---

## Mental model

`decoders` ≈ zod's functional-combinator cousin — same role, different ergonomics.

---

> **Note:** snippets are illustrative; exact `decoders` API not verified against the repo.

---

## Python is in the same boat

Python is in the same situation as TypeScript, for the same reason — type hints are **not** enforced at runtime (CPython ignores PEP 484 hints; they're advisory). So `def f(x: int)` will happily accept a string; only a static checker complains. Runtime data (JSON from an API) is unchecked unless you validate it.

### Same split as TS

| Concern | TypeScript | Python |
|---|---|---|
| **Static check (dev-time)** | `tsc` + type system | mypy / pyright + type hints |
| **Runtime boundary validation** | zod / `decoders` | Pydantic / marshmallow / DRF serializers |
| **Plain typed struct, no validation** | `interface` / `type` | `@dataclass` / `TypedDict` / `NamedTuple` |

### The analogs

- **Pydantic** — the direct equivalent of zod/`decoders`. Define a `BaseModel` with typed fields; it validates + coerces input at runtime, raising `ValidationError` or returning a typed object. Pydantic v2 (Rust core) is the de-facto standard and powers FastAPI — the same role zod plays in tRPC.
- **DRF serializers / marshmallow** — the older framework-world version: validate/deserialize incoming data at the boundary, serialize outgoing. DRF serializers are Django's; marshmallow is framework-agnostic.
- **`@dataclass` / `TypedDict`** — give you structure but **no** runtime validation (like a bare TS `interface`): a typed shape, not a validated shape.

### Where modern Python typing actually solves it

- mypy/pyright cover the static layer — but only for code you control, never external runtime data (exactly like `tsc`).
- Pydantic reuses the standard type hints to drive runtime validation — you write ordinary annotations and get the runtime guard for free, so one declaration serves both static checking and runtime validation.
- Contrast: in TS the zod schema and the TS type are separate artifacts (bridged via `z.infer<typeof schema>`). Python collapses that into a single typed model.

```python
# illustrative — Pydantic model mirroring the Article shape used elsewhere in the doc
from pydantic import BaseModel

class Article(BaseModel):
    slug: str
    title: str
    body: str
    tag_list: list[str]

# At the boundary: validates raw dict, returns a typed Article or raises ValidationError
article = Article.model_validate(raw)
```

**Mental model:** Pydantic ≈ "zod for Python," except the schema IS the type annotations — no infer-the-type-from-the-schema step.

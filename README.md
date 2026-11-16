# Project Halberd — public progress log

A personal R&D project: an automation harness for **authorized, scope-bounded security testing**
(bug-bounty and penetration-testing workflows).

**This repository is a progress log — not the product.** It intentionally contains **no source code**
and **no internal design detail**. It exists to record milestones and show the work is real and ongoing.
The implementation lives in a separate private repository.

**Current development milestone: [v-1.1](./DEVLOG.md#2026-11-15--v-11-persistent-investigation-graph-cross-agent-discovery--adversarial-execution-integrity)**
— supervised R&D, dated 2026-11-15. Unites the persistent investigation graph (PIG), autonomous lead loops, cross-agent discovery, and an 18-defect adversarial execution audit.

**Latest development update:** [2026-11-15 — v-1.1: Persistent investigation graph, cross-agent discovery & adversarial execution integrity](./DEVLOG.md#2026-11-15--v-11-persistent-investigation-graph-cross-agent-discovery--adversarial-execution-integrity).
Includes the adversarial defect repair matrix, cross-program graph activity snapshot, and 250+ deterministic offline validation passes.

## What the project is (high level)

An operator points the system at a target they are authorized to test. The system runs its testing
procedures through a scoped gateway with rate limits, request budgets and recorded evidence.
Model calls require explicit authorization; configured hard model-spend limits currently refuse launch
when enforcement cannot be established. Omitted model-spend limits mean uncapped execution. It is built safety-first:

- All outbound test traffic is forced through one authorized gateway. Anything out of scope is refused.
- The automation acts only through a small, audited set of tools — never raw network or shell access.
- Nothing is ever submitted or disclosed automatically. A human stays in the loop.

## What's here

- [`DEVLOG.md`](./DEVLOG.md) — dated entries: what was added, what was fixed, what's next.

## What's deliberately *not* here

No code, no architecture, no exploit details, no target data, no credentials. If you're looking for those,
that's by design.

---

_Testing is only ever performed against targets the operator is authorized to assess (e.g. public
bug-bounty scopes and intentionally-vulnerable practice labs)._

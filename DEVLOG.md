# Dev log

High-level progress only. No code, no internal design, no exploit or target detail.
Most recent first.

---

## 2026-10-24 — Independent finding-verification stage ✅

**Milestone.** The independent validation stage flagged as "next" earlier today now exists. Before any
finding can be treated as real, it is first checked **by deterministic code against its own recorded
evidence** — and only if that passes does a separate second-opinion model get involved.

### Added
- **Deterministic evidence replay** — for every finding, the system confirms each piece of evidence it
  cites genuinely exists and is **unaltered** (tamper-evident, content-addressed), and that the recorded
  evidence actually backs the claim. A finding that cites missing, altered, or unsupporting evidence is
  **rejected automatically and never escalated** — no model is even consulted. This kills fabricated or
  misleading findings at the door.
- **Second-opinion gate** — only findings that pass the deterministic check go to an independent review
  by a **different model than the one that produced them**. A finding counts as confirmed only when both
  the deterministic check and the independent review agree. Fails closed otherwise.
- Each verification is recorded with its own outcome state, so a finding's status is always auditable.

### Rough next steps
- Detect stalled runs (alive but making no progress) and surface them.
- Resume safely after an interruption without repeating completed work.

---

## 2026-10-24 — Reliability groundwork + full internal validation audit

**Milestone.** A day spent hardening foundations instead of adding attack coverage. The system now keeps a
durable, structured record of every automated run, and the whole harness went through a comprehensive
internal validation and regression audit. No live target run was performed — this validated the machinery,
not a new investigation.

### Added
- **Per-run record keeping** — every automated step now writes a record with a *typed outcome*
  (ran / blocked / failed / skipped-for-budget) and a failure reason, so a run's result is always knowable
  after the fact. First piece of the observability-and-recovery layer.
- **Per-run resource accounting** — each run now captures how much model work it used, recorded honestly
  (left blank rather than guessed when a provider doesn't report it). Groundwork for spend awareness.
- **Standing automated test suite** — one command runs static checks plus deterministic, offline tests for
  the safety gateway, the state-tracking logic, and the run-routing error handling.

### Fixed
- Corrected an outcome-classification edge case so a run is only marked "blocked" when it genuinely was.

### Validation audit (safety-first, offline)
- Static checks (compile/vet) and deterministic tests across the gateway and supporting layers — **all
  green**. Confirmed the gateway **fails closed**: out-of-scope hosts, disallowed methods, unknown risk
  levels, unanswered approvals, cross-program requests, and spent budgets are all refused.
- Confirmed the automation can reach a target **only** through the audited tool set, and that state
  tracking never marks a finding "verified" on its own.
- Produced a prioritized, incremental improvement roadmap, and a readiness assessment: the deterministic
  core is reliable; the model-driven and end-to-end paths remain to be validated under controlled live runs.

### Rough next steps
- Build the independent **finding-verification stage** — a deterministic replay plus a second-opinion
  review before any finding is treated as real.
- Turn the captured per-run usage into a spend budget with a stop-when-exhausted limit.
- Broaden automated coverage across more vulnerability classes.

---

## 2026-10-23 — First autonomous end-to-end run ✅

**Milestone.** The system completed its first fully autonomous test run from start to finish, entirely
inside the safety boundary: it independently probed a target, identified a genuine **access-control flaw**,
and produced tamper-evident evidence for it — without ever stepping outside the authorized scope.

Target was a local, intentionally-vulnerable practice lab (OWASP Juice Shop). The finding was recorded,
not acted on — a human reviews everything before it goes anywhere.

### Added
- **Enforcement gateway** — the single path from the automation to any target. Keeps every request inside
  authorized scope and within rate/spend limits, and records evidence of each one. Fails closed by default.
- **Constrained tool layer** — the automation reaches the outside world only through a small, audited set
  of tools. No raw network or shell access.
- **Orchestration layer** — runs testing procedures against an authorized target and aggregates results
  into a single review board for the operator.
- **Procedure library** — a growing set of structured, reviewable playbooks covering common web-vulnerability
  classes.
- Codebase brought under version control.

### Fixed
- The enforcement gateway now correctly **refuses all traffic when its configuration changes mid-run** —
  caught a real configuration-drift case during testing. Safety-first behavior confirmed.
- Improved stability of the gateway under heavy request load.
- Hardened error handling across the automation layer so failures surface honestly instead of being masked.

### Rough next steps
- Broaden automated coverage across more vulnerability classes.
- Add an independent **validation stage** — a second-opinion review before any finding is treated as real.
- Add a lightweight **planning component** so the system can prioritize what to test next.

---

_Entries are intentionally high level. The point of this log is to show steady, real progress and
responsible, safety-first engineering — not to publish the work itself._

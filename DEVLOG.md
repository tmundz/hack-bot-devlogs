# Dev log

High-level progress only. No code, no internal design, no exploit or target detail.
Most recent first.

Versioned milestones start at **v-1.0**. Earlier entries remain an unversioned history.

---

## 2026-11-15 — v-1.1: Persistent investigation graph, cross-agent discovery & adversarial execution integrity

**Milestone.** Project Halberd reaches development milestone **v-1.1**, uniting persistent investigation state, autonomous team-lead loops, cross-discipline discovery events, and an extensive adversarial hardening pass. Research observations no longer live in transient agent memory: they form a queryable, typed investigation graph with cryptographically backed evidence provenance. At the same time, an 18-defect adversarial audit has reinforced execution integrity, transport isolation, deduplication, and cross-model finding verification.

This is a **supervised R&D milestone**; it is not a claim of production readiness or dependable unattended vulnerability discovery.

### What the system is doing
- **Persistent Shared Investigation Graph (PIG)** — Observations, endpoints, dataflows, trust boundaries, hypotheses, primitives, and gadgets are tracked as typed nodes and relationships in a persistent graph. Every node enforces explicit knowledge states (`observed`, `hypothesis`, `validated`, `refuted`, `unknown`) and links back to tamper-evident content-addressed evidence. A hypothesis never silently morphs into an established fact.
- **Autonomous lead loops & event-driven discovery** — Discipline leads (Recon, JavaScript, API, Client, Server) execute persistent, restartable `OBSERVE -> FORM_HYPOTHESIS -> PRIORITIZE -> PLAN -> DELEGATE -> EXECUTE -> EVALUATE -> RECORD -> REPLAN` cycles. Typed discovery events (e.g. unmapped endpoints or auth boundaries revealed by JavaScript analysis) automatically notify relevant leads without requiring monolithic reprompting.
- **Adversarial execution integrity & transport hardening** — Repaired 18 subtle failure modes across HTTP egress, session isolation, loop deduplication, and verification gates. Pre-network credential validation rejects expired tokens; per-principal session rotations lock atomically; redirects preserve GraphQL bodies rather than transforming them; and partial/truncated responses fail closed rather than producing ungrounded findings.
- **Deterministic static analysis & reverse engineering (REA)** — AST parsing (Babel) and code pattern extraction (Semgrep) run deterministically offline without model spend. An integrated reverse-engineering pipeline enables offline APK manifest auditing, deep-link mapping, and exported-component inspection with scoped ADB loopback forwarding.
- **Mobile application attack-surface expansion** — Integrated 22 structured Android vulnerability classes (AL-01 to AL-22) into the application surface analysis path, covering mobile authorization, IPC interfaces, TLS/trust verification, crypto misuse, and hybrid webview bridges.
- **Cost-aware multi-provider model routing** — Dynamic role routing pairs routine note-taking and structured documentation with fast, cost-effective models (Gemini-first reporting), while reserving frontier reasoning for planning and verification. Strict cross-family model independence remains mandatory for peer reviews.
- **State-machine reporting pipeline** — A deterministic lifecycle (`DRAFT -> EVIDENCE_REQUIRED -> READY_FOR_REVIEW -> APPROVED -> SUBMITTED`) governs vulnerability report drafts. Reports must satisfy offline schema checks, credential redaction, attachment verification, and human operator approval before submission.
- **Isolated browser profiles & session management** — Browser automation operates on dedicated, disposable program profiles strictly isolated from operator workstations. Multi-identity sessions maintain private cookie jars with automatic expiration tracking.

### Adversarial audit & repair summary

A systematic adversarial review of current-worktree execution, graph, and reviewer contracts identified 18 specific failure modes. All were addressed with focused failing-before baselines and passing-after regression checks on controlled local fixtures:

| Ref | Category | Condition addressed | Enforced invariant & contract |
|---|---|---|---|
| **E1** | Egress / Auth | Stale or inapplicable captured credentials hitting targets. | Pre-network validation rejects expired or mismatched credentials; preference cookies cannot mask expired auth tokens; named 401s halt execution. |
| **E2** | Session Isolation | Concurrent workers conflicting on credential rotation or revocation. | Per-principal session locking serializes rotation/revocation; atomic private cookie-jar updates prevent cross-worker pollution. |
| **E3** | HTTP Transport | HTTP redirects mutating POST bodies or breaking GraphQL queries. | Redirects are never followed automatically; original POST bodies, variables, and headers are preserved; redirects are treated as distinct observations. |
| **E4** | Evidence Integrity | Upstream network disconnects or body truncation masked as success. | Partial/error evidence is preserved with intact status/headers but marked `complete=false` and `body_unavailable=true`; partial evidence cannot verify a finding. |
| **E5** | Protocol Safety | Model-injected control headers or host framing conflicts. | Host framing is validated pre-network; unsupported framing or internal control headers are refused before egress; internal gateway references never leak to targets. |
| **I1** | Investigation Logic | Inconclusive tool executions falsely closing valid hypotheses. | Inconclusive technique failures preserve open hypotheses; refutations require explicit, completed falsification with intact evidence and recorded rationale. |
| **I2** | Traceability | Experiments disconnected from originating hypothesis IDs. | Task experiments maintain bidirectional links across all hypothesis identifier formats, preserving decision context without claiming premature confirmation. |
| **I3** | Loop Scheduling | Repetitive tasks causing infinite retry loops or skipping new evidence. | Identical intents deduplicate; tasks re-dispatch only when canonical inputs, scope, or evidence hashes materially change; failed tasks enter quarantine. |
| **O1** | Replay Verification | Loose string matches or request-only assertions passing verification. | Replay verifier enforces strict, field-specific assertion matching; request-only assertions prove collection, not vulnerability verification. |
| **O2** | Checklist Gates | Incomplete checklists or broken artifact references passing review. | Completion reviews require 100% unique checklist step coverage and nonempty, verified artifact citations; missing steps or symlink escapes deny sign-off. |
| **O3** | Reviewer Independence | Self-review or provider-family collusion during second-opinion reviews. | Review scaffolding verifies actual model family responder independence; an identical model family or unverified identity cannot approve a finding. |
| **O4** | Review Durability | In-flight review obligations lost across process crashes. | Review obligations persist canonically across restarts; missing model provenance holds task completion for operator intervention rather than guessing. |
| **G1** | Static Analysis | Static code patterns treated as confirmed runtime vulnerabilities. | Static source/sink relationships project candidate entities with explicit uncertainty scores; static discovery is an investigation input, not exploit proof. |
| **G2** | Graph Integrity | Repeated agent writes creating duplicate primitive entities. | Shared graph maintains idempotent writes and contributor history, preserving multi-agent provenance without entity duplication. |

### Research & operational activity snapshot

Cumulative anonymized activity across active testing programs, frozen at **2026-11-15 19:30 UTC**:

| Measure | Recorded total | Change since Nov 2 |
|---|---:|---:|
| Confirmed findings — operator tally | 1 | — |
| Informational findings — operator tally | 1 | — |
| Tracked primitives / conditional leads | 6 | — |
| Refuted false positives / closed negative controls | 5 | — |
| Total research task cards | 317 | +38 |
| Task cards marked done | 185 | +13 |
| Persistent graph events recorded (`pig_events`) | 33,672 | *New in v-1.1* |
| Mapped investigation nodes (`pig_nodes`) | 14,192 | *New in v-1.1* |
| Tracked investigation relationships (`pig_edges`) | 18,548 | *New in v-1.1* |
| Cross-agent discovery events (`discovery_events`) | 689 | *New in v-1.1* |
| API inventory rows | 842 | — |
| API baseline records | 5,912 | — |
| Differential test-case records | 10,515 | — |

Finding counts reflect the operator's verified tally, **not platform acceptance or payout figures**. Expected behavior, refuted leads, and unproven primitives are rigorously excluded from confirmed totals.

### What verification changed

Anonymized examples from recent validation and triage notes:

| Initial signal | What the verification checks established | How it is tracked in v-1.1 |
|---|---|---|
| Inconclusive HTTP timeout during state-change test | Network disruption interrupted the request; response body was truncated (`body_unavailable=true`). | Marked `incomplete=true`; hypothesis preserved as open rather than falsely marked refuted. |
| Static AST detection of potential injection sink | Source parameter is parsed through a strict schema sanitizer before reaching the sink; no dynamic execution demonstrated. | Recorded as static candidate with uncertainty score; no finding claimed without dynamic PoC. |
| Second-opinion review agreeing with finding | Reviewer model was an alias from the same provider family as the producing agent. | Flagged as family collision; review obligation held in pending state until an independent model family reviews. |
| Apparent auth bypass on mobile deep link | Exported intent handler accepts parameter, but target activity validates caller signature before processing data. | Refuted by reverse-engineering component inspection; recorded in investigation graph as refuted. |
| Credentialed differential test returning 200 OK | Both privileged and unprivileged sessions received HTTP 200, but payload contained a standard public login shell. | Correctly classified as public edge behavior, not broken object-level authorization. |

### Controlled validation for this release

All repairs and new components were exercised against offline, controlled local fixtures with zero live bug-bounty target traffic:

| Controlled validation suite | Checks passed | Scope & invariants exercised |
|---|---:|---|
| Go core CLI & MCP transport (`./mcp`, `./cmd/hbctl`) | 100% (All PASS) | Egress transport, session locking, credential expiry, host framing, LOC ceilings. |
| Real loopback execution integrity (`test_execution_integrity.py`) | 6 / 6 | Real `hb-mcp` -> `hb-egress` -> loopback proxy; credential isolation & body truncation. |
| Egress policy & scope boundary (`test_policy_boundary.py`) | 25 / 25 | Host allowlists, method restrictions, risk-tier gating, config-drift refusal. |
| Multi-identity session manager (`test_sessions.py`) | 21 / 21 | Cookie rotation, private jar persistence, HttpOnly export, cross-worker isolation. |
| Deterministic finding replay (`test_finding_replay.py`) | 27 / 27 | Typed assertion matching, response-field validation, rejection of error evidence. |
| Review checklist scaffolding (`test_review_scaffolding.py`) | 22 / 22 | Step uniqueness, artifact-line resolution, rejection of symlink escapes. |
| Persistent investigation graph (`test_pig.py`) | 21 / 21 | Referential integrity, state transitions, provenance history, adversarial dedup. |
| Deterministic research tools (`test_research_tools.py`) | 16 / 16 | Real Babel AST extraction, Semgrep pattern queries, offline artifact analysis. |
| Shared research primitives (`test_shared_primitives.py`) | 9 / 9 | Cross-agent observation sharing, primitive attribution, idempotent writes. |
| Cross-model peer review (`test_peer_review.py`) | 19 / 19 | Model-family independence, fail-closed disagreement, obligation recovery. |
| Autonomous loop cycle (`test_loop_cycle.py`) | 10 / 10 | Coverage-signature triggers, token ceiling stops, stagnation detection. |
| Orchestration & scheduler (`test_orchestration.py`) | 32 / 32 | Task leasing, concurrency bounds, failure quarantine, safe resumption. |
| Structured reporting pipeline (`test_report_pipeline.py`) | 15 / 15 | Lifecycle state machine, schema linting, credential redaction, video PoC integrity. |
| Model routing & admission (`test_model_admission.py`) | 23 / 23 | Role-based model selection, availability fallbacks, data-retention gates. |
| Account flow authorization (`test_account_flow.py`) | 6 / 6 | Controlled multi-tenant contrast, positive/negative session authorization. |

- **Zero live target traffic**: All suites executed against local mock servers, recorded HTTP fixtures, or disposable SQLite databases.
- **Zero model spend**: Verification used scripted and deterministic stand-ins; live provider calls were not required for regression validation.
- **Fail-closed guarantees**: Replay verifiers, egress gateways, and peer review gates confirmed 100% fail-closed semantics under error injection.

### Operational limits
- Mitmproxy body truncation during interrupted flows is recorded honestly (`body_unavailable=true`); unavailable bytes are never synthesized.
- Passing offline checklist verification does not guarantee live vulnerability discovery on target applications.
- Generic HTTP 200 responses do not prove authenticated identity recognition; target-specific data assertions remain required.
- Hard model-spend limits refuse execution when unsupported by native provider APIs; iteration and token caps act as backstops.

### Next focus
- Continuous live monitoring across authorized target programs using the new persistent lead loops.
- Benchmarking cost per validated finding across Gemini-first and frontier model configurations.
- Expanding offline mobile test coverage for hybrid app bridges and native library interfaces.

---

## 2026-11-02 — Post-v-1.0: Authenticated research & verification outcomes

**Development update.** Project Halberd now has controlled end-to-end evidence for authenticated browser,
HTTP and API work, alongside clearer coverage, investigation status and operator control. The practical
goal is not to generate more alarming responses: it is to distinguish a real finding from expected
behavior, a blocked test or a chain whose impact still depends on an unproven prerequisite.

This continues the **v-1.0 supervised R&D milestone**; it is not a new release or a claim of dependable
unattended vulnerability discovery.

### What the system is doing
- **Session-aware research** — explicitly selected, operator-owned sessions can be used across browser
  inspection and request comparisons while credentials remain private. Account identity is checked
  against actual returned data, not just a successful HTTP status.
- **More honest coverage** — completed work is preserved, missing inputs remain visible, and refreshed
  views no longer imply that an unperformed or blocked test succeeded.
- **Actionable investigations** — related signals are grouped, decisions retain their reasons, and
  follow-up work stays linked to the originating observation. A signal is not automatically a finding.
- **Deliberate model use** — routine work starts on smaller profiles; stronger planning and independent
  critique remain separate. Availability fallbacks are explicit, not a way around safety refusals.
- **Clearer supervision** — waiting, blocked, failed and active work are distinguished. Explicit
  reassignment preserves history instead of silently restarting failed investigations.

### Research snapshot

Anonymized cumulative activity, frozen at **2026-11-02 04:06 UTC**; the live run continues afterward.

| Measure | Recorded total |
|---|---:|
| Confirmed findings — operator tally | 1 |
| Informational findings — operator tally | 1 |
| Research task cards | 279 |
| Task cards marked done | 172 |
| API inventory rows | 842 |
| API baseline records | 5,912 |
| Differential test-case records | 10,515 |

The finding totals are the operator's current assessment, **not platform acceptance or payout figures**.
Activity rows include different identities, variants and outcomes; they are not counts of distinct
vulnerabilities, successful requests or fully tested endpoints. Expected-behavior, refuted and
conditional leads are not added to the confirmed total.

### What verification changed

Anonymized examples from existing triage notes and controlled validation, not new public bug disclosures:

| Initial signal | What the checks established | How it is tracked |
|---|---|---|
| Reflected cross-origin access headers | The observed responses did not grant credentialed sharing, and no authenticated cross-origin data read was demonstrated. | Not a confirmed credentialed data leak; no claim beyond the checked responses. |
| A success-like response where access had been refused | The response was an edge-security challenge rather than protected application content; negative controls matched it. | False positive for authorization bypass, not a confirmed finding. |
| A credentialed cross-origin chain | Supporting behavior was recorded, but script execution on a trusted origin and the authenticated read pre-check remained unproven. | Conditional impact; no additional confirmed finding claimed. |
| A successful HTTP status during an identity check | In the controlled lab, an empty user response also had a successful status. Owned identities were distinguished using returned account data and negative controls. | Expected protocol behavior, not an authentication bypass. |

The useful outcome is sometimes a closed lead or a narrower claim. A recorded primitive is not the
same as a demonstrated attack chain, and agreement between models does not supply a missing prerequisite.

### Controlled validation for this work

Separate from the research totals above:

| Controlled check | Result |
|---|---:|
| Acceptance criteria passed in independent review | 12 / 12 |
| Separately owned test identities | 2 |
| Explicitly approved authentication actions, each used once | 2 |
| Authenticated baseline requests completed | 20 |
| Authenticated differential requests completed | 12 |
| Recorded evidence files passing integrity checks | 48 |

- Positive owned-account and unauthenticated contrasts were exercised through browser, HTTP and API
  consumers. Authentication actions were not retried, and no additional write was performed.
- Fast validation and selected regression checks passed. The full slower suite was **not** rerun.
- Availability fallback was checked with a controlled provider-boundary failure and real downstream
  handoffs; this was not an experiment that forced a live provider outage.
- Independent completion review passed. Its documentary follow-ups were applied. This validates the
  exercised machinery, **not a vulnerability-detection success rate or production-readiness guarantee**.

### Next focus
- Keep operator finding decisions and generated summaries aligned.
- Measure fresh reproduction quality and cost per validated result, rather than raw signal volume.
- Continue controlled evaluation without publishing target, account, credential or exploit details.

---


## 2026-10-30 — v-1.0: Coordinated research, shared evidence & execution reliability

**Milestone.** The first versioned development milestone brings together the October 28–30 runtime,
research-state and orchestration work. Hackbot can coordinate concurrent specialist tasks, preserve
shared discoveries and corrections, and give the operator a clearer view of readiness, failures and
unfinished work. This is a **supervised R&D milestone**, not a claim of production readiness or
dependable unattended vulnerability discovery.

### Added
- **Lead-aware concurrent execution** — specialist tasks run under their owning leads, with explicit
  dependencies, bounded concurrency and operator priorities. Leads return to evaluate new child results
  instead of being rerun unconditionally.
- **Background operation & steering** — the operator can pause, resume, reprioritize, instruct, cancel
  or stop work while unrelated investigations continue. Workspace views refresh as runs finish.
- **Unified startup report** — read-only checks show readiness, unfinished work, model routes and
  whether to start, attach to an existing run, wait for instructions or resolve a blocker.
- **Shared research memory** — all seven leads can contribute and retrieve program-local observations
  and reusable primitives, retaining attribution and correction history.
- **API investigation workflow** — a unified inventory connects discovery sources, identity-aware
  baselines, controlled differential cases, evidence references and explicit incomplete or gated outcomes.
  Anomalies remain investigation candidates, not automatically confirmed vulnerabilities.
- **Bounded browser integration** — authorized workers can reuse a validated operator-owned browser
  for read-only navigation and limited diagnostics without receiving unrestricted browser tools.
- **Layered independent critique** — finding triage now includes specialist, reproduction and impact
  critiques. Evidence-bearing specialist results can also receive a separate cross-family peer opinion.
- **Explicit model routing & report learning** — workers follow saved operator model roles with named
  availability fallbacks. A human-gated local learning workflow preserves report outcomes and corrections;
  it does not submit reports, train models or automatically adopt changes.

### Fixed
- Nested planner results are preserved and schema-checked; failed commands and database reads surface
  as failures rather than successful empty work.
- Per-task leases prevent duplicate launches. Interrupted tasks may resume once; failed or repeatedly
  interrupted work is quarantined without stopping unrelated eligible tasks.
- Shared-record readers and projections preserve original provenance, corrections and independently
  discovered inventory. Terminal agent records resist replacement, and checklist changes retain history.
- Proxy lifecycle checks verify process ownership before reuse or teardown. Model-attempt records
  preserve usage and producer information without double-counting linked summaries.

### Validation & limits
- The offline benchmark passed **all six scenarios**: concurrent work with lead relaunch and failure
  isolation; one-time interruption recovery; failed-run quarantine across restarts; waiting for an
  in-flight child; cross-family peer work; and append-only record protections. It completed in **24.2
  seconds** on the local workstation, with no model calls or target traffic.
- Controlled-lab acceptance records include passing API-engine, orchestration and startup reviews.
  The peer-review implementation has fixes and regression evidence after a **CHANGES** verdict, but
  no subsequent review round is recorded. These records are not a finding-quality benchmark.
- **Budget clarification:** configured hard model-spend limits currently refuse launch when native
  enforcement cannot be established. Omitted limits mean uncapped execution; iteration bounds and
  nominal token settings are not a spend guarantee. This supersedes the stronger October 27 wording.
- Model agreement and stored-evidence integrity checks are not fresh reproduction or demonstrated
  impact. Worker isolation is configuration/tool isolation, not an operating-system sandbox.
- A fast test mode and offline benchmark exist; broader acceptance-policy simplification remains
  proposed. Historical acceptance records do not constitute a new full-suite pass for this update.

### Rough next steps
- Make review dispositions a dependable gate for dependent work.
- Strengthen fresh reproduction and class-specific impact proof before confirming findings.
- Establish enforceable model-spend bounds and safer continuation after interrupted side effects.
- Extend controlled live evaluation without treating role ownership or review coverage as detection accuracy.

---

## 2026-10-27 — Bounded autonomous loop, operator workspace & programmatic report linting ✅

**Milestone.** The orchestration layer reached end-to-end maturity. The automation can now execute bounded autonomous testing loops with automatic stagnation detection, project complete state and evidence into an operator dashboard graph, and programmatically verify vulnerability reports before human sign-off.

### Added
- **Bounded autonomous testing loop** — orchestrates cycles (think → assign → dispatch → execute → gate) with strict iteration caps, per-run token limits, and no-progress early stopping to guarantee bounded spend.
- **Operator workspace & provenance graph** — projects run records, evidence hashes, and coverage status into a unified visual graph and dashboard, maintaining 100% auditability without model guesswork.
- **Programmatic report verification ("report-lint")** — a deterministic verifier that enforces standardized report structure, validates all cited attachments, redacts accidental secrets/credentials, and flags prohibited AI speech markers.
- **Independent cross-model review** — findings produced by one model family must pass an independent second-opinion review conducted by an entirely different model family, failing closed on disagreement.

### Fixed
- **CLI version precedence resolution** — resolved an environment path conflict where legacy local CLI binaries shadowed updated packages, ensuring reviewer checks run with supported runtime capabilities.
- **Proxy configuration-drift recovery** — hardened the enforcement gateway to recover cleanly from mid-flight configuration adjustments without silently dropping valid test passes.

### Rough next steps
- Multi-provider model diversity: integrate alternative local and hosted model runtimes behind strict data-retention authorization gates.
- Deepen per-vulnerability-class proof criteria for automated finding-triage gates.

---

## 2026-10-26 — Attack surface expansion, auth-bypass & cache-testing families ✅

**Milestone.** Substantially expanded the automated procedure catalog across high-impact vulnerability classes, and instituted a durable submission ledger tracking vulnerability validity independently from platform decisions.

### Added
- **Authentication bypass testing family** — reviewable playbooks covering modern authentication primitives: SAML assertions, OAuth 2.0 grant types, JWT integrity checks, MFA validation sequences, and password-reset invariants.
- **Web cache deception & unkeyed input suite** — automated detectors for cache-key discrepancies, unkeyed header reflections, and path delimiter misinterpretations.
- **DNS rebinding detection** — automated dual-stage prober pairing deterministic network checks with headless browser confirmation to safely identify host-validation weaknesses.
- **Submission & duplicate intelligence ledger** — a structured ledger recording all findings, platform outcomes, and root-cause analyses to systematically benchmark automation accuracy and improve coverage.

### Fixed
- **Scoped planning model calibration** — tightened planning prompt bounds to eliminate empty candidate leads and enforce strict adherence to target scope definitions.

### Rough next steps
- Bounded autonomous orchestration loop to pump task queues automatically.
- Unified operator dashboard and evidence provenance graph.

---

## 2026-10-25 — Specialized multi-agent hierarchy & tool-bounded execution ✅

**Milestone.** Decomposed the monolithic test runner into a disciplined hierarchy of specialized agents. Each agent operates under strict least-privilege constraints with deterministic checklists to prevent uncontrolled tool invocation.

### Added
- **Domain-specialized agent hierarchy** — introduced 5 discipline leads (Reconnaissance, JavaScript analysis, API endpoints, Client-side behavior, Server-side logic) coordinating 29 targeted worker subagents.
- **Enforced tool boundary (least privilege)** — worker subagents are strictly restricted to small, audited tool allowlists (≤14 tools per agent) with zero raw network or shell access. All target interaction must traverse the authorized enforcement gateway.
- **Mandatory checklist gates** — every subagent task is governed by an explicit checklist; tasks cannot resolve as complete until each item is deterministically proven or refuted with recorded evidence hashes.

### Confirmed by validation
- Subagent boundaries hold firm: agents cannot invoke tools outside their defined manifest, and state-changes without explicit operator approval remain impossible.
- Evidence integrity is preserved: every request/response pair generated during agent execution is captured as tamper-evident content-addressed storage.

### Rough next steps
- Add procedures for advanced authentication and caching vulnerability classes.
- Build autonomous task pumping and state graph projection.

---

## 2026-10-24 — Offline test harness for failure & verification paths ✅

**Milestone.** The hardest-to-test parts of the system — what happens when the AI provider misbehaves,
and how the verification stage treats a reviewer's verdict — can now be exercised **offline,
deterministically, with no model spend**, using a stand-in that imitates the provider.

### Added
- **Simulated-provider test harness** — a stand-in for the AI provider that returns scripted outcomes
  (normal result, provider error, garbled/truncated response, missing output) so the system's error
  handling can be tested repeatably and for free, instead of only against a live paid model.

### Confirmed by the new tests
- Provider failures **surface honestly and never corrupt a run's state** — a bad or missing response is
  turned into a clean, typed failure, not a silent wrong answer.
- The verification stage handles a reviewer's verdict correctly: a rejected finding stays unconfirmed,
  and it **fails closed** — if the independent review flags a serious gap, the finding is not confirmed
  even when the top-line verdict looked positive.

_Scope note: these tests validate how the system **handles** provider failures and review verdicts, not
the judgement quality of a real model — that still needs live runs, done under an explicit budget._

### Rough next steps
- Richer, longer-lived application knowledge that survives across runs.
- A small, budget-capped live run to validate real-model judgement end to end.

---

## 2026-10-24 — Resilience + readiness preflight ✅

**Milestone.** The run itself got harder to break. The system can now notice when a run is stuck,
pick up safely after an interruption without redoing finished work, refuse to run the same job twice
at once, and confirm the whole setup is healthy **before** spending anything.

### Added
- **Stall detection** — a run that is alive but making no progress past a time bound is flagged for the
  operator. It is never force-killed (that would throw away work already done); the operator decides.
- **Safe resume + single-run guard** — re-running skips work already completed for the current state,
  refuses to start a second copy of a job that is already in flight, and recovers cleanly if an earlier
  run died partway through (a half-finished run is treated as unfinished, not as done).
- **Readiness preflight ("doctor")** — one command checks the whole chain before a run: configuration is
  valid, the enforcement gateway and supporting pieces are reachable, and the required components are
  present. It fails loudly and clearly if anything is missing, so no budget is spent on a half-ready setup.

### Fixed
- A latent liveness-check bug: a job lock held by another *live* process could be mistakenly treated as
  stale and reclaimed. Now a live holder is always respected; only a genuinely dead one is cleared.

### Rough next steps
- Richer, longer-lived application knowledge that survives across runs.
- Broaden automated coverage across more vulnerability classes.

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

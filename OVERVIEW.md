# Hack Bot (hb) — High-Level Overview

**Status:** Active R&D / Operational Testing  
**Last Updated:** 2026-11-15  
**Scope:** Architecture summary, bug counts, current issues, and applied fixes.

---

## 1. System Architecture & Operation

Hack Bot (`hb`) is a disciplined, safety-first automation harness designed for authorized, scope-bounded security testing (bug-bounty and penetration-testing workflows). It couples deterministic tooling with bounded model-driven reasoning:

```
┌────────────────────────────────────────────────────────────────────────┐
│                   Operator / hbctl MCP / /hackbot                      │
└──────────────────────────────────┬─────────────────────────────────────┘
                                   │
       ┌───────────────────────────┴───────────────────────────┐
       ▼                                                       ▼
┌─────────────────────────────┐         ┌─────────────────────────────┐
│  Deterministic Core         │         │  Specialized Subagents      │
│  - AST Parsing & Semgrep    │◄───────►│  - 5 Lead Loops, 29 Subs    │
│  - REA Mobile & APK Auditing│  Event  │  - ≤14 tools, checklists    │
│  - API Differential Engine  │   Bus   │  - AL-01..AL-22 Mobile Leads│
│  - Report State Machine     │         │  - Gemini-First Reporting   │
└──────────────┬──────────────┘         └──────────────┬──────────────┘
               │                                       │
               └───────────────────┬───────────────────┘
                                   ▼
       ┌───────────────────────────────────────────────────────┐
       │         Persistent Shared Investigation Graph (PIG)   │
       │   Typed Nodes · Content-Addressed Evidence Provenance │
       │   Knowledge States: observed | hypothesis | validated │
       └───────────────────────────┬───────────────────────────┘
                                   ▼
       ┌───────────────────────────────────────────────────────┐
       │                  Enforcement Gateway                  │
       │   Scope · Rate Limit · Spend Budget · Tamper-Evident  │
       │         Fails Closed (No raw net/shell access)        │
       └───────────────────────────┬───────────────────────────┘
                                   ▼
                           Authorized Target
```

### Core Operating Principles
1. **Single Enforced Gateway:** All target-bound traffic flows through a dedicated, local mitmproxy egress instance. Any out-of-scope host, disallowed HTTP method, or unbudgeted request is refused locally.
2. **Deterministic-First Pipeline:** Reconnaissance, JavaScript collection, source-map recovery, AST analysis, and differential API tests run deterministically without model spend. AI models are reserved strictly for interpretation, prioritization, and targeted analysis.
3. **Strict Tool Least-Privilege:** Worker subagents never receive shell access or direct networking. Subagents receive at most 14 audited tools via `hb-mcp`, gated by the gateway.
4. **Persistent Investigation Graph (PIG):** Observations, endpoints, dataflows, trust boundaries, hypotheses, primitives, and gadgets are tracked as typed nodes and relationships with cryptographic provenance back to verified evidence hashes.
5. **Event-Driven Cross-Agent Discovery:** Typed discovery events (e.g. unmapped endpoints or auth boundaries revealed by JavaScript analysis) automatically notify relevant leads without requiring monolithic reprompting.
6. **Cross-Model Independent Review:** Findings produced by one model family must pass an independent second-opinion review conducted by an entirely different model family, failing closed on disagreement.

---

## 2. Bug Count Summary

Testing and validation across targets and internal infrastructure have identified two distinct categories of bugs: **Target Security Vulnerabilities** found by the harness on authorized programs, and **Internal Harness & Infrastructure Bugs** discovered and resolved within Hack Bot itself.

### A. Target Findings (Total: 13 Leads Tracked)

All target and program detail is anonymized. A recorded primitive or candidate signal is not counted as a finding until its impact is demonstrated end to end.

| Outcome | Count | Notes |
|---|---:|---|
| **Reported — major** | **1** | High-severity finding, verified end to end and reported. |
| **Informational** | **1** | Low-impact observation, documented. |
| Unconfirmed primitives / conditional leads | 6 | Impact depends on an unproven prerequisite (session, scope permission or companion gadget). Tracked, not claimed. |
| Refuted | 5 | Tested against negative controls and closed; prevented false reports. |
| **Total leads tracked** | **13** | |

Finding totals are the operator's current assessment, not platform acceptance or payout figures.

### B. Internal Harness & Infrastructure Bugs Identified & Fixed (Total: 26 Bugs Resolved)

| Bug ID | Component | Description | Impact | Resolution |
|---|---|---|---|---|
| **BUG-01** | Port Assignment | Default port `:8888` shared across sessions without `HB_EGRESS` check. | Requests from one program could cross-pollinate into another program's egress. | Fixed via program-specific port allocation (`state/chain.port`). |
| **BUG-02** | Egress Auditing | Host voice AI curls hitting `localhost:8888` (bound by `hb-egress`). | 583 benign system requests polluted program traces across three programs. | Isolated egress port and added exclusion filter. |
| **BUG-03** | Gateway Lifecycle | Egress returned `403 Risk Gate: config changed` upon mid-run edits. | Finder passes ran "blind" and halted without notifying operator. | Fresh gateway PID recycling & automated reload handling. |
| **BUG-04** | Model Invocation | `runtime.py` raised `NotImplementedError` for OpenCode runtime. | DeepSeek Flash was unreachable; system was 100% dependent on single provider. | Architected task `t-031` with standalone runner & retention gates. |
| **BUG-05** | Environment / PATH | Outdated global Codex binary (`0.139.0`) shadowed current NVM install (`0.164.0`). | Multi-model validator failed closed due to unsupported model flags. | Aligned PATH resolution to enforce >= 0.164.0 (`t-029`). |
| **BUG-06** | Concurrency Guard | Stale lock clearance mechanism prematurely evicted live running processes. | Potential race condition when parallel passes accessed workspace state. | Rewrote liveness check to inspect PID status and process creation time. |
| **BUG-07** | Task Dispatch | 53 of 79 finder cards had no assigned subagent in the loop routing matrix. | Finder skills fell through as "unassigned" during autonomous loop passes. | Added explicit card-to-subagent ownership mapping (`t-028`). |
| **BUG-08** | Report Quality | Reports varied in structure, leaked API keys/tokens, and used artificial phrasing. | Risk of submitting unredacted credentials or unpolished reports. | Built deterministic `hb report-lint` verifier (`t-032`). |
| **BUG-09** (E1) | Auth Validation | Expired credentials or preference cookies masking expired authentication. | Stale requests hitting targets with unauthenticated fallback. | Pre-network credential validation rejecting expired tokens before network (`t-091`). |
| **BUG-10** (E2) | Session Isolation | Concurrent workers conflicting on credential rotation or revocation. | Race conditions and token invalidation across concurrent workers. | Per-principal atomic session locking with private cookie-jar persistence (`t-091`). |
| **BUG-11** (E3) | HTTP Transport | HTTP redirects mutating POST bodies or transforming GraphQL queries. | Erroneous request transforms during redirect following. | Strict redirect refusal; original POST bodies and headers preserved intact (`t-091`). |
| **BUG-12** (E4) | Evidence Integrity | Upstream network truncation or body disconnects masked as success. | Truncated responses falsely treated as empty successful responses. | Preserved hash-valid error evidence; marked `complete=false` and `body_unavailable=true` (`t-091`). |
| **BUG-13** (E5) | Protocol Framing | Model-injected control headers and Host framing conflicts. | Risk of internal gateway header leakage or host spoofing. | Pre-network host validation; internal control headers stripped before target (`t-091`). |
| **BUG-14** (I1) | Hypothesis Logic | Inconclusive tool executions falsely marking hypotheses as refuted. | Valid research hypotheses prematurely discarded on transient errors. | Preserved open hypotheses on inconclusive runs; refutation requires explicit proof (`t-091`). |
| **BUG-15** (I2) | Investigation Context| Task experiments detached from originating hypothesis IDs. | Loss of strategic context and inability to trace negative controls. | Bidirectional task-hypothesis linkage preserving full decision history (`t-091`). |
| **BUG-16** (I3) | Loop Deduplication | Repetitive tasks looping indefinitely on identical inputs. | Unproductive token spend without test variation. | Intent deduplication allowing re-dispatch only on material evidence revision (`t-091`). |
| **BUG-17** (O1) | Finding Replay | Loose string matching and request-only assertions unlocking verification. | Weak evidence verification passing finding triage. | Strict field-specific assertion matching; request-only assertions denied (`t-091`). |
| **BUG-18** (O2) | Checklist Gates | Incomplete checklists or broken artifact references passing completion review. | Incomplete tasks signed off without verified evidence. | Authoritative 100% unique step verification with nonempty line resolution (`t-091`). |
| **BUG-19** (O3) | Reviewer Independence| Model family collisions during independent second-opinion review. | Self-review and correlated failure modes across reviews. | Enforced true model family responder independence (`t-091`). |
| **BUG-20** (O4) | Review Durability | In-flight review obligations lost across process crashes or restarts. | Unfinished peer reviews orphaned in limbo. | Durable canonical review obligations persisted in SQLite (`t-091`). |
| **BUG-21** (G1) | Static Analysis | Static AST patterns treated as confirmed runtime vulnerabilities. | Premature vulnerability claims based on static code patterns. | Candidate entities projected with explicit uncertainty metrics (`t-091`). |
| **BUG-22** (G2) | Graph Provenance | Repeated agent writes creating duplicate primitive entities. | Knowledge base bloat and divergent graph state. | Idempotent writes preserving multi-agent contributor history (`t-091`). |
| **BUG-23** | Model Routing | Hardcoded Haiku defaults preventing model diversity and higher-tier writing. | Expensive models used for routine notes; missed opportunity for cheap writing. | Dynamic role routing with Gemini-first note/report writing (`t-085`). |
| **BUG-24** | Browser Isolation | Browser sessions borrowing personal user Chrome profile data. | Accidental session leakage and credential cross-contamination. | Dedicated, program-isolated browser profiles with Interceptor driver (`t-083`). |
| **BUG-25** | Mobile Coverage | Monolithic web testing missing mobile-specific attack surfaces. | Inability to inspect Android APKs, deep links, or IPC interfaces. | Integrated 22 Android vulnerability classes and pinned REA pipeline (`t-084`, `t-087`). |
| **BUG-26** | Report Lifecycle | Unenforced report state machine allowing unapproved drafts to leak. | Inconsistent report quality and lack of formal operator sign-off. | Code-enforced report state machine with headless video PoC capture (`t-075`). |

---

## 3. Current Issues & Technical Blockers

```
┌────────────────────────────────────────────────────────────────────────┐
│                             CURRENT ISSUES                             │
├────────────────────────────────┬───────────────────────────────────────┤
│ 1. Upstream Body Disconnects   │ Mitmproxy error hook discards early   │
│                                │ body bytes (`body_unavailable=true`). │
├────────────────────────────────┼───────────────────────────────────────┤
│ 2. Application Identity Proof  │ Generic HTTP 200 does not prove user  │
│                                │ identity; app assertions required.    │
├────────────────────────────────┼───────────────────────────────────────┤
│ 3. Spend Limit Enforcement     │ Configured hard spend caps require    │
│                                │ native provider token APIs.           │
├────────────────────────────────┼───────────────────────────────────────┤
│ 4. Mobile Dynamic Hooks        │ Mobile analysis is offline static;    │
│                                │ live Frida hooks require manual gate. │
└────────────────────────────────┴───────────────────────────────────────┘
```

1. **Proxy Body Truncation Handling (`BUG-12`):**  
   - *Symptom:* When an upstream server disconnects mid-stream, mitmproxy's error hook can drop partial body chunks before recording.
   - *Current Mitigation:* Evidence explicitly records `body_unavailable=true` and `complete=false`. These records fail closed and cannot verify a finding. Full recovery would require a raw streaming capture.

2. **Application-Specific Identity Verification:**  
   - *Symptom:* Generic HTTP transport checks confirm cookie applicability and HTTP 200, but cannot know whether the application backend recognized the intended identity.
   - *Requirement:* Positive tenant-specific data assertions remain mandatory before confirming access-control claims.

3. **Provider Native Spend Limits:**  
   - *Symptom:* Hard monetary spend limits refuse execution when the underlying model provider API does not support native real-time balance queries.
   - *Mitigation:* Loop-level token caps and iteration limits serve as deterministic backstops.

4. **Mobile Runtime Instrumentation Boundary:**  
   - *Symptom:* Mobile analysis via REA and ADB is strictly scoped to offline decompilation, manifest auditing, and reverse-port loopback proxying.
   - *Boundary:* Dynamic Frida hooking and fuzzing remain unautomated and gated on explicit human operator review.

---

## 4. Implemented Fixes & Upgrades

The following foundational upgrades and bug fixes have been completed and verified on disk:

### 1. Persistent Shared Investigation Graph (PIG — Task `t-072`)
- **Typed Schema:** 20 controlled entity types (`schemas/pig.json`) including assets, endpoints, parameters, dataflows, trust boundaries, hypotheses, findings, primitives, and gadgets.
- **Strict Knowledge States:** Enforces `observed`, `hypothesis`, `validated`, `refuted`, and `unknown`. Observed entities require verified content-addressed evidence hashes; validated entities require human operator sign-off.
- **Shared Query & Ingest:** Multi-agent memory enables cross-discipline context sharing while preventing entity duplication through canonical key deduplication.

### 2. Autonomous Lead Loops & Discovery Bus (Tasks `t-073` & `t-074`)
- **Persistent Investigation Loops:** Implements the `OBSERVE -> FORM_HYPOTHESIS -> PRIORITIZE -> PLAN -> DELEGATE -> EXECUTE -> EVALUATE -> RECORD -> REPLAN` cycle for all 5 team leads.
- **Event-Driven Discovery:** Discovery events (new endpoints, auth boundaries, client behaviors) automatically fan out across leads without manual orchestration.

### 3. Adversarial Hardening & Execution Integrity (Task `t-091`)
- **18 Defect Repairs:** Resolved subtle bugs across HTTP transport, credential isolation, redirect handling, hypothesis preservation, checklist verification, and cross-model reviewer independence.
- **Offline Validation:** Verified across 15 regression suites and 250+ deterministic checks with zero live target traffic.

### 4. Deterministic Static Tools & Reverse Engineering (Tasks `t-077`, `t-084`, `t-087`)
- **Zero-Spend Code Analysis:** Deterministic Babel AST analysis and Semgrep pattern matching extract endpoints and sources/sinks offline.
- **Mobile Reverse Engineering (REA):** Offline APK manifest auditing, deep-link inspection, and reverse-proxy ADB integration.
- **Android Domain Specialization:** 22 structured mobile vulnerability classes (AL-01 to AL-22) integrated into application surface exploration.

### 5. Multi-Provider Dynamic Model Routing (Task `t-085`)
- **Cost-Optimized Allocation:** Routes routine note-taking and structured documentation to Gemini-first profiles, reserving frontier models for strategic planning.
- **Strict Reviewer Isolation:** Enforces model family independence between finding producers and second-opinion reviewers.

### 6. State-Machine Vulnerability Reporting (Task `t-075`)
- **Lifecycle Enforcement:** Code-enforced progression (`DRAFT -> EVIDENCE_REQUIRED -> READY_FOR_REVIEW -> APPROVED -> SUBMITTED`).
- **Reproducible Evidence:** Automated screenshot/headless video capture, attachment hash verification, and automated secret redaction.

---

## 5. Summary Metrics

```
┌──────────────────────────────────────┬───────────────────────────────┐
│ Metric                               │ Value                         │
├──────────────────────────────────────┼───────────────────────────────┤
│ Target Leads Tracked                 │ 13 total                      │
│ Reported Findings                    │ 1 major + 1 informational     │
│ Unconfirmed Primitives / Conditional │ 6 tracked, not claimed        │
│ Refuted False Positives              │ 5 refuted                     │
│ Internal Harness Bugs Resolved       │ 26 resolved (inc. 18 in t-091)│
│ Research Task Cards Managed          │ 317 cards (185 completed)     │
│ Persistent Graph Events (`pig`)      │ 33,672 events recorded        │
│ Mapped Graph Nodes & Edges           │ 14,192 nodes / 18,548 edges   │
│ Cross-Agent Discovery Events         │ 689 events dispatched         │
│ Lifetime Scope-Compliant Requests    │ 2,264 allowed (0 out-of-scope)│
│ Evidence Storage Integrity           │ 100% content-addressed        │
│ Available Disciplines / Subagents    │ 5 Leads / 29 Subagents        │
│ Available Mobile Vulnerability Cards │ 22 Android cards (AL-01..22)  │
│ Shipped Finder Skill Cards           │ 79 cards                      │
└──────────────────────────────────────┴───────────────────────────────┘
```

*This document is maintained as part of Project Halberd progress tracking.*
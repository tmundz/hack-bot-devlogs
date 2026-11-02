# Hack Bot (hb) — High-Level Overview

**Status:** Active R&D / Operational Testing  
**Last Updated:** 2026-11-02  
**Scope:** Architecture summary, bug counts, current issues, and applied fixes.

---

## 1. System Architecture & Operation

Hack Bot (`hb`) is a disciplined, safety-first automation harness designed for authorized, scope-bounded security testing (bug-bounty and penetration-testing workflows). It couples deterministic tooling with bounded model-driven reasoning:

```
┌────────────────────────────────────────────────────────────────────────┐
│                        Operator / hbctl MCP                             │
└──────────────────────────────────┬─────────────────────────────────────┘
                                   │
               ┌───────────────────┴───────────────────┐
               ▼                                       ▼
  ┌────────────────────────┐              ┌────────────────────────┐
  │  Deterministic Core    │              │  Specialized Subagents │
  │  Recon, JS, API diff,  │              │  5 Leads, 29 Subs      │
  │  Organizer, Workspace  │              │  ≤14 tools, checklists │
  └────────────┬───────────┘              └────────────┬───────────┘
               │                                       │
               └───────────────────┬───────────────────┘
                                   ▼
          ┌─────────────────────────────────────────────────┐
          │               Enforcement Gateway               │
          │  Scope · Rate Limit · Spend Budget · Evidence   │
          │         Fails Closed (No raw net/shell)         │
          └────────────────────────┬────────────────────────┘
                                   ▼
                         Authorized Target
```

### Core Operating Principles
1. **Single Enforced Gateway:** All target-bound traffic flows through a dedicated, local mitmproxy egress instance. Any out-of-scope host, disallowed HTTP method, or unbudgeted request is refused locally.
2. **Deterministic-First Pipeline:** Reconnaissance, JavaScript collection, source-map recovery, and differential API tests run deterministically without model spend. AI models are reserved strictly for interpretation, prioritization, and targeted analysis.
3. **Strict Tool Least-Privilege:** Worker subagents never receive shell access or direct networking. Subagents receive at most 14 audited tools via `hb-mcp`, gated by the gateway.
4. **100% Traceability:** Every allowed request and response pair is hashed and stored in content-addressed storage (`state/evidence/<sha256>.json`).
5. **Cross-Model Independent Review:** Findings produced by one model family must pass an independent second-opinion review conducted by a different model family, failing closed on disagreement.

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

### B. Internal Harness & Infrastructure Bugs Identified & Fixed (Total: 8 Bugs)

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

---

## 3. Current Issues & Technical Blockers

```
┌────────────────────────────────────────────────────────────────────────┐
│                             CURRENT ISSUES                             │
├────────────────────────────────┬───────────────────────────────────────┤
│ 1. Multi-Model Runtime Gap     │ OpenCode / DeepSeek Flash lacks paid  │
│                                │ quota; free-tier quota exhausted.     │
├────────────────────────────────┼───────────────────────────────────────┤
│ 2. Data Retention Compliance   │ Third-party free tiers retain data;   │
│                                │ need strict provider-retention gate.  │
├────────────────────────────────┼───────────────────────────────────────┤
│ 3. Target Credential Rotation  │ Secondary test identity expired;      │
│                                │ cross-tenant tests paused.            │
├────────────────────────────────┼───────────────────────────────────────┤
│ 4. Scope Method Boundary       │ Egress scope is read-only (GET/HEAD); │
│                                │ unauth POST verify probe blocked.     │
├────────────────────────────────┼───────────────────────────────────────┤
│ 5. Unassigned Finder Cards     │ Long-tail finder cards need explicit  │
│                                │ subagent mappings in `agents/*.md`.   │
└────────────────────────────────┴───────────────────────────────────────┘
```

1. **Model Diversity Blocked on OpenCode / DeepSeek Quota (`t-031`):**  
   - *Symptom:* The harness relies almost exclusively on `claude-haiku-5-5` for finders because `opencode/deepseek-v4.1-flash-free` reports quota exhaustion and `ollama/deepseek-v4.1-flash:cloud` reports model unavailable.
   - *Impact:* Evaluator cannot benchmark cheap local/alternative models against Claude.
   - *Requirement:* A keyed provider (DeepSeek API key, OpenCode Go subscription, or Ollama auth).

2. **Data-Retention Policy Enforcement (`t-031`):**  
   - *Symptom:* Free model tiers do not guarantee Zero Data Retention (ZDR).
   - *Resolution:* Introducing `retention = "provider"` metadata in profiles and requiring `[authorization] allow_retaining_models = true` in program configuration before any non-ZDR model can be invoked.

3. **Target Credential Expiration:**  
   - *Symptom:* The authentication token for a secondary test identity expired.
   - *Impact:* Cross-tenant comparison tests on the affected program cannot run automatically until refreshed.

4. **Egress Scope Method Boundary on a Pending Verification Probe:**  
   - *Symptom:* A pending verification probe requires an unauthenticated `POST` request. The program's scope policy strictly enforces `methods = ["GET", "HEAD", "OPTIONS"]`.
   - *Impact:* The automated model cannot fire the definitive verification probe; operator must explicitly authorize `POST` and restart egress.

5. **Subagent Skill Coverage Distribution:**  
   - *Symptom:* 53 out of 79 finder cards lack an explicit owning subagent.
   - *Impact:* When the thinker proposes these cards, the loop flags them as "unassigned" instead of routing them to a worker.

---

## 4. Implemented Fixes & Upgrades

The following foundational upgrades and bug fixes have been completed and verified on disk:

### 1. Programmatic Report Verifier (`hb report-lint` — Task `t-032`)
- **Deterministic Validation:** Operates completely offline without model spend.
- **Structural Enforcement:** Verifies H1 title, Asset/Severity/Weakness tables, CVSS vector presence, and standard sections (Summary, Impact, Steps, Fix, Attachments).
- **Tone & Voice Policing:** Disallows AI clichés ("delve", "leverage", "seamless", "it is important to note") and enforces active human voice.
- **Credential Redaction:** Scans reports and attachments for accidentally unredacted API keys, JWTs, and session tokens.
- **Attachment Verification:** Confirms all cited evidence files exist on disk before submission.

### 2. Bounded Autonomous Loop (`hbctl loop` — Task `t-028`)
- **Closed Execution Pipeline:** Thinker proposing cards → Deterministic assignment to subagents → Task dispatch via `hb-mcp` → Execution pump → Validation gate.
- **Stagnation & Budget Protection:** Halts immediately if remaining daily requests reach zero, if token ceilings are exceeded, or if an iteration makes no forward progress.
- **Single-Run Guard:** Prevents duplicate tasks for already-evaluated (agent, skill, target) tuples.

### 3. Operator Workspace & Provenance Graph (`hbctl workspace` — Task `t-027`)
- **Disposable Projection:** Deterministically compiles `hb.db` tasks, evidence files, agent notes, and coverage records into a unified operator dashboard (`dashboard.md`) and graph (`graph.md`, `graph.json`).
- **Audit Compliance:** Enables the operator to trace any finding directly back through every intermediate hypothesis, prompt, tool execution, and evidence hash.

### 4. Cross-Model Finding Reviewer (`hb review` — Task `t-026`)
- **Vendor Independence:** Gated finding verification using an independent model family (Codex) to review Claude-generated findings.
- **Fail-Closed Semantics:** If the review packet cannot prove evidence integrity or if the reviewer encounters errors, the finding remains strictly unconfirmed.

### 5. Multi-Agent Specialization & Tool Sandboxing (Tasks `t-014` to `t-019`)
- **5 Disciplines:** Partitioned testing into Recon, JavaScript, API, Client-side, and Server-side leads.
- **Tool Allowlist Gate:** Enforced tool maximums (≤14 tools per agent) verified at the MCP layer; subagents are physically prevented from executing unauthorized commands or raw network sockets.

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
│ Internal Harness Bugs Fixed          │ 8 resolved                    │
│ Lifetime Scope-Compliant Requests    │ 2,264 allowed (0 out-of-scope)│
│ Evidence Storage Integrity           │ 100% content-addressed        │
│ Available Disciplines / Subagents    │ 5 Leads / 29 Subagents        │
│ Available Finder Skill Cards         │ 79 cards                      │
└──────────────────────────────────────┴───────────────────────────────┘
```

*This document is maintained as part of Project Harvester progress tracking.*
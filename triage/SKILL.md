---
name: triage
description: Diagnose codebase issues from ticket text. Use when given a bug report, crash, test failure, unexpected behavior, or integration issue to investigate. Traces the codebase autonomously, confirms root cause, then fixes in the same session.
---

# Triage — Code Diagnosis

Given a ticket, trace the codebase to root cause, then fix. No project management. No handoff. One session.

## Scope

**In:** correctness bugs, unexpected behavior, test failures, crashes, integration issues, latency regression, resource exhaustion (memory leak, connection pool, GC pressure)
**Out:** performance optimization requests, security audits, architecture reviews, feature requests, infra/DevOps issues

## Invocation

Paste ticket text (ADO, Jira, or freeform) after `/triage`. Claude classifies, investigates, diagnoses, and fixes.

---

## Phase 1 — Classify

Identify the issue type before touching any code. Ask the questions below — answers determine investigation order.

| Type | Signal |
|---|---|
| Stack trace / crash | Error message, exception, unhandled rejection |
| Unexpected behavior | "X should do Y but does Z" |
| Test failure | Failing assertion, expected vs actual mismatch |
| Integration issue | Two systems not communicating, missing data at boundary |
| Latency / resource bug | Regression in response time, memory growth, connection exhaustion |
| Vague / mixed | Extract the symptom, treat as unexpected behavior |

**For test failures, determine sub-type before tracing:**
- **Regression** — was passing, now failing → check recent changes first
- **Flaky** — intermittent → suspect concurrency, timing, or state leakage
- **New** — never passed → trace logic against spec

**For any issue, flag early if environment-specific:**
- "Is this only in staging / prod / one region?"
- If yes: check config, secrets, feature flags, data differences before touching code

**For cascading failures:**
- Identify the first failure, not the loudest symptom
- What did the failing component depend on? Trace from that dependency, not from the visible error

---

## Phase 2 — Find entry point

For **production incidents** (errors live, users affected), use observability signals first — they point to the specific handler faster than any code trace:

1. **Logs** — error rate spike, last successful request timestamp, error message pattern, correlation IDs
2. **Metrics** — P99 latency, queue depth, error rate by handler, resource saturation
3. **Recent changes** — deployment, config change, traffic pattern shift (git log on suspected module, recent merged PRs)

For **isolated bug reports** (no live signal), find the code entry point:

1. **Read structural files** — routes, API definitions, service indexes, main entrypoints. Map ticket keywords to a specific handler, controller, or module.
2. **Keyword inference** — if no structural match, infer from ticket terms ("login" → auth, "checkout" → payment, "notification" → email/messaging service)
3. **Ask one specific question** — only if both above fail. Ask only: "Which service or module owns [specific flow named in ticket]?"

Never ask: "Can you describe the issue more?" — extract what you can from the ticket, ask only what the codebase cannot answer.

**Escape valve:** If investigation points outside the codebase (deployment config, DNS, data corruption, infra drift) — stop, surface findings with evidence, do not attempt a code fix.

---

## Phase 3 — Investigate

Trace the execution path from entry point toward the symptom. Read files, follow imports, check types, run tests when available.

**For async / concurrent code:** trace across all goroutines, promise chains, or event listeners — not just the happy path. Check shared mutable state for races. A linear trace misses deadlocks, race conditions, and async handler bugs.

**Surface key findings only.** Silent on routine reads. Report:

- **Repro confirmed** — how and where
- **Hypothesis ruled out** — what and why
- **Unexpected discovery** — includes: missing null/range guards, missing defensive copies of shared state, resource leaks on error paths (connections, file handles), silent failures where errors are swallowed
- **Blocked** — what's missing and why it matters to the diagnosis

No narration. No "now I'm reading X" commentary.

If first hypothesis is wrong:
1. Check if repro fails because the bug is environment-specific or data-dependent (not because hypothesis was right)
2. If confirmed wrong: stop, revert reasoning, restart Phase 2 from next most likely entry point
3. Never stack a fix on a wrong hypothesis

---

## Phase 4 — Terminate

### Primary path — target this

Reproduction confirmed + root cause located in code.

Output exactly this block, then fix immediately:

```
Root cause:    [one sentence — what the code does wrong and why]
Evidence:      [specific function / condition / code path]
Reproduced:    [yes — how] | [no — static analysis only]
Blast radius:  [who/what is affected and at what scale]
Data safety:   [does this cause corruption, inconsistency, or loss?]
Rollback:      [instant] | [needs migration — describe]
Proposed fix:  [one sentence]
```

### Fallback path — when investigation hits a wall

Trigger conditions: can't reproduce, ambiguous code path, needs runtime data unavailable locally.

**Always use fallback (wait for approval) when:** fix touches user data, transactions, authentication, or shared API contracts.

**OK to propose fix without approval when:** observability confirms repro (logs/metrics match), code is clearly wrong, blast radius is low (single handler, no side effects, instant rollback).

Output this block, then wait for direction:

```
Best hypothesis: [what and why]
Confidence:      [high / medium / low]
Blocker:         [what's missing]
Options:
  A) Proceed with fix — risk: [what could be wrong]
  B) Dig deeper — need: [specific missing information]
```

---

## Phase 5 — Fix

After root cause confirmed (or user approves fallback):

- Minimal diff only — fix the root cause, nothing else
- No refactoring, no cleanup, no scope creep
- Follow project conventions — read sibling files if conventions aren't documented
- If fix touches API contracts or shared schemas: verify no live consumers break before shipping

**Self-review before done:**
- [ ] Typecheck passes, lint passes
- [ ] Acceptance criteria met, no unflagged TODOs
- [ ] Data consistency maintained — if transactional, invariants hold
- [ ] Backward compat — doesn't break in-flight requests or live consumers
- [ ] Observability — if this bug was silent (no alert, no log), add instrumentation in this same fix so regressions are caught

---

## Hard rules

1. Never ask what you can find in the code yourself
2. Never narrate routine investigation steps
3. Never start a fix before root cause is confirmed — or user approves fallback for low-risk cases
4. Never refactor or clean up beyond the bug scope
5. Never stack a second hypothesis on top of a wrong first — revert and re-diagnose
6. Never fix code when root cause points outside the codebase — surface findings and stop
7. Never ship a fix for a previously silent bug without adding instrumentation — the next regression must be catchable

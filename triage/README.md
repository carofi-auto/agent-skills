# triage — Code Diagnosis Skill

Paste or describe any issue. Traces codebase to root cause, outputs a structured diagnosis, waits for your confirmation, then fixes. No guessing. No auto-fix before root cause is confirmed.

## Install

```bash
npx skills add carofi-auto/agent-skills --skill triage
```

## Usage

```
/triage <paste issue, ticket, or freeform description>
```

Works with ADO, Jira, Slack messages, or plain text. Classifies, investigates, and diagnoses without asking questions it can answer itself.

## What it covers

| In scope | Out of scope |
|---|---|
| Correctness bugs | Performance optimization requests |
| Unexpected behavior | Security audits |
| Test failures (regression, flaky, new) | Architecture / design reviews |
| Crashes and unhandled exceptions | Feature requests |
| Integration issues | Infra / DevOps issues |
| Latency regression, resource exhaustion | |

## How it works

**Phase 1 — Classify + Reproduce**
Identifies issue type and confirms reproducibility before touching code. If not reliably reproducible, gathers more data — does not guess.

**Phase 2 — Find entry point**
For live incidents: observability first (logs → metrics → recent deploys).
For isolated reports: reads structural files → keyword inference → asks one specific question only if both fail.

**Phase 3 — Investigate**
Finds a working example first, then forms one hypothesis at a time. Tests minimally. Never stacks a fix on an unconfirmed hypothesis.

For multi-layer systems: adds diagnostic logging at each boundary before tracing logic — only when failure point is ambiguous AND crosses a layer boundary.

Surfaces only:
- Reproduction confirmed (how and where)
- Hypothesis ruled out (what and why, with evidence)
- Unexpected discovery (missing guards, resource leaks, swallowed errors)
- Blocked (what's missing and why it matters)

**3-hypothesis ceiling:** After 3 failed hypotheses, stops and outputs an architectural signal block — directs to `/feature-discovery` instead of attempting a 4th guess.

**Phase 4 — Terminate**
Root cause confirmed → outputs this block, then **waits for your confirmation**:

```
Root cause:    [what the code does wrong and why]
Evidence:      [specific function / condition / code path]
Reproduced:    [yes — how] | [no — static analysis only]
Blast radius:  [who/what is affected]
Data safety:   [corruption / inconsistency risk]
Rollback:      [instant] | [needs migration]
Proposed fix:  [one sentence]
```

If investigation was long, suggests `/clear` + paste the diagnosis block to start the fix in a clean context.

Fallback (when blocked): presents hypothesis + confidence + options and waits for direction. Always waits for approval before touching user data, transactions, auth, or shared API contracts.

**Phase 5 — Fix**
Executes after you say "fix it". Minimal diff. Follows project conventions. If the bug was previously silent, flags instrumentation as a separate follow-up commit.

## What you never see

- "Can you describe the issue more?"
- "I'm now reading file X..."
- A fix attempted before root cause is confirmed
- Refactoring outside the bug scope
- A 4th hypothesis attempt after 3 have failed

## Example

```
/triage Users report login fails intermittently after password reset.
         Error rate ~5%, no pattern by browser or device.
         Started appearing after Tuesday's deploy. (ADO #4821)
```

Claude checks recent deploy changes, finds a working pre-deploy auth path, compares it to the broken one, traces the token ordering race, confirms repro, outputs the diagnosis block, waits for your "fix it", then applies a minimal patch.

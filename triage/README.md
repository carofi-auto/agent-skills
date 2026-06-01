# triage — Code Diagnosis Skill

Given a ticket, trace the codebase to root cause, then fix. No project management. No handoff. One session.

## Install

```bash
npx skills add carofi-auto/agent-skills --skill triage
```

## Usage

```
/triage <paste ticket text>
```

Paste any ADO, Jira, or freeform issue description. The skill classifies, investigates, and fixes without asking questions it can answer itself.

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

**Phase 1 — Classify**
Identifies the issue type (crash, behavior, test failure, integration, resource) and detects sub-type signals: regression vs flaky, environment-specific, cascading failure.

**Phase 2 — Find entry point**
For live incidents: reads logs → metrics → recent changes first.
For isolated bug reports: reads routes/API definitions → keyword inference → asks one specific question only if both fail.

**Phase 3 — Investigate**
Traces execution path silently. Only surfaces key findings:
- Reproduction confirmed
- Hypothesis ruled out
- Unexpected discovery (missing guards, resource leaks, swallowed errors)
- Blocked (with reason)

No narration. No "now I'm reading X" commentary.

**Phase 4 — Diagnose**
Primary: reproduction confirmed + root cause located. Outputs a structured block:

```
Root cause:    [what the code does wrong and why]
Evidence:      [specific function / condition / code path]
Reproduced:    [yes — how] | [no — static analysis only]
Blast radius:  [who/what is affected]
Data safety:   [corruption / inconsistency risk]
Rollback:      [instant] | [needs migration]
Proposed fix:  [one sentence]
```

Fallback (when blocked): presents hypothesis + confidence + options and waits for your direction. Always waits for approval before touching user data, transactions, or auth.

**Phase 5 — Fix**
Minimal diff. Follows project conventions. Adds instrumentation if the bug was previously silent.

## What you never see

- "Can you describe the issue more?"
- "I'm now reading file X..."
- A fix attempted before root cause is confirmed
- Refactoring outside the bug scope

## Example

```
/triage Users report login fails intermittently after password reset.
         Error rate ~5%, no pattern by browser or device.
         Started appearing after Tuesday's deploy. (ADO #4821)
```

Claude checks recent deploy changes, traces auth flow from the login handler, finds the cookie/token ordering race, confirms repro, outputs the diagnosis block, and fixes it.

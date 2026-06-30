---
name: pr-review
description: Self-review your changes before raising a PR. Runs on your local branch/diff, scans every changed line for real bugs, security holes, missing tests, and design problems, filters false positives, and reports severity-tagged findings plus a ready-to-raise verdict. Triggers on "review my changes", "review my diff before I push", "is this ready to raise a PR", "self-review this branch".
---

# PR Review — Pre-Raise Self Review

> **Attribution:** Coverage and severity model from the *PR & Code Review Playbook*
> by Mohamed ElZanaty (Engineering Manager, VOXI — vfuk-digital/Digital). Review
> standard from Google's *Code Review Developer Guide* (google.github.io/eng-practices).

Run this on your own branch **before you raise the PR**. It reads your local diff,
reviews every changed line the way a senior reviewer would, and reports only findings
worth acting on — so the PR you raise is already clean. No GitHub, no CI, no posting:
it runs locally and reports back to you.

## When this skill activates

- "Review my changes" / "review my diff" / "self-review this branch"
- "Is this ready to raise a PR?" / "what would a reviewer flag?"
- After finishing a feature/fix, before pushing or opening the PR

## The standard

Judge the change against one question: **does this improve the overall code health of
the system?** Report what blocks that. Do not demand perfection — a clean, correct,
self-contained change is ready, even if not ideal.

**Signal over noise.** This is a self-review tool, not a linter. Report real issues a
senior engineer would stop on. Suppress nitpicks a human wouldn't bother raising, and
anything a linter/typechecker/CI already catches.

---

## Phase 0 — Get the diff

1. **Read the change.** `git diff <base>...HEAD` (or staged: `git diff --cached`).
   `<base>` = the branch you'll target (usually `main`/`master`/`develop`).
2. **Read the intent.** Infer what this change is for from the diff and commit messages.
   If intent is genuinely unclear, say so — don't invent it.
3. **Classify** — picks what to weight hardest:

| Type | Weight hardest on |
|---|---|
| Feature | Design, logic correctness, tests, separation of concerns, UX/a11y |
| Bugfix | Root cause vs symptom, regression test added, blast radius |
| Refactor | Behavior preserved (no functional change smuggled in), test parity |
| API / schema | Backward compatibility, versioning, consumer impact |
| Config / infra | Secrets, rollback path, environment differences |

**Size note:** > ~400 LOC or mixed concerns (refactor + feature + fix in one) → say so
up front and recommend splitting before raising. Small, self-contained PRs review faster
and ship fewer bugs.

---

## Phase 1 — Design pass (before line nits)

A line-perfect change on the wrong approach is still wrong. Quick check first:

- **Does this belong here?** Right layer, right module — or bolted where it doesn't fit.
- **Sound approach?** Fights existing patterns? Reinvents a helper that already exists?
- **Right time?** Or speculative work (YAGNI) for a need that doesn't exist yet.

A wrong-approach finding is a **Blocker** no matter how clean the code is — surface it
first, before line-level comments.

---

## Phase 2 — Scan every changed line

Read every line of human-written code in the diff. (Generated/vendored/large data files:
scan, don't read line-by-line — and say so.) Look for:

**Bugs & correctness**
- Logic does what was intended; edge/failure paths handled (null/empty/boundary/error).
- No unexpected side effects or shared/global state mutation.
- Async/concurrent: every promise chain / listener traced, no races on shared state.

**Security & safety**
- No secrets, tokens, credentials, or debug code committed.
- No PII or sensitive data in logs.
- Inputs validated at true boundaries (user input, external APIs).

**Tests**
- Critical paths and edge cases covered. Bugfix → has a regression test.
- A major logic change with no test update is a **Blocker**.

**Complexity** *(over-engineering is a finding)*
- A future reader can understand it quickly. No speculative generality.

**Contracts & compatibility**
- API/schema changes are backward-compatible or versioned; no silent consumer break.

**Structure & consistency**
- Right layer, clear naming, follows codebase conventions; reuses existing helpers.
- Comments explain **why**, not what.

**Performance**
- Acceptable under expected load — loops, N+1 queries, DOM thrash, memory growth.

**User-facing (if UI)**
- Matches design; a11y (ARIA, keyboard, focus); responsive at key breakpoints.

---

## Red flags — always a Blocker

| Red flag | Why |
|---|---|
| Hardcoded secret / credential | Leak — rotate and remove from history |
| Logging PII or sensitive data | Privacy/compliance breach |
| Empty `catch` with no context | Silently swallows failures |
| Major logic change, no test update | Unverifiable; regression risk |
| Wrong design / wrong layer | Polished code on a bad foundation |
| Leftover debug code, logs, commented-out blocks | Ships noise |

---

## Filter before reporting (keep it low-noise)

Drop these — they are not worth a finding:

- Pre-existing issues on lines this change didn't touch.
- Anything a linter, typechecker, formatter, or CI already catches (imports, types,
  style, newlines, test failures) — assume CI runs separately.
- Pedantic nitpicks a senior engineer wouldn't raise.
- Intentional changes clearly related to the broader goal.
- Suggestions you can't tie to a concrete failing case — if you're not confident it's
  real, don't report it.

Each reported finding gets one severity:

| Severity | Tag | Blocks raising? |
|---|---|---|
| **Blocker** | `issue:` | Yes — fix before raising |
| **Issue** | `issue:` | Yes — fix before raising |
| **Suggestion** | `suggestion:` | No — author's call |
| **Question** | `question:` | No — clarify intent |

Phrase findings as [Conventional Comments](https://conventionalcomments.org):
`tag: file:line — <problem>. <fix>.`

---

## Output

```
Verdict:      Ready to raise | Fix blockers first | Split recommended
Code health:  Improves | Neutral | Degrades
Change type:  [feature | bugfix | refactor | api | config]
Size:         [LOC, atomic? or "split recommended"]
Blockers:     [count, or "none"]

Findings (most severe first):
  issue:       file:line — <problem>. <fix>.
  suggestion:  file:line — <better approach>.
  question:    file:line — <what's unclear>.
```

Verdict rules:
- **Ready to raise** — no open Blockers/Issues and the change improves code health.
  Suggestions can remain; they don't block.
- **Fix blockers first** — any Blocker/Issue, or design that degrades code health.
- Found nothing? Say "Ready to raise — no issues." Don't invent findings to look thorough.

---

## Hard rules

1. Run on the local diff — never invent code that isn't in it.
2. Report real issues only — signal over noise; suppress what CI/linters catch.
3. Review design before lines — a wrong approach blocks no matter how clean the code.
4. Severity gates the verdict — Blockers/Issues block raising; suggestions never do.
5. Secrets, PII-in-logs, empty catch blocks are always Blockers — no exceptions.
6. A major logic change with no test update is always a Blocker.
7. Don't demand perfection — a clean, correct, self-contained change is ready to raise.
8. Never report a finding you can't tie to a concrete failing case.

# feature-plan — Execution-Ready Plan Skill

Converts completed `/feature-discovery` output into a plan a real engineering team can execute. Phased tasks, dependency table, risk matrix, success metrics, and binary definition of done. Refuses to proceed if critical unknowns remain.

## Install

```bash
npx skills add carofi-auto/agent-skills --skill feature-plan
```

## Usage

Run after `/feature-discovery` completes:

```
/feature-plan
```

## Gate check

Before generating anything, verifies discovery is actually complete. Stops and surfaces gaps if:

- Business value is directional, not measurable
- Scope has no explicit out-of-bounds list
- A dependency is named but ownership is unconfirmed
- A risk has no mitigation or owner
- Success metrics require manual investigation to evaluate
- Any open question would change phasing, architecture, or scope

**Refusing to proceed is correct behavior when discovery is incomplete.** A bad plan executed well is worse than no plan.

## Output structure

| Section | What it contains |
|---|---|
| Problem | One sentence — what breaks or doesn't exist today |
| Outcome | One sentence — measurable change when Phase 1 ships |
| Scope | Explicit in-bounds + out-of-bounds lists |
| Architecture | Key decisions with what was chosen, rejected, and why |
| Phases | Phase 0 (foundation), Phase 1 (minimal shippable), Phase 2+ (hardening) — each with exit conditions |
| Dependencies | System/team, what's needed, owner, confirmed? |
| Risks | Ordered by severity × probability with mitigations |
| Success metrics | Specific, measurable, with baseline and measurement tool |
| Definition of done | Binary checklist — each item is definitively completable |
| Open questions | Known unknowns with blocker assessment |
| Stakeholder actions | Concrete asks, not FYIs |

## Before running /executing-plans

Save this plan as `docs/plans/<feature-slug>.md` using kebab-case.

Context is heavy from planning. Clear before executing:
`/clear` then paste: `/executing-plans docs/plans/<feature-slug>.md`

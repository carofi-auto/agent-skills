# feature-discovery — Idea Stress-Test Skill

You have a rough idea. This skill stress-tests it before you commit to a plan. Acts as a skeptical PM, Engineering Manager, Principal Architect, QA Lead, and Security Engineer simultaneously — one hard question at a time.

## Install

```bash
npx skills add carofi-auto/agent-skills --skill feature-discovery
```

## Usage

```
/feature-discovery <describe your idea, initiative, or direction>
```

## What it does

Runs a structured discovery conversation that challenges every assumption. Probes value, scope, users, dependencies, risks, and success metrics until all are locked.

**One question at a time.** Each question includes why it matters, a recommended answer, and tradeoffs. Every answer you give is checked for new assumptions, risks, and gaps — vague answers get pushed back.

**State block maintained throughout:**
- Current understanding (confirmed facts)
- Decisions made (locked, with reasoning)
- Open questions (ordered by impact)
- Assumptions (each is a potential failure point)
- Risks (ordered by severity × probability)

**Discovery is complete when:**
- Business value is specific and measurable
- Scope has explicit in-bounds AND out-of-bounds lists
- All dependencies confirmed with ownership
- Top risks documented with mitigations
- Success metrics observable without manual investigation
- No open questions that would change scope, architecture, or phasing

## What it never does

- Generates implementation plans, task lists, or code during discovery
- Accepts "we'll figure it out" or "it depends" without resolving the ambiguity
- Says discovery is complete before all criteria are met

## When discovery completes

```
"Discovery is complete. Context is heavy from this session — clear before planning.
`/clear` then paste: `/feature-plan`"
```

The skill gates itself: partial discovery produces false confidence and bad plans. One more hard question is always worth asking.

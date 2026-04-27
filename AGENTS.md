# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, Cline, etc.) when working with code in this repository.

## Repository Overview

A collection of agent skills published by [carofi-auto](https://github.com/carofi-auto). Each skill packages domain knowledge into a format that AI coding agents can install and use automatically.

Skills are plain markdown — no build step, no dependencies.

## Directory Structure

```
carofi-auto/agent-skills/
├── README.md               # skill directory and install commands
├── AGENTS.md               # this file
├── CLAUDE.md               # Claude Code-specific guidance (same content)
├── LICENSE                 # MIT
└── {skill-name}/           # one directory per skill, kebab-case
    ├── SKILL.md            # required: skill definition and instructions
    └── references/         # optional: supporting docs read on demand
        └── {topic}.md
```

## Adding a New Skill

### 1. Create the skill directory

Use kebab-case. The directory name becomes the `--skill` argument at install time.

```
{skill-name}/
├── SKILL.md
└── references/         # add only if SKILL.md would exceed ~400 lines without it
    └── {topic}.md
```

### 2. Write SKILL.md

Required frontmatter:

```markdown
---
name: {skill-name}
description: "{One sentence written as a trigger condition — when should an agent activate this skill? Include key nouns: library names, env var names, domain terms.}"
---
```

Body structure (adapt sections to the skill's complexity):

```markdown
# {Skill Title}

> **Attribution:** {Credit the upstream source if this skill is built on
> official documentation. Link to the source.}

Brief one-paragraph description of what this skill covers.

## When this skill activates

- Bullet list of concrete trigger signals (env vars, keywords, file patterns)

## Non-negotiable rules

Numbered list of rules the agent must enforce regardless of user intent.
Security rules, data integrity rules, and irreversibility warnings belong here.

## Before writing any integration code

Table mapping tasks to reference files (if references/ exists).

## {Any domain-specific sections}
```

### 3. Write reference files (if needed)

Reference files extend the skill without bloating SKILL.md. The agent reads them on demand when the user's task maps to that topic.

- One file per topic (authentication, error-handling, a specific API, etc.)
- Keep each file focused — one clear purpose per file
- Name files in kebab-case: `create-intention.md`, `webhooks-hmac.md`
- SKILL.md should have a table mapping tasks → reference files so the agent knows what to read

### 4. Update README.md

Add a row to the Skills table:

```markdown
| [skill-name](./skill-name) | One-line description | `npx skills add carofi-auto/agent-skills --skill skill-name` |
```

### 5. Open a pull request

Branch name: `feat/skill-{skill-name}`
PR title: `feat: add {skill-name} skill`
PR body: describe the use case, the upstream source (if any), and any open questions.

---

## Best Practices

**SKILL.md**
- Write the `description` as a trigger condition, not a feature list — agents use it to decide whether the skill is relevant
- Keep SKILL.md under 400 lines; use `references/` for anything longer
- Non-negotiable rules go in SKILL.md, not in references — they must always be in context
- Cite the upstream documentation source if the skill is built from official docs

**Reference files**
- Each file should answer one question: authentication, a specific API endpoint, a flow, error debugging
- Cross-link with backtick paths: `references/webhooks-hmac.md`
- Write for an agent that has no prior context — be explicit about field names, types, and order

**Attribution**
- If a skill is built from official LLM-friendly docs (e.g., `llms.txt` files), include an attribution blockquote at the top of SKILL.md
- Link directly to the source URL

**Naming**
- Skill directories: `kebab-case`
- Reference files: `kebab-case.md`
- Avoid version numbers in directory names — update the content in place

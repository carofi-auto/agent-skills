# carofi-auto/agent-skills

Agent skills for [carofi-auto](https://github.com/carofi-auto) — installable capabilities that extend AI coding agents with specialized, production-ready knowledge.

Works with Claude Code, GitHub Copilot, Cursor, Cline, and 15+ other agents.

## Install all skills

```bash
npx skills add carofi-auto/agent-skills
```

## Install a specific skill

```bash
npx skills add carofi-auto/agent-skills --skill <skill-name>
```

## Skills

| Skill | Description | Install |
|-------|-------------|---------|
| [paymob](./paymob) | Paymob payment integration for MENA — intentions, checkout flows, HMAC-verified webhooks, payment links | `npx skills add carofi-auto/agent-skills --skill paymob` |

## Contributing

Each skill lives in its own directory:

```
<skill-name>/
├── SKILL.md          # agent instructions (required)
└── references/       # supporting docs the agent reads on demand (optional)
```

`SKILL.md` must include YAML frontmatter with `name` and `description` fields. The `description` is what the agent uses to decide when to activate the skill — write it as a trigger condition, not a feature list.

To propose a new skill, open a pull request with the skill directory and a short description of the use case in the PR body.

## License

MIT

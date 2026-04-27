# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## What this repo is

A collection of installable agent skills published by [carofi-auto](https://github.com/carofi-auto). Skills are plain markdown — no build step, no dependencies, no tests to run.

See [AGENTS.md](./AGENTS.md) for the full contributing guide: directory structure, SKILL.md format, naming conventions, and best practices.

## When working in this repo

- **Adding a skill**: follow the structure in AGENTS.md exactly — directory name, frontmatter fields, and README table update are all required
- **Editing an existing skill**: prefer minimal diffs; if an API detail changes, update only the affected reference file
- **No code to run**: there is nothing to compile, lint, or test — verify changes by reading the files
- **Commits**: use `feat: add {skill-name} skill` for new skills, `fix:` for corrections, `docs:` for copy/attribution changes

## Package manager

None. This repo contains only markdown files.

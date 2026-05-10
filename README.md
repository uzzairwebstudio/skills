# agent-skills

A collection of skills for AI coding agents. Skills are packaged instructions that extend agent capabilities with procedural knowledge.

[![skills.sh](https://skills.sh/b/uzzairwebstudio/skills)](https://skills.sh/uzzairwebstudio/skills)

Skills follow the [Agent Skills](https://skills.sh/docs) format and work with Claude Code, Cursor, Codex, Windsurf, and [more](https://skills.sh).

## Available Skills

### changelog

Generate human-readable changelogs from git commit history, strictly following the [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/) format.

## Installation

```
npx skills add yourusername/agent-skills
```

To install a specific skill:

```
npx skills add yourusername/agent-skills --skill changelog
```

## Usage

Skills are automatically available once installed. The agent will use them when relevant tasks are detected.

**Examples:**

```
Generate a changelog since the last tag
```

```
Write release notes for v2.3.0
```

```
What changed since v1.2.0?
```

## Skill Structure

Each skill contains:

- `SKILL.md` — Instructions for the agent
- `scripts/` — Helper scripts (optional)
- `references/` — Additional context loaded on demand (optional)

```
skills/
└── changelog/
    └── SKILL.md
```

## Contributing

To add a new skill, create a directory under `skills/` with a `SKILL.md` file:

```markdown
---
name: my-skill
description: What it does and when to use it.
---

# My Skill

Instructions for the agent...
```

See [skills.sh/docs](https://skills.sh/docs) for the full spec.
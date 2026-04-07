# claude-skills

![Skills](https://img.shields.io/badge/skills-1-blue?style=flat-square)
![Platform](https://img.shields.io/badge/platform-Claude%20Code-blueviolet?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)

Jerry's personal library of [Claude Code](https://claude.ai/code) skills — reusable instruction sets that extend Claude with custom slash commands.

---

## What is a Skill?

A **skill** is a `SKILL.md` file that Claude Code loads as a set of instructions, making it available as a `/slash-command` inside any Claude Code session.

Skills live locally at:

```
~/.claude/skills/<skill-name>/SKILL.md
```

When you type `/skill-name` in Claude Code, Claude reads the corresponding `SKILL.md` and executes the instructions within it.

---

## Available Skills

| Skill | Invoke | Description |
|-------|--------|-------------|
| `project-viz-report` | `/project-viz-report` | Weekly 5-section project report: Ghibli art panels, project overview, tech stack, deploy links, token usage — sent as one Gmail |

---

## How to Use a Skill

1. Copy the skill folder into your local skills directory:

   ```bash
   cp -r project-viz-report ~/.claude/skills/
   ```

2. Invoke it in Claude Code:

   ```
   /project-viz-report
   ```

Claude will load the `SKILL.md` and run the skill automatically.

---

## Skill Structure

Each skill follows this layout:

```
<skill-name>/
└── SKILL.md          # frontmatter + markdown instructions
```

`SKILL.md` format:

```markdown
---
name: skill-name
description: One-line description shown in skill picker
---

# Skill Title

Step-by-step instructions Claude follows when the skill is invoked.
```

The YAML frontmatter sets the skill's `name` and `description`. Everything below the `---` divider is the instruction body.

---

## Repository Layout

```
claude-skills/
├── README.md
└── project-viz-report/
    └── SKILL.md
```

---

*Built for personal use with [Claude Code](https://claude.ai/code).*

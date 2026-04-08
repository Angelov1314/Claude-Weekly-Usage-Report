# Claude Weekly Usage Report

A Claude Code skill that generates a weekly project report with 5 sections — Ghibli-style art panels, project overview, tech stack + API links, deployment links, and token/model usage — sent as one Gmail email.

## What It Does

The `project-viz-report` skill scans your active projects, generates Studio Ghibli-inspired watercolor art panels via the Nano Banana API, collects token usage stats, and compiles everything into a single HTML email with 5 sections:

| # | Section | Content |
|---|---------|---------|
| 1 | Project Visualization | Ghibli art panels (3 groups of 5) hosted on GitHub |
| 2 | Project Overview | Name, date, core features table |
| 3 | Tech Stack & APIs | Tech stack, API services, GitHub repos |
| 4 | Deployment Links | Platform badges + live/repo URLs |
| 5 | Token Usage | Totals, per-model breakdown, daily bar chart |

## Installation

Copy the `project-viz-report` directory into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/project-viz-report
cp project-viz-report/skill.md ~/.claude/skills/project-viz-report/skill.md
```

## Usage

```
/project-viz-report
```

Claude will scan project directories, generate art panels, collect token usage, and draft a Gmail email with all 5 sections.

## Requirements

- Gmail MCP (for drafting/sending the email)
- Nano Banana API key (for Ghibli art generation)
- `token-usage` skill (for section 5 data)

## License

MIT

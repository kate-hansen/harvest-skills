# Harvest skills

Agent skills for Harvest time tracking, for Claude Code and Codex. They talk to Harvest through the Harvest MCP server.

## Setup

1. Add the Harvest MCP server and sign in to it:
   - Claude Code: `claude mcp add --transport http --scope user harvest https://api.harvestapp.com/mcp`, then run `/mcp` in Claude Code and authenticate `harvest`.
   - Codex: add an MCP server named `harvest` with the same URL.
2. Link or copy each folder under `skills/` into `~/.claude/skills/` and/or `~/.codex/skills/`.
3. Run `refresh-harvest-projects` once to build your project cache.

## Skills

| Skill | What it does |
|---|---|
| `refresh-harvest-projects` | Caches your active Harvest projects and tasks, with their IDs. |
| `weekly-harvest-report` | Summarizes a week of logged time by project. Read-only. |
| `monthly-harvest-report` | Summarizes a month of logged time by project, with weekly totals and an email-ready version. Read-only. |
| `harvest-timesheet` | Turns a day's work into Harvest time entries, from wherever you keep your daily updates or from what you tell it, and submits them once you approve. |

## Your data

The skills hold no personal data. Everything about you lives in `~/.cache/harvest/`:

- `projects.json`: your project and task cache, written by `refresh-harvest-projects`.
- `timesheet.md`: where `harvest-timesheet` finds your daily updates, plus your saved project mappings. The first run sets it up with you; see `skills/harvest-timesheet/CONFIG.md`.

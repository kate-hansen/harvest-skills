# Harvest skills

Agent skills for Harvest time tracking, for Claude Code and Codex. They talk to Harvest through the Harvest MCP server.

## Install

```bash
npx skills add nothingalike/harvest-skills --global
```

Then connect the Harvest MCP server (below) and run `refresh-harvest-projects` once to build your project cache.

## Connect the Harvest MCP server

The skills reach Harvest through its MCP server at `https://api.harvestapp.com/mcp`, which you sign in to with your own Harvest account. Name the server `harvest`, since the skills look for its tools under that name.

### Claude Code

Add the server for all your projects:

```bash
claude mcp add --transport http --scope user harvest https://api.harvestapp.com/mcp
```

Then sign in: start `claude`, run `/mcp`, pick **harvest**, and choose **Authenticate**. Your browser opens to Harvest; log in and approve access. Check the connection with:

```bash
claude mcp get harvest
```

It should no longer say "Needs authentication". Start a new session to pick up the Harvest tools.

### Codex

Add the server:

```bash
codex mcp add harvest --url https://api.harvestapp.com/mcp
```

Then sign in, which opens your browser to Harvest:

```bash
codex mcp login harvest
```

Start a new Codex session to pick up the Harvest tools.

## Skills

| Skill | What it does |
|---|---|
| `refresh-harvest-projects` | Caches your active Harvest projects and tasks, with their IDs. |
| `weekly-harvest-report` | Summarizes a week of logged time by project. Read-only. |
| `monthly-harvest-report` | Summarizes a month of logged time by project, with weekly totals and an email-ready version. Read-only. |
| `time-off-harvest-report` | Summarizes time out of the office by category — vacation, sick, holiday, and other leave — with a subtotal each and a grand total in hours and days. Read-only. |
| `harvest-timesheet` | Turns a day's work into Harvest time entries, from wherever you keep your daily updates or from what you tell it, and submits them once you approve. |

## Your data

The skills hold no personal data. Everything about you lives in `~/.cache/harvest/`:

- `projects.json`: your project and task cache, written by `refresh-harvest-projects`.
- `timesheet.md`: where `harvest-timesheet` finds your daily updates, plus your saved project mappings. The first run sets it up with you; see `skills/harvest-timesheet/CONFIG.md`.

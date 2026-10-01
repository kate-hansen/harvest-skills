---
name: refresh-harvest-projects
description: Refresh the shared JSON cache of Kyle Johns' active Harvest project assignments through the Harvest MCP server. Use when the user asks to refresh Harvest projects, look up Kyle's current Harvest projects, clients, or tasks, or when another skill needs Kyle's assigned project data.
---

# Refresh Harvest Projects

Rebuild the cache of Kyle Johns' active Harvest project assignments so other skills can read it. All Harvest data comes from the authenticated Harvest MCP server (`harvest`); its tools appear as `mcp__harvest__<tool>` in Claude Code and `mcp__harvest.<tool>` in Codex.

## Cache

The one cache file, shared by every agent:

`~/.cache/harvest/kyle-johns-project-assignments.json`

Other skills read this file directly. It holds project and assignment data only; credentials stay with the MCP connection.

## Refresh

1. Call `list_projects` with `is_active: true`.
2. For each active project, call `list_project_assignments` with `assignment_type: "users"`, following `next_cursor` while `truncated` is true.
3. Keep the projects whose user roster includes Kyle Johns: match the name or email fields case-insensitively on `Kyle Johns`, or a clearly equivalent Kyle Johns identifier.
4. For each kept project, call `list_project_assignments` with `assignment_type: "tasks"`, following `next_cursor` the same way.
5. Write the cache in the shape below. Leave billable rates out unless the user asks for them.
6. Report the cache path and the project count.

The cache lists only assignments Harvest returned. When `list_projects` itself comes back truncated, narrow the call by client or project where Harvest allows it, and tell the user the cache may need a more specific refresh.

## Cache shape

- `cache_version`: `1`.
- `owner`: `Kyle Johns`.
- `source`: Harvest MCP metadata (service, connection, tools used, scope).
- `refreshed_at`: ISO timestamp of the refresh.
- `needs_refresh`: `false` after a successful refresh, `true` in an unpopulated starter cache.
- `project_count`: number of projects kept.
- `projects`: compact list sorted by client, then project name. Each has `project_id`, `project_name`, `project_code`, `client_id`, `client_name`, `display_name`, `is_billable`, `is_fixed_fee`, `starts_on`, `ends_on`, `updated_at`, and `task_names`.
- `assignments`: Kyle Johns' normalized user assignments.
- `task_assignments`: normalized task assignments for the kept projects.
- `raw`: MCP responses or compact excerpts for fields not yet normalized.

`display_name` is `Client: Project` when the project has a client. It is the label to use for timesheets.

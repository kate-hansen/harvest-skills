---
name: refresh-harvest-projects
description: Refresh the local cache of the signed-in user's active Harvest projects and tasks through the Harvest MCP server. Use when the user asks to refresh their Harvest projects, look up their current Harvest projects, clients, or tasks, or when another skill needs their project data.
---

# Refresh Harvest Projects

Rebuild the cache of the signed-in Harvest user's active project assignments so other skills can read it. All Harvest data comes from the authenticated Harvest MCP server (`harvest`); its tools appear as `mcp__harvest__<tool>` in Claude Code and `mcp__harvest.<tool>` in Codex.

## Cache

The cache file is `~/.cache/harvest/projects.json`. It lives in the user's home folder, outside the skills, and holds only that user's project and assignment data; credentials stay with the MCP connection. Other skills read it directly.

## Refresh

1. **Identify the user.** Find the signed-in Harvest user's ID and name: through `list_users`, or from the user named on their own recent `list_time_entries` results. When neither settles it, ask the user for their Harvest name and confirm it against `list_users`.
2. Call `list_projects` with `is_active: true`.
3. For each active project, call `list_project_assignments` with `assignment_type: "users"`, following `next_cursor` while `truncated` is true.
4. Keep the projects whose user roster includes the signed-in user, matched by user ID (or by name, case-insensitively, when the roster gives no IDs).
5. For each kept project, call `list_project_assignments` with `assignment_type: "tasks"`, following `next_cursor` the same way.
6. Write the cache in the shape below. Leave billable rates out unless the user asks for them.
7. Report the cache path and the project count.

The cache lists only assignments Harvest returned. When `list_projects` itself comes back truncated, narrow the call by client or project where Harvest allows it, and tell the user the cache may need a more specific refresh.

## Cache shape

- `cache_version`: `1`.
- `owner`: the signed-in user's Harvest name; `owner_id`: their Harvest user ID.
- `source`: Harvest MCP metadata (service, connection, tools used, scope).
- `refreshed_at`: ISO timestamp of the refresh.
- `needs_refresh`: `false` after a successful refresh, `true` in an unpopulated starter cache.
- `project_count`: number of projects kept.
- `projects`: compact list sorted by client, then project name. Each has `project_id`, `project_name`, `project_code`, `client_id`, `client_name`, `display_name`, `is_billable`, `is_fixed_fee`, `starts_on`, `ends_on`, `updated_at`, and `task_names`.
- `assignments`: the user's normalized user assignments.
- `task_assignments`: normalized task assignments for the kept projects.
- `raw`: MCP responses or compact excerpts for fields not yet normalized.

`display_name` is `Client: Project` when the project has a client. It is the label to use for timesheets.

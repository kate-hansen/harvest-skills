---
name: weekly-harvest-report
description: Summarize a week of Harvest time by project, read live from the Harvest MCP server. Use when the user asks to pull, review, summarize, or export their weekly Harvest time, timesheet, or logged hours for a week or date range.
---

# Weekly Harvest Report

Summarize a week of Harvest time entries by project. The report is **read-only**: it reads entries through the Harvest MCP server's `list_time_entries` tool and leaves Harvest unchanged. The tool appears as `mcp__harvest__list_time_entries` in Claude Code and `mcp__harvest.list_time_entries` in Codex. Each entry carries its own client, project, and task labels, so the report works from the entries alone.

## Steps

1. **Set the period.** Default to the current Monday through Sunday, in local time. "Last week" is the previous Monday through Sunday. Exact `YYYY-MM-DD` start and end dates are used as given. The report states the exact range.
2. **Pull every entry.** Call `list_time_entries` with `from`, `to`, and `limit: 500`. Include `user_id` only when the user asks about someone else or several people; otherwise the tool returns the authenticated user's entries. While the response has `truncated: true`, call again with its `next_cursor` and the same filters. If any response has `scope_limited: true`, warn in the report that it may be incomplete. This step is done when the last response comes back untruncated.
3. **Normalize.** For each entry, record the date, client, project, task, hours, notes, billable status, invoiced status, and timer start and end when present. Use `rounded_hours` when the entry has it, otherwise `hours`. Group by client and project, then by task. Sort projects by total hours (highest first), then by name. Sort entries within a project by date, then task.
4. **Summarize.** For each project, give the total hours, the task breakdown, and a short narrative drawn from the entries' notes:
   - Merge repeated notes into themes such as implementation, meetings, discovery, support, planning, review, or administration.
   - Keep the concrete nouns from the notes: story IDs, client names, deliverables, and system names.
   - Describe work as "logged", "worked on", or "supported". Say "completed" only when a note says so.
   - Draw every summary line from the notes. When the notes are sparse, say the summary rests on limited notes.
5. **Check for gaps.** Show the total logged hours. List days with no entries under **No Harvest Entries**. Count the entries with blank notes and name their project and task. If the total looks unusually low or high, state it plainly and ask whether the user wants a closer look.

## Output

Default structure:

```markdown
# Harvest Weekly Report: YYYY-MM-DD to YYYY-MM-DD

Total logged: 0.0h

## Project Name
Total: 0.0h

Summary:
- Concise work theme drawn from the notes.
- Another theme the entries support.

Task breakdown:
| Task | Hours |
| --- | ---: |
| Task Name | 0.0 |

Source entries:
| Date | Task | Hours | Notes |
| --- | --- | ---: | --- |
| YYYY-MM-DD | Task Name | 0.0 | Harvest note text |
```

- **Short update:** totals and project summaries only.
- **Export:** write a Markdown file to the path the user gives or confirms.

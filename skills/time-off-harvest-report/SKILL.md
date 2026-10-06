---
name: time-off-harvest-report
description: Summarize time out of the office from Harvest — planned vacation, unplanned vacation, sick, holiday, and other leave — grouped by category with a subtotal for each and a grand total, over any date range. Read-only.
---

# Time Off Harvest Report

Summarize time logged **out of the office** — vacation, sick time, holidays, and other leave — from Harvest time entries, grouped by category with a subtotal for each and a grand total. The report is **read-only**: it reads entries through the Harvest MCP server's `list_time_entries` tool and leaves Harvest unchanged. The tool appears as `mcp__harvest__list_time_entries` in Claude Code and `mcp__harvest.list_time_entries` in Codex.

Harvest has no built-in "time off" flag. Leave is logged either to non-billable **tasks** named for each leave type ("Planned Vacation", "Sick Time", "Holiday", …) or under a **project** named for time off ("Out of Office", "PTO"). This report finds those entries by task name and by project name, then totals them by category.

## Steps

1. **Set the period.** Default to the current calendar year (Jan 1 to today), since time off is usually tracked by the year. "Last year", a quarter, a month name, `YYYY-MM`, and exact `YYYY-MM-DD` start and end dates are all accepted. The report states the exact range.

2. **Learn the account's time-off categories.** Call `list_tasks` (active) once. A time-off category is a task whose name matches the keyword set below, case-insensitive:

   `vacation · pto · sick · holiday · bereavement · jury · leave · parental · maternity · paternity · furlough · sabbatical · natural disaster · environmental · out of office · ooo`

   Keep each matched task as its own category, so "Planned Vacation" and "Unplanned Vacation" stay distinct. Also note any **project** whose name matches `out of office · ooo · pto · time off · leave · vacation · holiday` — entries under it count as time off even when their task isn't a leave type (a "Company Event" logged under an "Out of Office" project is still time out of the office). List the categories you found; when a match is ambiguous, confirm with the user and let them add or drop one.

3. **Pull every entry.** Call `list_time_entries` with `from`, `to`, and `limit: 500`. Include `user_id` only when the user asks about someone else or several people; otherwise the tool returns the authenticated user's entries. While a response has `truncated: true`, call again with its `next_cursor` and the same filters. If any response has `scope_limited: true`, warn in the report that it may be incomplete. This step is done when the last response comes back untruncated — a year of entries can span several pages, so don't stop at the first.

4. **Filter and categorize.** Keep an entry when its **task** matches the keyword set OR its **project** matches a time-off project. Group kept entries by their task — that task name is the category. A task caught only by its project (not by a leave word) still gets its own category line and still counts toward the grand total; never fold it into a leave category. For each entry, record the date, category, hours, project, and notes; use `rounded_hours` when the entry has it, otherwise `hours`.

5. **Total.** For each category, sum its hours into a **subtotal**. Sum every category into a **grand total**. Give both in hours and in **days** — default 8 hours to a day; state the assumption and let the user set a different workday length. Sort categories by hours, highest first, then by name.

## Output

Default structure:

```markdown
# Harvest Time Off Report: YYYY-MM-DD to YYYY-MM-DD

Grand total: 0.0h (0.0 days)

## By category
| Category | Hours | Days |
| --- | ---: | ---: |
| Planned Vacation | 0.0 | 0.0 |
| Holiday | 0.0 | 0.0 |
| **Grand total** | **0.0** | **0.0** |

## Planned Vacation — 0.0h (0.0 days)
| Date | Hours | Project | Notes |
| --- | ---: | --- | --- |
| YYYY-MM-DD | 0.0 | Out of Office | Harvest note text |

## No time taken
Categories with no entries in this period:
- Bereavement
- Jury Duty
- Leave
```

- **Summary only:** the "By category" table and the grand total — no per-entry lists.
- **Email-ready:** plain-text headings and lines that paste cleanly into Outlook, without tables.
- **Export:** write a Markdown file to the path the user gives or confirms.

The **No time taken** section lists every category from step 2 that had no entries in the period, so the report reads the same from one run to the next. When the whole period has no time off, say so plainly and show the grand total as 0.

---
name: search-task
description: Search for Tasks in HP Nova by various conditions such as title, task code, status, owner (username or email), keyword, date range, plan code, phase, test site, and task type. Supports pagination and sorting. Use this skill whenever the user wants to find, search, or list Tasks in Nova.
version: 0.0.1
author: Matt Xiao
---

# Search Task Skill

Search for Tasks in the HP Nova test management platform using the `nova_search_task` and `nova_search_user` MCP tools.

## When to Use

Invoke this skill when the user says something like:

- "Search for tasks with title containing xxx"
- "Find all Testing tasks in Nova"
- "List tasks owned by xxx"
- "Show me all Draft tasks"
- "Look up task code TASKxxxxxxxx"
- "Search tasks by keyword"
- "Find tasks assigned to bjorn_deng"
- "Show tasks in Testing status with title nova test"
- "Find tasks created between Jan and Mar 2026"

## Workflow

### Step 1 — Resolve Owner (if specified)

If the user provides an owner name or email, call `nova_search_user` first to look up the `userId`:

- Map the username/email to `userId` (integer).
- If multiple users match, pick the one whose username exactly matches; otherwise list candidates and ask the user to clarify.
- If no user is found, inform the user and stop.

### Step 2 — Search Tasks

Call `nova_search_task` with the resolved `owner` (userId integer) and all other extracted parameters.

## Supported Query Parameters

| Parameter | Description | Example Values |
|-----------|-------------|----------------|
| `tasktitle` | Fuzzy match on task title | `nova test`, `regression` |
| `taskcode` | Exact match on task code | `TASK0000041124` |
| `keywords` | Full-text keyword search | `smoke`, `release` |
| `statuss` | Filter by status (array of integers) | See status table below |
| `owner` | Filter by owner userId (integer) — resolve via `nova_search_user` first | `1658` |
| `plancodes` | Filter by associated plan code(s) | `PLN300000009` |
| `startdate` | Task start date (lower bound) | `2026-01-01` |
| `enddate` | Task end date (upper bound) | `2026-03-31` |
| `phase` | Filter by phase | `DVT`, `PVT` |
| `testsite` | Filter by test site | `BJS`, `SHH` |
| `taskType` | Filter by task type (integer) | `1`, `3` |
| `categorya` | Level-1 category ID | |
| `categoryb` | Level-2 category ID | |
| `categoryc` | Level-3 category ID | |
| `createuserHpOdm` | Creator's HP ODM identifier | |
| `createuserHpOdmName` | Creator's HP ODM name | |
| `orderColumn` | Sort column (default: `updatedate`) | `updatedate`, `title` |
| `orderWay` | Sort direction (default: `desc`) | `asc`, `desc` |
| `pageNum` | Page number (default: 1) | `1`, `2`, `3` |
| `pageSize` | Results per page (default: 20) | `10`, `20`, `50` |

## Task Status Codes

| Status Code | Status Name |
|-------------|-------------|
| `0` | Draft |
| `1` | Testing |
| `2` | Cancelled |
| `3` | Paused |
| `4` | Complete |
| `5` | Reviewing |
| `6` | Returned |
| `9` | All (Exclude Cancelled) |

When the user specifies a status name, convert it to the corresponding integer before passing to `statuss`. Multiple statuses can be combined, e.g., `[1, 5]` for Testing + Reviewing.

## Behavior

1. Extract query conditions from the user's natural language input and map them to the corresponding parameters.
2. If an owner name/email is provided, call `nova_search_user` to resolve `userId`.
3. Call `nova_search_task` with the resolved parameters.
4. Present results in a clear Markdown table (see Output Format below).
5. Show total count and current pagination info below the table.
6. If there are more pages, ask the user whether they want to see the next page.
7. If no results are found, inform the user politely and suggest relaxing the search criteria (e.g., remove status filter, broaden title keywords).

## Output Format Example

Found **N** task(s) (page X, page size 20):

| # | Task Code | Title | Status | Owner | Progress | Start Date | End Date | Last Updated |
|---|-----------|-------|--------|-------|----------|-----------|----------|--------------|
| 1 | TASK0000041124 | nova test Copy | Testing | Bjorn_Deng(HP) | 0.00% | 2026-01-07 | 2026-01-08 | 2026-01-07 |

> Total N result(s), page X of Y. Would you like to see the next page?

## Notes

- `statuss` accepts an **array** of integers — always wrap in brackets, e.g., `[1]` for Testing only.
- `owner` must be an **integer** userId — never pass the username string directly; always resolve via `nova_search_user` first.
- Only pass parameters explicitly provided or clearly implied by the user; leave all others at their defaults.
- If the user provides both a title and keywords, prefer `tasktitle` for title-based filtering and `keywords` for full-text search. Do not set both unless clearly needed.
- Date formats should be `YYYY-MM-DD`.

---
name: search-component
description: Search for Components (hardware, firmware, or software entries) in HP Nova by various conditions such as name keyword, part number (PN), pulsar ID, platform ID, creator, date range, etc. Use this skill whenever the user wants to find, search, or list Components in Nova.
version: 0.0.1
author: Matt Xiao
---

# Search Component Skill

Search for Components (hardware / firmware / software entries) in the HP Nova test management platform using the `nova_search_component` MCP tool.

## When to Use

Invoke this skill when the user says something like:

- "Search for component named Intel CSME"
- "Find component with part number ABC123"
- "List components for platform ID P001"
- "Show components with pulsar ID 12345"
- "Find components created by bjorn_deng"
- "Show components updated between Jan and Mar 2026"
- "查询组件 Embedded Controller 的详情"
- "搜索 Nova 里的 firmware component"

## Supported Query Parameters

| Parameter | Description | Example Values |
|-----------|-------------|----------------|
| `content` | Fuzzy match on component name / content keyword | `Intel CSME`, `Embedded Controller`, `BIOS` |
| `PN` | Filter by part number (PN) | `ABC123`, `L12345-001` |
| `pulsar_id` | Filter by Pulsar ID | `12345` |
| `platformID` | Filter by platform ID | `P001` (use platform ID from `nova_search_platform`) |
| `createuser` | Filter by the username who created the component | `bjorn_deng` |
| `start` | Filter by creation date (lower bound, inclusive) | `2026-01-01` |
| `end` | Filter by creation date (upper bound, inclusive) | `2026-03-31` |
| `isShowAll` | Show all versions/levels (0 = top-level only, default; 1 = show all) | `0`, `1` |

## Workflow

### Step 1 — Resolve Platform (if user specifies a platform name instead of ID)

If the user provides a platform **name** (e.g., "Nuvolari") instead of a numeric platform ID, first call `nova_search_platform` with that name to retrieve the platform record and obtain the `platform_id`. Then use that ID as `platformID` when calling `nova_search_component`.

### Step 2 — Search Components

Call `nova_search_component` with all extracted parameters and display the results.

## Behavior

1. Extract query conditions from the user's natural language input and map them to the corresponding parameters.
2. If a platform name (not ID) is provided, resolve it via `nova_search_platform` first.
3. Call `nova_search_component` with the resolved parameters.
4. Present results in a clear Markdown table (see Output Format below).
5. If the result set is large, summarize by level/type and offer to show details.
6. If no results are found, inform the user politely and suggest relaxing the search criteria.

## Output Format Example

Found **N** component(s):

| # | Component ID | Name | Level | Supplier | HW | FW/Ver | Part Number | Pulsar ID | Parent |
|---|-------------|------|-------|----------|-----|--------|-------------|-----------|--------|
| 1 | C001 | Intel CSME FW for PTL | 2 | Intel | - | 21.0.2.1482 | - | 67890 | C000 |

## Notes

- Only pass parameters explicitly provided or clearly implied by the user; leave all others at their defaults.
- `content` supports fuzzy/partial matching on the component name — no need to type the full name.
- `platformID` is a numeric ID. If the user gives a platform name, resolve it with `nova_search_platform` first.
- Date formats should be `YYYY-MM-DD`.
- `isShowAll=0` (default) returns only top-level component entries; set `isShowAll=1` to include all sub-levels/versions.
- Components are hierarchical — the result includes `level` and `parentId` to indicate the hierarchy.


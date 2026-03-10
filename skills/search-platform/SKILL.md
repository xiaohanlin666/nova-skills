---
name: search-platform
description: Search for Platforms (Products) in HP Nova by various conditions such as platform name, market name, product cycle year, etc. Supports pagination. Use this skill whenever the user wants to find, search, or list Platforms or Products in Nova.
version: 0.0.1
author: Matt Xiao
---

# Search Platform Skill

Search for Platforms (Products) in the HP Nova test management platform using the `nova_search_platform` MCP tool.

## When to Use

Invoke this skill when the user says something like:

- "Search for platform named Nuvolari"
- "Find platforms in cycle 2026"
- "List all Nova platforms with market name MerinoW14"
- "Show me products with cycle year 2025"
- "Look up platform Cashmere"
- "查询平台 Nuvolari 的详情"
- "搜索 Nova 里的 product"

## Supported Query Parameters

| Parameter | Description | Example Values |
|-----------|-------------|----------------|
| `platform` | Fuzzy match on platform name (plat_name) | `Nuvolari`, `Cashmere`, `Merino` |
| `MarketName` | Fuzzy match on market name | `MerinoW14`, `CashmereI` |
| `Cycle` | Filter by product cycle year | `2025`, `2026` |
| `pageSize` | Results per page (default: 10) | `10`, `20`, `50` |
| `pageindex` | Page number (default: 1) | `1`, `2`, `3` |

## Behavior

1. Extract query conditions from the user's natural language input and map them to the corresponding parameters.
2. Call `nova_search_platform` with the extracted parameters.
3. Present results in a clear Markdown table (see Output Format below).
4. Show total count and current pagination info below the table.
5. If there are more pages, ask the user whether they want to see the next page.
6. If no results are found, inform the user politely and suggest relaxing the search criteria.

## Output Format Example

Found **N** platform(s) (page X, page size 10):

| # | Platform ID | Platform Name | Market Name | Product Line | BU | Cycle | ODM | Architecture |
|---|-------------|--------------|-------------|-------------|-----|-------|-----|--------------|
| 1 | P001 | Nuvolari | Nuvolari 14 | NB | PC | 2026 | Compal | Intel |

> Total N result(s), page X of Y. Would you like to see the next page?

## Notes

- Only pass parameters explicitly provided or clearly implied by the user; leave all others at their defaults.
- `platform` and `MarketName` support fuzzy/partial matching — no need to type the full name.
- `Cycle` refers to the product cycle/generation year, e.g., `2025` or `2026`.
- If the user says "product" instead of "platform", treat it the same way.


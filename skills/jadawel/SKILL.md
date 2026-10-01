---
name: jadawel
description: Operate Jadawel (جداول), the Arabic-first spreadsheet-database at app.jadawl.site. Use for its workspaces, databases, tables, fields, rows and page views (صفحة); for its REST API, webhooks, health or backups; for releasing a new build; or for Arabic product wording.
---

# Jadawel

Jadawel is an Arabic-first, RTL fork of Baserow, self-hosted in Saudi Arabia.
Production is `https://app.jadawl.site`. The marketing site `jadawl.site` is a
separate static repository.

The data model, with the Arabic terms to use:

```
Workspace (مساحة عمل)
└─ Database (قاعدة بيانات)  ·  Dashboard (لوحة التحكم)  ·  Automation (أتمتة)  ·  Application
   └─ Table (جدول)
      ├─ Fields (حقول): text, long_text, number, boolean, date, single_select,
      │   multiple_select, link_row, file, email, url, phone_number, rating, formula, lookup
      ├─ Rows (صفوف)
      └─ Views (عروض): grid, gallery, form (نموذج), kanban, calendar, page (صفحة)
```

## Three ways in

| Surface | Use it for | Auth | Read |
|---|---|---|---|
| **MCP** (`https://app.jadawl.site/mcp/<KEY>/sse`, SSE) | Schema, rows and page views in one workspace | The key in the URL. Acts as the user who created the endpoint. | [references/mcp.md](references/mcp.md) |
| **REST API** (`https://app.jadawl.site/api/`) | Work with no MCP tool: views, filters, sorts, webhooks, trash, exports, health, admin | JWT or database token | [references/rest-api.md](references/rest-api.md) |
| **GitHub** (`code92-dev/jadawel_cranl`, `main`) | Code, CI and releases | `gh` CLI | [references/release.md](references/release.md) |

Use MCP first. It carries field protection and works with display names. Use REST
only for work MCP has no tool for.

## The data loop

Every data task follows this loop. Each step has its own done-when.

1. **Locate.** Call `list_databases`, then `list_tables`, then
   `get_table_schema`. Done when you hold the table ID, plus the exact name, type
   and options of every field you will touch.
2. **Read.** Call `list_table_rows` with `search`, at most 200 rows per page.
   Done when you have the row IDs you will change, or the count you were asked
   for.
3. **Gate.** If the plan deletes anything, changes a field type, touches more
   than 50 rows or creates a share or credential, show it (IDs, counts, how to
   undo) and wait for approval.
4. **Write.** Send at most 200 rows per call. Done when every batch has
   returned.
5. **Verify.** Read the changed rows back. Done when you can report IDs, counts
   and every failure verbatim.

## Facts that bite

- **Field names are keys.** `create_rows` and `update_rows` take display names
  exactly as `get_table_schema` returns them: Arabic letters, spaces and
  diacritics included. An unknown name fails, and the error lists the valid
  names.
- **Values have shapes.** `link_row` takes a list of row IDs from the linked
  table. Select fields take an option ID from the schema's `select_options` (the
  option's exact text also works). Dates
  are ISO `YYYY-MM-DD`. Numbers use Western digits.
- **Field creation has an order.** Regular fields, then `link_row`, then
  `lookup`, then `formula`. `create_fields` sorts a batch this way. Across
  separate calls, keep that order yourself.
- **`list_table_rows` only searches.** It has no filters or sorts and returns
  rows in grid order. For "rows where X > 5", use REST
  ([references/rest-api.md](references/rest-api.md)) or search and filter
  locally.
- **Deletes go to trash for 72 hours.** Then they are permanent. Restore through
  REST (`/api/trash/restore/`) or the trash UI.
- **One background worker serves everyone.** Production runs a single Celery
  worker at concurrency 1. Every row-triggered automation and webhook from a bulk
  write queues behind the others, so a 2,000-row import delays every user's
  automations. Split large jobs and space them out.
- **Protected fields return mask tokens.** See
  [references/mcp.md](references/mcp.md#protected-fields). Pass a token back
  verbatim to keep a value, or write a new literal value to replace it.
- **The formula engine has quirks.** Rounding is half to even. Division by zero
  fails even inside `if()`. `date_diff('mm')` counts month boundaries crossed.
  Schedules run in UTC.

## Deeper guides in the repository

Sanad, the in-app assistant, has expert guides that are accurate for any agent.
Read the matching one before advanced work. Their tool names are Sanad's, not
MCP's, so map them to the tools you have.

`backend/src/arabase/sanad/skills/<name>/SKILL.md`, where `<name>` is one of:

| `<name>` | Covers |
|---|---|
| `formulas` | Both formula languages, precision, dates, finance recipes |
| `dashboards` | Widgets, aggregations, chart choice |
| `automations` | Triggers, actions, loops, routers, limits |
| `forms` | Form views, conditions, prefills, sharing |
| `html-pages` | Page views: sandbox, data contract, RTL design |
| `app-builder` | Pages, layout, data sources, themes, Arabic fonts |

## Branches

- Page view (صفحة) HTML pages: [references/mcp.md](references/mcp.md#page-views)
- Views, filters, webhooks, trash, exports, health, backups, admin:
  [references/rest-api.md](references/rest-api.md)
- Release, deploy or "what's in production":
  [references/release.md](references/release.md)
- Arabic names, labels, field names or messages:
  [references/arabic.md](references/arabic.md)

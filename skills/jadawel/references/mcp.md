# Jadawel MCP server

## Connection

- URL: `https://app.jadawl.site/mcp/<KEY>/sse`. The transport is **SSE**, not
  Streamable HTTP. With a Claude CLI engine, register it with
  `claude mcp add --transport sse jadawel <URL>`.
- One endpoint gives access to **one workspace**, with the permissions of the
  user who created it. A second workspace needs a second endpoint.
- An endpoint is created in Jadawel under **Settings → MCP server**, or through
  `POST /api/mcp/endpoints/`. The key inside the URL is the only credential, so
  anyone holding the URL acts as that user.
- If the owner loses access to the workspace, the endpoint is **suspended**
  rather than deleted, and calls fail until access returns.

## Tools (20)

| Area | Tool | Arguments |
|---|---|---|
| Discover | `list_databases` | — |
| | `list_tables` | `database_id?` |
| | `get_table_schema` | `table_ids[]`. Returns fields with ID, name, type, `select_options` and `link_row_table_id`. |
| Build | `create_database` | `name` |
| | `create_table` | `database_id`, `name`, `fields[]?` (`{name, type, …extras}`) |
| | `update_table` | `table_id`, `name` (rename only) |
| | `delete_table` ⚠ | `table_id` (to trash) |
| Fields | `create_fields` | `table_id`, `fields[]` |
| | `update_fields` | `fields[]` (`{id, name?, type?, …options}`) |
| | `delete_fields` ⚠ | `field_ids[]` |
| Rows | `list_table_rows` | `table_id`, `search?`, `page=1`, `size=100` (maximum 200). Returns `{count, results}`. |
| | `create_rows` | `table_id`, `rows[]` (`{field name: value}`) |
| | `update_rows` | `table_id`, `rows[]` (`{id, field name: value}`) |
| | `delete_rows` ⚠ | `table_id`, `row_ids[]` |
| Pages | `list_page_views` | `table_id` |
| | `get_page_view` | `view_id`, `include_rows?` |
| | `create_page_view` | `table_id`, `name`, `html?`, `protected_field_ids?` |
| | `update_page_view` | `view_id`, `html` |
| | `list_page_view_revisions` | `view_id` |
| | `restore_page_view_revision` | `view_id`, `revision_id` |

⚠ marks a tool that takes effect immediately. MCP has no approval step, so the
gate in SOUL is the only check. Get approval before calling it.

Field-type extras for `create_table` and `create_fields`:

| Type | Extra |
|---|---|
| `number` | `number_decimal_places` |
| `date` | `date_include_time` |
| `single_select`, `multiple_select` | `select_options=[{value, color}]` |
| `link_row` | `link_row_table_id` |

## Limits

- At most 200 rows per call, for both reads and writes.
- A response is capped at 4 MB. Wide tables with long text need smaller pages.
- Each endpoint can hold 10,000 live mask tokens. Reading a large protected table
  page after page exhausts them, so read only what the task needs.

## Protected fields

An endpoint owner can put fields in the endpoint's **protection policy** (سياسة
حماية الحقول). Their non-empty values never leave Jadawel over MCP. Each one
arrives as a **mask token** (رمز إخفاء) instead.

- To keep the value: write the token back unchanged, into **the same cell** it
  came from.
- To replace it: write a new literal value. This is always allowed.
- Every other use fails closed: a token in another cell, row or field, an edited
  token, or a token older than 24 hours. A token is also **stale** once the row
  changes after you read it. To recover, re-read the row and use the fresh token.
- You cannot compute with a protected value: no sums, comparisons or formatting.
  If the task needs that, tell your owner. The work belongs inside Jadawel, in a
  formula, dashboard or Sanad, where the plaintext already is.
- Fields derived from a protected field, such as lookups and formulas that
  reproduce it, are masked too.

## Page views

A page view (صفحة) is a self-contained HTML document that Jadawel renders with
the view's live rows, and it can be shared on a public link.

1. Call `create_page_view(table_id, name)`. To edit an existing page, find it
   with `list_page_views(table_id)`.
2. Call `get_page_view(view_id)`. It returns the current HTML, the fields, a
   five-row sample, and the **runtime contract**. Read the contract every time,
   because it defines how the page receives data.
3. Call `update_page_view(view_id, html=…)`. Each write is a new revision, and
   `restore_page_view_revision` undoes one.

Page constraints:

- Design is entirely up to you. The page runs **sealed off from the network**,
  so inline all CSS, JS and graphics (SVG or `data:` URIs).
- Honour `view.dir`. Lay the page out with logical CSS (`margin-inline-start`,
  `inset-inline-end`) so it works in both RTL and LTR.
- A page that would show protected values needs approval from someone inside
  Jadawel before it renders them.

Read `docs/PAGE_VIEW.md` and
`backend/src/arabase/sanad/skills/html-pages/SKILL.md` in the repository for the
full contract and design guidance.

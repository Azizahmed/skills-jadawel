# Jadawel REST API

Base URL: `https://app.jadawl.site/api/`. The OpenAPI schema is at
`/api/schema.json`, and each database has its own generated documentation at
`https://app.jadawl.site/api-docs`. When an endpoint below is not enough, look it
up there.

**MCP field protection does not apply here.** REST returns plaintext for every
field the account can read. Use REST only for work MCP has no tool for, and
request only the fields you need (`include=`).

## Authentication

| Kind | Header | Reaches | Get it |
|---|---|---|---|
| **Database token** | `Authorization: Token <t>` | Row endpoints and field listing, in one workspace, with per-table create/read/update/delete rights | Jadawel → Settings → Database tokens |
| **JWT** | `Authorization: JWT <access_token>` | Everything the account may do: views, webhooks, trash, exports, admin | `POST /api/user/token-auth/` with `{email, password}` returns `access_token` and `refresh_token` |

The access token lasts **10 minutes**. Renew it with `POST /api/user/token-refresh/`
and `{refresh_token}`, which lasts 7 days. Prefer a database token for row work;
use JWT only for what needs it.

## Rows

```
GET    /api/database/rows/table/{table_id}/?user_field_names=true
POST   /api/database/rows/table/{table_id}/?user_field_names=true          one row
POST   /api/database/rows/table/{table_id}/batch/?user_field_names=true    {"items":[…]}, max 200
PATCH  /api/database/rows/table/{table_id}/batch/?user_field_names=true    {"items":[{"id":…,…}]}
POST   /api/database/rows/table/{table_id}/batch-delete/                   {"items":[ids]}
```

Always send `user_field_names=true`. Without it, keys are `field_<id>`.

Listing parameters:

| Parameter | Meaning |
|---|---|
| `page`, `size` | Pagination. `size` is at most 200. |
| `search` | Full-text search across fields |
| `order_by` | Comma-separated field names. Prefix `-` for descending, and quote names that contain commas. |
| `include` / `exclude` | Limit the fields returned |
| `filter__{field}__{type}=value` | Example: `filter__field_12__higher_than=5`. Combine filters with `filter_type=AND\|OR`. |
| `filters={json}` | Nested filter groups. The format is in the per-database documentation. |
| `view_id` | Apply a saved view's filters and sorts |

## Structure that MCP lacks

| Need | Endpoint |
|---|---|
| Fields (full detail) | `GET /api/database/fields/table/{table_id}/` |
| Views | `GET/POST /api/database/views/table/{table_id}/`, `PATCH/DELETE /api/database/views/{view_id}/` |
| View filters and sorts | `POST /api/database/views/{view_id}/filters/`, `…/sortings/`. Individual items live at `views/filter/{id}/` and `views/sort/{id}/`. |
| Webhooks | `GET/POST /api/database/webhooks/table/{table_id}/`, `…/test-call/`, `PATCH/DELETE /api/database/webhooks/{id}/` |
| Export | `POST /api/database/export/table/{table_id}/` starts a job. Poll `GET /api/database/export/{job_id}/`. |
| Trash | `GET /api/trash/workspace/{workspace_id}/`, `PATCH /api/trash/restore/` |
| Workspaces and applications | `GET /api/workspaces/`, `GET /api/applications/workspace/{id}/` |

Outgoing webhooks carry `X-Jadawel-Event` and `X-Jadawel-Delivery` headers.
Because production has one background worker, webhook calls queue behind
automations and imports.

## Fork endpoints (`/api/arabase/`)

| Endpoint | Purpose | Who |
|---|---|---|
| `workspace/{id}/activity/` | Recent activity in a workspace | Member |
| `workspace/{id}/database-stats/` | Counts of databases, tables and rows | Member |
| `workspace/{id}/table-access/` | Guest access to individual tables (وصول الجداول) | Workspace admin |
| `my-dashboards/…` | The user's collected dashboards (لوحاتي) | Self |
| `dashboard/{id}/share/` | A dashboard's public link and password | Editor |
| `admin/backup/`, `admin/backup/runs/` | Backup configuration and history | Staff |
| `admin/backup/run/` (POST) | Start a backup now | Staff, approval |
| `admin/backup/restore/` | **Overwrites the production database.** Leave it to your owner. | Staff |
| `admin/generative-ai/` | AI provider settings | Staff |

The billing and organizations plugins add `/api/billing/admin/plans/` and
`/api/organizations/admin/`, both for staff. An anonymous call to either must
return an authentication error, not `URL_NOT_FOUND`. A not-found error means the
plugin is missing from the deployed image.

## Health

| Check | Endpoint | Healthy |
|---|---|---|
| Liveness | `GET /api/_health/` | HTTP 200, no authentication needed |
| Full check | `GET /api/_health/full/` | Staff JWT. Runs every backend check and returns each result under `checks`. |
| Front end | `GET https://app.jadawl.site/` | HTTP 200 with HTML |

When you report a health problem, include the status code, the response body's
first lines, the time in UTC and the endpoint you called.

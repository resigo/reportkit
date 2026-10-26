---
name: building-reportkit
description: "REQUIRED before ANY change to, or debugging of, .rk.json, .rk-model.json, .rk-conn.json, or .rk-theme.json files. Also use when asked to create, edit, fix, or extend dashboards, reports, charts, KPIs, data tables, connections, models, or data visualizations. Always invoke this skill FIRST – read its workflow and the Scope rule before making changes. You build reports only by editing these JSON files; you never edit ReportKit's own code."
user-invocable: true
---

# Build ReportKit Reports

## Scope

You build and fix reports **only** by editing this project's files: `reports/*.rk.json`, `models/*.rk-model.json`, `connections/*.rk-conn.json`, `themes/*.rk-theme.json`, seeds, and dbt models. ReportKit's own code (its server, frontend, CLI, and `@reportkit/*` packages) is off-limits – never edit it or reimplement it in application code.

If a request seems to need a feature ReportKit doesn't have, **stop and tell the user it's a framework limitation** rather than editing the product or building your own version. But check the references first – the feature often exists. For example, to make column names human-readable, set a `label` on the model column or `labels` on the panel (see `references/panel-types.md`); never alias columns in SQL to a name with spaces or parentheses, which breaks on warehouses like BigQuery.

## Project structure

```
connections/     # Warehouse connections (.rk-conn.json)
models/          # Registered models (.rk-model.json)
reports/         # Report definitions (.rk.json)
themes/          # Theme files (.rk-theme.json)
seeds/           # (optional) CSV files loaded into SQLite on startup
```

## Workflow

### Before you start

1. Read `models/*.rk-model.json` – understand available data, columns, and table names
2. Read `connections/*.rk-conn.json` – know which warehouses and SQL dialects to use
3. If a report exists, read it before making changes
4. If no model exists for the data you need, create one first (see `references/data-blocks.md`)

### Building a report

1. Create or edit `reports/<id>.rk.json` – the `id` field must match the filename
2. A report has four sections: metadata, filters, data blocks, and pages (each page has a layout)
3. Use the panel type reference in `references/panel-types.md` for available chart types and their fields
4. Use the data block guide in `references/data-blocks.md` for query patterns

**If the report has a temporary timestamp id** (matches `report_\d+`): choose a short meaningful snake_case id (e.g. `monthly_revenue`), write to `reports/<new_id>.rk.json`, and delete the old file.

### Verify after every edit (MANDATORY)

You MUST run this loop after every report change. Do NOT consider the task done until `healthy: true`.

The server runs on `http://localhost:3001` by default (or `PORT` env var, or `port` in `.reportkit/settings.yml`).

1. `POST /api/reports/:id/check` – runs schema check, reference check, queries, and data diagnostics in one call
2. Read the response:
   - `healthy: true` → done
   - `phase: "schema"` → fix schema errors (missing fields, bad types), repeat from step 1
   - `phase: "reference"` → fix model/connection references or empty SQL, repeat from step 1
   - `phase: "data"` → fix data errors (unmapped columns, query failures, type mismatches), repeat from step 1
3. The `fix` field on each issue tells you exactly what to change
4. Repeat from step 1 until `healthy: true`

If health reports an error, you must fix it before doing anything else. Never ignore health errors.

See `references/diagnostics.md` for response format and issue categories.

## Report structure (quick reference)

```jsonc
{
  "version": 1,
  "id": "my_report",
  "title": "Report Title",
  "description": "What this report shows",
  "theme": "default",
  "filters": [ /* see references/filters.md */ ],
  "data": [ /* see references/data-blocks.md */ ],
  "pages": [
    { "id": "main", "title": "Overview", "layout": [ /* panels on a 24-column grid */ ] }
  ]
}
```

## Grid layout

24 columns wide, 1-indexed. Place panels using `row`, `col`, `span` (width), and `height`.

- `col + span` must not exceed 25
- Side-by-side: `col: 1, span: 12` + `col: 13, span: 12`
- Full width: `col: 1, span: 24`

## Pending instructions

`pendingInstructions` appears on panels.

- **Placeholder panel**: change `type` to `targetType`, add required fields, create data blocks, remove `pendingInstructions` and `targetType`.
- **Existing panel**: apply requested changes, remove `pendingInstructions`. Keep the panel type.

## Rules

- **Scope (see top):** only ever edit this project's `.rk.json` / `.rk-model.json` / `.rk-conn.json` / `.rk-theme.json` files, seeds, and dbt models. Never edit or reimplement ReportKit's server, frontend, CLI, or `@reportkit/*` packages – if a task seems to need that, stop and tell the user it's a framework limitation.
- Write SQL in the dialect of the target warehouse
- Connection files contain credentials – never log or expose them
- Never guess dataset/schema names – use `GET /api/data/search-tables?q=<pattern>` to discover them

## Reference files

Detailed documentation is in the `references/` directory:

- `references/panel-types.md` – all panel types with required/optional fields, style options, and defaults
- `references/data-blocks.md` – writing data blocks, SQL templates, aggregate, filters
- `references/metrics.md` – metrics system (aggregate, ratio, derived), proposing and referencing metrics
- `references/connectors.md` – warehouse connectors, connection file examples, query routing
- `references/filters.md` – filter types (dropdown, multiselect, daterange, range, text, toggle) and SQL usage
- `references/themes.md` – theme fields, style overrides, palette customization
- `references/diagnostics.md` – health endpoint, issue categories, troubleshooting workflow
- `references/schema-exploration.md` – search-tables API for discovering tables and columns; query API for validating SQL and previewing data

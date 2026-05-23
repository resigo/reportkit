# Diagnostics Reference

## Check endpoint (primary – use this)

```
POST /api/reports/:id/check
```

Runs schema validation, reference checks, queries, and data diagnostics in a single call. Fast-fails before touching the warehouse if structural errors are found.

Response:
```json
{
  "reportId": "my_report",
  "healthy": false,
  "phase": "data",
  "materializedAt": "2026-04-26T...",
  "counts": { "errors": 2, "warnings": 1 },
  "summary": "2 error(s) and 1 warning(s) found.",
  "issues": [
    {
      "code": "overflow",
      "severity": "error",
      "category": "visual",
      "message": "Panel 'table1' overflows the page.",
      "panelId": "table1",
      "pageId": "retention",
      "fix": "Reduce height to 22 or increase theme pageRows to at least 60.",
      "details": { "bottom": 60, "pageRows": 52 }
    }
  ],
  "blocks": {
    "monthly_revenue": {
      "rows": 12,
      "columns": ["month", "revenue", "orders"],
      "columnTypes": { "month": "string", "revenue": "number", "orders": "number" },
      "sample": [{ "month": "2024-01", "revenue": 45000, "orders": 120 }],
      "issues": []
    }
  }
}
```

`healthy` is `false` when any errors exist (warnings are OK).

The `phase` field indicates how far validation progressed:
- `"schema"` – stopped at schema errors (no DB queries run)
- `"reference"` – stopped at reference/SQL errors (no DB queries run)
- `"data"` – queries ran; `blocks` contains full column/sample metadata

## Health endpoint (read-only alternative)

```
GET /api/reports/:id/health
```

Returns diagnostics for the last materialized state without re-running queries.

## Issue categories

- **visual**: layout problems (overflow, overlap, out-of-bounds, undersized panels, excessive whitespace)
- **requirement**: structural problems (placeholders, pending instructions, empty SQL, missing data refs, missing required fields, duplicate ids, invalid model/connection/metric/dimension references, unwired filters)
- **data**: query problems (empty blocks, all-null columns, unmapped fields, type mismatches, KPI multi-row, excessive rows, failed filter options queries) – requires materialization first

## Warnings (severity: warning)

Warnings never flip `healthy` to `false`, but they flag reports that will look broken, mislead, or waste effort – treat them as a punch list and clear them. Each issue carries a `fix` string and, where useful, a `details` object with the raw numbers.

**Visual** – the panel is mis-sized for what it draws:
- **undersized** – panel `height` is below the recommended minimum for its type. Increase `height`.
- **excessive_whitespace** – a large vertical gap between panels. Move panels up or fill the gap.
- **orphan_data** – a data block no panel references. Add a panel that uses it, or remove the block.
- **legend_overflow** – too narrow for the legend; chips wrap or overflow the right edge. Increase `span`, drop a series, or set `"showLegend": false`.
- **legend_overlaps_plot** – the legend fits horizontally but sits on top of the data. Increase `span`, or hide the legend.
- **axis_labels_overlap** – the x-axis (or a HorizontalBarChart's y-axis) has more distinct values than fit at the configured `labelAngle`. Rotate (`"chartStyle": { "labelAngle": -45 }`), format the column (`xFormat`), increase `span`/`height`, or aggregate to fewer ticks.
- **y_ticks_crowded** – the chart is too short for a readable value axis; y labels stack. Increase `height`.

The squeeze warnings (`legend_*`, `axis_labels_overlap`, `y_ticks_crowded`) put the raw numbers (`numSeries`, `iw`, `ih`, `numTicks`, `longestLabelChars`, `labelAngle`, `projectedPx`, `slotPx`, `recommendedSpan`, `recommendedRows`) in `details` so you can compute a fix without re-deriving them.

**Requirement** – structural loose ends:
- **instructions_pending** – a panel still has `pendingInstructions`. Apply the requested change, then remove the field.
- **filter_unused** – a filter is defined but never referenced. Use `@filter_id` in a data block's SQL / `filterColumns`, or remove the filter.
- **invalid_filter_ref / invalid_filter_model / invalid_filter_column** – a filter (or a block's `filterColumns`) points at a filter id, model, or column that doesn't exist. Fix the reference.

**Data** – query output won't render as intended (requires materialization):
- **excessive_rows** – a block returned a very large result set (likely a missing GROUP BY or bad join). Aggregate or LIMIT.
- **y_not_numeric / kpi_not_numeric** – a y / KPI value column isn't numeric. Cast it in SQL or point at a numeric column.
- **kpi_multiple_rows** – a KPI block returns more than one row (only the first shows). Aggregate or LIMIT 1.
- **single_row_chart** – a chart block returned a single row. Remove the aggregate/LIMIT or add a GROUP BY.
- **filter_options_empty** – a filter's options query returned nothing (dropdown shows only "All"). Check the query.

## Troubleshooting workflow

When something is wrong (charts show "No data", panels look broken, etc.), **always check first**. Do NOT guess at the problem.

1. `POST /api/reports/:id/check` – runs all validation phases in one call
2. Read `phase` to know where the problem is:
   - `"schema"` → fix missing/invalid fields
   - `"reference"` → fix model/connection references or empty SQL
   - `"data"` → fix query failures, unmapped columns, type mismatches
3. Fix every error using the `fix` suggestion in each issue
4. Repeat until `healthy: true`

## Using the blocks response

- `columns` – verify panel field mappings match actual query output
- `sample` – verify the data looks correct
- `columnTypes` – check that chart y-axes have numeric data

## Post-build verification

After health returns `healthy: true`, verify:
- Every panel's x/y/value field matches a column in the data block's result
- Sample data matches what was asked for
- Filters are used: each filter ID appears in at least one data block's SQL (`@filter_id`) or `filters` list
- KPI cards have single-row results (use aggregate or LIMIT 1)
- No double aggregation: if SQL has GROUP BY, don't also set `aggregate`

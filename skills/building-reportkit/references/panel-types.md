# Panel Types Reference

## Overview

| Type | Category | Description | Data required |
|------|----------|-------------|:---:|
| `LineChart` | Chart | Trends over time, multi-series, dual Y-axis | Yes |
| `BarChart` | Chart | Categorical comparison, grouped/stacked | Yes |
| `AreaChart` | Chart | Area with gradient fill, multi-series, stacking, colorBy | Yes |
| `ScatterChart` | Chart | Correlation between two numeric values, bubble sizing | Yes |
| `PieChart` | Chart | Proportions and part-to-whole | Yes |
| `FunnelChart` | Chart | Conversion pipelines and drop-off | Yes |
| `HorizontalBarChart` | Chart | Rankings/leaderboards | Yes |
| `CohortChart` | Chart | Retention heatmap grid | Yes |
| `CustomChart` | Chart | Agent-generated visx chart | Yes |
| `KPICard` | Widget | Single-number summary | Yes |
| `DataTable` | Widget | Sortable, paginated table | Yes |
| `Title` | Content | Heading text | No |
| `Text` | Content | Body text block | No |
| `Image` | Content | Image panel | No |

## Default layout sizes

| Type | span | height |
|------|:----:|:------:|
| `LineChart` | 12 | 12 |
| `BarChart` | 12 | 12 |
| `AreaChart` | 12 | 12 |
| `ScatterChart` | 12 | 12 |
| `PieChart` | 12 | 12 |
| `FunnelChart` | 10 | 10 |
| `HorizontalBarChart` | 10 | 10 |
| `CohortChart` | 12 | 12 |
| `CustomChart` | 12 | 12 |
| `KPICard` | 6 | 4 |
| `DataTable` | 12 | 12 |
| `Title` | 24 | 3 |
| `Text` | 24 | 6 |
| `Image` | 12 | 16 |

---

## LineChart

Trends over time. Supports multiple series, long-format colorBy, and dual Y-axis.

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `x` | string | Yes | Category/time column for x-axis |
| `y` | string \| string[] | Yes | Value column(s). Array for wide-format multi-line |
| `colorBy` | string | | Long-format multi-series: one line per unique value. Takes precedence over y[] |
| `y2` | string \| string[] | | Secondary Y-axis (dashed lines, independent right-side scale) |
| `yLabels` | string[] | | Legend labels for y series |
| `y2Labels` | string[] | | Legend labels for y2 series |
| `xFormat` | string | | d3-time-format specifier for x-axis labels, e.g. `"%Y-%m"`, `"W%V"` (ISO week), `"%b %Y"` |
| `color` | string | | Line color (single-line only; multi-line uses palette) |
| `yFormat` | string | | d3-format specifier, e.g. `"$,.0f"`, `".0%"`, `"~s"` |
| `referenceLines` | ReferenceLine[] | | `[{ y: 500, label: "target", style: "dashed" }]` |
| `xLabel` / `yLabel` | string | | Axis titles |
| `yDomain` | [number, number] | | Override y-axis range |
| `showLegend` | boolean | | Show/hide legend (default: true) |

| chartStyle field | Type | Default | Description |
|------------------|------|---------|-------------|
| `strokeWidth` | number | 2 | Line thickness |
| `showGrid` | boolean | true | Horizontal grid lines |
| `fillOpacity` | number | 0.06 | Area fill under line (single-line) |
| `showValues` | boolean | false | Value labels above data points |

Multi-series: (1) `colorBy` long-format with GROUP BY, (2) `y: string[]` wide-format, (3) `y2` dual-axis.

---

## BarChart

Vertical bars for categorical comparison. Single series, grouped, or stacked.

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `x` | string | Yes | Category column |
| `y` | string \| string[] | Yes | Value column(s). Array for multi-series |
| `stack` | boolean | | When true + y array, render stacked bars (default: false) |
| `yLabels` | string[] | | Legend labels for multi-series |
| `xFormat` | string | | d3-time-format specifier for x-axis labels |
| `color` | string | | Bar fill (single series only) |
| `yFormat` | string | | d3-format specifier |
| `referenceLines` | ReferenceLine[] | | Threshold lines |
| `xLabel` / `yLabel` | string | | Axis titles |
| `yDomain` | [number, number] | | Override y-axis range |
| `showLegend` | boolean | | Show/hide legend (default: true) |
| `sort` | "asc" \| "desc" \| "none" | | Sort bars by value |

| chartStyle field | Type | Default | Description |
|------------------|------|---------|-------------|
| `barRadius` | number | 4 | Corner radius of bars |
| `showGrid` | boolean | true | Horizontal grid lines |
| `showValues` | boolean | false | Value labels above bars |

---

## AreaChart

Area chart with gradient fill. Supports multi-series, stacking, and long-format colorBy.

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `x` | string | Yes | Category/time column |
| `y` | string \| string[] | Yes | Value column(s). Array for wide-format multi-series. Used as value column when colorBy is set. |
| `yLabels` | string[] | | Legend labels for multi-series |
| `colorBy` | string | | Long-format multi-series: group by this column, one area per group |
| `stack` | boolean \| string | | `true` for stacked multi-y areas. String pivots long-format data by that column. |
| `xFormat` | string | | d3-time-format specifier for x-axis labels |
| `color` | string | | Area color (single series; multi-series uses palette) |
| `yFormat` | string | | d3-format specifier |
| `referenceLines` | ReferenceLine[] | | Threshold lines |
| `xLabel` / `yLabel` | string | | Axis titles |
| `yDomain` | [number, number] | | Override y-axis range |
| `showLegend` | boolean | | Show/hide legend (default: true) |

| chartStyle field | Type | Default | Description |
|------------------|------|---------|-------------|
| `strokeWidth` | number | 2 | Line thickness |
| `fillOpacity` | number | 0.1 | Gradient fill opacity |
| `showGrid` | boolean | true | Horizontal grid lines |

Multi-series: (1) `colorBy` – long-format, one area per group; (2) `y: string[]` – wide-format. Add `stack: true` for stacked areas. `stack: "column_name"` pivots long-format data into stacked areas.

---

## ScatterChart

Scatter plot for correlation. Supports colorBy and bubble sizing.

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `x` | string | Yes | Numeric column for x-axis |
| `y` | string | Yes | Numeric column for y-axis |
| `colorBy` | string | | Categorical color encoding per unique value |
| `color` | string | | Uniform point color (ignored when colorBy set) |
| `size` | string | | Numeric column for bubble size |
| `xFormat` | string | | d3-format for x-axis |
| `yFormat` | string | | d3-format for y-axis |
| `referenceLines` | ReferenceLine[] | | Threshold lines |
| `xLabel` / `yLabel` | string | | Axis titles |
| `xDomain` / `yDomain` | [number, number] | | Override axis range |
| `showLegend` | boolean | | Show/hide legend (default: true) |

| chartStyle field | Type | Default | Description |
|------------------|------|---------|-------------|
| `pointRadius` | number | 5 | Base point size |
| `showGrid` | boolean | true | Grid lines (both axes) |

---

## PieChart

Donut/pie chart for proportions.

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `label` | string | Yes | Category names (slice labels) |
| `value` | string | Yes | Numeric values (slice size) |
| `formatValue` | string | | d3-format specifier |
| `showLegend` | boolean | | Show legend (default: true) |

One row per category. Center displays total or hovered value.

---

## FunnelChart

Conversion pipeline with drop-off between stages.

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `label` | string | Yes | Stage names |
| `value` | string | Yes | Numeric counts per stage |
| `formatValue` | string | | d3-format specifier |

One row per stage, ordered widest (top) to narrowest (bottom). First row = 100%.

---

## HorizontalBarChart

Horizontal bars, auto-sorts descending. Good for rankings/leaderboards.

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `x` | string | Yes | Value column (bar length) – numeric |
| `y` | string | Yes | Category column (bar labels) |
| `color` | string | | Bar fill color |
| `limit` | number | | Max bars shown (default: 10) |
| `yFormat` | string | | d3-format for bar values |
| `referenceLines` | ReferenceLine[] | | Vertical reference lines |
| `xLabel` / `yLabel` | string | | Axis titles |
| `xDomain` | [number, number] | | Override value axis range |
| `sort` | "asc" \| "desc" \| "none" | | Sort order (default: "desc") |

| chartStyle field | Type | Default | Description |
|------------------|------|---------|-------------|
| `barRadius` | number | 4 | Corner radius |
| `showValues` | boolean | false | Value labels next to bars |

Note: `x` is numeric (bar length), `y` is category (label) – opposite of vertical charts.

---

## CohortChart

Heatmap grid for cohort retention analysis.

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `x` | string | Yes | Period column (numeric) |
| `y` | string | Yes | Cohort column (group label) |
| `value` | string | Yes | Cell intensity (0-100) |
| `color` | string | | Heatmap accent color |
| `formatValue` | string | | d3-format for cell values |

One row per cohort+period combination. Value interpreted as percentage 0-100.

---

## CustomChart

Agent-generated visx chart running in a sandbox. Use **only** when built-in chart types can't express what you need.

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `data` | string | Yes | **Single** data block id (same as every other panel). Not an array. |
| `render` | string | Yes | JS function body. Signature: `(data, width, height, theme, React, visx, utils) => React.ReactNode` |

### Runtime contract

The render function runs in a `new Function(...)` sandbox with `"use strict"`. These globals are explicitly set to `undefined`: `window`, `document`, `globalThis`, `self`, `top`, `parent`, `frames`, `fetch`, `XMLHttpRequest`, `WebSocket`, `localStorage`, `sessionStorage`, `indexedDB`, `importScripts`, `require`. No imports. No async. Return a React element (use `React.createElement`, no JSX).

**Arguments:**

| Arg | Type | Notes |
|-----|------|-------|
| `data` | `Array<Record<string, unknown>>` | Rows from the **one** data block referenced by `panel.data`. Already filtered. Never a keyed map. |
| `width`, `height` | `number` | Panel inner size in px. Subtract your margins before scaling. |
| `theme` | `{ accent: string; palette: string[] }` | Use `theme.accent` for single-series, `theme.palette[i]` for multi-series. |
| `React` | React module | Use `React.createElement(...)`. No JSX. |
| `visx` | flat namespace (see below) | **No sub-namespaces** – `visx.scaleBand`, NOT `visx.scale.scaleBand`. |
| `utils` | small helper bag (see below) | No `filters`, no `panel`, no DOM. |

### visx allowlist (flat – nothing else exists)

```
Scales:  scaleLinear  scalePoint  scaleBand  scaleOrdinal  scaleLog  scaleTime
Shapes:  LinePath  AreaClosed  Bar  Circle  Line  Polygon  Arc
Axes:    AxisBottom  AxisLeft  AxisRight  AxisTop
Grid:    GridRows  GridColumns
Layout:  Group
Curves:  curveMonotoneX  curveLinear  curveStep  curveBasis  curveCardinal  curveNatural
Event:   localPoint
```

Anything outside this list (`visx.scale.X`, `visx.shape.X`, `visx.axis.X`, `Tooltip`, `Legend`, `Pattern`, `Pie`, `Stack`, `Tree`, `extent`, `min`, `max`, `d3.*`, custom imports) is `undefined` and will throw "Cannot read properties of undefined".

### utils allowlist

```
fmt(n)               // 1234 -> "1.2K", 1500000 -> "1.5M"
extent(numArr)       // [min, max]
sum(numArr)
unique(arr)
groupBy(arr, key)    // Record<string, T[]>
colors               // string[] fallback palette
```

There is **no** `utils.filters`, `utils.theme`, or `utils.panel`. Filter values are not exposed to the render function – see "Reacting to filters" below.

### Reacting to filters

A CustomChart panel references **one** data block. When a filter changes, the dashboard re-runs that block's SQL and calls `render` again with the new rows. To make the chart respond to a filter:

1. Drive the change from the data block's SQL using `@filter_id` (see `data-blocks.md`).
2. The render function just draws whatever rows arrive – it doesn't need to know which filter changed.

Do not try to switch between multiple data blocks inside `render`; only one block's rows are available.

### Correct minimal example

```jsonc
{
  "id": "revenue_over_time",
  "type": "model",
  "model": "stg_orders",
  "sql": "SELECT strftime(CASE WHEN @granularity = 'day' THEN DATE_TRUNC('day', ordered_at) WHEN @granularity = 'week' THEN DATE_TRUNC('week', ordered_at) ELSE DATE_TRUNC('month', ordered_at) END, '%Y-%m-%d') AS period, SUM(amount) AS revenue FROM {{table}} GROUP BY 1 ORDER BY 1"
}
```

```jsonc
{
  "id": "revenue_chart",
  "type": "CustomChart",
  "title": "Revenue Over Time",
  "data": "revenue_over_time",
  "row": 1, "col": 1, "span": 24, "height": 12,
  "render": "if (!data.length) return React.createElement('div', null, 'No data'); const m = { top: 16, right: 24, bottom: 32, left: 56 }; const iw = width - m.left - m.right; const ih = height - m.top - m.bottom; const x = visx.scaleBand({ range: [0, iw], domain: data.map(d => d.period), padding: 0.2 }); const y = visx.scaleLinear({ range: [ih, 0], domain: [0, Math.max(...data.map(d => Number(d.revenue))) * 1.1] }); return React.createElement('svg', { width, height }, React.createElement('g', { transform: `translate(${m.left},${m.top})` }, data.map((d, i) => React.createElement(visx.Bar, { key: i, x: x(d.period), y: y(Number(d.revenue)), width: x.bandwidth(), height: ih - y(Number(d.revenue)), fill: theme.accent })), React.createElement(visx.AxisBottom, { scale: x, top: ih }), React.createElement(visx.AxisLeft, { scale: y, tickFormat: v => '$' + utils.fmt(v) })));"
}
```

### Common mistakes

- `"data": ["a", "b"]` – arrays are rejected; `panel.data` must be a single block id.
- `visx.scale.scaleBand` / `visx.shape.Bar` / `visx.axis.AxisBottom` – flat namespace only.
- `utils.filters?.x` – `utils.filters` is undefined; drive filter logic from the data block's SQL.
- Numeric columns may arrive as strings from some warehouses – `Number(d.col)` before scaling.
- Returning JSX – this is a runtime function body, not TSX. Use `React.createElement`.
- Calling `fetch`, reading `window`, importing modules – all blocked by the sandbox.

---

## KPICard

Single-number summary. Pair with `aggregate` in the data block.

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `value` | string | Yes | Column name for displayed value |
| `unit` | string | | Unit label – `"$"` renders before value |
| `subtitle` | string | | Context text below value (don't repeat title) |
| `formatValue` | string | | d3-format specifier |

| kpiStyle field | Type | Default | Description |
|----------------|------|---------|-------------|
| `valueSize` | "sm" \| "md" \| "lg" \| "xl" | "xl" | Display size |

---

## DataTable

Sortable, paginated data table.

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `columns` | string[] | | Column whitelist (omit for all) |
| `limit` | number | | Max rows loaded (default: 100) |

| tableStyle field | Type | Default | Description |
|------------------|------|---------|-------------|
| `compact` | boolean | false | Compact row padding |
| `striped` | boolean | true | Alternating row backgrounds |
| `pageSize` | number | 10 | Rows per page |

---

## Title

Heading text. No `data` field needed.

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `title` | string | Yes | Heading text |
| `level` | 1 \| 2 \| 3 | | Heading level (h1/h2/h3) |
| `align` | "left" \| "center" \| "right" | | Text alignment |
| `textColor` | string | | Font color override |
| `paddingRight` | number | | Right padding (px) |
| `paddingTop` | number | | Top padding (px) |

---

## Text

Body text block. No `data` field needed.

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `content` | string | Yes | Text content |
| `align` | "left" \| "center" \| "right" | | Text alignment |
| `textColor` | string | | Font color override |
| `paddingRight` | number | | Right padding (px) |

---

## Image

Image panel. No `data` field needed.

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `src` | string | Yes | Image URL |
| `alt` | string | | Alt text |

---

## Shared fields (all panel types with data)

| Field | Type | Description |
|-------|------|-------------|
| `colors` | string[] | Per-series color overrides (index-aligned with y[]) |
| `xFormat` | string | d3-format for x-axis (numeric axes only) |
| `yFormat` | string | d3-format for y-axis |
| `formatValue` | string | d3-format for displayed values |

## ReferenceLine format

```json
{ "y": 500, "label": "$500 target", "style": "dashed", "color": "#ff0000" }
{ "x": "2024-01", "label": "launch", "style": "solid" }
```

Fields: `x` (number | string), `y` (number), `label` (string), `color` (string), `style` ("solid" | "dashed" | "dotted").

## Usage guidance

- **KPICards** for single-number summaries (pair with `aggregate` in data block)
- **Charts** for trends and comparisons
- **DataTable** for detailed drill-downs
- **Title/Text** for section headers and annotations

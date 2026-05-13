# Panel Types Reference

## Overview

| Type | Category | Description | Data required |
|------|----------|-------------|:---:|
| `LineChart` | Chart | Trends over time, multi-series, dual Y-axis | Yes |
| `BarChart` | Chart | Categorical comparison, grouped/stacked | Yes |
| `AreaChart` | Chart | Area with gradient fill, multi-series, stacking, colorBy | Yes |
| `ScatterChart` | Chart | Correlation between two numeric values, bubble sizing | Yes |
| `PieChart` | Chart | Proportions and part-to-whole | Yes |
| `SparklineChart` | Chart | Compact mini line chart, no axes | Yes |
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
| `SparklineChart` | 6 | 3 |
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
| `xFormat` | string | | Axis label format tokens: YYYY, MM, DD, HH, mm, ss |
| `color` | string | | Line color (single-line only; multi-line uses palette) |
| `formatY` | string | | d3-format specifier, e.g. `"$,.0f"`, `".0%"`, `"~s"` |
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
| `xFormat` | string | | Axis label format tokens |
| `color` | string | | Bar fill (single series only) |
| `formatY` | string | | d3-format specifier |
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
| `xFormat` | string | | Axis label format tokens |
| `color` | string | | Area color (single series; multi-series uses palette) |
| `formatY` | string | | d3-format specifier |
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
| `formatX` | string | | d3-format for x-axis |
| `formatY` | string | | d3-format for y-axis |
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

## SparklineChart

Compact mini line chart – no axes, no labels. Place next to KPICards (span 4-8, height 3-4).

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `x` | string | Yes | Category/time column |
| `y` | string | Yes | Value column |
| `color` | string | | Line color |

| chartStyle field | Type | Default | Description |
|------------------|------|---------|-------------|
| `strokeWidth` | number | 2 | Line thickness |
| `fillOpacity` | number | 0.1 | Area fill under line |

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
| `formatY` | string | | d3-format for bar values |
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

Agent-generated visx chart running in a sandbox.

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `render` | string | Yes | JS function body. Receives `(data, width, height, theme, React, visx, utils)` |

Use only when built-in types are insufficient. Has access to visx scales, shapes, axes, grid, and curves.

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
| `formatX` | string | d3-format for x-axis (numeric axes only) |
| `formatY` | string | d3-format for y-axis |
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
- **SparklineChart** next to KPICard for "value + trend" layouts
- **Title/Text** for section headers and annotations

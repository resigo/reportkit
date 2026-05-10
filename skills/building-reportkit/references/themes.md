# Themes Reference

Create a `.rk-theme.json` in `themes/` and reference it by name in your report's `"theme"` field.

## Example

```json
{
  "name": "my_theme",
  "accent": "#0066FF",
  "palette": ["#0066FF", "#7C3AED", "#10B981", "#F43F5E", "#F59E0B"],
  "panel": { "border": "subtle", "radius": "medium" },
  "charts": {
    "strokeWidth": 2,
    "pointRadius": 3,
    "fillOpacity": 0.06,
    "barRadius": 4,
    "showGrid": true,
    "showDots": false,
    "showValues": false
  },
  "table": { "compact": false, "striped": true, "pageSize": 10 },
  "kpi": { "valueSize": "xl" },
  "accents": { "KPICard": "#000000" },
  "palettes": { "BarChart": ["#0066FF", "#7C3AED"] }
}
```

## Theme fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Theme identifier (matches filename) |
| `mode` | `"light"` / `"dark"` | Color mode |
| `accent` | string | Primary UI color |
| `background` | string | Background color |
| `palette` | string[] | Chart series colors (cycle in order) |
| `palettes` | Record<PanelType, string[]> | Per-panel-type palette overrides |
| `accents` | Record<PanelType, string> | Per-panel-type accent color overrides |
| `panel.border` | `"none"` / `"subtle"` / `"visible"` | Panel border style |
| `panel.radius` | `"none"` / `"small"` / `"medium"` | Panel corner radius |
| `nav.position` | `"left"` / `"top"` | Navigation bar position |
| `pageRows` | number | Rows per page (default 52) |

## Style sub-objects

**`charts`** (ChartStyle) – applies to all chart types:
| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `strokeWidth` | number | 2 | Line thickness |
| `pointRadius` | number | 3 | Point size |
| `fillOpacity` | number | 0.06 | Area fill opacity |
| `barRadius` | number | 4 | Bar corner radius |
| `showGrid` | boolean | true | Show grid lines |
| `showDots` | boolean | false | Show data points |
| `showValues` | boolean | false | Show value labels |
| `margins` | { top, right, bottom, left } | – | Chart area margins in px |
| `labelAngle` | number | -30 | X-axis label rotation in degrees |

**`table`** (TableStyle):
| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `compact` | boolean | false | Compact row padding |
| `striped` | boolean | true | Alternating row backgrounds |
| `pageSize` | number | 10 | Rows per page |

**`kpi`** (KPIStyle):
| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `valueSize` | `"sm"` / `"md"` / `"lg"` / `"xl"` | `"xl"` | Display size of value |

## Per-panel overrides

Individual panels can override theme styles via `chartStyle`, `tableStyle`, or `kpiStyle`:

```json
{
  "type": "BarChart",
  "chartStyle": { "showValues": true, "barRadius": 0 }
}
```

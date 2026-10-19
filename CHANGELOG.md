# Changelog

All notable changes to `qalainau/filament-warp-pivot` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- `PivotPage`, `PivotWidget` and `InteractsWithPivot` to put a canvas-rendered pivot table on a page, on a dashboard or in any Livewire component.
- Aggregation in the database with `GROUP BY`: `Count`, `CountDistinct`, `Sum`, `Avg`, `Min`, `Max`, `Ratio` and SQL expressions, with subtotals and grand totals that are computed from the records.
- Dimensions: `Column`, `Relationship` (any depth of `belongsTo`), `Date` (periods and cycles in the display timezone), `Bins` and `Expression`, with `limit()`, `orderByValue()` and `formatLabelUsing()`.
- Headers that open on demand, expand and collapse all, sorting rows by a column, "Other" for the rest of a top N, and `maxCells()`.
- Values shown as shares of totals, differences from the previous column and running totals.
- A field panel to rearrange rows, columns and values, saved layouts with filters (private, shared and default), and the layout in the URL.
- Number formats, heatmaps, data bars and colors by thresholds.
- Drilling down from a cell to its records, with links to each record and to another screen.
- Searching and sorting the drill-down list (`drillDownSearchable()`, `drillDownSortable()`), and Filament actions on its records (`drillDownRecordActions()`) that open on top of it and update the totals when they finish.
- Excel and CSV export and printing. The CSV writes values that spreadsheets would read as formulas (`=`, `+`, `-`, `@`) as text.
- Filament table filters, `viewAny` policies and tenancy.
- Cell selection with the keyboard, copying as tab-separated text, a live count, sum and average, and an ARIA grid for screen readers.
- `StdDev` and `Variance` measures, computed from the records for every subtotal.
- `Formula` measures and calculated values users write in the field panel, evaluated in PHP from the aggregated values.
- Pivot charts: column, stacked column, bar, stacked bar, line, area, column and line, pie, donut and scatter, with tooltips, a legend that hides series, drilling down from a mark and PNG export.
- Member filters with a searchable checklist, a filters area in the field panel, sorting headers by label or value, and the top or bottom N from the field panel or `limit(bottom: true)` and `orderByLabel()`.
- Users can change the aggregation, decimals, heatmap, data bars and conditional formatting (text or fill) of a value.
- Values shown as an index or a rank among their siblings.
- Compact, classic and flat layouts, grand totals and subtotals shown or hidden per axis, columns resized by dragging, and the display saved with layouts and in the URL.
- Context menus on headers and cells, expanding or collapsing one level, HTML export and full-screen mode.
- Conditions on dimensions: on the label (contains, begins with, between…), on a value of each header, and on dates (between two days, the last, current or next period).
- `Median` and `Product` measures, durations (`duration()`), negative numbers in parentheses, text for empty cells and divisions by zero, and custom separators.
- Conditional formatting with formulas over the cell and the other values of the cell (`formatWhen()`), with bold and italic, for all cells, values only or totals only.
- Calculated values over the columns of each record (`recordFormulaColumns()`), and `and`, `or` and `not` in formulas.
- The grand total row at the top (`grandTotalsPosition()`), the table and the chart together, a title, value labels with their aggregation, pivots that cannot be opened and closed, and headers in the user's own order.
- Choosing the columns of the drill-down and editing its cells in place (`drillDownEditable()`).
- PDF and PNG exports of the table, headers and footers for every export, the xlsx sheet name, exports handed to the server (`saveExportsUsing()`), and layouts saved to and opened from a file.
- Cell actions (`cellActions()`, `cellClickAction()`, `cellQuery()`), a JavaScript API, events, and hooks to draw cells and build the context menu.
- English, Japanese, German, French, Spanish, Italian, Brazilian Portuguese, Korean and Chinese (simplified and traditional) translations.

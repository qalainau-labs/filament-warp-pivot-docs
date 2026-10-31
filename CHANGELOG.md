# Changelog

All notable changes to `qalainau/filament-warp-pivot` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.2.0] - 2026-10-31

### Added

- `drillDownFormats()`: a closure per drill-down column to show its value your own way (money in cents, percentages,
  your own date format), also for columns that are not attributes and are worked out from the record.
  Columns of amounts and percentages are aligned to the right.

### Fixed

- Date-only columns of related models in the drill-down (`'order.order_date' => 'Order date'` with a `date` cast on
  the order) were shown with a time ("12:00 AM"), and could be shown as the day before in a timezone behind UTC.
  The cast of the related model is now used, also through nested relationships (`order.customer.signed_up_on`).

## [1.1.1] - 2026-10-25

### Fixed

- After publishing the migrations (`php artisan vendor:publish --tag=filament-warp-pivot-migrations`, as the README
  says for editing them), `php artisan migrate` stopped with "table already exists": Warp Pivot ran its own copy as
  well. A published migration now replaces the bundled one; migrations you have not published still run as before.

## [1.1.0] - 2026-10-25

### Upgrading

- Run `php artisan migrate` after updating. The new `update_warp_pivot_views_keys_to_strings` migration turns
  `warp_pivot_views.owner_id` and `tenant_id` into text columns and keeps the saved layouts.

### Added

- A "Tenancy" section in the README: a full panel example, what is kept to the current tenant, how the relationship is found, and things to know.

### Changed

- `warp_pivot_views.owner_id` and `tenant_id` are text columns, so users and tenants with UUID or ULID keys can save layouts (MySQL, MariaDB and PostgreSQL refused them before). Run `php artisan migrate` after updating: a new migration changes the columns of an existing table and keeps the saved layouts. `PivotView::$owner_id` and `$tenant_id` are now strings.
- When the model has no relationship to the tenant, the pivot says how to fix it (name another relationship with `scopeToTenant(relationship: …)`, or turn the scope off) instead of failing with "Call to undefined method".

## [1.0.1] - 2026-10-20

### Added

- Widgets that use `InteractsWithPageFilters` are queried again when the dashboard's filters change.
- When records are changed from a drill-down or a cell action, other pivots on the same page that read the same table are queried again.

### Fixed

- After a record is edited in the drill-down, the list is loaded again, so columns that the model changes along with the edited one (an amount worked out from the quantity, say) show their new values.
- Pressing Esc or opening another cell while an edit was being saved left the totals and the list as they were.
- Hiding or showing a drill-down column while editing could save the value to the neighbouring column.
- Editing a date and time in the drill-down moved it by the difference between Filament's timezone and the app's. Date-only columns are now shown and saved without a timezone.
- When a model event stopped the save of a drill-down edit, the edit looked saved.
- Saved layouts now keep the display settings and the column widths, and applying one no longer leaves the previous display in place.
- "Reset" now also puts the filters back to their defaults.
- Columns grow when their values get wider after the pivot is queried again (after an edit, say), and are measured again when the web font has loaded.
- The context menu closes when the pivot is queried again, so its items no longer act on a different cell.
- The selected cells stay selected when the pivot is queried again or a header is opened from the keyboard.
- The chart's measure, the sorted column and the column widths follow their measure when values are removed or reordered.
- A saved layout that was deleted or unshared is taken out of the list when it is chosen.

## [1.0.0] - 2026-10-20

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

# Changelog

All notable changes to `qalainau/filament-warp-pivot` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.6.0] - 2026-11-06

### Added

- `money('USD', divideBy: 100)` (and `divideBy()`) for amounts stored in cents: the integers are added up in SQL and each
  cell is divided once, so screens, formulas, filters and Excel cells get exact values (no `44.9999998`).
- `DrillDownFormat::money('USD', divideBy: 100)` formats drill-down columns of cents in `drillDownFormats()`.
- Values in rows: `valuesOn('rows')`, and a "Σ Values" field in the field panel to drag between Rows and Columns
  (saved as `valuesAxis` in the layout). Exports, totals, sorting and the chart work with it.
- `drillDownModalWidth()` changes the width of the drill-down dialog.
- Browser tests: `rowHeaderRect(row)` gives the position of a row header and its open/close triangle, and
  `fieldRect(name)` the position of a field in the field panel.

### Changed

- The drill-down dialog is wider (`Width::SevenExtraLarge`, was `FiveExtraLarge`), and columns of short values
  (numbers, dates, text up to 20 characters such as order numbers) no longer wrap; only long text wraps. Call
  `drillDownModalWidth(Width::FiveExtraLarge)` for the old width.

### Fixed

- Excel: amounts with a currency get the decimals of the currency (`$#,##0.00` for USD) when `decimals()` is not set,
  as on screen; they were written with no decimals.

## [1.5.0] - 2026-11-06

### Added

- Browser tests: `cellPoint({ row, column, measure })` gives the position of a cell (found by header paths or labels)
  to click, with its value and text, and `whenIdle()` waits until queries, the drill-down and saves have finished.

## [1.4.1] - 2026-11-06

### Fixed

- README: the "Tenancy" example put `rows(['store'])` without registering `store` in `dimensions()`, so it stopped with
  "The pivot has no dimension named [store]". It now passes `Relationship::make('store')`.

### Changed

- README, "Filters and permissions": the policy's `viewAny` (and `authorize()`) only refuse the pivot's data calls with
  a 403; the page still opens and stays in the navigation. An example shows Filament's `canAccess()` on the page to lock
  it and hide it from the navigation.
- README, "Dimensions": strings in `rows()` and `columns()` must be registered names, and a `Date` dimension is always
  named `column:granularity` (`ordered_at:month`), even alone, in `rows()`, `columns()`, `$conditions` and tests.

## [1.4.0] - 2026-11-06

### Added

- Instructions for AI coding assistants (Claude Code, Cursor, Codex) through [Laravel Boost](https://laravel.com/docs/boost):
  a guideline that Boost adds to `CLAUDE.md` and `AGENTS.md`, and the `warp-pivot-development` skill for building
  pivots and testing them. Select `qalainau/filament-warp-pivot` in `php artisan boost:install`, or add it to
  `"packages"` in `boost.json`. See [AI coding assistants (Laravel Boost)](README.md#ai-coding-assistants-laravel-boost).
- `php artisan warp:assign-tenant "App\Models\Team" 1`: for apps that add Filament's tenancy after saving layouts. Saved
  layouts without a tenant are hidden from every tenant; the command gives them (`warp_pivot_views`) to one tenant, together with
  the rows without a tenant of every other installed Warp plugin, in one transaction. `--dry-run` only counts them.
  See "Adding tenancy to an app that already has saved layouts" in the README.
- README: "Testing with tenancy". Another tenant's records and saved layouts are treated like ones that do not exist
  (no exception or 403), with a table of what each call returns and an example test of what a tenant sees.

## [1.3.0] - 2026-11-02

### Fixed

- Relative date conditions read naturally in every language: the current period is one word or phrase ("today",
  "this week", 「今日」, 「今天」, 「本週」, 「이번 달」) instead of "this" + "day" (「本天」), the units are plural after a
  number ("in the last 3 months", 「直近 3 か月」), and the words before the number agree with the period in Spanish,
  Italian and Portuguese ("en las últimas 2 semanas"). The language files have `filter.relative.*` in place of
  `filter.periods.*`; if you published and changed `filter.periods`, move your wording to `filter.relative`.
- Translations read more naturally: the Korean name of the Compact layout is a noun (「압축 형식」, it was the adverb
  「간단히」), Spanish speaks to the user with tú everywhere (it mixed usted and tú), and French says
  « Ordre personnalisé » for a custom sort order.

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

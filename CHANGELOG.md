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
- Excel and CSV export and printing. The CSV writes values that spreadsheets would read as formulas (`=`, `+`, `-`, `@`) as text.
- Filament table filters, `viewAny` policies and tenancy.
- Cell selection with the keyboard, copying as tab-separated text, a live count, sum and average, and an ARIA grid for screen readers.
- English and Japanese translations.

# Warp Pivot

![Sales by region and year with data bars, a heatmap and subtotals](https://raw.githubusercontent.com/qalainau-labs/filament-warp-pivot-docs/main/art/pivot.png)

**A pivot table for Filament that aggregates in the database and draws the result on a `<canvas>`: a million source rows are no heavier than a hundred, and a result of thousands of rows and hundreds of columns still scrolls smoothly. Filters, actions, policies and tenancy are Filament's own.**

Pivot components for the browser load every record into JavaScript and add them up there. That stops working long before a sales table reaches a million rows. Warp Pivot sends `GROUP BY` queries to your database instead, only for the headers that are open, and sends only the totals to the browser.

```php
use Qalainau\FilamentWarpPivot\Dimensions\Date;
use Qalainau\FilamentWarpPivot\Dimensions\Relationship;
use Qalainau\FilamentWarpPivot\Measures\Ratio;
use Qalainau\FilamentWarpPivot\Measures\Sum;
use Qalainau\FilamentWarpPivot\Pages\PivotPage;
use Qalainau\FilamentWarpPivot\Pivot;

class SalesPivot extends PivotPage
{
    public function pivot(Pivot $pivot): Pivot
    {
        return $pivot
            ->query(Sale::class)
            ->rows([
                Relationship::make('store.prefecture.region')->label('Region'),
                Relationship::make('store.prefecture')->label('Prefecture'),
            ])
            ->columns([Date::make('sold_at')->year()->label('Year')])
            ->values([
                Sum::make('amount')->label('Sales')->money('JPY'),
                Ratio::make('margin')->of(Sum::make('profit')->expression('amount - cost'), Sum::make('amount'))->percent(),
            ]);
    }
}
```

## Why

Measured on the demo app with 1,000,000 sales rows, 300 stores in three levels of regions and 5,000 products (MySQL 8.4 on a laptop, Chrome):

| | |
| --- | --- |
| First table on screen (regions × years) | about 1.2 s |
| Opening a region (one more query) | about 1.3 s |
| Opening every level at once | about 1.2 s |
| Drawing a frame of a 2,000 × 500 cell result while scrolling | about 1 ms |

Whatever is open, the source table is scanned once: the headers under closed headers are folded with `CASE` in the same query, and subtotals and grand totals are added up from that one result.

## Features

![The field panel to rearrange rows, columns and values](https://raw.githubusercontent.com/qalainau-labs/filament-warp-pivot-docs/main/art/fields.png)

- **Database aggregation**: count, distinct count, sum, average, minimum, maximum, ratios of two measures and SQL expressions. Averages and ratios of subtotals are computed from the records, never as averages of averages.
- **Dimensions**: columns (enum casts give the labels and order), `belongsTo` relationships of any depth (`store.prefecture.region`), dates by year, quarter, month, week, day or hour or by month of year, weekday and hour of day in the user's timezone (daylight saving included), numeric bins and SQL expressions.
- **Open what you need**: headers open on demand and only the opened part is queried. Expand or collapse everything, sort rows by any column, keep the top N per group with the rest as "Other", and cap the number of cells.
- **Show values as**: percentage of the row, column or grand total or of the parent row or column, difference or percentage difference from the previous column, and running totals.
- **Users arrange it**: a field panel to drag dimensions into rows and columns and measures into values, save named layouts (private or shared, with a default per user, filters included) and share a link with the layout in the URL.
- **Formats and colors**: decimals, prefixes and suffixes, currencies and percentages; heatmaps, data bars and colors by thresholds.
- **Drill down**: double-click a cell, or press Enter, to list the records behind it, with links to each record and to another screen (a Warp Sheet, a Warp Table or a resource's list) for the same records.
- **Export and print**: Excel (.xlsx) with merged headers, frozen labels and number formats, CSV, and printing with the headers repeated on every page.
- **Filament inside**: Filament table filters, policies (`viewAny`), tenancy, actions, notifications, dark mode and translations (English and Japanese).
- **Keyboard and screen readers**: arrow keys, Shift to extend a selection, Ctrl/Cmd+A and Ctrl/Cmd+C (tab-separated, ready to paste into a spreadsheet), and a live count, sum and average of the selection. The visible cells are exposed as an ARIA grid.

## Requirements

- PHP 8.3+
- Laravel 12 or 13
- Filament 5.x (Livewire 4)
- MySQL 8 / MariaDB 10.6+, PostgreSQL 14+ or SQLite

## Installation

> **Warp Pivot is a commercial plugin.** A license is required. After purchase you receive credentials for the private Composer repository.

Add the repository and authenticate:

```bash
composer config repositories.qalainau composer https://qalainau.privato.pub/composer
composer config --auth http-basic.qalainau.privato.pub "your-email" "your-license-key"
```

Install the package, run the migrations and publish Filament's assets:

```bash
composer require qalainau/filament-warp-pivot
php artisan migrate
php artisan filament:assets
```

The migration creates one table, `warp_pivot_views`, for the layouts users save. Your tables are not changed. To edit the migration, publish it with `php artisan vendor:publish --tag=filament-warp-pivot-migrations`.

> Run `php artisan filament:assets` again after every update of the package.

## Usage

### Where to put a pivot

Extend `PivotPage` for a full page or `PivotWidget` for a dashboard widget, and implement `pivot()`.

In any other Livewire component, implement `HasPivot`, `HasSchemas` and `HasActions`, use `InteractsWithPivot`, `InteractsWithSchemas` and `InteractsWithActions`, and render `{{ $this->getWarpPivot() }}`. The pivot's properties and methods are prefixed with `warpPivot`. `HasPivot` includes Filament's `HasTable`: the pivot uses a hidden table for its filters, so the component cannot have a Filament table of its own.

### Records

`query()` takes a model class, an Eloquent query or a closure returning one. Global scopes such as soft deletes apply.

```php
$pivot->query(fn (): Builder => Sale::query()->where('status', 'paid'));
```

### Dimensions

Put dimensions in `rows()` and `columns()`, outermost first. `dimensions()` registers more that users can add from the field panel.

```php
use Qalainau\FilamentWarpPivot\Dimensions\Bins;
use Qalainau\FilamentWarpPivot\Dimensions\Column;
use Qalainau\FilamentWarpPivot\Dimensions\Date;
use Qalainau\FilamentWarpPivot\Dimensions\Expression;
use Qalainau\FilamentWarpPivot\Dimensions\Relationship;

Column::make('channel');                                     // enum casts give labels and order
Column::make('status')->options(['open' => 'Open', 'closed' => 'Closed']);
Relationship::make('product.category')->titleAttribute('name')->orderBy('sort');
Date::make('sold_at')->month();                              // year, quarter, week, day, hour
Date::make('sold_at')->dayOfWeek()->weekStartsOn(0);         // monthOfYear, quarterOfYear, hourOfDay
Date::make('sold_at')->month()->timezone('Europe/Berlin');   // defaults to FilamentTimezone
Bins::make('amount')->width(1000);                           // 0 – 1,000, 1,000 – 2,000, …
Bins::make('amount')->ranges([1000, 5000, 20000]);           // under 1,000 … 20,000 and over
Expression::make('size')->sql("case when amount >= 5000 then 'large' else 'small' end");
```

Every dimension accepts `label()`, `formatLabelUsing(fn (string $value, string $label): string => …)`, `orderByValue()` (largest total first) and `limit(10)` (the ten largest per group, the rest as "Other"). Several `Date` dimensions on the same column are named `column:granularity`, such as `sold_at:year` and `sold_at:month`.

Relationships are grouped by their foreign key, and their labels are read afterwards, so a `belongsTo` with a million rows behind it costs nothing extra. Only the relationships in the middle of a path are joined.

### Measures

```php
use Qalainau\FilamentWarpPivot\Enums\ShowAs;
use Qalainau\FilamentWarpPivot\Measures\Avg;
use Qalainau\FilamentWarpPivot\Measures\Count;
use Qalainau\FilamentWarpPivot\Measures\CountDistinct;
use Qalainau\FilamentWarpPivot\Measures\Max;
use Qalainau\FilamentWarpPivot\Measures\Min;
use Qalainau\FilamentWarpPivot\Measures\Ratio;
use Qalainau\FilamentWarpPivot\Measures\Sum;

Count::make();
CountDistinct::make('customer_id');
Sum::make('amount')->money('USD');
Avg::make('amount')->decimals(0)->suffix(' avg');
Sum::make('profit')->expression('amount - cost');
Ratio::make('margin')->of(Sum::make('profit')->expression('amount - cost'), Sum::make('amount'))->percent();
Sum::make('amount')->name('share')->showAs(ShowAs::PercentOfColumnTotal);
Sum::make('amount')->name('growth')->showAs(ShowAs::PercentDifferenceFromPrevious);
```

A measure's name defaults to `column_function` (`amount_sum`); set another with `name()`, for example when the same column appears twice. Users can change how each value is shown in the field panel.

Color cells with `heatmap('success', 'danger')` (high and low colors), `dataBars()` and `colorWhen('<', 0, 'danger')`. Colors are Filament color names. Heatmaps and data bars are scaled to the visible detail cells, without subtotals and totals.

### Opening headers

All levels are open by default. For large results, open fewer levels at first; the rest is queried when a user opens a header:

```php
$pivot->defaultExpanded(rows: 0, columns: 1);
```

`maxCells(200_000)` (the default) caps the cells sent at once. When a result would be larger, the extra headers are left out with a warning, and subtotals and totals still include them.

### Filters, permissions and tenancy

`filters()` takes Filament table filters. They appear in a dropdown above the pivot, like on a table.

```php
$pivot->filters([
    SelectFilter::make('channel')->options(SalesChannel::class)->multiple(),
    Filter::make('this_year')->query(fn (Builder $query) => $query->whereYear('sold_at', now()->year)),
]);
```

When the model has a policy, `viewAny` decides who may see the pivot. Override it with `authorize(fn (): bool => …)`. With panel tenancy, records are scoped to the current tenant like resources are; use `scopeToTenant(false)` or `scopeToTenant(relationship: 'team')` to change that.

### Layouts

Users can rearrange the pivot with the field panel unless you call `configurable(false)`. Signed-in users can save layouts, with the filters, and share them with everyone (`shareableViews(false)` to prevent sharing, `savedViews(false)` to turn saving off). Layouts are stored per pivot under `key()`, which defaults to the Livewire component's class, and per tenant.

On pages, the current layout is also kept in the `pivot` query string parameter, so the address can be bookmarked or shared. Change the name with `urlKey('sales')`, turn it off with `urlKey(false)`, or turn it on for a widget or another component by giving it a name.

### Drill down

Double-click a cell (or press Enter) to list the records behind it, 25 per page.

```php
$pivot
    ->drillDownColumns(['sold_at' => 'Date', 'store.name' => 'Store', 'amount' => 'Amount'])
    ->recordUrl(fn (Sale $record): string => SaleResource::getUrl('view', ['record' => $record]))
    ->drillDownUrl(fn (array $conditions): string => SaleResource::getUrl('index', [
        'tableFilters' => ['channel' => ['value' => $conditions['channel'] ?? null]],
    ]));
```

`$conditions` maps each dimension name of the cell to its value. The closure also receives the `$query` of the records. Turn drilling down off with `drillDown(false)`. Cells of "Other" headers cannot be drilled into.

### Export

The export menu writes the table as it is on screen (open headers and sorting included) to Excel or CSV, or prints it. Limit the formats with `exportable(['csv'])`, or remove the menu with `exportable([])`. Name the file with `exportFileName('sales')`.

## Keyboard

| Key | Action |
| --- | --- |
| Arrow keys | Move between cells (Ctrl/Cmd to jump to the edge) |
| Shift + arrow keys | Extend the selection |
| Page Up / Page Down, Home / End | Move by a page, or to the first or last column |
| Ctrl/Cmd + A | Select every cell |
| Ctrl/Cmd + C | Copy the selection with its headers, tab-separated |
| Space | Open or close the group of the current row |
| Enter / double-click | List the records behind the cell |
| Escape | Clear the selection |

Click a row or column header to open or close it, and click a value header to sort the rows by it (largest first, then smallest first, then the original order).

## Limitations

- Distinct counts cannot be added up, so each visible level is queried separately when they are shown, and "Other" headers show no distinct count.
- Sorting by a column happens in the browser for the loaded rows. `orderByValue()` and `limit()` sort on the server.
- The drill-down list shows record attributes as text; it is not a Filament table.
- Dates in MySQL, MariaDB and SQLite are shifted to the display timezone with the daylight saving rules from 1990 to 2050. PostgreSQL uses its own timezone database.

## Troubleshooting

**Nothing is drawn.** Run `php artisan filament:assets` after installing or updating, and check that the page shows no JavaScript errors.

**A warning says some headers are left out.** The result is larger than `maxCells()`. Open fewer headers, filter, or use `limit()` on the dimension with many values.

**A relationship dimension throws an exception.** Every step of its path must be a `belongsTo` relationship.

## Changelog

See [CHANGELOG.md](CHANGELOG.md).

## Support

Open an issue at https://github.com/qalainau-labs/filament-warp-pivot-docs/issues. Do not post your license key in the issue.

## License

Warp Pivot is commercial software. See [LICENSE.md](LICENSE.md) for the license terms.

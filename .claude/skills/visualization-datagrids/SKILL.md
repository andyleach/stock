---
name: visualization-datagrids
description: Build a DataGrid with the dashworthy/visualizations package — generate the class, define typed columns, register the route, and test it with Pest. Use when the user wants a sortable/filterable data table or grid in a Laravel app.
boost-tags: [dashworthy, visualizations, datagrids]
---

# Building DataGrids

A DataGrid turns a query into a sortable, filterable, paginated table. The
package handles filtering, sorting, pagination, and schema generation from the
columns you define.

## 1. Generate

```bash
php artisan make:datagrid UserDataGrid
```

Creates `app/DataGrids/UserDataGrid.php` extending
`Dashworthy\Visualizations\DataGrids\Abstracts\DataGrid`. Implement
`getColumns()` and `getQuery()`.

## 2. Build

Columns are created with `make($sqlExpression, $field)` — the first argument is
the SQL select expression (usually a column reference), the second is the field
name.

```php
use Illuminate\Database\Query\Builder;
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\DB;
use Dashworthy\Visualizations\DataGrids\Abstracts\DataGrid;
use Dashworthy\Visualizations\DataGrids\Columns\Boolean;
use Dashworthy\Visualizations\DataGrids\Columns\DateTime;
use Dashworthy\Visualizations\DataGrids\Columns\Number;
use Dashworthy\Visualizations\DataGrids\Columns\Text;

class UserDataGrid extends DataGrid
{
    public function getColumns(): Collection
    {
        return collect([
            Number::make('users.id', 'ID')->asRowKey(),
            Text::make('users.name', 'Name'),
            Text::make('users.email', 'Email'),
            Boolean::make('users.is_active', 'Active'),
            DateTime::make('users.created_at', 'Joined'),
        ]);
    }

    public function getQuery(): Builder
    {
        return DB::table('users');
    }
}
```

Column types (in `Dashworthy\Visualizations\DataGrids\Columns`): `Text`, `Number`,
`Date`, `DateTime`, `Time`, `Boolean`, `Chip`.

Per-column modifiers: `->asRowKey()`, `->withoutSorting()`, `->withoutFiltering()`,
`->hidden()`, `->withoutExport()`, `->pinLeft()`, `->pinRight()`, `->header('...')`.

Optional overrides: `getFloatingFilters()` for filters on fields not shown as
columns, and `getDefaultSorts()` for the initial sort order.

## 3. Register the route

```php
// routes/api.php
use App\DataGrids\UserDataGrid;

Route::dataGrid(UserDataGrid::class);
```

This always registers POST `grids/users/data` and `grids/users/schema` (named
`grids.users.data` and `.schema`), plus view/export routes when those handler
methods exist. The path is derived from the class name minus the `DataGrid`
suffix. Override `getRoutePrefix()` to change the `grids` prefix.

## 4. Test with Pest

```bash
composer require dashworthy/pest-plugin-visualizations --dev
```

```php
use function Dashworthy\PestPluginVisualizations\dataGrid;

it('describes its schema', function () {
    dataGrid(UserDataGrid::class)
        ->assertColumnCount(5)
        ->assertHasColumn('Name')
        ->assertColumnIsSortable('Name')
        ->assertColumnIsFilterable('Email');
});

it('returns rows', function () {
    dataGrid(UserDataGrid::class)
        ->usingFilterSets(fn ($data) => $data->addAndFilterSet(/* ... */))
        ->assertRowCount(1)
        ->assertRowMatches(['Name' => 'Andrew']);
});
```

## Rules

- Name the class with a `DataGrid` suffix — the route name depends on it.
- Give exactly one column `->asRowKey()` (typically the id) so the front end can
  identify rows for selection.
- `getQuery()` returns a query `Builder` (`DB::table(...)`). Eloquent queries
  work too — chain `->toBase()` to hand back the underlying query builder
  (e.g. `User::query()->where('is_active', true)->toBase()`).
- Reference fully-qualified columns (`users.email`) so filtering/sorting and any
  joins resolve unambiguously.

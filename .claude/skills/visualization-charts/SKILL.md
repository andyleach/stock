---
name: visualization-charts
description: Build a Chart with the dashworthy/visualizations package — generate the class, define a label and datasets, register the route, and test it with Pest. Use when the user wants to add a chart, graph, or time-series visualization in a Laravel app.
boost-tags: [dashworthy, visualizations, charts]
---

# Building Charts

A Chart turns a query into one or more series (datasets) grouped by a label
(the x-axis / category field).

## 1. Generate

```bash
php artisan make:chart RevenueChart
```

Creates `app/Charts/RevenueChart.php` extending
`Dashworthy\Visualizations\Charts\Abstracts\Chart`. Implement three methods:
`getLabel()`, `getDatasets()`, and `getQuery()`.

## 2. Build

Labels and datasets are created with `make($sqlExpression, $field)` — the first
argument is the SQL select expression (an aggregate for datasets), the second is
the field name.

```php
use Illuminate\Database\Query\Builder;
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\DB;
use Dashworthy\Visualizations\Charts\Abstracts\Chart;
use Dashworthy\Visualizations\Charts\Datasets\Bar;
use Dashworthy\Visualizations\Charts\Datasets\Line;
use Dashworthy\Visualizations\Charts\Labels\Label;

class RevenueChart extends Chart
{
    public function getLabel(): Label
    {
        return Label::make('DATE(orders.created_at)', 'Date');
    }

    public function getDatasets(): Collection
    {
        return collect([
            Bar::make('SUM(orders.total)', 'Revenue'),
            Line::make('COUNT(orders.id)', 'Orders'),
        ]);
    }

    public function getQuery(): Builder
    {
        return DB::table('orders');
    }
}
```

Available dataset types (all in `Dashworthy\Visualizations\Charts\Datasets`):
`Bar`, `Line`, `Area`, `Pie`, `Doughnut`, `Scatter`. `Bar` also supports
`->stacked()` and `->stackGroup('name')`.

The package adds grouping/aggregation automatically from the label and dataset
expressions — write the bare `SELECT`/aggregate expressions, not a full
`GROUP BY` query.

## 3. Register the route

```php
// routes/api.php
use App\Charts\RevenueChart;

Route::chart(RevenueChart::class);
```

This registers POST `charts/revenues/data` and `charts/revenues/schema`
(named `charts.revenues.data` / `.schema`). The path is derived from the class
name minus the `Chart` suffix. Override `getRoutePrefix()` on the class to change
the `charts` prefix.

## 4. Test with Pest

```bash
composer require dashworthy/pest-plugin-visualizations --dev
```

```php
use function Dashworthy\PestPluginVisualizations\chart;

it('describes its schema', function () {
    chart(RevenueChart::class)
        ->assertHasLabel('date')
        ->assertHasDataset('revenue')
        ->assertDatasetCount(2);
});

it('returns data', function () {
    chart(RevenueChart::class)
        ->usingFilterSets(fn ($data) => $data->addAndFilterSet(/* ... */))
        ->assertResultCount(1)
        ->assertResultContains(['date' => '2024-01-01']);
});
```

## Rules

- Name the class with a `Chart` suffix — the route name depends on it.
- `getQuery()` returns a query `Builder` (`DB::table(...)`). Eloquent queries
  work too — chain `->toBase()` to hand back the underlying query builder
  (e.g. `Order::query()->where('status', 'paid')->toBase()`).
- Dataset expressions should be aggregates (`SUM`, `COUNT`, `AVG`, ...); the
  label is the grouping field.
- Use `->header('...')` for the display label; otherwise it defaults to the field.

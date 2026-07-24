---
name: visualization-metrics
description: Build a Metric with the dashworthy/visualizations package — generate the class, define a single scalar aggregate value, register the route, and test it with Pest. Use when the user wants a single-number stat, KPI, or summary tile in a Laravel app.
boost-tags: [dashworthy, visualizations, metrics]
---

# Building Metrics

A Metric resolves a query to a single scalar aggregate value (a KPI such as
total revenue or active users).

## 1. Generate

```bash
php artisan make:metric RevenueMetric
```

Creates `app/Metrics/RevenueMetric.php` extending
`Dashworthy\Visualizations\Metrics\Abstracts\Metric`. Implement two methods:
`getValue()` and `getQuery()`.

## 2. Build

The value is created with `Value::make($sqlExpression, $field)` — the first
argument is the SQL aggregate expression, the second is the field name.

```php
use Illuminate\Database\Query\Builder;
use Illuminate\Support\Facades\DB;
use Dashworthy\Visualizations\Metrics\Abstracts\Metric;
use Dashworthy\Visualizations\Metrics\Value;

class RevenueMetric extends Metric
{
    public function getValue(): Value
    {
        return Value::make('SUM(orders.total)', 'Revenue');
    }

    public function getQuery(): Builder
    {
        return DB::table('orders');
    }
}
```

## 3. Register the route

```php
// routes/api.php
use App\Metrics\RevenueMetric;

Route::metric(RevenueMetric::class);
```

This registers POST `metrics/revenues/data` and `metrics/revenues/schema`
(named `metrics.revenues.data` / `.schema`). The path is derived from the class
name minus the `Metric` suffix. Override `getRoutePrefix()` to change the
`metrics` prefix.

## 4. Test with Pest

```bash
composer require dashworthy/pest-plugin-visualizations --dev
```

```php
use function Dashworthy\PestPluginVisualizations\metric;

it('describes its schema', function () {
    metric(RevenueMetric::class)->assertHasValue('revenue');
});

it('returns the aggregate', function () {
    metric(RevenueMetric::class)
        ->usingFilterSets(fn ($data) => $data->addAndFilterSet(/* ... */))
        ->assertAggregateEquals(350.0);
});
```

## Rules

- Name the class with a `Metric` suffix — the route name depends on it.
- `getValue()` returns exactly one `Value` built from an aggregate expression
  (`SUM`, `COUNT`, `AVG`, ...).
- `getQuery()` returns a query `Builder` (`DB::table(...)`). Eloquent queries
  work too — chain `->toBase()` to hand back the underlying query builder
  (e.g. `Order::query()->where('status', 'paid')->toBase()`).

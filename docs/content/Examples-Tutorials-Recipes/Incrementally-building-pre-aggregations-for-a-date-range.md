---
title: Incrementally building pre-aggregations for a date range
permalink: /recipes/incrementally-building-pre-aggregations-for-a-date-range
category: Examples & Tutorials
subCategory: Query acceleration
tags: FILTER_PARAMS,incremental,pre-aggregations,partitions
menuOrder: 2
---

In scenarios where a large dataset spanning multiple years is pre-aggregated
with partitioning, it is often useful to only rebuild pre-aggregations between a
certain date range (and therefore only a subset of all the partitions). This is
because recalculating all partitions is often an expensive and/or time-consuming
process.

[comment]: <> (You only want to &#40;re&#41;build pre-aggregations between a certain date range, rather than)
[comment]: <> (every single partition largely because it can often be an expensive process to)
[comment]: <> (recalculate every partition.)
[comment]: <> (When building pre-aggregations, it is often useful to be able to control the date)
[comment]: <> (range of the pre-aggregations being refreshed. We can do this with)

> Add an example of a cube with a complex SQL query that has a time dimension
> filter inside a nested `WITH` clause

Let's use an example of a cube with a nested SQL query:

```javascript
cube('Orders', {

  sql: `
WITH base_orders AS (
  SELECT * FROM orders
),

base_line_items AS (
  SELECT * FROM line_items
)

SELECT * FROM base_orders
LEFT JOIN base_line_itemes
  ON base_orders.id = base_line_items.order_id
`,

  ...,

});
```

> Explain the issue; that doing a filter AFTER selecting all that data can be
> expensive, so it is smarter to apply that filter in the nested clause. Then
> show the same SQL query with the `FILTER_PARAMS` global being used in the
> nested clause.

Pre-aggregations within cubes can be configured to only build within a range by
using the [`buildRangeStart` and `buildRangeStart`
properties][ref-schema-ref-preagg-buildrange].

Now that we've added `FILTER_PARAMS` to the SQL query, we can update our
pre-aggregation definition with a build range:

```javascript
cube('Orders', {

  ...,

  preAggregations: {
    main: {
      measures: [],
      dimensions: [],
      timeDimension: CUBE.createdAt,
      granularity: `day`,
      partitionGranularity: `month`,
      refreshKey: {
        every: '1 day',
        incremental: true
      },
      // Now we'll define the following two properties to be used
      // as start and end values for the `timeDimension` above
      buildRangeStart: { sql: `SELECT DATE('2021-01-01')` },
      buildRangeEnd: { sql: `SELECT NOW()` }
    },
  }
});
```

[ref-schema-ref-preagg-buildrange]:
  /schema/reference/pre-aggregations#parameters-build-range-start-and-build-range-end

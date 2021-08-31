---
title: Optimizing complex cubes
permalink: /recipes/optimizing-complex-cubes
category: Examples & Tutorials
subCategory: Query Acceleration
menuOrder: 4
---

For cubes that are built from an expensive SQL query, we can

...

```javascript
cube('Orders', {
  sql: ``,

  preAggregations: {
    base: {
      type: `originalSql`,
      external: false,
    },
    main: {
      dimensions: [],
      measures: [],
    }
  },
})
```

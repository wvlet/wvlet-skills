---
name: testing-wvlet
description: Writes tests for Wvlet queries using embedded test statements. Use when writing tests for Wvlet queries, validating query results, or when the user mentions testing Wvlet or query assertions.
---

# Testing Wvlet Queries

Wvlet supports embedded `test` statements that run only in test-run mode.

## Result Inspection

| Function | Description |
|----------|-------------|
| `_.size` | Row count |
| `_.columns` | Column names |
| `_.rows` | Rows as array of arrays |
| `_.output` | Pretty print |
| `_.json` | JSON line format |

## Assertions

```sql
x should be y            -- Equality
x should not be y        -- Inequality
x should contain y       -- Membership/substring
x should not contain y
x < y, x <= y, x > y, x >= y
```

## Examples

```sql
-- Row count
from users where active = true
test _.size should be 3

-- Column schema
from users select id, name, age
test _.columns should be ['id', 'name', 'age']

-- All rows
from users select id, name, age
test _.rows should be [
  [1, "alice", 10],
  [2, "bob", 24],
]

-- Row containment
from users
test _.rows should contain [1, "alice", 10]

-- Multiple assertions
from orders where status = 'active'
test _.size > 0
test _.columns should contain 'order_id'
```

## Reference

https://wvlet.org/wvlet/docs/syntax/test-syntax

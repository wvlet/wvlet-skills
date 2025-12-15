---
name: testing-wvlet
description: Writes tests for Wvlet queries using embedded test statements. Use when writing tests for Wvlet queries, validating query results, or when the user mentions testing Wvlet or query assertions.
---

# Testing Wvlet Queries

## Overview

Wvlet supports embedded `test` statements to validate query results. Tests only run in test-run mode and are ignored during normal execution.

## Result Inspection Functions

| Function | Description |
|----------|-------------|
| `_.size` | Number of rows in the query result |
| `_.columns` | Column names in the query result |
| `_.output` | Pretty print query result |
| `_.json` | JSON line format of result rows |
| `_.rows` | Rows in array of array representation |

## Test Assertions

```sql
x should be y            -- Equality
x should not be y        -- Inequality
x should contain y       -- Substring/membership check
x should not contain y   -- Negated check
x = y, x is y            -- Alternative equality
x != y, x is not y       -- Alternative inequality
x < y, x <= y            -- Less than comparisons
x > y, x >= y            -- Greater than comparisons
```

## Examples

**Validate row count:**
```sql
from users
where active = true
test _.size should be 3
```

**Validate column schema:**
```sql
from users
select id, name, age
test _.columns should be ['id', 'name', 'age']
```

**Validate all rows (preferred):**
```sql
from users
select id, name, age
test _.rows should be [
  [1, "alice", 10],
  [2, "bob", 24],
  [3, "clark", 40],
]
```

**Validate row containment:**
```sql
from users
test _.rows should contain [1, "alice", 10]
```

**Multiple assertions:**
```sql
from orders
where status = 'active'
test _.size > 0
test _.columns should contain 'order_id'
```

## Running Tests

Tests are executed in test-run mode. During normal query execution, test statements are ignored.

## Reference

For complete test syntax documentation, see: https://wvlet.org/wvlet/docs/syntax/test-syntax

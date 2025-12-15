---
name: writing-wvlet
description: Writes queries using Wvlet, a flow-style query language alternative to SQL. Use when writing Wvlet queries, transforming data, or when the user mentions Wvlet, .wv files, or flow-style queries.
---

# Writing Wvlet Queries

## Design Principles

Wvlet is a brand-new query language designed for modern development:

- **Left-to-right, top-to-bottom flow** - Write queries in the order data is processed
- **Column-at-a-time operators** - Dedicated operators (`add`, `exclude`, `shift`) instead of complex SELECT
- **Reduced syntactic overhead** - Minimal parentheses, multiple WHERE clauses allowed
- **Lowercase keywords** - Less typing, consistent style
- **Dot notation** - Method chaining like `c1.sum.abs.round(1)`
- **Quote differentiation** - Single/double quotes for strings, backticks for identifiers
- **Braces for subqueries** - `{...}` for subqueries, `(...)` for functions
- **LLM-friendly** - Clean syntax optimized for AI code generation

## Core Concept

Wvlet uses flow-style syntax matching data processing order:

```
from → where → add → group by → agg → order by → limit
```

## Basic Query

```sql
from customers
where country = 'US'
select name, email
limit 10
```

## Data Source Operators

```sql
from table_name           -- Read table
from 'file.parquet'       -- Read file
show tables               -- List tables
show schemas              -- List schemas
```

## Subqueries

Subqueries are enclosed in `{ ... }` braces:

```sql
from {
  from orders
  where status = 'active'
}
where amount > 100

-- Named subquery with 'with'
with active_orders as {
  from orders
  where status = 'active'
}
from active_orders
where amount > 100
```

## Filtering

```sql
from orders
where status = 'active'
where amount > 100        -- Multiple where clauses allowed
```

## Column Operations

Wvlet favors column-at-a-time operators for clarity. Instead of SQL's complex SELECT:

```sql
-- SQL style (hard to understand intent)
SELECT sum(c1), c2 as c2_new, c4 + c5 as c101, c8, ..., c100, c6, c7 FROM tbl
```

Wvlet clarifies the intention of each column operation:

```sql
from tbl
add c1.sum                    -- Add aggregation
rename c2 as c2_new           -- Rename column
exclude c3                    -- Remove column
add c4 + c5 as c101           -- Add computed column
shift to right c6, c7         -- Move columns to end
```

**Column operators:**

```sql
from users
add full_name = concat(first_name, ' ', last_name)  -- Add column to the right
prepend id = generate_id()                           -- Add column to the left
exclude password                                     -- Remove column
rename email as contact_email                        -- Rename column
shift created_at                                     -- Move column to left
shift to right name                                  -- Move column to right
```

## Aggregation

```sql
from orders
group by customer_id
agg
  total = amount.sum,
  count = _.count,
  avg_amount = amount.avg
```

Common aggregations: `_.count`, `column.sum`, `column.avg`, `column.min`, `column.max`, `_.count_distinct(column)`, `_.array_agg(column)`

## Count Functions

`count` as a standalone operator:

```sql
from orders
count
-- Returns the total number of rows
```

In aggregations:

```sql
from orders
group by store_id
agg
  total_orders = _.count,                        -- Count all rows
  unique_products = _.count_distinct(product),   -- Count distinct values
  approx_users = _.count_approx_distinct(user_id) -- Approximate distinct (faster for large data)
```

Without grouping:

```sql
from orders
agg
  total = _.count,
  unique_customers = _.count_distinct(customer_id),
```

## Joins

```sql
from orders
join customers on orders.customer_id = customers.id
left join products on orders.product_id = products.id
```

## Sorting and Limiting

```sql
from products
order by price desc, name asc
limit 20
```

## Sampling

```sql
from large_table
sample 1000              -- Fixed number of rows
sample 10 percent        -- Percentage-based
```

## Expressions

**Type casting:**
```sql
'2025-01-01'::date
'100'::int
```

**Intervals:**
```sql
'7 days':interval
'1 month':interval
```

**Conditionals (no 'end' required, unlike SQL):**
```sql
if status = 'active' then 'Yes' else 'No'

case
  when score >= 90 then 'A'
  when score >= 80 then 'B'
  else 'C'
```

**Arrays and Structs:**
```sql
[1, 2, 3]                    -- Array (1-indexed)
{name: 'John', age: 30}      -- Struct
```

**Lambda functions:**
```sql
array.transform(x -> x * 2)
```

## Window Functions

Window functions compute data against the whole query result.

- `partition by (key)` - Split the query result into multiple partitions based on the key
- `order by (key)` - Define ordering of the rows within each partition
- `rows [start, end]` - Define the range of rows in each window

```sql
from orders
select
  customer_id,
  order_date,
  amount,
  amount.sum() over (
    partition by customer_id
    order by order_date
    rows [,0]
  ) as cumulative_amount
```

**Row frame specifications:**

| Wvlet Syntax | Description |
|--------------|-------------|
| `rows [,0]` | Unbounded preceding to current row (default) |
| `rows [-1,1]` | 1 row before to 1 row after |
| `rows [-1,0]` | 1 row before to current row |
| `rows [0,1]` | Current row to 1 row after |
| `rows [0,]` | Current row to unbounded following |

## Set Operations

```sql
from table1
concat                   -- UNION ALL
from table2

from table1
intersect
from table2

from table1
except
from table2
```

## Pivot and Unpivot

```sql
-- Pivot: rows to columns
from sales
pivot on region for total

-- Unpivot: columns to rows
from wide_table
unpivot value for category in (col1, col2, col3)
```

## Variables

```sql
val threshold = 100

from orders
where amount > threshold
```

## String Interpolation

Use `s"..."` syntax for easy string concatenation:

```sql
val name = 'wvlet'
select s"Hello ${name}!" as msg
-- Returns: 'Hello wvlet!'

from users
add greeting = s"Welcome, ${first_name} ${last_name}!"
```

## Table Variables

Define inline data tables for testing or lookup:

```sql
val products(id, name, price) = [
  [1, "Laptop", 999.99],
  [2, "Mouse", 29.99],
  [3, "Keyboard", 79.99],
]

from products
where price < 100
```

With type annotations:

```sql
val users(id: int, name: string, active: boolean) = [
  [1, "Alice", true],
  [2, "Bob", false],
]

from users
where active = true
```

## Comments

```sql
-- Single line comment

---
## Multi-line documentation comment (supports Markdown)
...
---
```

## Trailing Commas

Wvlet supports trailing commas for easier editing and cleaner diffs:

```sql
from orders
select
  id,
  customer_id,
  amount,   -- trailing comma allowed
```

## Best Practices

- Chain operators in logical data flow order
- Use multiple `where` clauses instead of complex AND conditions
- Use `add`/`exclude`/`shift` instead of listing all columns
- Apply `where` after `agg` for HAVING-like filtering
- Use `val` for reusable values
- Prefer lowercase keywords

## Reference

For complete syntax documentation, see: https://wvlet.org/wvlet/docs/syntax/

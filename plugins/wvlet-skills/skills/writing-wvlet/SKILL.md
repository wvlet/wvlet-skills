---
name: writing-wvlet
description: Writes queries using Wvlet, a flow-style query language alternative to SQL. Use when writing Wvlet queries, transforming data, or when the user mentions Wvlet, .wv files, or flow-style queries.
---

# Writing Wvlet Queries

Wvlet uses flow-style syntax: `from → where → add → group by → agg → order by → limit`

Pipe `|` can optionally be used between operators for readability:
```sql
from orders | where status = 'active' | limit 10
```

## Basic Query

```sql
from customers
where country = 'US'
select name, email
limit 10
```

## Data Sources

```sql
from table_name           -- Table
from 'file.parquet'       -- File
show tables               -- List tables
```

## Subqueries

Use `{ ... }` braces for subqueries:

```sql
from { from orders where status = 'active' }
where amount > 100

-- Named subquery
with active_orders as { from orders where status = 'active' }
from active_orders
```

## Filtering

```sql
from orders
where status = 'active'
where amount > 100        -- Multiple where clauses allowed
```

## Column Operations

```sql
from users
add full_name = concat(first_name, ' ', last_name)  -- Add column
prepend id = generate_id()                           -- Add to left
exclude password                                     -- Remove column
rename email as contact_email                        -- Rename
shift created_at                                     -- Move to left
shift to right name                                  -- Move to right
```

## Aggregation

```sql
from orders
group by customer_id
agg
  total = amount.sum,
  count = _.count,
  avg_amount = amount.avg

-- Without grouping
from orders
agg total = _.count, unique = _.count_distinct(customer_id)

-- Standalone count
from orders
count
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
sample 1000              -- Fixed rows
sample 10 percent        -- Percentage
```

## Expressions

```sql
'2025-01-01'::date                    -- Type casting
'7 days':interval                     -- Intervals
if status = 'active' then 'Yes' else 'No'  -- Conditional (no 'end' needed)
case when score >= 90 then 'A' else 'B'    -- Case expression
[1, 2, 3]                             -- Array (1-indexed)
{name: 'John', age: 30}               -- Struct
array.transform(x -> x * 2)           -- Lambda
```

## Window Functions

```sql
from orders
select
  customer_id,
  amount.sum() over (
    partition by customer_id
    order by order_date
    rows [,0]              -- Unbounded preceding to current
  ) as cumulative_amount
```

Row frames: `rows [,0]` (unbounded to current), `rows [-1,1]` (1 before to 1 after), `rows [0,]` (current to unbounded)

## Set Operations

```sql
from table1 concat from table2    -- UNION ALL
from table1 intersect from table2
from table1 except from table2
```

## Pivot/Unpivot

```sql
from sales pivot on region for total
from wide_table unpivot value for category in (col1, col2, col3)
```

## Variables

```sql
val threshold = 100
from orders where amount > threshold

-- String interpolation
val name = 'wvlet'
select s"Hello ${name}!" as msg

-- Table variables
val products(id, name, price) = [
  [1, "Laptop", 999.99],
  [2, "Mouse", 29.99],
]
from products
```

## Comments

```sql
-- Single line

---
## Multi-line (supports Markdown)
---
```

Trailing commas are allowed in select lists.

## Reference

https://wvlet.org/wvlet/docs/syntax/

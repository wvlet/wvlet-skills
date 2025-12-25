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
show schemas              -- List schemas
describe users            -- Show table schema
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
where amount > 100              -- Multiple where clauses allowed
where status in ('a', 'b')      -- Membership test
where age between 18 and 65     -- Range check
where name like 'J%'            -- Pattern matching
where deleted_at is null        -- Null check
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

-- Additional aggregation functions
agg
  unique = _.count_distinct(product_id),
  items = _.array_agg(name),
  top_product = product_id.max_by(amount)   -- product_id with max amount

-- Standalone count
from orders
count
```

## Dedup

```sql
from events
dedup                     -- Remove duplicate rows
```

## Joins

```sql
from orders
join customers on orders.customer_id = customers.id
left join products on orders.product_id = products.id
right join suppliers on products.supplier_id = suppliers.id
cross join regions
asof join prices on orders.ts = prices.ts   -- Time-based join
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

## Conditionals (no `end` keyword needed, unlike SQL)

```sql
-- if expression (NOT if...then...else...end)
add status_label = if status = 'active' then 'Yes' else 'No'

-- case expression (NOT case...when...then...else...end)
add grade = case
  when score >= 90 then 'A'
  when score >= 80 then 'B'
  else 'C'
```

## Expressions

```sql
'2025-01-01'::date                    -- Type casting
'7 days':interval                     -- Intervals
[1, 2, 3]                             -- Array (1-indexed)
{name: 'John', age: 30}               -- Struct
map {a: 1, b: 2}                      -- Map
array.transform(x -> x * 2)           -- Lambda
`column name`                         -- Identifier with spaces
```

## String Interpolation and Raw SQL

```sql
val name = 'wvlet'
select s"Hello ${name}!" as msg       -- String interpolation

-- Execute raw SQL when needed
sql"SELECT * FROM legacy_table"
```

## Unnest Arrays

```sql
from users
unnest tags as tag                    -- Expand array to rows
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

-- Table variables
val products(id, name, price) = [
  [1, "Laptop", 999.99],
  [2, "Mouse", 29.99],
]
from products
```

## Data Modification

```sql
from transformed_data
save to new_table                     -- Create/overwrite table

from new_records
append to existing_table              -- Append to table

from users where inactive = true
delete                                -- Delete matching rows
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

- Full syntax: https://wvlet.org/wvlet/docs/syntax/
- See also: [syntax-reference.md](syntax-reference.md) for complete operator/expression tables

# Wvlet Syntax Reference

## Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `from` | Read table/file | `from users`, `from 'data.parquet'` |
| `where` | Filter rows (multiple allowed) | `where status = 'active'` |
| `select` | Choose columns | `select id, name` |
| `add` | Append column | `add total = price * qty` |
| `prepend` | Insert column at left | `prepend row_id = 1` |
| `exclude` | Remove column | `exclude password` |
| `rename` | Rename column | `rename email as mail` |
| `shift` | Move column left | `shift created_at` |
| `shift to right` | Move column right | `shift to right name` |
| `group by` | Group rows | `group by category` |
| `agg` | Aggregate expressions | `agg total = amount.sum` |
| `count` | Count rows | `from users count` |
| `order by` | Sort rows | `order by price desc` |
| `limit` | Restrict rows | `limit 100` |
| `sample` | Random sample | `sample 1000`, `sample 10 percent` |
| `dedup` | Remove duplicates | `dedup` |
| `join` | Inner join | `join t2 on t1.id = t2.id` |
| `left join` | Left outer join | `left join t2 on ...` |
| `right join` | Right outer join | `right join t2 on ...` |
| `cross join` | Cartesian product | `cross join t2` |
| `asof join` | Time-based join | `asof join t2 on t1.ts = t2.ts` |
| `concat` | Union all | `from t1 concat from t2` |
| `intersect` | Set intersection | `from t1 intersect from t2` |
| `except` | Set difference | `from t1 except from t2` |
| `pivot` | Rows to columns | `pivot on region for total` |
| `unpivot` | Columns to rows | `unpivot val for cat in (a, b)` |
| `unnest` | Expand array to rows | `unnest tags as tag` |
| `with` | Named subquery | `with t as { ... }` |
| `describe` | Show schema | `describe users` |
| `show tables` | List tables | `show tables` |
| `show schemas` | List schemas | `show schemas` |
| `show catalogs` | List catalogs | `show catalogs` |
| `debug` | Debug output | `debug` |
| `save to` | Create/overwrite table | `save to new_table` |
| `append to` | Append to table | `append to existing` |
| `delete` | Delete rows | `from t where ... delete` |

## Expressions

| Expression | Description | Example |
|------------|-------------|---------|
| `::` | Type cast | `'2025-01-01'::date`, `'100'::int` |
| `:interval` | Interval literal | `'7 days':interval` |
| `in` | Membership test | `status in ('a', 'b')` |
| `not in` | Negated membership | `id not in (1, 2, 3)` |
| `between` | Range check | `age between 18 and 65` |
| `like` | Pattern match | `name like 'J%'` |
| `not like` | Negated pattern | `email not like '%test%'` |
| `is null` | Null check | `deleted_at is null` |
| `is not null` | Not null check | `email is not null` |
| `and`, `or`, `not` | Boolean logic | `a > 0 and b < 10` |
| `[...]` | Array literal | `[1, 2, 3]` |
| `{...}` | Struct literal | `{name: 'Jo', age: 30}` |
| `map {...}` | Map literal | `map {a: 1, b: 2}` |
| `arr[i]` | Array access (1-indexed) | `tags[1]` |
| `x -> expr` | Lambda | `x -> x * 2` |
| `s"..."` | String interpolation | `s"Hello ${name}"` |
| `sql"..."` | Raw SQL | `sql"SELECT * FROM t"` |
| `_` | Previous input reference | `from t add _.count` |
| `` `name` `` | Quoted identifier | `` `column name` `` |

## Aggregation Functions

| Function | Description |
|----------|-------------|
| `_.count` | Count rows |
| `_.count_distinct(col)` | Count distinct values |
| `col.sum` | Sum values |
| `col.avg` | Average |
| `col.min` | Minimum |
| `col.max` | Maximum |
| `col.max_by(val)` | col value at max val |
| `col.min_by(val)` | col value at min val |
| `_.array_agg(col)` | Collect into array |

## Conditionals (no `end` keyword needed, unlike SQL)

```sql
-- if expression
if status = 'active' then 'Yes' else 'No'

-- case expression
case
  when score >= 90 then 'A'
  when score >= 80 then 'B'
  else 'C'
```

## Window Functions

```sql
col.sum() over (
  partition by key
  order by ts
  rows [start, end]
)
```

| Frame | Description |
|-------|-------------|
| `rows [,0]` | Unbounded preceding to current |
| `rows [-1,1]` | 1 before to 1 after |
| `rows [0,]` | Current to unbounded following |

# Table of contents

- [Guiding principles](#guiding-principles)
- [Structure](#structure)
  * [CTEs](#ctes)
- [General syntax and style](#general-syntax-and-style)
  * [Line length](#line-length)
  * [Capitalization](#capitalization)
  * [Alignment](#alignment)
  * [Single vs multiple lines](#single-vs-multiple-lines)
  * [Beginning vs end of lines](#beginning-vs-end-of-lines)
  * [Quotations](#quotations)
  * [Comments](#comments)
  * [Naming things](#naming-things)
  * [Data Types](#data-types)
- [Clauses](#clauses)
  * [`select`](#-select-)
  * [`from`](#-from-)
  * [`join`](#-join-)
  * [`where`](#-where-)
  * [`group by`](#-group-by-)
  * [`order by`](#-order-by-)
  * [Set clauses](#set-clauses)
- [Operators](#operators)
  * [Arithmetic Operators](#arithmetic-operators)
  * [Logical Operators](#logical-operators)
- [Functions](#functions)
  * [Date functions](#date-functions)
  * [Window functions](#window-functions)



# Guiding principles

- Optimize primarily for readability, maintainability, and robustness rather than for fewer lines of code. Newlines are cheap; people's time is expensive.
- Do not obsess over fewer lines, however, aim to be reasonably DRY (see [the DRY Principal](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)). If you type the same line twice, it needs to be maintained in two places.

# Structure

## CTEs

- Use [Common Table Expressions](http://www.postgresql.org/docs/8.4/static/queries-with.html) (CTEs) early and often - they will make your queries more straightforward to read/reason about, can be referenced multiple times, and are easier to adapt/refactor later
    - Strongly prefer CTEs over subqueries - only consider a subquery when an equivalent CTE is overwhelmingly more verbose

- Where performance permits, each CTE should perform a single, logical unit of work
- CTEs with confusing or noteable logic should be commented
- CTE names should be as concise as possible while still being clear
    - Avoid long names like `replace_sfdc_account_id_with_master_record_id` and prefer a shorter name with a comment in the CTE. This will help avoid table aliasing in joins.

- Format CTEs in a consistent way for readability
    - Begin the first CTE on a new line after the `with` keyword. Open a CTE on the same line as the name.
    - The body of a CTE should be one indent further than the CTE name
    - Close the CTE on a new line (the same indentation level as the CTE name)
- Always create a `final` CTE that you `select *` from at the end. That way you can quickly inspect the output of other CTEs used in the query to debug the results.

```
/* Good */

with

events as (
    select
    ...
),

/* CTE comments go here */
final as (
    select
    ...
)

select * from final

/* Bad */

with events as (
select *
from events),

test as (
select *
from test)

select *
from events
left join tests

```

# General syntax and style

## Line length

- Lines should ideally not be longer than 120 characters.
- Very long lines are harder to read, especially in situations where space may be limited like on smaller screens or in side-by-side version control diffs.

## Capitalization

- Use lowercase SQL: It's just as readable as uppercase SQL and you won't have to constantly be holding down a shift key.

```
/* Good */
select * from users

/* Bad */
SELECT * FROM users

/* Bad */
Select * From users

```

## Alignment

- Left align everything: This is easier to keep consistent, and is also easier to write.

```
/* Good */
select email
from customers
where email like '%@domain.com'

/* Bad */
select email
  from customers
 where email like '%@domain.com'

/* Bad */
select
  id    as account_id,
  name  as account_name,
  type  as account_type
from
```

## Single vs multiple lines

- If there's only one thing, put it on the same line as the opening keyword.
- If there are multiple things, put each one on its own line (including the first one), indented one level more than the opening keyword.

```
/* Good */
select *
from users
where email = 'example@domain.com'

/* Good */
select
    id,
    email,
    created_at
from users

/* Good */
select
    user_id,
    count(*) as total_charges
from charges
group by user_id

/* Bad */
select id, email, created_at
from users

/* Bad */
select id,
    email
from users
```

## Beginning vs end of lines

### Logic goes at the beginning of lines

- Never end a line with an operator like `and`, `or`, `+`, `||`, etc. If code containing such operators needs to be split across multiple lines, put the operators at the beginning of the subsequent lines.
- You should be able to scan the left side of the query text to see the logic being used without having to read to the end of every line.
- The operator is only there for/because of what follows it. If nothing followed the operator it wouldn't be needed, so putting the operator on the same line as what follows it makes it clearer why it's there.

```
/* Good */
select *
from customers
where
    email like '%@domain.com'
    and plan_name != 'free'

/* Bad */
select *
from customers
where
    email like '%@domain.com' and
    plan_name != 'free'
```

### Commas go at the the end of lines

- Commas don't provide much information other than correctly separating lines (compared to logical operators, for instance), and therefore they can go at the end of lines for readability.
- This is also more consistent with the syntax other programming languages.

```
/* Good */
select
    id,
    email
from users

/* Bad */
select
    id
    , email
from users
```

## Quotations

- Use single quotes for strings. Some SQL dialects like BigQuery support using double quotes or even triple quotes for strings, but for most dialects:
    - Double quoted strings represent identifiers.
    - Triple quoted strings will be interpreted like the value itself contains leading and trailing single quotes.

```
/* Good */
select *
from customers
where email like '%@domain.com'

/* Bad ...
Will probably result in an error like `column "%@domain.com" does not exist`. */
select *
from customers
where email like "%@domain.com"

/* Bad ...
Will probably be interpreted like '\\'%domain.com\\''. */
select *
from customers
where email like '''%@domain.com'''
```

## Comments

- Comments should go near the top of your query, or at least near the closest `select`
- Try to only comment on things that aren't obvious about the query (e.g., why a particular ID is hardcoded, etc.)
- Always use `/* */` comment syntax. This allows single-line comments to naturally expand into multi-line comments without having to change their syntax.
- When expanding a comment into multiple lines, keep the opening `/*` on the same line as the first comment text and the closing `/` on the same line as the last comment text.

```
/* Good */

-- Bad

/* Good:  Lorem ipsum dolor sit amet, consectetur adipiscing elit,
sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
Dolor sed viverra ipsum nunc aliquet bibendum enim. */

-- Bad:  Lorem ipsum dolor sit amet, consectetur adipiscing elit,
-- sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
-- Dolor sed viverra ipsum nunc aliquet bibendum enim.
```

- Respect the character line limit when making comments. Move to a new line or to the model documentation if the comment is too long

## Naming things

### Capitalization style

- All field names should be [snake-cased](https://en.wikipedia.org/wiki/Snake_case) and as stated above, lowercase.

```
  /* Good */
  select
    dvcecreatedtstamp as device_created_timestamp
  from table

  /* Bad */
  select
    dvcecreatedtstamp as DeviceCreatedTimestamp
  from table
```

### Aliases

- Always use the `as` keyword when aliasing columns, expressions, and tables.
- Never use reserved words as identifiers, otherwise, the identifier will have to be quoted everywhere it's used.

```
/* Good */
select
  name,
  sum(amount) as sum_amount
from customers

/* Good */
select max(id) as max_customer_id
from customers

/* Bad */
select max(id)
from customers

/* Bad */
select max(id) max_customer_id
from customers
```

### Table aliases

- When joining to any data from a different source, ALL fields should be prefixed with the data source (table name/alias) to avoid confusion. You should be able to tell at a glance where a column is coming from.

```
/* Good */
select
    customers.email as customer_email,
    orders.invoice_number as order_invoice_number,
    orders.total_amount as order_total_amount
from customers
inner join orders on customers.id = orders.customer_id

/* Bad */
select
    email,
    invoice_number,
    total_amount
from customers
inner join orders on customers.id = orders.customer_id
```

### Table naming

- Strongly prefer to reference the full table name instead of an alias. If the table name consists of 3 words or less don't alias it.
- When the table name is long (~20), consider aliasing to something descriptive. NEVER alias with an initialism.
- Tip: Use a subset of the words as the alias if it makes sense (e.g. if `partner_shipments_order_line_items` is the only line items table being referenced it could be reasonable to alias it as just `line_items`).

```
/* Good */
select
    customers.email as customer_email,
    orders.invoice_number as orders_invoice_number
from customers
inner join orders on customers.id = orders.customer_id

/* Acceptable, but not preferred */
select
  bfcopex.account_id,
  date_details.fiscal_year
  cost_category.cost_category_level_1
from budget_forecast_cogs_opex as bfcopex
left join date_details on date_details.first_day_of_month = bfcopex.accounting_period

/* Bad */
select
    c.email,
    invoice_number
from customers as c
inner join orders as o on c.id = o.customer_id
```

### Boolean column aliases

- Boolean columns should be prefixed with a present or past tense third-person singular verb, such as:
    - `is_` or `was_`
    - `has_` or `had_`
    - `does_` or `did_`

```
/* Good */
select
  deleted as is_deleted,
  contract as has_contract
from table

/* Bad */
select
  deleted,
  contract
from table
```

### Date/time column aliases

- Date columns based on UTC should be named like `<event>_date`.
- Date columns based on a specific timezone should be named like `<event>_date_<timezone indicator>` (e.g. `order_date_et`).
- Date+time columns based on UTC should be named like `<event>_at`.
- Date+time columns based on a specific timezone should be named like `<event>_at_<timezone indicator>` (e.g `created_at_pt`).
- US timezone indicators:
    - `et` = Eastern Time.
    - `ct` = Central Time.
    - `mt` = Mountain Time.
    - `pt` = Pacific Time.
- Months should be indicated as such and should always be truncated to a date format, e.g. `deal_closed_month`
- Always avoid key words like `date` or `month` as a column name

```
/* Good */
select
  deal_timestamp as deal_closed_at,
  date as deal_closed_date
from deals

/* Bad */
select
  deal_timestamp,
  deal_date
from deals
```

## Data Types

- Use default data types and not aliases. Review the [Snowflake summary of data types](https://docs.snowflake.com/en/sql-reference/intro-summary-data-types.html) for more details. The defaults are:
    - `number` instead of `decimal`, `numeric`, `integer`, `bigint`, etc.
    - `float` instead of `double`, `real`, etc.
    - `varchar` instead of `string`, `text`, etc.
    - `timestamp` instead of `datetime`
- The exception to this is for timestamps. Prefer `timestamp` to `time`. Note that the default for `timestamp` is `timestamp_ntz` which does not include a time zone.

# Clauses

## `select`

- Prefer using `select distinct` instead of grouping by all columns. This makes the intention more clear.

```
/* Good */
select distinct country

/* Good */
select distinct
    state,
    country

/* Bad */
select
    customer_id,
    created_at as purchase_date_at
from orders
group by customer_id, purchase_date_at
```

## `from`

- Only one table should be in the `from`. Never use `from`-joins:

```
/* Good */
select
  projects.name AS project_name,
  count(backings.id) as backings_count
from projects
inner join backings on backings.project_id = projects.id

/* Bad */
select
  projects.name as project_name,
  count(backings.id) as backings_count
from projects, backings
where backings.project_id = projects.id
```

## `join`

### Explicit joins

- Be explicit when joining, e.g. use `left join`instead of `join`. (Default joins are `inner`). Multiple lines of `inner join`s easier to scan.

### General style

- Put each `join` on its own line, at the same indentation level as `from`.
- If there is only one join condition, put it on the same line as the `join`.
- If there are multiple join conditions, end the `join` line with `on` and put each condition on its own line (including the first one), indented one level more than the `join`.

```
/* Good */
from customers
left join orders on customers.id = orders.customer_id

/* Good */
from customers
left join orders on customers.id = orders.customer_id
left join locations on customers.location_id = locations.location_id

/* Good */
from customers
left join orders on
    customers.id = orders.customer_id
    and customers.region_id = orders.region_id

/* Bad */
from customers
left join orders
    on customers.id = orders.customer_id

/* Bad */
from customers
left join orders
    on customers.id = orders.customer_id
left join locations 
		on customers.location_id = locations.location_id

/* Bad */
from customers
left join orders on customers.id = orders.customer_id
    and customers.region_id = orders.region_id

/* Bad */
from customers
left join orders
    on customers.id = orders.customer_id
    and customers.region_id = orders.region_id
```

- Begin with `inner join`s and then list `left join`s, order them semantically, and do not intermingle `left join`s with `inner join`s unless necessary:

```
/* Good */
select
	...
from projects 
inner join backings on ...
inner join orders on ...
left join customers on ...
left join locations on ...

/* Bad */
select
	...
from projects 
inner join backings on ...
left join customers on ...
inner join orders on ...
left join locations on ...
```

### Join conditions

- Don't use `using` in joins.
    - Having all joins use `on` is more consistent.
    - If additional join conditions need to be added later, `on` is easier to adapt.
    - `using` can produce inconsistent results with outer joins in some databases (Snowflake).

- In join conditions, put the table that was referenced first immediately after `on`.
    - This makes it easier to determine if the join is going to cause the results to fan out.

```
/* Good */
select *
from customers
left join orders on customers.id = orders.customer_id
/* primary key = foreign key --> one-to-many --> fan out */

/* Good */
select *
from orders
left join customers on orders.customer_id = customers.id
/* foreign key = primary key --> many-to-one --> no fan out */

/* Bad */
select *
from customers
left join orders on orders.customer_id = customers.id

```

- When inner joining, put filter conditions in the `where` clause instead of the `join` clause.
    - Only join conditions should be put in a `join` clause. All filter conditions should be put together in the `where` clause.

```
/* Good */
select
    ...
from orders
inner join customers on orders.customer_id = customers.id
where
    orders.total_amount >= 100
    and customers.email like '%@domain.com'

/* Bad */
select
    ...
from orders
inner join customers on
    orders.customer_id = customers.id
    and customers.email like '%@domain.com'
where orders.total_amount >= 100

```

## `where`

- Use `where` instead of `having` when either would suffice.
- Queries filter on the `where` clause earlier in their processing, so `where` filters are more performant.

## `group by`

- Always group by column name/aliases for better readability, especially in Snowflake, where you can reference an alias in the same statement

```
/* Good */
group by plan_name

/* Good */
group by
    plan_name,
    signup_month

/* Bad */
group by 1, 2, 3

/* Bad */
order by plan_name, signup_month

/* Bad */
order by plan_name,
    signup_month

```

## `order by`

- Always order by column name/aliases for better readability
- Avoid using an `order by` clause unless it's necessary to produce the correct result.
- There's no need to incur the performance hit. If consumers of the query need the results ordered they can normally do that themselves.

## Set clauses

- Use `union all` instead of `union` unless duplicate rows really do need to be removed.
- `union all` is more performant because it doesn't have to sort and de-duplicate the rows.

# Operators

## Arithmetic Operators

- Prefer `!=` to `<>`. This is because `!=` is more common in other programming languages and reads like "not equal" which is how we're more likely to speak

## Logical Operators

- Consider performance of your SQL engine. Understand the difference between `like` vs `ilike`, `is` vs `=`, and `not` vs `!` vs `<>` and use appropriately.

### Null

- Use `is null` instead of `isnull`, and `is not null` instead of `notnull`.
- `isnull` and `notnull` are specific to Redshift.

### Like

- Prefer `lower(column) like '%match%'` to `column ilike '%Match%'`. This lowers the chance of stray capital letters leading to an unexpected result

### `case`

- Use a `case` statement instead of `iff` or `if`. `case` statements are universally supported, whereas Redshift doesn't support `iff`, and in BigQuery the function is named `if` instead of `iff`.
- Consider simplifying a repetitive `case` statement where possible:

```
/* Good */
case
    when field_id = 1 then 'date'
    when field_id = 2 then 'integer'
    when field_id = 3 then 'currency'
    when field_id = 4 then 'boolean'
    when field_id = 5 then 'variant'
    when field_id = 6 then 'text'
end as field_type

/* Better, when possible */
case field_id
    when 1 then 'date'
    when 2 then 'integer'
    when 3 then 'currency'
    when 4 then 'boolean'
    when 5 then 'variant'
    when 6 then 'text'
end as field_type
```

### `in` lists

- Break long lists of `in` values into multiple indented lines with one value per line.

```
/* Good */
where email in (
        'user-1@example.com',
        'user-2@example.com',
        'user-3@example.com'
    )

/* Bad */
where email in ('user-1@example.com', 'user-2@example.com', 'user-3@example.com')
```

- Don't put extra spaces inside of parentheses.

```
/* Good */
select *
from customers
where plan_name in ('monthly', 'yearly')

/* Bad */
select *
from customers
where plan_name in ( 'monthly', 'yearly' )
```

### `||`

- Use `||` instead of `concat`.
- `||` is a standard SQL operator, and in some databases like Redshift `concat` only accepts two arguments.

### `coalesce`

- Use `coalesce` instead of `ifnull` or `nvl`.
- `coalesce` is universally supported, whereas Redshift doesn't support `ifnull` and BigQuery doesn't support `nvl`.
- `coalesce` is more flexible and accepts an arbitrary number of arguments.

# Functions

## Date functions

- Prefer the explicit date function over `date_part`, but prefer `date_part` over `extract`, e.g. `DAYOFWEEK(created_at)` > `DATE_PART(dayofweek, 'created_at')` > `EXTRACT(dow FROM created_at)`
    - Note that selecting a date's part is different from truncating the date. `date_trunc('month', created_at)` will produce the calendar month ('2019-01-01' for '2019-01-25') while `SELECT date_part('month', '2019-01-25'::date)` will produce the number 1

- Be careful using [DATEDIFF](https://docs.snowflake.net/manuals/sql-reference/functions/datediff.html), as the results are often non-intuitive.
    - For example, `SELECT DATEDIFF('days', '2001-12-01 23:59:59.999', '2001-12-02 00:00:00.000')` returns `1` even though the timestamps are different by one millisecond.
    - Similarly, `SELECT DATEDIFF('days', '2001-12-01 00:00:00.001', '2001-12-01 23:59:59.999')` return `0` even though the timestamps are nearly an entire day apart.
    - Using the appropriate interval with the `DATEDIFF` function will ensure you are getting the right results. For example, `DATEDIFF('days', '2001-12-01 23:59:59.999', '2001-12-02 00:00:00.000')` will provide a `1 day interval` and `DATEDIFF('ms', '2001-12-01 23:59:59.999', '2001-12-02 00:00:00.000')` will provide a `1 millisecond interval`.

- For functions that take date part parameters, specify the date part as a string rather than a keyword.
    - While some advanced SQL editors can helpfully auto-complete and validate date part keywords, if they get it wrong they'll show superfluous errors.
    - Less advanced SQL editors won't syntax highlight date part keywords, so using strings helps them stand out.
    - Using a string makes it unambiguous that it's not a column reference.

```
/* Good */
date_trunc('month', created_at)

/* Bad */
date_trunc(month, created_at)
```

## Window functions

- You can put a window function all on one line if it doesn't cause the line's length to be too long.
- If breaking a window function into multiple lines:
    - Put each sub-clause within `over ()` on its own line, indented one level more than the window function:
        - `partition by`
        - `order by`
        - `rows between` / `range between`
    - Put the closing `over ()` parenthesis on its own line at the same indentation level as the window function.

```
/* Good */
select
    customer_id,
    invoice_number,
    row_number() over (partition by customer_id order by created_at) as order_rank
from orders

/* Good */
select
    customer_id,
    invoice_number,
    row_number() over (
		    partition by customer_id
		    order by created_at
    ) as order_rank
from orders

/* Bad */
select
    customer_id,
    invoice_number,
    row_number() over (partition by customer_id
                       order by created_at) as order_rank
from orders
```
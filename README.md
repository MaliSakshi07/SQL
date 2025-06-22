# SQL (Structured Query Language)
## SQL Basics

### DDL - Data Definition Language

- `CREATE` - create objects in the database eg. schemas or tables within schemas
- `ALTER` - change objects in the database eg. tablees, indexes or schemas
- `DROP`-  delete objects from the database eg. tables or indexes
- `TRUNCATE` - deletes all data from a table
- `COMMENT` - adds comments to the data dictionary
- `RENAME` - renames an object in the database eg. table

### DML - Data Modification Language

- `SELECT` - get rows from a table or view
- `INSERT` -  add new rows into a table
- `UPDATE` - change rows in a table
- `DELETE` - delete rows from a table
- `MERGE` - perform an UPSERT operation (Insert or Update depending on whether a matching row already exists or not)
- `CALL` -  runs a PL/SQL procedure or Java program
- `EXPLAIN PLAN` - explain the way the data is loaded
- `LOCK TABLE` - control concurrency (TODO: by blocking writes? Explain this better)

### Query Predicates

- `WHERE` - returns rows that match the WHERE's expression argument
  - eg. `WHERE age >= 18` to return only people who are 18 years or older
  - eg. `WHERE age >= 18 AND age <= 25` - to return only people of prime biological age (which I am well past! :cry:)
  - can result in better performance to filtering early in the query execution to read less data from a table
    (especially in columnar [databases](databases.md) like MPP systems)
- `BETWEEN` - checks if the value of a field is in within a range of two values, usually numbers of some kind -
  - eg. `WHERE age BETWEEN 18 AND 25`
    integer, float / decimal etc.
- `IN` - checks if the value of a field is an exact match to one of a given enumeration of possible literal values
  - eg. `WHERE first_name in ("Hari", "Neo", "Morpheus")` - return only people with cool names from this pre-approved
    list
- `HAVING` - like `WHERE` but filters late in the query execution, evaluated after `GROUP BY` since `WHERE` filter
  cannot operate on values calculated by aggregate functions since they have no yet been calculated when `WHERE` is
  filtering rows
  - `WHERE` happens before the `GROUP BY`, so there is no way for `WHERE` clause to know what the value of the
    aggregate function is
  - eg. restricting the results to only rows where the `MAX(field)` is greater than `N`, you cannot `WHERE` on the
    `MAX()` because it doesn't exist at the time the WHERE is evaluated
  - eg. `SELECT MAX(age) max_age FROM my_table WHERE job = "Engineer" HAVING max_age > 40` - find only engineers over 40

### Functions

Functions following usual programming syntax `SOMEFUNCTION(value)` taking an input `value`
and replace themselves with the function's calculated value based on this input.

- `DISTINCT` - deduplicates records in a `SELECT` query

#### Aggregate Functions

Functions that summarize a column from rows data into a single value, usually using common mathematical functions like:

- `COUNT` - returns the number of rows or if given a column the number of non-`NULL` values in that column in the table
  or for the `GROUP BY` clause
- `SUM` - returns the sum of the numeric values in the given column (eg. bill total of all individual item purchases)
- `MIN` - returns the minimum numeric value for the given field in all the `SELECT`'d rows
- `MAX` - returns the maximum numeric value for the given field in all the `SELECT`'d rows
- `AVG` - returns everage value of the given field in all the `SELECT`'d rows

Often used with a `GROUP BY` clause to get the aggregates for each of a category defined by the `GROUP BY` clause.

COUNT(column) vs COUNT(DISTINCT(column)):

- COUNT(column) - returns number of non-`NULL` values in the given column
  - eg. `SELECT COUNT(category) FROM my_table` - how many rows have a category defined
- COUNT(DISTINCT(column)) - returns the number of unique non-`NULL` values in the given column
  - eg. `SELECT COUNT(DISTINCT(category)) FROM my_table GROUP BY category` - how many different categories are there

##### Nesting Aggregate Functions

Yyou can nest aggregate functions up to two levels deep.

Eg. this query finds the category with the most records:

```sql
SELECT category, MAX(COUNT(1)) FROM mytable GROUP BY category;
```

### JOINs

Returns a data set by merging the rows of two tables on given fields which are expected to have matching values
representing that their rows are linked.

- Inner Join - returns records that exist in both tables
  - Natural Join - an inner join on two tables that have the same column names
- `LEFT JOIN` / `LEFT OUTER JOIN` - returns records that exist in the left table regardless of whether they are
  found in the right table. Shows `NULL`s for fields that don't exist in the other table
- `RIGHT JOIN` / `RIGHT OUTER JOIN` - returns records that exist in the right table regardless of whether they are
  found in the left table. Shows `NULL`s for fields that don't exist in the other table
- `Full JOIN` / `FULL Outer JOIN` - returns records that exist in either table - shows `NULL`s for fields that don't
  exist in the other table
- `CROSS JOIN` - returns all combinations of all records in both tables (cartesian product)
  - there is no join field for this by definition since all rows from one table are multiplied by all rows from the
    other table
- `SELF JOIN` - a join from one table to another record in the same table
  - used where a field in the table refers to another field in the same table such as hierarchical structures
    - eg. in an employees table the `manager_id` field refers to the `employee_id` of the manager who exists in another
      row the same table

### Rows vs Records vs Columns vs Fields

- Row - a line of data in a table
- Record - same thing as a row
- Column - a data type that each row may populate, depending on whether `NULL` (no value) is allowed in the column
  definition via a definition constraint.
- Field - a specific intersection of Row and Column - a single value from a single row in the table in the specified
  column.

### Constraints

Definitions in columns that enforce behaviours like:

- `NOT NULL` - the fields in each row for that column may not be empty (`NULL`)
- `UNIQUE` - all rows must have a unique value for this column eg. a globally unique customer ID

### Keys - Primary vs Secondary

These are defined at table creation time. Some can be updated on an existing table, but others like Primaries keys
usually cannot be because their new definitions constraints may violate the stored date.

- Primary Key - one or more columns that together uniquely identify a row in a table
  - there can only be one Primary Key per table by definition, if there is one at all
  - the Primary key column has implicit `UNIQUE` and `NOT NULL` constraints
  - usually a numeric ID
  - best practice is to use a Surrogate Key (described below) rather than a natural key
- Foreign Key - a column of a table that references the Primary Key of another table to mark each record in this table
  as being linked to a unique row in the other table
  - eg. the purchases in this table all belong to customer ID `N` in the customers table


# Row Subqueries

A row subquery is a [subquery](/sql-statements-structure/sql-statements/data-manipulation/selecting-data/joins-subqueries/subqueries/) returning a single row, as opposed to a [scalar subquery](/sql-statements-structure/sql-statements/data-manipulation/selecting-data/joins-subqueries/subqueries/subqueries-scalar-subqueries/), which returns a single column from a row, or a literal.

## Examples

```sql
CREATE TABLE staff (name VARCHAR(10), age TINYINT);

CREATE TABLE customer (name VARCHAR(10), age TINYINT);

INSERT INTO staff VALUES ('Bilhah',37), ('Valerius',61), ('Maia',25);

INSERT INTO customer VALUES ('Thanasis',48), ('Valerius',61), ('Brion',51);

SELECT * FROM staff WHERE (name,age) = (SELECT name,age FROM customer WHERE name='Valerius');
+----------+------+
| name     | age  |
+----------+------+
| Valerius |   61 |
+----------+------+
```

Finding all rows in one table also in another:

```sql
SELECT name,age FROM staff WHERE (name,age) IN (SELECT name,age FROM customer);
+----------+------+
| name     | age  |
+----------+------+
| Valerius |   61 |
+----------+------+
```
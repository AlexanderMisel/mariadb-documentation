# VAR_POP

## Syntax

```sql
VAR_POP(expr)
```

## Description

Returns the population standard variance of `expr`. It considers rows as
the whole population, not as a sample, so it has the number of rows as
the denominator. You can also use [VARIANCE()](/built-in-functions/aggregate-functions/variance/), which is equivalent but
is not standard SQL.

Variance is calculated by

- working out the mean for the set
- for each number, subtracting the mean and squaring the result
- calculate the average of the resulting differences

It is an [aggregate function](/built-in-functions/aggregate-functions/), and so can be used with the [GROUP BY](/sql-statements-structure/sql-statements/data-manipulation/selecting-data/group-by/) clause.

From [MariaDB 10.2.2](/kb/en/mariadb-1022-release-notes/), VAR_POP() can be used as a [window function](/built-in-functions/special-functions/window-functions/).

VAR_POP() returns `NULL` if there were no matching rows.

## Examples

```sql
CREATE TABLE v(i tinyint);

INSERT INTO v VALUES(101),(99);

SELECT VAR_POP(i) FROM v;
+------------+
| VAR_POP(i) |
+------------+
|     1.0000 |
+------------+

INSERT INTO v VALUES(120),(80);

SELECT VAR_POP(i) FROM v;
+------------+
| VAR_POP(i) |
+------------+
|   200.5000 |
+------------+
```

As an [aggregate function](/built-in-functions/aggregate-functions/):

```sql
CREATE OR REPLACE TABLE stats (category VARCHAR(2), x INT);

INSERT INTO stats VALUES 
  ('a',1),('a',2),('a',3),
  ('b',11),('b',12),('b',20),('b',30),('b',60);

SELECT category, STDDEV_POP(x), STDDEV_SAMP(x), VAR_POP(x) 
  FROM stats GROUP BY category;
+----------+---------------+----------------+------------+
| category | STDDEV_POP(x) | STDDEV_SAMP(x) | VAR_POP(x) |
+----------+---------------+----------------+------------+
| a        |        0.8165 |         1.0000 |     0.6667 |
| b        |       18.0400 |        20.1693 |   325.4400 |
+----------+---------------+----------------+------------+
```

As a [window function](/built-in-functions/special-functions/window-functions/):

```sql
CREATE OR REPLACE TABLE student_test (name CHAR(10), test CHAR(10), score TINYINT);

INSERT INTO student_test VALUES 
    ('Chun', 'SQL', 75), ('Chun', 'Tuning', 73), 
    ('Esben', 'SQL', 43), ('Esben', 'Tuning', 31), 
    ('Kaolin', 'SQL', 56), ('Kaolin', 'Tuning', 88), 
    ('Tatiana', 'SQL', 87);

SELECT name, test, score, VAR_POP(score) 
  OVER (PARTITION BY test) AS variance_results FROM student_test;
+---------+--------+-------+------------------+
| name    | test   | score | variance_results |
+---------+--------+-------+------------------+
| Chun    | SQL    |    75 |         287.1875 |
| Esben   | SQL    |    43 |         287.1875 |
| Kaolin  | SQL    |    56 |         287.1875 |
| Tatiana | SQL    |    87 |         287.1875 |
| Chun    | Tuning |    73 |         582.0000 |
| Esben   | Tuning |    31 |         582.0000 |
| Kaolin  | Tuning |    88 |         582.0000 |
+---------+--------+-------+------------------+
```

## See Also

- [VARIANCE](/built-in-functions/aggregate-functions/variance/) (equivalent, non-standard SQL)
- [STDDEV_POP](/built-in-functions/aggregate-functions/stddev_pop/) (population standard deviation)
- [STDDEV_SAMP](/built-in-functions/aggregate-functions/stddev_samp/) (sample standard deviation)
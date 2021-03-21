# AVG

## Syntax

```sql
AVG([DISTINCT] expr)
```

## Description

Returns the average value of expr. The DISTINCT option can be used to return the average of the distinct values of expr. NULL values are ignored. It is an [aggregate function](/built-in-functions/aggregate-functions/), and so can be used with the [GROUP BY](/sql-statements-structure/sql-statements/data-manipulation/selecting-data/group-by/) clause.

AVG() returns NULL if there were no matching rows.

From [MariaDB 10.2.0](/kb/en/mariadb-1020-release-notes/), AVG() can be used as a [window function](/built-in-functions/special-functions/window-functions/).

## Examples

```sql
CREATE TABLE sales (sales_value INT);

INSERT INTO sales VALUES(10),(20),(20),(40);

SELECT AVG(sales_value) FROM sales;
+------------------+
| AVG(sales_value) |
+------------------+
|          22.5000 |
+------------------+

SELECT AVG(DISTINCT(sales_value)) FROM sales;
+----------------------------+
| AVG(DISTINCT(sales_value)) |
+----------------------------+
|                    23.3333 |
+----------------------------+
```

Commonly, AVG() is used with a [GROUP BY](/kb/en/select/#group-by) clause:

```sql
CREATE TABLE student (name CHAR(10), test CHAR(10), score TINYINT); 

INSERT INTO student VALUES 
  ('Chun', 'SQL', 75), ('Chun', 'Tuning', 73), 
  ('Esben', 'SQL', 43), ('Esben', 'Tuning', 31), 
  ('Kaolin', 'SQL', 56), ('Kaolin', 'Tuning', 88), 
  ('Tatiana', 'SQL', 87), ('Tatiana', 'Tuning', 83);

SELECT name, AVG(score) FROM student GROUP BY name;
+---------+------------+
| name    | AVG(score) |
+---------+------------+
| Chun    |    74.0000 |
| Esben   |    37.0000 |
| Kaolin  |    72.0000 |
| Tatiana |    85.0000 |
+---------+------------+
```

Be careful to avoid this common mistake, not grouping correctly and returning mismatched data:

```sql
SELECT name,test,AVG(score) FROM student;
+------+------+------------+
| name | test | MIN(score) |
+------+------+------------+
| Chun | SQL  |         31 |
+------+------+------------+
```

As a [window function](/built-in-functions/special-functions/window-functions/):

```sql
CREATE TABLE student_test (name CHAR(10), test CHAR(10), score TINYINT); 

INSERT INTO student_test VALUES 
    ('Chun', 'SQL', 75), ('Chun', 'Tuning', 73), 
    ('Esben', 'SQL', 43), ('Esben', 'Tuning', 31), 
    ('Kaolin', 'SQL', 56), ('Kaolin', 'Tuning', 88), 
    ('Tatiana', 'SQL', 87), ('Tatiana', 'Tuning', 83);

SELECT name, test, score, AVG(score) OVER (PARTITION BY test) 
    AS average_by_test FROM student_test;
+---------+--------+-------+-----------------+
| name    | test   | score | average_by_test |
+---------+--------+-------+-----------------+
| Chun    | SQL    |    75 |         65.2500 |
| Chun    | Tuning |    73 |         68.7500 |
| Esben   | SQL    |    43 |         65.2500 |
| Esben   | Tuning |    31 |         68.7500 |
| Kaolin  | SQL    |    56 |         65.2500 |
| Kaolin  | Tuning |    88 |         68.7500 |
| Tatiana | SQL    |    87 |         65.2500 |
| Tatiana | Tuning |    83 |         68.7500 |
+---------+--------+-------+-----------------+
```

## See Also

- [MAX](/built-in-functions/aggregate-functions/max/) (maximum)
- [MIN](/built-in-functions/aggregate-functions/min/) (minimum)
- [SUM](/built-in-functions/aggregate-functions/sum/) (sum total)
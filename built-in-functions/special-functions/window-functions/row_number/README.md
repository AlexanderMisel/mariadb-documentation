# ROW_NUMBER

##### MariaDB starting with [10.2](/kb/en/what-is-mariadb-102/)

ROW_NUMBER() was first introduced with [window functions](/built-in-functions/special-functions/window-functions) in [MariaDB 10.2.0](/kb/en/mariadb-1020-release-notes/).

## Syntax

```sql
ROW_NUMBER() OVER (
  [ PARTITION BY partition_expression ]
  [ ORDER BY order_list ]
) 
```

## Description

ROW_NUMBER() is a [window function](/built-in-functions/special-functions/window-functions) that displays the number of a given row, starting at one and following the [ORDER BY](/sql-statements-structure/sql-statements/data-manipulation/selecting-data/order-by) sequence of the window function, with identical values receiving different row numbers. It is similar to the [RANK()](/built-in-functions/special-functions/window-functions/rank) and [DENSE_RANK()](/built-in-functions/special-functions/window-functions/dense_rank) functions except that in that function, identical values will receive the same rank for each result.

## Examples

The distinction between [DENSE_RANK()](/built-in-functions/special-functions/window-functions/dense_rank), [RANK()](/built-in-functions/special-functions/window-functions/rank) and ROW_NUMBER():

```sql
CREATE TABLE student(course VARCHAR(10), mark int, name varchar(10));

INSERT INTO student VALUES 
  ('Maths', 60, 'Thulile'),
  ('Maths', 60, 'Pritha'),
  ('Maths', 70, 'Voitto'),
  ('Maths', 55, 'Chun'),
  ('Biology', 60, 'Bilal'),
   ('Biology', 70, 'Roger');

SELECT 
  RANK() OVER (PARTITION BY course ORDER BY mark DESC) AS rank, 
  DENSE_RANK() OVER (PARTITION BY course ORDER BY mark DESC) AS dense_rank, 
  ROW_NUMBER() OVER (PARTITION BY course ORDER BY mark DESC) AS row_num, 
  course, mark, name 
FROM student ORDER BY course, mark DESC;
+------+------------+---------+---------+------+---------+
| rank | dense_rank | row_num | course  | mark | name    |
+------+------------+---------+---------+------+---------+
|    1 |          1 |       1 | Biology |   70 | Roger   |
|    2 |          2 |       2 | Biology |   60 | Bilal   |
|    1 |          1 |       1 | Maths   |   70 | Voitto  |
|    2 |          2 |       2 | Maths   |   60 | Thulile |
|    2 |          2 |       3 | Maths   |   60 | Pritha  |
|    4 |          3 |       4 | Maths   |   55 | Chun    |
+------+------------+---------+---------+------+---------+
```

## See Also

- [RANK()](/built-in-functions/special-functions/window-functions/rank)
- [DENSE_RANK()](/built-in-functions/special-functions/window-functions/dense_rank)
- [ORDER BY](/sql-statements-structure/sql-statements/data-manipulation/selecting-data/order-by)
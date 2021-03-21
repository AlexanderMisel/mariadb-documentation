# Not Equal Operator: !=

## Syntax

```sql
<>, !=
```

## Description

Not equal operator.  Evaluates both SQL expressions and returns 1 if they are not equal and 0 if they are equal, or `NULL` if either expression is NULL.  If the expressions return different data types, (for instance, a number and a string), performs type conversion.

When used in row comparisons these two queries return the same results:

```sql
SELECT (t1.a, t1.b) != (t2.x, t2.y) 
FROM t1 INNER JOIN t2;

SELECT (t1.a != t2.x) OR (t1.b != t2.y)
FROM t1 INNER JOIN t2;
```

## Examples

```sql
SELECT '.01' <> '0.01';
+-----------------+
| '.01' <> '0.01' |
+-----------------+
|               1 |
+-----------------+

SELECT .01 <> '0.01';
+---------------+
| .01 <> '0.01' |
+---------------+
|             0 |
+---------------+

SELECT 'zapp' <> 'zappp';
+-------------------+
| 'zapp' <> 'zappp' |
+-------------------+
|                 1 |
+-------------------+
```
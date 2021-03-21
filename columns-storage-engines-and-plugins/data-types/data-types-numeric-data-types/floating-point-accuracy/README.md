# Floating-point Accuracy

Due to their nature, not all floating-point numbers can be stored with exact precision. Hardware architecture, the CPU or even the compiler version and optimization level may affect the precision.

If you are comparing [DOUBLEs](/columns-storage-engines-and-plugins/data-types/data-types-numeric-data-types/double) or [FLOATs](/columns-storage-engines-and-plugins/data-types/data-types-numeric-data-types/float) with numeric decimals, it is not safe to use the [equality](/sql-statements-structure/operators/comparison-operators/equal) operator.

Sometimes, changing a floating-point number from single-precision (FLOAT) to double-precision (DOUBLE) will fix the problem.

## Example

f1, f2 and f3 have seemingly identical values across each row, but due to floating point accuracy, the results may be unexpected.

```sql
CREATE TABLE fpn (id INT, f1 FLOAT, f2 DOUBLE, f3 DECIMAL (10,3));
INSERT INTO fpn VALUES (1,2,2,2),(2,0.1,0.1,0.1);

SELECT * FROM fpn WHERE f1*f1 = f2*f2;
+------+------+------+-------+
| id   | f1   | f2   | f3    |
+------+------+------+-------+
|    1 |    2 |    2 | 2.000 |
+------+------+------+-------+
```

The reason why only one instead of two rows was returned becomes clear when we see how the floating point squares were evaluated.

```sql
SELECT f1*f1, f2*f2, f3*f3 FROM fpn;
+----------------------+----------------------+----------+
| f1*f1                | f2*f2                | f3*f3    |
+----------------------+----------------------+----------+
|                    4 |                    4 | 4.000000 |
| 0.010000000298023226 | 0.010000000000000002 | 0.010000 |
+----------------------+----------------------+----------+
```
# YEAR Data Type

## Syntax

```sql
YEAR[(4)]
```

## Description

A year in two-digit or four-digit format. The default is four-digit format. Note that the two-digit format has been deprecated since [MariaDB 5.5.27](/kb/en/mariadb-5527-release-notes/).

In four-digit format, the allowable values are 1901 to 2155,
and 0000. In two-digit format, the allowable values are 70 to 69,
representing years from 1970 to 2069. MariaDB displays YEAR values in
YYYY format, but allows you to assign values to YEAR columns using
either strings or numbers.

Inserting numeric zero has a different result for YEAR(4) and YEAR(2). For YEAR(2), the value `00` reflects the year 2000. For YEAR(4), the value `0000` reflects the year zero. This only applies to numeric zero. String zero always reflects the year 2000.

## Examples

Accepting a string or a number:

```sql
CREATE TABLE y(y YEAR);

INSERT INTO y VALUES (1990),('2012');

SELECT * FROM y;
+------+
| y    |
+------+
| 1990 |
| 2012 |
+------+
```

With [strict_mode](/kb/en/sql-mode/#strict-mode) set, the default from [MariaDB 10.2.4](/kb/en/mariadb-1024-release-notes/):

Out of range:

```sql
INSERT INTO y VALUES (1005),('3080');
ERROR 1264 (22003): Out of range value for column 'y' at row 1

INSERT INTO y VALUES ('2013-12-12');
ERROR 1265 (01000): Data truncated for column 'y' at row 1

SELECT * FROM y;
+------+
| y    |
+------+
| 1990 |
| 2012 |
+------+
```

With [strict_mode](/kb/en/sql-mode/#strict-mode) unset, the default until [MariaDB 10.2.3](/kb/en/mariadb-1023-release-notes/):

Out of range:

```sql
INSERT INTO y VALUES (1005),('3080');
Query OK, 2 rows affected, 2 warnings (0.05 sec)
Records: 2  Duplicates: 0  Warnings: 2

SHOW WARNINGS;
+---------+------+--------------------------------------------+
| Level   | Code | Message                                    |
+---------+------+--------------------------------------------+
| Warning | 1264 | Out of range value for column 'y' at row 1 |
| Warning | 1264 | Out of range value for column 'y' at row 2 |
+---------+------+--------------------------------------------+

SELECT * FROM y;
+------+
| y    |
+------+
| 1990 |
| 2012 |
| 0000 |
| 0000 |
+------+
```

Truncating:

```sql
INSERT INTO y VALUES ('2013-12-12');
Query OK, 1 row affected, 1 warning (0.05 sec)

SHOW WARNINGS;
+---------+------+----------------------------------------+
| Level   | Code | Message                                |
+---------+------+----------------------------------------+
| Warning | 1265 | Data truncated for column 'y' at row 1 |
+---------+------+----------------------------------------+

SELECT * FROM y;
+------+
| y    |
+------+
| 1990 |
| 2012 |
| 0000 |
| 0000 |
| 2013 |
+------+
```

Difference between YEAR(2) and YEAR(4), and string and numeric zero:

```sql
CREATE TABLE y2(y YEAR(4), y2 YEAR(2));
Query OK, 0 rows affected, 1 warning (0.40 sec)

Note (Code 1287): 'YEAR(2)' is deprecated and will be removed in a future release. 
 Please use YEAR(4) instead

INSERT INTO y2 VALUES(0,0),('0','0');

SELECT YEAR(y),YEAR(y2) FROM y2;
+---------+----------+
| YEAR(y) | YEAR(y2) |
+---------+----------+
|       0 |     2000 |
|    2000 |     2000 |
+---------+----------+
```

## See Also

- [YEAR() function](/built-in-functions/date-time-functions/year/)
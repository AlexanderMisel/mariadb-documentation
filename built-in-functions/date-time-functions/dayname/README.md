# DAYNAME

## Syntax

```sql
DAYNAME(date)
```

## Description

Returns the name of the weekday for date. The language used for the name is controlled by the value
of the [lc_time_names](/kb/en/server-system-variables/#lc_time_names) system variable. See [server locale](/columns-storage-engines-and-plugins/data-types/string-data-types/character-sets/internationalization-and-localization/server-locale/) for more on the supported locales.

## Examples

```sql
SELECT DAYNAME('2007-02-03');
+-----------------------+
| DAYNAME('2007-02-03') |
+-----------------------+
| Saturday              |
+-----------------------+
```

```sql
CREATE TABLE t1 (d DATETIME);
INSERT INTO t1 VALUES
    ("2007-01-30 21:31:07"),
    ("1983-10-15 06:42:51"),
    ("2011-04-21 12:34:56"),
    ("2011-10-30 06:31:41"),
    ("2011-01-30 14:03:25"),
    ("2004-10-07 11:19:34");
```

```sql
SELECT d, DAYNAME(d) FROM t1;
+---------------------+------------+
| d                   | DAYNAME(d) |
+---------------------+------------+
| 2007-01-30 21:31:07 | Tuesday    |
| 1983-10-15 06:42:51 | Saturday   |
| 2011-04-21 12:34:56 | Thursday   |
| 2011-10-30 06:31:41 | Sunday     |
| 2011-01-30 14:03:25 | Sunday     |
| 2004-10-07 11:19:34 | Thursday   |
+---------------------+------------+
```

Changing the locale:

```sql
SET lc_time_names = 'fr_CA';

SELECT DAYNAME('2013-04-01');
+-----------------------+
| DAYNAME('2013-04-01') |
+-----------------------+
| lundi                 |
+-----------------------+
```
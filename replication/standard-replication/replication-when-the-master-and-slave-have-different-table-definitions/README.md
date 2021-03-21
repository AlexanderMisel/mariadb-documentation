# Replication When the Master and Slave Have Different Table Definitions

The terms <em>master</em> and <em>slave</em> have historically been used in replication, but the terms terms <em>primary</em> and <em>replica</em> are now preferred. The old terms are used throughout the documentation, and in MariaDB commands, although [MariaDB 10.5](/kb/en/what-is-mariadb-105/) has begun the process of renaming. The documentation will follow over time. See [MDEV-18777](https://jira.mariadb.org/browse/MDEV-18777) to follow progress on this effort.

While replication is usually meant to take place between masters and slaves with the same table definitions and this is recommended, in certain cases replication can still take place even if the definitions are identical.

Tables on the slave and the master do not need to have the same definition in order for [replication](/replication) to take place. There can be differing numbers of columns, or differing data definitions and, in certain cases, replication can still proceed.

## Different Column Definitions - Attribute Promotion and Demotion

It is possible in some cases to replicate to a slave that has a column of a different type on the slave and the master. This process is called attribute promotion (to a larger type) or attribute demotion (to a smaller type).

The conditions differ depending on whether [statement-based](/kb/en/binary-log-formats/#statement-based) or [row-based replication](/kb/en/binary-log-formats/#row-based) is used.

### Statement-Based Replication

When using [statement-based replication](/kb/en/binary-log-formats/#statement-based), generally, if a statement can run successfully on the slave, it will be replicated. If a column definition is the same or a larger type on the slave than on the master, it can replicate successfully. For example a column defined as [VARCHAR(10)](/columns-storage-engines-and-plugins/data-types/string-data-types/varchar) will successfully be replicated on a slave with a definition of `VARCHAR(12)`.

Replicating to a slave where the column is defined as smaller than on the master can also work. For example, given the following table definitions:

Master:

```sql
DESC r;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | tinyint(4)  | YES  |     | NULL    |       |
| v     | varchar(10) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
```

Slave

```sql
DESC r;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | tinyint(4)  | YES  |     | NULL    |       |
| v     | varchar(8) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
```

the statement

```sql
INSERT INTO r VALUES (6,'hi');
```

would successfully replicate because the value inserted into the `v` field can successfully be inserted on both the master and the smaller slave equivalent.

However, the following statement would fail:

```sql
INSERT INTO r VALUES (7,'abcdefghi')
```

In this case, the value fits in the master definition, but is too long for the slave field, and so replication will fail.

```sql
SHOW SLAVE STATUS\G
*************************** 1. row ***************************
...
Slave_IO_Running: Yes
Slave_SQL_Running: No
...
Last_Errno: 1406
Last_Error: Error 'Data too long for column 'v' at row 1' on query. 
   Default database: 'test'. Query: 'INSERT INTO r VALUES (7,'abcdefghi')'
...
```

### Row-Based Replication

When using [row-based replication](/kb/en/binary-log-formats/#row-based), the value of the [slave_type_conversions](/kb/en/replication-and-binary-log-server-system-variables/#slave_type_conversions) variable is important. The default value of this variable is empty, in which case MariaDB will not perform attribute promotion or demotion. If the column definitions do not match, replication will stop. If set to `ALL_NON_LOSSY`, safe replication is permitted. If set to `ALL_LOSSY` as well, replication will be permitted even if data loss takes place.

For example:

Master:

```sql
DESC r;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | smallint(6) | YES  |     | NULL    |       |
| v     | varchar(10) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
```

Slave:

```sql
SHOW VARIABLES LIKE 'slave_ty%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| slave_type_conversions |       |
+------------------------+-------+

 DESC r;
+-------+------------+------+-----+---------+-------+
| Field | Type       | Null | Key | Default | Extra |
+-------+------------+------+-----+---------+-------+
| id    | tinyint(4) | YES  |     | NULL    |       |
| v     | varchar(1) | YES  |     | NULL    |       |
+-------+------------+------+-----+---------+-------+
```

The following query will fail:

```sql
INSERT INTO r VALUES (3,'c');
```

```sql
SHOW SLAVE STATUS\G;
...
Slave_IO_Running: Yes
Slave_SQL_Running: No
...
Last_Errno: 1677
Last_Error: Column 0 of table 'test.r' cannot be converted from 
  type 'smallint' to type 'tinyint(4)'
...
```

By changing the value of the [slave_type_conversions](/kb/en/replication-and-binary-log-server-system-variables/#slave_type_conversions), replication can proceed:

```sql
SET GLOBAL slave_type_conversions='ALL_NON_LOSSY,ALL_LOSSY';

START SLAVE;
```

```sql
SHOW SLAVE STATUS\G;
*************************** 1. row ***************************
...
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
...
```

#### Supported Conversions

- Between [TINYINT](/columns-storage-engines-and-plugins/data-types/data-types-numeric-data-types/tinyint), [SMALLINT](/columns-storage-engines-and-plugins/data-types/data-types-numeric-data-types/smallint), [MEDIUMINT](/columns-storage-engines-and-plugins/data-types/data-types-numeric-data-types/mediumint), [INT](/columns-storage-engines-and-plugins/data-types/data-types-numeric-data-types/int) and [BIGINT](/columns-storage-engines-and-plugins/data-types/data-types-numeric-data-types/bigint). If lossy conversion is supported, the value from the master will be converted to the maximum or minimum permitted on the slave, which non-lossy conversions require the slave column to be large enough. For example, SMALLINT UNSIGNED can be converted to MEDIUMINT, but not SMALLINT SIGNED.

## Different Number or Order of Columns

Replication can also take place when the master and slave have a different number of columns if the following criteria are met:

- columns must be in the same order on the master and slave
- common columns must be defined with the same data type
- extra columns must be defined after the common columns

### Row-Based

The following example replicates incorrectly (replication proceeds, but the data is corrupted), as the columns are not in the same order.

Master:

```sql
CREATE OR REPLACE TABLE r (i1 INT, i2 INT);
```

Slave:

```sql
ALTER TABLE r ADD i3 INT AFTER i1; 
```

Master:

```sql
INSERT INTO r (i1,i2) VALUES (1,1);

SELECT * FROM r;
+------+------+
| i1   | i2   |
+------+------+
|    1 |    1 |
+------+------+
```

Slave:

```sql
SELECT * FROM r;
+------+------+------+
| i1   | i3   | i2   |
+------+------+------+
|    1 |    1 | NULL |
+------+------+------+
```

### Statement-Based

Using statement-based replication, the same example may work, even though the columns are not in the same order.

Master:

```sql
CREATE OR REPLACE TABLE r (i1 INT, i2 INT);
```

Slave:

```sql
ALTER TABLE r ADD i3 INT AFTER i1; 
```

Master:

```sql
INSERT INTO r (i1,i2) VALUES (1,1);

SELECT * FROM r;
+------+------+
| i1   | i2   |
+------+------+
|    1 |    1 |
+------+------+
```

Slave:

```sql
 SELECT * FROM r;
+------+------+------+
| i1   | i3   | i2   |
+------+------+------+
|    1 | NULL |    1 |
+------+------+------+
```
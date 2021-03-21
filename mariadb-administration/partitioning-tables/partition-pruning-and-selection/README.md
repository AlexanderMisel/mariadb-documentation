# Partition Pruning and Selection

When a WHERE clause is related to the partitioning expression, the optimizer knows which partitions are relevant for the query. Other partitions will not be read. This optimization is called <em>partition pruning</em>.

[EXPLAIN PARTITIONS](/sql-statements-structure/sql-statements/administrative-sql-statements/analyze-and-explain-statements/explain/) can be used to know which partitions will be read for a given query. A column called `partitions` will contain a comma-separated list of the accessed partitions. For example:

```sql
EXPLAIN PARTITIONS SELECT * FROM orders WHERE id < 15000000;
+------+-------------+--------+------------+-------+---------------+---------+---------+------+------+-------------+
| id   | select_type | table  | partitions | type  | possible_keys | key     | key_len | ref  | rows | Extra       |
+------+-------------+--------+------------+-------+---------------+---------+---------+------+------+-------------+
|    1 | SIMPLE      | orders | p0,p1      | range | PRIMARY       | PRIMARY | 4       | NULL |    2 | Using where |
+------+-------------+--------+------------+-------+---------------+---------+---------+------+------+-------------+
```

Sometimes the WHERE clause does not contain the necessary information to use partition pruning, or the optimizer cannot infer this information. However, we may know which partitions are relevant for the query. Since [MariaDB 10.0](/kb/en/what-is-mariadb-100/), we can force MariaDB to only access the specified partitions by adding a PARTITION clause. This feature is called <em>partition selection</em>. For example:

```sql
SELECT * FROM orders PARTITION (p3) WHERE user_id = 50;
```

The PARTITION clause is supported for all DML statements:

- [SELECT](/sql-statements-structure/sql-statements/data-manipulation/selecting-data/select/)
- [INSERT](/sql-statements-structure/sql-statements/data-manipulation/inserting-loading-data/insert/)
- [UPDATE](/sql-statements-structure/sql-statements/data-manipulation/changing-deleting-data/update/)
- [DELETE](/sql-statements-structure/sql-statements/data-manipulation/changing-deleting-data/delete/)
- [REPLACE](/sql-statements-structure/sql-statements/data-manipulation/changing-deleting-data/replace/)

## Partition pruning and triggers

In general, partition pruning is applied to statements contained in [triggers](/programming-customizing-mariadb/triggers-events/triggers/).

However, note that if a `BEFORE INSERT` or `BEFORE UPDATE` trigger is defined on a table, MariaDB doesn't know in advance if the columns used in the partitioning expression will be changed. For this reason, it is forced to lock all partitions.
# OQGRAPH Overview

The Open Query GRAPH computation engine, or OQGRAPH as the engine itself is called, allows you to handle hierarchies (tree structures) and complex graphs (nodes having many connections in several directions).

OQGRAPH is unlike other storage engines, consisting of an entirely different architecture to a regular storage engine such as Aria, MyISAM or InnoDB.

It is intended to be used for retrieving hierarchical information, such as those used for graphs, routes or social relationships, in plain SQL.

## Installing

See [Installing OQGRAPH](/columns-storage-engines-and-plugins/storage-engines/oqgraph-storage-engine/installing-oqgraph). Note that the [query cache](/replication/optimization-and-tuning/buffers-caches-and-threads/query-cache) needs to be disabled when using OQGRAPH with [MariaDB 5.5.35](/kb/en/mariadb-5535-release-notes/) and before, or 10.0.8 and before (see [MDEV-5744](https://jira.mariadb.org/browse/MDEV-5744)). With newer versions, the query cache can be enabled and OQGRAPH will not use it.

## Creating a Table

The following documentation is based upon [MariaDB 10.0.7](/kb/en/mariadb-1007-release-notes/) and OQGRAPH v3. Details have changed since older versions.

## Example with origin and destination nodes only

To create an OQGRAPH v3 table, a backing table must first be created. This backing table will store the actual data, and will be used for all INSERTs, UPDATEs and so on. It must be a regular table, not a view. Here's a simple example to start with:

```sql
CREATE TABLE oq_backing (
  origid INT UNSIGNED NOT NULL, 
  destid INT UNSIGNED NOT NULL,  
  PRIMARY KEY (origid, destid), 
  KEY (destid)
);
```

Some data can be inserted into the backing table to test with later:

```sql
INSERT INTO oq_backing(origid, destid) 
 VALUES (1,2), (2,3), (3,4), (4,5), (2,6), (5,6);
```

Now the read-only OQGRAPH table is created. The CREATE statement must match the format below - any difference will result in an error.

```sql
CREATE TABLE oq_graph (
  latch VARCHAR(32) NULL,
  origid BIGINT UNSIGNED NULL,
  destid BIGINT UNSIGNED NULL,
  weight DOUBLE NULL,
  seq BIGINT UNSIGNED NULL,
  linkid BIGINT UNSIGNED NULL,
  KEY (latch, origid, destid) USING HASH,
  KEY (latch, destid, origid) USING HASH
) 
ENGINE=OQGRAPH 
data_table='oq_backing' origid='origid' destid='destid';
```

An older format has the latch field as a SMALLINT rather than a VARCHAR. In [MariaDB 10.0.7](/kb/en/mariadb-1007-release-notes/) with OQGRAPH v3, the format is still valid, but gives an error by default:

```sql
CREATE TABLE oq_old (
  latch SMALLINT UNSIGNED NULL,
  origid BIGINT UNSIGNED NULL,
  destid BIGINT UNSIGNED NULL,
  weight DOUBLE NULL,
  seq BIGINT UNSIGNED NULL,
  linkid BIGINT UNSIGNED NULL,
  KEY (latch, origid, destid) USING HASH,
  KEY (latch, destid, origid) USING HASH
) 
ENGINE=OQGRAPH 
data_table='oq_backing' origid='origid' destid='destid';

ERROR 1005 (HY000): Can't create table `test`.`oq_old` (errno: 140 "Wrong create options")
```

The old, deprecated format can still be used if the value of the [oqgraph_allow_create_integer_latch](/kb/en/oqgraph-system-and-status-variables/#oqgraph_allow_create_integer_latch) system variable is changed from its default, `FALSE`, to `TRUE`.

```sql
SET GLOBAL oqgraph_allow_create_integer_latch=1;

CREATE TABLE oq_old (
  latch SMALLINT UNSIGNED NULL,
  origid BIGINT UNSIGNED NULL,
  destid BIGINT UNSIGNED NULL,
  weight DOUBLE NULL,
  seq BIGINT UNSIGNED NULL,
  linkid BIGINT UNSIGNED NULL,
  KEY (latch, origid, destid) USING HASH,
  KEY (latch, destid, origid) USING HASH
) 
ENGINE=OQGRAPH 
data_table='oq_backing' origid='origid' destid='destid';
Query OK, 0 rows affected, 1 warning (0.19 sec)

SHOW WARNINGS;
+---------+------+-----------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                           |
+---------+------+-----------------------------------------------------------------------------------------------------------------------------------+
| Warning | 1287 | 'latch SMALLINT UNSIGNED NULL' is deprecated and will be removed in a future release. Please use 'latch VARCHAR(32) NULL' instead |
+---------+------+-----------------------------------------------------------------------------------------------------------------------------------+
```

Data is only inserted into the backing table, not the OQGRAPH table.

Now, having created the `oq_graph` table linked to a backing table, it is now possible to query the `oq_graph` table directly. The `weight` field, since it was not specified in this example, defaults to `1`.

```sql
SELECT * FROM oq_graph;
+-------+--------+--------+--------+------+--------+
| latch | origid | destid | weight | seq  | linkid |
+-------+--------+--------+--------+------+--------+
|  NULL |      1 |      2 |      1 | NULL |   NULL |
|  NULL |      2 |      3 |      1 | NULL |   NULL |
|  NULL |      2 |      6 |      1 | NULL |   NULL |
|  NULL |      3 |      4 |      1 | NULL |   NULL |
|  NULL |      4 |      5 |      1 | NULL |   NULL |
|  NULL |      5 |      6 |      1 | NULL |   NULL |
+-------+--------+--------+--------+------+--------+
```

The data here represents one-directional starting and ending nodes. So node 2 has paths to node 3 and node 6, while node 6 has no paths to any other node.

## Manipulating Weight

There are three fields which can be manipulated: `origid`, `destid` (the example above uses these two), as well as `weight`. To create a backing table with a `weight` field as well, the following syntax can be used:

```sql
CREATE TABLE oq2_backing (
  origid INT UNSIGNED NOT NULL, 
  destid INT UNSIGNED NOT NULL, 
  weight DOUBLE NOT NULL, 
  PRIMARY KEY (origid, destid), 
  KEY (destid)
);
```

```sql
INSERT INTO oq2_backing(origid, destid, weight)  
 VALUES (1,2,1), (2,3,1), (3,4,3), (4,5,1), (2,6,10), (5,6,2);
```

```sql
CREATE TABLE oq2_graph (
  latch VARCHAR(32) NULL,
  origid BIGINT UNSIGNED NULL,
  destid BIGINT UNSIGNED NULL,
  weight DOUBLE NULL,
  seq BIGINT UNSIGNED NULL,
  linkid BIGINT UNSIGNED NULL,
  KEY (latch, origid, destid) USING HASH,
  KEY (latch, destid, origid) USING HASH
) 
ENGINE=OQGRAPH 
data_table='oq2_backing' origid='origid' destid='destid' weight='weight';
```

```sql
SELECT * FROM oq2_graph;
+-------+--------+--------+--------+------+--------+
| latch | origid | destid | weight | seq  | linkid |
+-------+--------+--------+--------+------+--------+
| NULL  |      1 |      2 |      1 | NULL |   NULL |
| NULL  |      2 |      3 |      1 | NULL |   NULL |
| NULL  |      2 |      6 |     10 | NULL |   NULL |
| NULL  |      3 |      4 |      3 | NULL |   NULL |
| NULL  |      4 |      5 |      1 | NULL |   NULL |
| NULL  |      5 |      6 |      2 | NULL |   NULL |
+-------+--------+--------+--------+------+--------+
```

See [OQGRAPH Examples](/columns-storage-engines-and-plugins/storage-engines/oqgraph-storage-engine/oqgraph-examples) for OQGRAPH usage examples.
# Fair Choice Between Range and Index_merge Optimizations

`index_merge` is a method used by the optimizer to retrieve rows from a
single table using several index scans. The results of the scans are then
merged.

When using [EXPLAIN](/sql-statements-structure/sql-statements/administrative-sql-statements/analyze-and-explain-statements/explain), if `index_merge` is the plan chosen by the
optimizer, it will show up in the "type" column. For example:

```sql
MariaDB [ontime]> SELECT COUNT(*) FROM ontime;
+--------+
|count(*)|
+--------+
| 1578171|
+--------+

MySQL [ontime]> EXPLAIN SELECT * FROM ontime WHERE (Origin='SEA' OR Dest='SEA');
+--+-----------+------+-----------+-------------+-----------+-------+----+-----+--------------------------------------+
|id|select_type|table |type       |possible_keys|key        |key_len|ref |rows |Extra                                 |
+--+-----------+------+-----------+-------------+-----------+-------+----+-----+--------------------------------------+
| 1|SIMPLE     |ontime|index_merge|Origin,Dest  |Origin,Dest|6,6    |NULL|92800|Using union (Origin,Dest); Using where|
+--+-----------+------+-----------+-------------+-----------+-------+----+-----+--------------------------------------+
```

The "rows" column gives us a way to compare efficiency between `index_merge`
and other plans.

It is sometimes necessary to discard index_merge in favor of a different plan
to avoid a combinatorial explosion of possible range and/or index_merge
strategies. But, the old logic in MySQL for when index_merge was rejected
caused some good index_merge plans to not even be considered. Specifically,
additional `AND` predicates in `WHERE` clauses could cause an index_merge
plan to be rejected in favor of a less efficient plan. The slowdown could be
anywhere from 10x to over 100x. Here are two examples (based on the previous
query) using MySQL:

```sql
MySQL [ontime]> EXPLAIN SELECT * FROM ontime WHERE (Origin='SEA' OR Dest='SEA') AND securitydelay=0;
+--+-----------+------+----+-------------------------+-------------+-------+-----+------+-----------+
|id|select_type|table |type|possible_keys            |key          |key_len|ref  |rows  |Extra      |
+--+-----------+------+----+-------------------------+-------------+-------+-----+------+-----------+
| 1|SIMPLE     |ontime|ref |Origin,Dest,SecurityDelay|SecurityDelay|5      |const|791546|Using where|
+--+-----------+------+----+-------------------------+-------------+-------+-----+------+-----------+

MySQL [ontime]> EXPLAIN SELECT * FROM ontime WHERE (Origin='SEA' OR Dest='SEA') AND depdelay < 12*60;
+--+-----------+------+----+--------------------+----+-------+----+-------+-----------+
|id|select_type|table |type|possible_keys       |key |key_len|ref |rows   |Extra      |
+--+-----------+------+----+--------------------+----+-------+----+-------+-----------+
| 1|SIMPLE     |ontime|ALL |Origin,DepDelay,Dest|NULL|NULL   |NULL|1583093|Using where|
+--+-----------+------+----+--------------------+----+-------+----+-------+-----------
```

In the above output, the "rows" column shows that the first is almost 10x less
efficient and the second is over 15x less efficient than `index_merge`.

Starting in [MariaDB 5.3](/kb/en/what-is-mariadb-53/), the optimizer will delay discarding potential
`index_merge` plans until the point where it is really necessary. See [MWL#24](http://askmonty.org/worklog/?tid=24)
for more information.

By not discarding potential `index_merge` plans until absolutely necessary,
the two queries stay just as efficient as the original:

```sql
MariaDB [ontime]> EXPLAIN SELECT * FROM ontime WHERE (Origin='SEA' or Dest='SEA');
+--+-----------+------+-----------+-------------+-----------+-------+----+-----+-------------------------------------+
|id|select_type|table |type       |possible_keys|key        |key_len|ref |rows |Extra                                |
+--+-----------+------+-----------+-------------+-----------+-------+----+-----+-------------------------------------+
| 1|SIMPLE     |ontime|index_merge|Origin,Dest  |Origin,Dest|6,6    |NULL|92800|Using union(Origin,Dest); Using where|
+--+-----------+------+-----------+-------------+-----------+-------+----+-----+-------------------------------------+

MariaDB [ontime]> EXPLAIN SELECT * FROM ontime WHERE (Origin='SEA' or Dest='SEA') AND securitydelay=0;
+--+-----------+------+-----------+-------------------------+-----------+-------+----+-----+-------------------------------------+
|id|select_type|table |type       |possible_keys            |key        |key_len|ref |rows |Extra                                |
+--+-----------+------+-----------+-------------------------+-----------+-------+----+-----+-------------------------------------+
| 1|SIMPLE     |ontime|index_merge|Origin,Dest,SecurityDelay|Origin,Dest|6,6    |NULL|92800|Using union(Origin,Dest); Using where|
+--+-----------+------+-----------+-------------------------+-----------+-------+----+-----+-------------------------------------+

MariaDB [ontime]> EXPLAIN SELECT * FROM ontime WHERE (Origin='SEA' or Dest='SEA') AND depdelay < 12*60;
+--+-----------+------+-----------+--------------------+-----------+-------+----+-----+-------------------------------------+
|id|select_type|table |type       |possible_keys       |key        |key_len|ref |rows |Extra                                |
+--+-----------+------+-----------+--------------------+-----------+-------+----+-----+-------------------------------------+
| 1|SIMPLE     |ontime|index_merge|Origin,DepDelay,Dest|Origin,Dest|6,6    |NULL|92800|Using union(Origin,Dest); Using where|
+--+-----------+------+-----------+--------------------+-----------+-------+----+-----+-------------------------------------+
```

This new behavior is always on and there is no need to enable it. There are no known issues or gotchas with this new optimization.

## See Also:

- [What is MariaDB 5.3](/kb/en/what-is-mariadb-53/)
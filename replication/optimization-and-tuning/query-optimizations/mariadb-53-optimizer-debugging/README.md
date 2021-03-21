# MariaDB 5.3 Optimizer Debugging

[MariaDB 5.3](/kb/en/what-is-mariadb-53/) has an optimizer debugging patch. The patch is pushed into

[lp:maria-captains/maria/5.3-optimizer-debugging](https://code.launchpad.net/~maria-captains/maria/5.3-optimizer-debugging)

The patch is wrapped in #ifdef, but there is a #define straight in
mysql_priv.h so simply compiling that tree should produce a binary with
optimizer debugging enabled.

The patch adds two system variables:

- `@@debug_optimizer_prefer_join_prefix`
- `@@debug_optimizer_dupsweedout_penalized`

the variables are present as session/global variables, and are also settable
via the server command line.

## debug_optimizer_prefer_join_prefix

If this variable is non-NULL, it is assumed to specify a join prefix as a
comma-separated list of table aliases:

```sql
set debug_optimizer_prefer_join_prefix='tbl1,tbl2,tbl3';
```

The optimizer will try its best to build a join plan which matches the
specified join prefix. It does this by comparing join prefixes it is
considering with `@@debug_optimizer_prefer_join_prefix`, and multiplying cost
by a million if the plan doesn't match the prefix.

As a result, you can more-or-less control the join order. For example, let's
take this query:

```sql
MariaDB [test]> explain select * from ten A, ten B, ten C;
+----+-------------+-------+------+---------------+------+---------+------+------+------------------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra                              |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------------------------+
|  1 | SIMPLE      | A     | ALL  | NULL          | NULL | NULL    | NULL |   10 |                                    |
|  1 | SIMPLE      | B     | ALL  | NULL          | NULL | NULL    | NULL |   10 | Using join buffer (flat, BNL join) |
|  1 | SIMPLE      | C     | ALL  | NULL          | NULL | NULL    | NULL |   10 | Using join buffer (flat, BNL join) |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------------------------+
3 rows in set (0.00 sec)
```

and request a join order of C,A,B:

```sql
MariaDB [test]> set debug_optimizer_prefer_join_prefix='C,A,B';
Query OK, 0 rows affected (0.00 sec)

MariaDB [test]> explain select * from ten A, ten B, ten C;
+----+-------------+-------+------+---------------+------+---------+------+------+------------------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra                              |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------------------------+
|  1 | SIMPLE      | C     | ALL  | NULL          | NULL | NULL    | NULL |   10 |                                    |
|  1 | SIMPLE      | A     | ALL  | NULL          | NULL | NULL    | NULL |   10 | Using join buffer (flat, BNL join) |
|  1 | SIMPLE      | B     | ALL  | NULL          | NULL | NULL    | NULL |   10 | Using join buffer (flat, BNL join) |
+----+-------------+-------+------+---------------+------+---------+------+------+------------------------------------+
3 rows in set (0.00 sec)
```

We got it.

Note that this is still a <em>best-effort</em> approach:

- you won't be successful in forcing join orders which the optimizer considers
  invalid (e.g. for "t1 LEFT JOIN t2" you won't be able to get a join order of
  t2,t1).
- The optimizer does various plan pruning and may discard the requested join
  order before it has a chance to find out that it is a million-times cheaper
  than any other.

### Semi-joins

It is possible to force the join order of joins plus semi-joins. This may cause
a different strategy to be used:

```sql
MariaDB [test]> set debug_optimizer_prefer_join_prefix=NULL;
Query OK, 0 rows affected (0.00 sec)

MariaDB [test]> explain select * from ten A where a in (select B.a from ten B, ten C where C.a + A.a < 4);
+----+-------------+-------+------+---------------+------+---------+------+------+----------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra                      |
+----+-------------+-------+------+---------------+------+---------+------+------+----------------------------+
|  1 | PRIMARY     | A     | ALL  | NULL          | NULL | NULL    | NULL |   10 |                            |
|  1 | PRIMARY     | B     | ALL  | NULL          | NULL | NULL    | NULL |   10 | Using where                |
|  1 | PRIMARY     | C     | ALL  | NULL          | NULL | NULL    | NULL |   10 | Using where; FirstMatch(A) |
+----+-------------+-------+------+---------------+------+---------+------+------+----------------------------+
3 rows in set (0.00 sec)

MariaDB [test]> set debug_optimizer_prefer_join_prefix='C,A,B';
Query OK, 0 rows affected (0.00 sec)

MariaDB [test]> explain select * from ten A where a in (select B.a from ten B, ten C where C.a + A.a < 4);
+----+-------------+-------+------+---------------+------+---------+------+------+-------------------------------------------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra                                           |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------------------------------------------+
|  1 | PRIMARY     | C     | ALL  | NULL          | NULL | NULL    | NULL |   10 | Start temporary                                 |
|  1 | PRIMARY     | A     | ALL  | NULL          | NULL | NULL    | NULL |   10 | Using where; Using join buffer (flat, BNL join) |
|  1 | PRIMARY     | B     | ALL  | NULL          | NULL | NULL    | NULL |   10 | Using where; End temporary                      |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------------------------------------------+
3 rows in set (0.00 sec)

```

Semi-join materialization is a somewhat special case, because "join prefix" is
not exactly what you see in the EXPLAIN output. For semi-join materialization:

- don't put "`&lt;subqueryN&gt;`" into `@@debug_optimizer_prefer_join_prefix`
- instead, put all of the materialization tables into the place where you want
  the `&lt;subqueryN&gt;` line.
- Attempts to control the join order inside the materialization nest will be
  unsuccessful. Example: we want A-C-B-AA:

```sql
MariaDB [test]> set debug_optimizer_prefer_join_prefix='A,C,B,AA';
Query OK, 0 rows affected (0.00 sec)

MariaDB [test]> explain select * from ten A, ten AA where A.a in (select B.a from ten B, ten C);
+----+-------------+-------------+--------+---------------+--------------+---------+------+------+------------------------------------+
| id | select_type | table       | type   | possible_keys | key          | key_len | ref  | rows | Extra                              |
+----+-------------+-------------+--------+---------------+--------------+---------+------+------+------------------------------------+
|  1 | PRIMARY     | A           | ALL    | NULL          | NULL         | NULL    | NULL |   10 |                                    |
|  1 | PRIMARY     | <subquery2> | eq_ref | distinct_key  | distinct_key | 5       | func |    1 |                                    |
|  1 | PRIMARY     | AA          | ALL    | NULL          | NULL         | NULL    | NULL |   10 | Using join buffer (flat, BNL join) |
|  2 | SUBQUERY    | B           | ALL    | NULL          | NULL         | NULL    | NULL |   10 |                                    |
|  2 | SUBQUERY    | C           | ALL    | NULL          | NULL         | NULL    | NULL |   10 |                                    |
+----+-------------+-------------+--------+---------------+--------------+---------+------+------+------------------------------------+
5 rows in set (0.00 sec)
```

but we get A-B-C-AA.

## debug_optimizer_dupsweedout_penalized

There are four semi-join execution strategies:

1 `FirstMatch`
2 `Materialization`
3 `LooseScan`
4 `DuplicateWeedout`

The first three strategies have flags in @@optimizer_switch that can be used to
disable them. The `DuplicateWeedout` strategy does not have a flag. This was
done for a reason, as that strategy is the catch-all strategy and it can handle
all kinds of subqueries, in all kinds of join orders. (We're slowly moving to
the point where it will be possible to run with `FirstMatch` enabled and
everything else disabled but we are not there yet.)

Since `DuplicateWeedout` cannot be disabled, there are cases where it "gets
in the way" by being chosen over the strategy you need. This is what
`debug_optimizer_dupsweedout_penalized` is for. if you set:

```sql
MariaDB [test]> set debug_optimizer_dupsweedout_penalized=TRUE;
```

...the costs of query plans that use `DuplicateWeedout` will be multiplied by
a millon. This doesn't mean that you will get rid of `DuplicateWeedout`
<span>—</span> due to [Bug #898747](https://bugs.launchpad.net/bugs/898747) it is still possible to have
`DuplicateWeedout` used even if a cheaper plan exits. A partial remedy to this
is to run with

```sql
MariaDB [test]> set optimizer_prune_level=0;
```

It is possible to use both `debug_optimizer_dupsweedout_penalized` and
`debug_optimizer_prefer_join_prefix` at the same time. This should give you
the desired strategy and join order.

## Further reading

- See mysql-test/t/debug_optimizer.test (in the MariaDB source code) for examples
# Extended Keys

## Syntax

Enable:

```sql
set optimizer_switch='extended_keys=on';
```

Disable:

```sql
set optimizer_switch='extended_keys=off';
```

## Description

##### MariaDB starting with [5.5](/kb/en/what-is-mariadb-55/)

Extended Keys was first introduced in [MariaDB 5.5](/kb/en/what-is-mariadb-55/).

##### MariaDB until [10.0.8](/kb/en/mariadb-1008-release-notes/)

Default is <code class="fixed" style="white-space:pre-wrap"><span class="s1">'extended_keys=off'</span>
</code>.

##### MariaDB starting with [10.0.9](/kb/en/mariadb-1009-release-notes/)

Default is <code class="fixed" style="white-space:pre-wrap"><span class="s1">'extended_keys=on'</span>
</code>.

Extended Keys is an optimization, set with the [optimizer_switch](/kb/en/server-system-variables/#optimizer_switch) system variable, which makes use of existing components of InnoDB/XtraDB keys to
generate more efficient execution plans. Using these components in many cases
allows the server to generate execution plans which employ index-only look-ups.

Extended keys can be used with:

- ref and eq-ref accesses
- range scans
- index-merge scans
- loose scans
- min/max optimizations

## Examples

An example of how extended keys could be employed for a query built over a
[DBT-3/TPC-H database](http://www.tpc.org/tpch/specs.asp) with one added index
defined on `p_retailprice`:

```sql
select o_orderkey
from part, lineitem, orders
where p_retailprice > 2095 and o_orderdate='1992-07-01'
      and o_orderkey=l_orderkey and p_partkey=l_partkey;
```

The above query asks for the `orderkeys` of the orders placed on 1992-07-01
which contain parts with a retail price greater than $2095.

Using Extended Keys, the query could be executed by the following execution
plan:

1 Scan the entries of the index `i_p_retailprice`
  where `p_retailprice&gt;2095` and read `p_partkey` values from the extended
  keys.
2 For each value `p_partkey` make an index look-up into the table lineitem
  employing index `i_l_partkey` and fetch the values of `l_orderkey` from
  the extended index.
3 For each fetched value of `l_orderkey`, append it to the
  date `'1992-07-01'` and use the resulting key for an index look-up by
  index `i_o_orderdate` to fetch the values of `o_orderkey` from the found
  index entries.

All access methods of this plan do not touch table rows, which results in much
better performance.

Here is the explain output for the above query:

```sql
MariaDB [dbt3sf10]> explain
   -> select o_orderkey
   ->   from part, lineitem, orders
   ->   where p_retailprice > 2095 and o_orderdate='1992-07-01'
   ->         and o_orderkey=l_orderkey and p_partkey=l_partkey\G
*************************** 1. row ***************************
          id: 1
 select_type: SIMPLE
       table: part
        type: range
possible_keys: PRIMARY,i_p_retailprice
         key: i_p_retailprice
     key_len: 9
         ref: NULL
        rows: 100
       Extra: Using where; Using index
*************************** 2. row ***************************
          id: 1
 select_type: SIMPLE
       table: lineitem
        type: ref
possible_keys: PRIMARY,i_l_suppkey_partkey,i_l_partkey,i_l_orderkey,i_l_orderkey_quantity
         key: i_l_partkey
     key_len: 5
         ref: dbt3sf10.part.p_partkey
        rows: 15
       Extra: Using index
*************************** 3. row ***************************
          id: 1
 select_type: SIMPLE
       table: orders
        type: ref
possible_keys: PRIMARY,i_o_orderdate
         key: i_o_orderdate
     key_len: 8
         ref: const,dbt3sf10.lineitem.l_orderkey
        rows: 1
       Extra: Using index
3 rows in set (0.00 sec)
```

## See Also

- [MWL#247](http://askmonty.org/worklog/?tid=247)
- [Blog post about the development of this feature](http://igors-notes.blogspot.com/2011/12/3-way-join-that-touches-only-indexes.html)
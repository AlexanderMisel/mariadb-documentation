# Tips on Converting to Galera

These topics will be discussed in more detail below.

Dear Schema Designer:

- InnoDB only, always have PK.

Dear Developer:

- Check for errors, even after COMMIT.
- Moderate sized transactions.
- Don't make assumptions about AUTO_INCREMENT values.
- Handling of "critical reads" is quite different (arguably better).
- Read/Write split is not necessary, but is still advised in case the underlying structure changes in the future.

Dear DBA:

- Building the machines is quite different. (Not covered here)
- ALTERs are handled differently.
- TRIGGERs and EVENTs may need checking.
- Tricks in replication (eg, BLACKHOLE) may not work.
- Several variables need to be set differently.

## Galera is available in many places

Galera's High Availability replication is available via:

- [MariaDB 10.1](/kb/en/what-is-mariadb-101/) and later
- Percona XtraDB Cluster
- Codership's Galera Cluster for MySQL

## Overview of cross-colo writing

(This overview is valid even for same-datacenter nodes, but the issues of latency vanish.)

Cross-colo latency is an 'different' than with traditional replication, but not necessarily better or worse with Galera. The latency happens at a very different time for Galera.

In 'traditional' replication, these steps occur:

- Client talks to Master. If Client and Master are in different colos, this has a latency hit.
- Each SQL to Master is another latency hit, including(?) the COMMIT (unless using autocommit).
- Replication to Slave(s) is asynchronous, so this does not impact the client writing to the Master.
- Since replication is asynchronous, a Client (same or subsequent) cannot be guaranteed to see that data on the Slave. This is a "critical read". The async Replication delay forces apps to take some evasive action.

In Galera-based replication:

- Client talks to any Master -- possibly with cross-colo latency. Or you could arrange to have Galera nodes co-located with clients to avoid this latency.
- At COMMIT time (or end of statement, in case of autocommit=1), galera makes one roundtrip to other nodes.
- The COMMIT usually succeeds, but could fail if some other node is messing with the same rows. (Galera retries on autocommit failures.)
- Failure of the COMMIT is reported to the Client, who should simply replay the SQL statements from the BEGIN.
- Later, the whole transaction will be applied (with possibility of conflict) on the other nodes.
- Critical Read -- details below

For an N-statement transaction: In a typical 'traditional' replication setup:

- 0 or N (N+2?) latency hits, depending on whether the Client is co-located with the Master.
- Replication latencies and delays lead to issues with "Critical Reads".

In Galera:

- 0 latency hits (assuming Client is 'near' some node)
- 1 latency hit for the COMMIT.
- 0 (usually) for Critical Read (details below)

Bottom line: Depending on where your Clients are, and whether you clump statements into BEGIN...COMMIT transacitons, Galera may be faster or slower than traditional replication in a WAN topology.

## AUTO_INCREMENT

By using wsrep_auto_increment_control = ON, the values of auto_increment_increment and auto_increment_offset will be automatically adjusted as nodes come/go.

If you are building a Galera cluster by starting with one node as a Slave to an existing non-Galera system, and if you have multi-row INSERTs that depend on AUTO_INCREMENTs, the read this Percona blog

Bottom line: There may be gaps in AUTO_INCREMENT values. Consecutive rows, even on one connection, will not have consecutive ids.

Beware of Proxies that try to implement a "read/write split". In some situations, a reference to LAST_INSERT_ID() will be sent to a "Slave".

## InnoDB only

For effective replication of data, you must use only InnoDB. This eliminates

- FULLTEXT index (until 5.6)
- SPATIAL index
- MyISAM's PK as second column

You can use MyISAM and MEMORY for data that does not need to be replicated.

Also, you should use "START TRANSACTION READONLY" wherever appropriate.

## Check after COMMIT

Check for errors after issuing COMMIT. A "deadlock" can occur due to writes on other node(s).

Possible exception (could be useful for legacy code without such checks): Treat the system as single-Master, plus Slaves. By writing only to one node, COMMIT should always succeed(?)

What about autocommit = 1? wsrep_retry_autocommit tells Galera to retry if a single statement that is autocommited N times. So, there is still a chance (very slim) of getting a deadlock on such a statement. The default setting of "1" retry is probably good.

## Always have PRIMARY KEY

"Row Based Replication" will be used; this requires a PK on every table.

A non-replicated table (eg, MyISAM) does not have to have a PK.

## Transaction "size"

(This section assumes you have Galera nodes in multiple colos.) Because of some of the issues discussed, it is wise to group your write statements into moderate sized BEGIN...COMMIT transactions. There is one latency hit per COMMIT or autocommit. So, combining statements will decrease those hits. On the other hand, it is unwise (for other reasons) to make huge transactions, such as inserting/modifying millions of rows in a single transaction.

To deal with failure on COMMIT, design your code so you can redo the SQL statements in the transaction without messing up other data. For example, move "normalization" statements out of the main transaction; there is arguably no compelling reason to roll them back if the main code rolls back.

In any case, doing what is "right" for the business logic overrides other considerations.

Galera's tx_isolation is between Serializable and Repeatable Read. tx_isolation variable is ignored.

Set wsrep_log_conflicts to get errors put in the regular MySQL mysqld.err.

XA transactions cannot be supported. (Galera is already doing a form of XA in order to do its thing.)

## Critical reads

Here is a 'simple' (but not 'free') way to assure that a read-after-write, even from a different connection, will see the updated data.

SET SESSION wsrep_sync_wait = 1;
   SELECT ...
   SET SESSION wsrep_sync_wait = 0;

For non-SELECTs, use a different bit set for the first select. (TBD: Would 0xffff always work?) (Before Galera 3.6, it was wsrep_causal_reads = ON.) Doc for wsrep_sync_wait

This setting stalls the SELECT until all current updates have been applied to the node. That is sufficient to guarantee that a previous write will be visible. The time cost is usually zero. However, a large UPDATE could lead to a delay. Because of RBR and parallel application, delays are likely to be less than on traditional replication. Zaitsev's blog

It may be more practical (for a web app) to simply set wsrep_sync_wait right after connecting.

## MyISAM and MEMORY

As said above, use InnoDB only. However, here is more info on the MyISAM (and hence FULLTEXT, SPATIAL, etc) issues.

MyISAM and MEMORY tables are not replicated.

Having MyISAM not replicated can be a big benefit -- You can "CREATE TEMPORARY TABLE ... ENGINE=MyISAM" and have it exist on only one node. RBR assures that any data transferred from that temp table into a 'real' table can still be replicated.

## Replicating GRANTs

GRANTs and related operations act on the MyISAM tables in the database `mysql`. The GRANT statements will(?) be replicated, but the underlying tables will not.

## ALTERs

Many DDL changes on Galera can be achieved without downtime, even if they take a long time.

[RSU vs TOI](https://galeracluster.com/documentation-webpages/documentation/schema-upgrades.html):

- Rolling Schema Upgrade (RSU):  manually execute the DDL on each node in the cluster. The node will desync while executing the DDL.
- Total Order Isolation (TOI): Galera automatically replicates the DDL to each node in the cluster, and it synchronizes each node so that the statement is executed at same time (in the replication sequence) on all nodes.

Caution: Since there is no way to synchronize the clients with the DDL, you must make sure that the clients are happy with either the old or the new schema. Otherwise, you will probably need to take down the entire cluster while simultaneously switching over both the schema and the client code.

Fast DDL operations can usually be executed in TOI mode:

- DDL operations that support the `NOCOPY` and `INSTANT` algorithms are usually very fast.
- DDL operations that support the `INPLACE` algorithm may be fast or slow, depending on whether the table needs to be rebuilt.
- DDL operations that only support the `COPY` algorithm are usually very slow.

For a list of which operations support which algorithms, see [InnoDB Online DDL](/columns-storage-engines-and-plugins/storage-engines/innodb/innodb-online-ddl/).

If you need to use RSU mode, then do the following separately for each node:

```sql
SET SESSION wsrep_OSU_method='RSU';
ALTER TABLE tab <alter options here>;
SET SESSION wsrep_OSU_method='TOI';
```

[More discussion of RSU procedures](http://www.severalnines.com/blog/online-schema-upgrade-mysql-galera-cluster-using-rsu-method)

## Single "Master" Configuration

You can 'simulate' Master + Slaves by having clients write only to one node.

- No need to check for errors after COMMIT.
- Lose the latency benefits.

## DBA tricks

- Remove node from cluster; back it up; put it back in. Syncup is automatic.
- Remove node from cluster; use it for testing, etc; put it back in. Syncup is automatic.
- Rolling hardware/software upgrade: Remove; upgrade; put back in. Repeat.

## Variables that may need to be different

- [auto_increment_increment](/kb/en/replication-and-binary-log-server-system-variables/#auto_increment_increment) - If you are writing to multiple nodes, and you use AUTO_INCREMENT, then auto_increment_increment will automatically be equal the current number of nodes.
- [binlog-do](/kb/en/mysqld-options/#-binlog-do-db)/[ignore-db](/kb/en/mysqld-options/#-binlog-ignore-db) - Do not use.
- [binlog_format](/kb/en/replication-and-binary-log-server-system-variables/#binlog_format) - ROW is required for Galera.
- [innodb_autoinc_lock_mode](/kb/en/replication-and-binary-log-server-system-variables/#auto_increment_increment) - 2
- [innodb_doublewrite](/kb/en/xtradbinnodb-server-system-variables/#innodb_doublewrite) - ON: When an IST occurs, want there to be no torn pages? (With FusionIO or other drives that guarantee atomicity, OFF is better.)
- [innodb_flush_log_at_trx_commit](/kb/en/xtradbinnodb-server-system-variables/#innodb_doublewrite) - 2 or 0. IST or SST will recover from loss if you have 1.
- [query_cache_size](/kb/en/server-system-variables/#query_cache_size) - 0
- [query_cache_type](/kb/en/server-system-variables/#query_cache_type) - 0: The Query cache cannot be used in a Galera context.
- [wsrep_auto_increment_control](/kb/en/galera-cluster-system-variables/#wsrep_auto_increment_control) - Normally want ON
- [wsrep_on](/kb/en/galera-cluster-system-variables/#wsrep_on) - ON
- [wsrep_provider_options](/kb/en/galera-cluster-system-variables/#wsrep_provider_options) - Various settings may need tuning if you are using a WAN.
- [wsrep_slave_threads](/kb/en/galera-cluster-system-variables/#wsrep_slave_threads) - use for parallel replication
- [wsrep_sync_wait](/kb/en/galera-cluster-system-variables/#wsrep_sync_wait) (previously wsrep_causal_reads) - used transiently to dealing with "critical reads".

## Miscellany

Until recently, FOREIGN KEYs were buggy.

LOAD DATA is auto-chunked. That is, it is passed to other nodes piecemeal, not all at once.

[MariaDB's known issues with Galera](/replication/galera-cluster/mariadb-galera-cluster-known-limitations/)

DROP USER may not replicate?

A slight difference in ROLLBACK for conflict: InnoDB rolls back smaller transaction; Galera rolls back last.

[Slide Deck for Galera](http://www.slideshare.net/skysql/mariadb-galera-cluster-simple-transparent-highly-available)

SET GLOBAL wsrep_debug = 1; leads to a lot of debug info in the error log.

Large UPDATEs / DELETEs should be broken up. This admonition is valid for all databases, but there are additional issues in Galera.

WAN: May need to increase (from the defaults) wsrep_provider_options = evs...

MySQL/Perona 5.6 or MariaDB 10 is recommended when going to Galera.

[Cluster limitations](/replication/galera-cluster/mariadb-galera-cluster-known-limitations/)
[Slide show](https://www.percona.com/files/presentations/percona-live/nyc-2012/PLNY12-galera-cluster-best-practices.pdf)

## GTIDs

See [Using MariaDB GTIDs with MariaDB Galera Cluster](/replication/galera-cluster/using-mariadb-replication-with-mariadb-galera-cluster/using-mariadb-gtids-with-mariadb-galera-cluster/).

## How many nodes to have in a cluster

If all the servers are in the same 'vulnerability zone' -- eg, rack or data center -- Have an odd number (at least 3) of nodes.

When spanning colos, you need 3 (or more) data centers in order to be 'always' up, even during a colo failure. With only 2 data centers, Galera can automatically recover from one colo outage, but not the other. (You pick which.)

If you use 3 or 4 colos, these number of nodes per colo are safe:

- 3 nodes: 1+1+1 (1 node in each of 3 colos)
- 4 nodes: 1+1+1+1 (4 nodes won't work in 3 colos)
- 5 nodes: 2+2+1, 2+1+1+1 (5 nodes spread 'evenly' across the colos)
- 6 nodes: 2+2+2, 2+2+1+1
- 7 nodes: 3+2+2, 3+3+1, 2+2+2+1, 3+2+1+1
There may be a way to "weight" the nodes differently; that would allow a few more configurations. With "weighting", give each colo the same weight; then subdivide the weight within each colo evenly. Four nodes in 3 colos: (1/6+1/6) + 1/3 + 1/3 That way, any single colo failure cannot lead to "split brain".

## Postlog

Posted 2013; VARIABLES: 2015; Refreshed Feb. 2016

## See also

Rick James graciously allowed us to use this article in the Knowledge Base.

[Rick James' site](http://mysql.rjweb.org/) has other useful tips, how-tos,
optimizations, and debugging tips.

Original source: [http://mysql.rjweb.org/doc.php/galera](http://mysql.rjweb.org/doc.php/galera)
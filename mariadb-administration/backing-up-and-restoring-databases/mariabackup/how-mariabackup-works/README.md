# How Mariabackup Works

This is a description of the different stages in Mariabackup, what they do and why they are needed.

Note that a few items are marked with `TODO`; these are things we are working on and will be in next version of Mariabackup.

## Execution Stages

### Initialization Phase

- Connect to mysqld instance, find out important variables (datadir ,InnoDB pagesize, encryption keys, encryption plugin etc)
- Scan the database directory, `datadir`, looking for InnoDB tablespaces, load the tablespaces (basically, it is an “open” in InnoDB sense)
- If --lock-ddl-per-table is used:
<ul start="1"><li>Do MDL locks, for InnoDB tablespaces that we want to copy. This is to ensure that there are no ALTER, RENAME , TRUNCATE or DROP TABLE on any of the tables that we want to copy.
</li><li>This is implemented with:
</li></ul>

```sql
BEGIN
For each affected table
SELECT 1 from <table> LIMIT 0
```

- If lock-ddl-per-table is not done, then Mariabackup would have to know all tables that were created or altered during the backup. See [MDEV-16791](https://jira.mariadb.org/browse/MDEV-16791).

### Redo Log Handling

Start a dedicated thread in Mariabackup to copy InnoDB redo log (`ib_logfile*`).

- This is needed to record all changes done while the backup is running. (The redo log logically is a single circular file, split into [innodb_log_files_in_group](/kb/en/xtradbinnodb-server-system-variables/#innodb_log_files_in_group) files.)
- The log is also used to see detect if any truncate or online alter tables are used.
- The assumption is that the copy thread will be able to keep up with server. It should always be able keep up, if the redo log is big enough.

### Copy-phase for InnoDB Tablespaces

- Copy all selected tablespaces, file by file, in dedicated threads in Mariabackup without involving the mysqld server.
- This is special “careful” copy, it looks for page-level consistency by checking the checksum.
- The files are not point-in-time consistent as data may change during copy.
- The idea is that InnoDB recovery would make it point-in-time consistent.
- Copy Aria log files (TODO)

### Create a Consistent Backup Point

- Execute [FLUSH TABLE WITH READ LOCK](/sql-statements-structure/sql-statements/administrative-sql-statements/flush-commands/flush). This is default, but may be omitted with the `-–no-lock` parameter. The reason why `FLUSH` is needed is to ensure that all tables are in a consistent state at the exact same point in time, independent of storage engine.
- If `--lock-ddl-per-table` is used and there is a user query waiting for MDL, the user query will be killed to resolve a deadlock. Note that these are only queries of type ALTER, DROP, TRUNCATE or RENAME TABLE. ([MDEV-15636](https://jira.mariadb.org/browse/MDEV-15636))

### Last Copy Phase

- Copy `.frm`, `MyISAM`, `Aria` and other storage engine files
- If `MyRocks` is used, create rocksdb checkpoint   via  "set rocksdb_create_checkpoint=$rocksdb_data_dir/mariabackup_rocksdb_checkpoint " command. The result of it is a directory with hardlinks to MyRocks files. Copy the checkpoint directory to the backup (or  create hardlinks in backup directory is on the same partition as data directory). Remove the checkpoint directory.
- Copy tables that were created while the backup was running and do rename files that were changed during backup (since [MDEV-16791](https://jira.mariadb.org/browse/MDEV-16791))
- Copy the rest of InnoDB redo log, stop redo-log-copy thread
- Copy changes to Aria log files (They are append only, so this is easy to do) (TODO)
- Write some metadata info (binlog position)

### Release Locks

- If [FLUSH TABLE WITH READ LOCK](/sql-statements-structure/sql-statements/administrative-sql-statements/flush-commands/flush) was done:
<ul start="1"><li>execute: `UNLOCK TABLES`
</li></ul>
- If `--lock-ddl-per-table`  was done:
<ul start="1"><li>execute `COMMIT`
</li></ul>

### Handle Log Tables (TODO)

- If log tables exists:
<ul start="1"><li>Take MDL lock for log tables
</li><li>Copy part of log tables that wasn't copied before
</li><li>Unlock log tables
</li></ul>

## Notes

- If [FLUSH TABLE WITH READ LOCK](/sql-statements-structure/sql-statements/administrative-sql-statements/flush-commands/flush) is not used, then only InnoDB tables will be consistent (not the privilege tables in the mysql database or the binary log). The backup point depends on the content of the redo log within the backup itself.
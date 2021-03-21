# SHOW VARIABLES

## Syntax

```sql
SHOW [GLOBAL | SESSION] VARIABLES
    [LIKE 'pattern' | WHERE expr]
```

## Description

`SHOW VARIABLES` shows the values of MariaDB [system variables](/replication/optimization-and-tuning/system-variables/server-system-variables/). This
information also can be obtained using the [mysqladmin](/clients-utilities/mysqladmin/) variables
command. The `LIKE` clause, if present, indicates which variable names
to match. The `WHERE` clause can be given to select rows using more
general conditions.

With the `GLOBAL` modifier, `SHOW VARIABLES` displays the values that are
used for new connections to MariaDB. With `SESSION`, it displays the
values that are in effect for the current connection. If no modifier
is present, the default is `SESSION`. `LOCAL` is a synonym for `SESSION`.
With a `LIKE` clause, the statement displays only rows for those
variables with names that match the pattern. To obtain the row for a
specific variable, use a `LIKE` clause as shown:

```sql
SHOW VARIABLES LIKE 'maria_group_commit';
SHOW SESSION VARIABLES LIKE 'maria_group_commit';
```

To get a list of variables whose name match a pattern, use the "`%`"
wildcard character in a `LIKE` clause:

```sql
SHOW VARIABLES LIKE '%maria%';
SHOW GLOBAL VARIABLES LIKE '%maria%';
```

Wildcard characters can be used in any position within the pattern to
be matched. Strictly speaking, because "`_`" is a wildcard that matches
any single character, you should escape it as "`\_`" to match it
literally. In practice, this is rarely necessary.

The <code class="highlight fixed" style="white-space:pre-wrap">WHERE</code> and <code class="highlight fixed" style="white-space:pre-wrap">LIKE</code> clauses can be given to select rows using more general conditions, as discussed in [Extended SHOW](/sql-statements-structure/sql-statements/administrative-sql-statements/show/extended-show/).

See [SET](/sql-statements-structure/sql-statements/administrative-sql-statements/set-commands/set/) for information on setting server system variables.

See [Server System Variables](/replication/optimization-and-tuning/system-variables/server-system-variables/) for a list of all the variables that can be set.

You can also see the server variables by querying the [Information Schema GLOBAL_VARIABLES and SESSION_VARIABLES](/kb/en/information-schema-global_variables-and-session_variables-tables/) tables.

## Examples

```sql
SHOW VARIABLES LIKE 'aria%';
+------------------------------------------+---------------------+
| Variable_name                            | Value               |
+------------------------------------------+---------------------+
| aria_block_size                          | 8192                |
| aria_checkpoint_interval                 | 30                  |
| aria_checkpoint_log_activity             | 1048576             |
| aria_force_start_after_recovery_failures | 0                   |
| aria_group_commit                        | none                |
| aria_group_commit_interval               | 0                   |
| aria_log_file_size                       | 1073741824          |
| aria_log_purge_type                      | immediate           |
| aria_max_sort_file_size                  | 9223372036853727232 |
| aria_page_checksum                       | ON                  |
| aria_pagecache_age_threshold             | 300                 |
| aria_pagecache_buffer_size               | 134217728           |
| aria_pagecache_division_limit            | 100                 |
| aria_recover                             | NORMAL              |
| aria_repair_threads                      | 1                   |
| aria_sort_buffer_size                    | 134217728           |
| aria_stats_method                        | nulls_unequal       |
| aria_sync_log_dir                        | NEWFILE             |
| aria_used_for_temp_tables                | ON                  |
+------------------------------------------+---------------------+
```

```sql
SELECT VARIABLE_NAME, SESSION_VALUE, GLOBAL_VALUE FROM
  INFORMATION_SCHEMA.SYSTEM_VARIABLES WHERE
  VARIABLE_NAME LIKE 'max_error_count' OR
  VARIABLE_NAME LIKE 'innodb_sync_spin_loops';
+---------------------------+---------------+--------------+
| VARIABLE_NAME             | SESSION_VALUE | GLOBAL_VALUE |
+---------------------------+---------------+--------------+
| MAX_ERROR_COUNT           | 64            | 64           |
| INNODB_SYNC_SPIN_LOOPS    | NULL          | 30           |
+---------------------------+---------------+--------------+

SET GLOBAL max_error_count=128;

SELECT VARIABLE_NAME, SESSION_VALUE, GLOBAL_VALUE FROM
  INFORMATION_SCHEMA.SYSTEM_VARIABLES WHERE
  VARIABLE_NAME LIKE 'max_error_count' OR
  VARIABLE_NAME LIKE 'innodb_sync_spin_loops';
+---------------------------+---------------+--------------+
| VARIABLE_NAME             | SESSION_VALUE | GLOBAL_VALUE |
+---------------------------+---------------+--------------+
| MAX_ERROR_COUNT           | 64            | 128          |
| INNODB_SYNC_SPIN_LOOPS    | NULL          | 30           |
+---------------------------+---------------+--------------+

SET GLOBAL max_error_count=128;

SHOW VARIABLES LIKE 'max_error_count';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_error_count | 64    |
+-----------------+-------+

SHOW GLOBAL VARIABLES LIKE 'max_error_count';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_error_count | 128   |
+-----------------+-------+
```

Because the following variable only has a global scope, the global value is returned even when specifying SESSION (in this case by default):

```sql
SHOW VARIABLES LIKE 'innodb_sync_spin_loops';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| innodb_sync_spin_loops | 30    |
+------------------------+-------+
```
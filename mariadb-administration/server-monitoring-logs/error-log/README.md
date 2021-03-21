# Error Log

The error log contains a record of critical errors that occurred during the server's operation, table corruption, start and stop information.

SQL errors can also be logged in a separate file using the [SQL_ERROR_LOG plugin](/kb/en/sql_error_log-plugin/).

## Configuring the Error Log Output Destination

MariaDB always writes its error log, but the destination is configurable.

### Writing the Error Log to a File

To configure the error log to be written to a file, you can set the <a undefined>log_error</a> system variable. You can configure a specific file name. However, if a specific file name is not configured, then the log will be written to the `${hostname}.err` file in the <a undefined>datadir</a> directory by default.

The <a undefined>log_error</a> system variable can be set in a server [option group](/kb/en/configuring-mariadb-with-option-files/#option-groups) in an [option file](/mariadb-administration/getting-installing-and-upgrading-mariadb/configuring-mariadb-with-option-files/) prior to starting up the server. For example, to write the error log to the default `${hostname}.err`  file, you could configure the following:

```sql
[mariadb]
...
log_error
```

If you configure a specific file name as the <a undefined>log_error</a> system variable, and if it is not an absolute path, then it will be relative to the <a undefined>datadir</a> directory. For example, if you configured the following, then the error log would be written to `mariadb.err` in the <a undefined>datadir</a> directory:

```sql
[mariadb]
...
log_error=mariadb.err
```

If it is a relative path, then the <a undefined>log_error</a> is relative to the <a undefined>datadir</a> directory.

However, the <a undefined>log_error</a> system variable can also be an absolute path. For example:

```sql
[mariadb]
...
log_error=/var/log/mysql/mariadb.err
```

Another way to configure the error log file name is to set the <a undefined>log-basename</a> option, which configures MariaDB to use a common prefix for all log files (e.g. [general query log](/mariadb-administration/server-monitoring-logs/general-query-log/), [slow query log](/mariadb-administration/server-monitoring-logs/slow-query-log/), error log, [binary logs](/mariadb-administration/server-monitoring-logs/binary-log/), etc.). The error log file name will be built by adding a `.err` extension to this prefix. For example, if you configured the following, then the error log would still be written to `mariadb.err` in the <a undefined>datadir</a> directory:

```sql
[mariadb]
...
log-basename=mariadb
log_error
```

The <a undefined>log-basename</a> cannot be an absolute path. The log file name is relative to the <a undefined>datadir</a> directory.

### Writing the Error Log to Stderr on Unix

On Unix, if the <a undefined>log_error</a> system variable is not set, then errors are written to `stderr`, which usually means that the log messages are output to the terminal that started `mysqld`.

If the <a undefined>log_error</a> system variable was set in an [option file](/mariadb-administration/getting-installing-and-upgrading-mariadb/configuring-mariadb-with-option-files/) or on the command-line, then it can still be unset by specifying `--skip-log-error`.

### Writing the Error Log to Syslog on Unix

On Unix, the error log can also be redirected to the [syslog](https://linux.die.net/man/8/rsyslogd). How this is done depends on how you [start](/kb/en/starting-and-stopping-mariadb-starting-and-stopping-mariadb/) MariaDB.

#### Syslog with mysqld_safe

If you [start](/kb/en/starting-and-stopping-mariadb-starting-and-stopping-mariadb/) MariaDB with [mysqld_safe](/mariadb-administration/getting-installing-and-upgrading-mariadb/starting-and-stopping-mariadb/mysqld_safe/), then the error log can be redirected to the syslog. See [mysqld_safe: Configuring MariaDB to Write the Error Log to Syslog](/kb/en/mysqld_safe/#configuring-mariadb-to-write-the-error-log-to-syslog) for more information.

#### Syslog with Systemd

If you [start](/kb/en/starting-and-stopping-mariadb-starting-and-stopping-mariadb/) MariaDB with [systemd](/mariadb-administration/getting-installing-and-upgrading-mariadb/starting-and-stopping-mariadb/systemd/), then the error log can also be redirected to the syslog. See [Systemd: Configuring MariaDB to Write the Error Log to Syslog](/kb/en/systemd/#configuring-mariadb-to-write-the-error-log-to-syslog) for more information.

[systemd](/mariadb-administration/getting-installing-and-upgrading-mariadb/starting-and-stopping-mariadb/systemd/) also has its own logging system called the `journal`, and some errors may get logged there instead. See [Systemd:Systemd Journal](/kb/en/systemd/#systemd-journal) for more information.

### Writing the Error Log to Console on Windows

On Windows, if the <a undefined>console</a> option is specified, and if the <a undefined>log_error</a> system variable is not used, then errors are written to the console. If both options are specified, then the last option takes precedence.

### Writing the Error Log to the Windows Event Viewer

On Windows, error log messages are also written to the Windows Event Viewer. You can find MariaDB's error log messages by browsing <strong>Windows Logs</strong>, and then selecting <strong>Application</strong> or <strong>Application Log</strong>, depending on the Windows version.

In [MariaDB 10.3](/kb/en/what-is-mariadb-103/) and before, you can find MariaDB's error log messages by searching for the <strong>Source</strong> `MySQL`.

In [MariaDB 10.4](/kb/en/what-is-mariadb-104/) and later, you can find MariaDB's error log messages by searching for the <strong>Source</strong> `MariaDB`.

## Configuring the Error Log Verbosity

##### MariaDB starting with [10.2.4](/kb/en/mariadb-1024-release-notes/)

In [MariaDB 10.2.4](/kb/en/mariadb-1024-release-notes/) and later, the default value of the <a undefined>log_warnings</a> system variable is `2`.

##### MariaDB until [10.2.3](/kb/en/mariadb-1023-release-notes/)

In [MariaDB 10.2.3](/kb/en/mariadb-1023-release-notes/) and before, the default value of the <a undefined>log_warnings</a> system variable is `1`.

The <a undefined>log_warnings</a> system variable can be used to configure the verbosity of the error log. It can be changed dynamically with <a undefined>SET GLOBAL</a>. For example:

```sql
SET GLOBAL log_warnings=3;
```

It can also be set either on the command-line or in a server [option group](/kb/en/configuring-mariadb-with-option-files/#option-groups) in an [option file](/mariadb-administration/getting-installing-and-upgrading-mariadb/configuring-mariadb-with-option-files/) prior to starting up the server. For example:

```sql
[mariadb]
...
log_warnings=3
```

Some of the warnings included in each verbosity level are described below.

The <a undefined>log_warnings</a> system variable only has an effect on some log messages. Some log messages are <strong>always</strong> written to the error log, regardless of the error log verbosity. For example, most warnings from the InnoDB storage engine are not affected by <a undefined>log_warnings</a>. For a complete list of log messages affected by <a undefined>log_warnings</a>, see the description of the <a undefined>log_warnings</a> system variable.

### Verbosity Level 0

If <a undefined>log_warnings</a> is `0`, then many optional warnings will not be logged. However, this does not prevent all warnings from being logged, because there are certain core warnings that will always be written to the error log. For example:

- If [InnoDB strict mode](/columns-storage-engines-and-plugins/storage-engines/innodb/innodb-strict-mode/) is disabled, and if DDL is performed on a table that triggers a ["Row size too large" error](/columns-storage-engines-and-plugins/storage-engines/innodb/innodb-row-formats/troubleshooting-row-size-too-large-errors-with-innodb/), then InnoDB will log a warning:

```sql
[Warning] InnoDB: Cannot add field col25 in table db1.tab because after adding it, the row size is 8477 which is greater than maximum allowed size (8126) for a record on index leaf page.
```

However, if [InnoDB strict mode](/columns-storage-engines-and-plugins/storage-engines/innodb/innodb-strict-mode/) is enabled, then the same message will be logged as an error.

### Verbosity Level 1

If <a undefined>log_warnings</a> is `1`, then many types of warnings are logged. Some useful warnings are:

- Replication-related messages:

```sql
[Note] Error reading relay log event: slave SQL thread was killed
[Note] Slave SQL thread exiting, replication stopped in log 'dbserver-2-bin.000033' at position 181420; GTID position '0-263316466-368886'
[Note] Slave I/O thread exiting, read up to log 'dbserver-2-bin.000034', position 642; GTID position 0-263316466-368887
```

- Messages related to DNS lookup failures:

```sql
[Warning] IP address '192.168.1.193' could not be resolved: Name or service not known
```

- Messages related to the [event scheduler](/programming-customizing-mariadb/triggers-events/event-scheduler/):

```sql
[Note] Event Scheduler: Loaded 0 events
```

- Messages related to [unsafe statements for statement-based replication](/replication/standard-replication/unsafe-statements-for-statement-based-replication/):

```sql
[Warning] Unsafe statement written to the binary log using statement format since 
  BINLOG_FORMAT = STATEMENT. The statement is unsafe because it uses a LIMIT clause. This 
  is unsafe because the set of rows included cannot be predicted.
```

##### MariaDB starting with [10.0.14](/kb/en/mariadb-10014-release-notes/)

Frequent warnings about [unsafe statements for statement-based replication](/replication/standard-replication/unsafe-statements-for-statement-based-replication/) can cause the error log to grow very large. In [MariaDB 10.0.14](/kb/en/mariadb-10014-release-notes/) and later, MariaDB will automatically detect frequent duplicate warnings about [unsafe statements for statement-based replication](/replication/standard-replication/unsafe-statements-for-statement-based-replication/). After 10 identical warnings are detected, MariaDB will prevent that same warning from being written to the error log again for the next 5 minutes.

### Verbosity Level 2

If <a undefined>log_warnings</a> is `2`, then a couple other different kinds of warnings are printed. For example:

- Messages related to access denied errors:

```sql
[Warning] Access denied for user 'root'@'localhost' (using password: YES)
```

- Messages related to connections that are aborted due to errors or timeouts:

```sql
[Warning] Aborted connection 35 to db: 'unconnected' user: 'user1@host1' host: '192.168.1.40' (Got an error writing communication packets)
[Warning] Aborted connection 36 to db: 'unconnected' user: 'user1@host2' host: '192.168.1.230' (Got an error writing communication packets)
[Warning] Aborted connection 38 to db: 'db1' user: 'user2' host: '192.168.1.60' (Unknown error) 
[Warning] Aborted connection 51 to db: 'db1' user: 'user2' host: '192.168.1.50' (Got an error reading communication packets)
[Warning] Aborted connection 52 to db: 'db1' user: 'user3' host: '192.168.1.53' (Got timeout reading communication packets)
```

- Messages related to table handler errors:

```sql
[Warning] Can't find record in 'tab1'.
[Warning] Can't write; duplicate key in table 'tab1'.
[Warning] Lock wait timeout exceeded; try restarting transaction.
[Warning] The number of locks exceeds the lock table size.
[Warning] Update locks cannot be acquired during a READ UNCOMMITTED transaction.
```

- Messages related to the files used to [persist replication state](/kb/en/change-master-to/#option-persistence):
<ul start="1"><li>Either the default `master.info` file or the file that is configured by the <a undefined>master_info_file</a> option.
</li><li>Either the default `relay-log.info` file or the file that is configured by the <a undefined>relay_log_info_file</a> system variable.
</li></ul>

```sql
[Note] Reading Master_info: '/mariadb/data/master.info'  Relay_info:'/mariadb/data/relay-log.info'
[Note] Initialized Master_info from '/mariadb/data/master.info'
[Note] Reading of all Master_info entries succeded
[Note] Deleted Master_info file '/mariadb/data/master.info'.
[Note] Deleted Master_info file '/mariadb/data/relay-log.info'.
```

- Messages about a master's [binary log dump thread](/kb/en/replication-threads/#binary-log-dump-thread):

```sql
[Note] Start binlog_dump to slave_server(263316466), pos(, 4)
```

### Verbosity Level 3

If <a undefined>log_warnings</a> is `3`, then a couple other different kinds of warnings are printed. For example:

- Messages related to old-style language options:

```sql
[Warning] An old style --language value with language specific part detected: /usr/local/mysql/data/
[Warning] Use --lc-messages-dir without language specific part instead.
```

- Messages related to [progress of InnoDB online DDL](https://mariadb.org/monitoring-progress-and-temporal-memory-usage-of-online-ddl-in-innodb/):

```sql
[Note] InnoDB: Online DDL : Start
[Note] InnoDB: Online DDL : Start reading clustered index of the table and create temporary files
[Note] InnoDB: Online DDL : End of reading clustered index of the table and create temporary files
[Note] InnoDB: Online DDL : Start merge-sorting index PRIMARY (1 / 3), estimated cost : 18.0263
[Note] InnoDB: Online DDL : merge-sorting has estimated 33 runs
[Note] InnoDB: Online DDL : merge-sorting current run 1 estimated 33 runs
[Note] InnoDB: Online DDL : merge-sorting current run 2 estimated 17 runs
[Note] InnoDB: Online DDL : merge-sorting current run 3 estimated 9 runs
[Note] InnoDB: Online DDL : merge-sorting current run 4 estimated 5 runs
[Note] InnoDB: Online DDL : merge-sorting current run 5 estimated 3 runs
[Note] InnoDB: Online DDL : merge-sorting current run 6 estimated 2 runs
[Note] InnoDB: Online DDL : End of  merge-sorting index PRIMARY (1 / 3)
[Note] InnoDB: Online DDL : Start building index PRIMARY (1 / 3), estimated cost : 27.0395
[Note] InnoDB: Online DDL : End of building index PRIMARY (1 / 3)
[Note] InnoDB: Online DDL : Completed
[Note] InnoDB: Online DDL : Start merge-sorting index ux1 (2 / 3), estimated cost : 5.7895
[Note] InnoDB: Online DDL : merge-sorting has estimated 2 runs
[Note] InnoDB: Online DDL : merge-sorting current run 1 estimated 2 runs
[Note] InnoDB: Online DDL : End of  merge-sorting index ux1 (2 / 3)
[Note] InnoDB: Online DDL : Start building index ux1 (2 / 3), estimated cost : 8.6842
[Note] InnoDB: Online DDL : End of building index ux1 (2 / 3)
[Note] InnoDB: Online DDL : Completed
[Note] InnoDB: Online DDL : Start merge-sorting index ix1 (3 / 3), estimated cost : 6.1842
[Note] InnoDB: Online DDL : merge-sorting has estimated 3 runs
[Note] InnoDB: Online DDL : merge-sorting current run 1 estimated 3 runs
[Note] InnoDB: Online DDL : merge-sorting current run 2 estimated 2 runs
[Note] InnoDB: Online DDL : End of  merge-sorting index ix1 (3 / 3)
[Note] InnoDB: Online DDL : Start building index ix1 (3 / 3), estimated cost : 9.2763
[Note] InnoDB: Online DDL : End of building index ix1 (3 / 3)
[Note] InnoDB: Online DDL : Completed
```

### Verbosity Level 4

If <a undefined>log_warnings</a> is `4`, then a couple other different kinds of warnings are printed. For example:

- Messages related to killed connections:

```sql
[Warning] Aborted connection 53 to db: 'db1' user: 'user2' host: '192.168.1.50' (KILLED)
```

- Messages related to <strong>all</strong> closed connections:

```sql
[Warning] Aborted connection 56 to db: 'db1' user: 'user2' host: '192.168.1.50' (CLOSE_CONNECTION)
```

- Messages related to released connections, such as when a transaction is committed and <a undefined>completion_type</a> is set to `RELEASE`:

```sql
[Warning] Aborted connection 58 to db: 'db1' user: 'user2' host: '192.168.1.50' (RELEASE)
```

### Verbosity Level 9

If <a undefined>log_warnings</a> is `9`, then some <strong>very</strong> verbose warnings are printed. For example:

- Messages about initializing plugins:

```sql
[Note] Initializing built-in plugins
[Note] Initializing plugins specified on the command line
[Note] Initializing installed plugins
```

### MySQL's log_error_verbosity

MariaDB does not support the <a undefined>log_error_verbosity</a> system variable added in MySQL 5.7.

## Format

Until [MariaDB 10.1.4](/kb/en/mariadb-1014-release-notes/), the format consisted of the date (yymmdd) and time, followed by the type of error (Note, Warning or Error) and the error message, for example:

```sql
160615 16:53:08 [Note] InnoDB: The InnoDB memory heap is disabled
```

From [MariaDB 10.1.5](/kb/en/mariadb-1015-release-notes/), the date format has been extended to yyyy-mm-dd, and the thread ID has been added, for example:

```sql
2016-06-15 16:53:33 139651251140544 [Note] InnoDB: The InnoDB memory heap is disabled
```

## Rotating the Error Log on Unix and Linux

Unix and Linux distributions offer the <a undefined>logrotate</a> utility, which makes it very easy to rotate log files. See [Rotating Logs on Unix and Linux](/mariadb-administration/server-monitoring-logs/rotating-logs-on-unix-and-linux/) for more information on how to use this utility to rotate the error log.

## Error Messages File

Many error messages are ready from an error messages file that contains localized error messages. If the server can't find this file when it starts up, then you might see errors like the following:

```sql
[ERROR] Can't find messagefile '/usr/share/errmsg.sys'
```

If this error is occurring because the file is in a custom location, then you can configure this location by setting the <a undefined>lc_messages_dir</a> system variable either on the command-line or in a server [option group](/kb/en/configuring-mariadb-with-option-files/#option-groups) in an [option file](/mariadb-administration/getting-installing-and-upgrading-mariadb/configuring-mariadb-with-option-files/) prior to starting up the server. For example:

```sql
[mariadb]
...
lc_messages_dir=/usr/share/mysql/
```

If you want to use a different locale for error messages, then you can also set the <a undefined>lc_messages</a> system variable. For example:

```sql
[mariadb]
...
lc_messages_dir=/usr/share/mysql/
lc_messages=en_US
```

See [Setting the Language for Error Messages](/columns-storage-engines-and-plugins/data-types/string-data-types/character-sets/internationalization-and-localization/setting-the-language-for-error-messages/) for more information.
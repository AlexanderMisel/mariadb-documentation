# Performance Schema prepared_statements_instances Table

##### MariaDB starting with [10.5.2](/kb/en/mariadb-1052-release-notes/)

The prepared_statements_instances table was introduced in [MariaDB 10.5.2](/kb/en/mariadb-1052-release-notes/).

The `prepared_statements_instances` table contains aggregated statistics of prepared statements.

The maximum number of rows in the table is determined by the [performance_schema_max_prepared_statement_instances](/kb/en/performance-schema-system-variables/#performance_schema_max_prepared_statement_instances) system variable, which is by default autosized on startup.

The table contains the following columns:

```sql
+-----------------------------+--------------------------------------------------------+------+-----+---------+-------+
| Field                       | Type                                                   | Null | Key | Default | Extra |
+-----------------------------+--------------------------------------------------------+------+-----+---------+-------+
| OBJECT_INSTANCE_BEGIN       | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| STATEMENT_ID                | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| STATEMENT_NAME              | varchar(64)                                            | YES  |     | NULL    |       |
| SQL_TEXT                    | longtext                                               | NO   |     | NULL    |       |
| OWNER_THREAD_ID             | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| OWNER_EVENT_ID              | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| OWNER_OBJECT_TYPE           | enum('EVENT','FUNCTION','PROCEDURE','TABLE','TRIGGER') | YES  |     | NULL    |       |
| OWNER_OBJECT_SCHEMA         | varchar(64)                                            | YES  |     | NULL    |       |
| OWNER_OBJECT_NAME           | varchar(64)                                            | YES  |     | NULL    |       |
| TIMER_PREPARE               | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| COUNT_REPREPARE             | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| COUNT_EXECUTE               | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_TIMER_EXECUTE           | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| MIN_TIMER_EXECUTE           | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| AVG_TIMER_EXECUTE           | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| MAX_TIMER_EXECUTE           | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_LOCK_TIME               | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_ERRORS                  | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_WARNINGS                | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_ROWS_AFFECTED           | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_ROWS_SENT               | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_ROWS_EXAMINED           | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_CREATED_TMP_DISK_TABLES | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_CREATED_TMP_TABLES      | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_SELECT_FULL_JOIN        | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_SELECT_FULL_RANGE_JOIN  | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_SELECT_RANGE            | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_SELECT_RANGE_CHECK      | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_SELECT_SCAN             | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_SORT_MERGE_PASSES       | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_SORT_RANGE              | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_SORT_ROWS               | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_SORT_SCAN               | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_NO_INDEX_USED           | bigint(20) unsigned                                    | NO   |     | NULL    |       |
| SUM_NO_GOOD_INDEX_USED      | bigint(20) unsigned                                    | NO   |     | NULL    |       |
+-----------------------------+--------------------------------------------------------+------+-----+---------+-------+
```
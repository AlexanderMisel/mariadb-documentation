# InnoDB File-Per-Table Tablespaces

When you create a table using the [InnoDB storage engine](/kb/en/xtradb-and-innodb/), data written to that table is stored on the file system in a data file called a tablespace.  Tablespace files contain both the data and indexes.

When <a undefined>innodb_file_per_table=ON</a> is set, InnoDB uses one tablespace file per InnoDB table. These tablespace files have the `.ibd` extension. When <a undefined>innodb_file_per_table=OFF</a> is set, InnoDB stores all tables in the [InnoDB system tablespace](/columns-storage-engines-and-plugins/storage-engines/innodb/innodb-tablespaces/innodb-system-tablespaces/).

InnoDB versions in MySQL 5.7 and above also support an additional type of tablespace called [general tablespaces](https://dev.mysql.com/doc/refman/5.7/en/general-tablespaces.html) that are created with <a undefined>CREATE TABLESPACE</a>. However, InnoDB versions in MariaDB Server do not currently support general tablespaces or [CREATE TABLESPACE](/sql-statements-structure/sql-statements/data-definition/create/create-tablespace/).

## File-Per-Table Tablespace Locations

By default, InnoDB's file-per-table tablespaces are created in the system's data directory, which is defined by the <a undefined>datadir</a> system variable.  If you want to store InnoDB data separate from the data for other storage engines, then you change this by setting the the <a undefined>innodb_data_home_dir</a> system variable.

In the event that you have a specific tablespace that you need stored in a dedicated path, you can set the location using the <a undefined>DATA DIRECTORY</a> table option when you create the table.

For instance,

```sql
CREATE TABLE test.t1 (
   id INT PRIMARY KEY AUTO_INCREMENT,
   name VARCHAR(50)
) ENGINE=InnoDB
DATA DIRECTORY = "/data/contact";
```

MariaDB then creates a database directory on the configured path and the file-per-table tablespace will be created inside that directory.  On Unix-like operating systems, you can see the file using the ls command:

```sql
# ls -al /data/contact/test
drwxrwx--- 2 mysql mysql  4096 Dec 8 18:46 .
drwxr-xr-x 3 mysql mysql  4096 Dec 8 18:46 ..
-rw-rw---- 1 mysql mysql 98304 Dec 8 20:41 t1.ibd
```

Note, the system user that runs the MariaDB Server process (which is usually `mysql`) must have write permissions on the given path.

## Copying Transportable Tablespaces

InnoDB's file-per-table tablespaces are transportable, which means that you can copy a file-per-table tablespace from one MariaDB Server to another server.  You may find this useful in cases where you need to transport full tables between servers and don't want to use backup tools like [mariabackup](/mariadb-administration/backing-up-and-restoring-databases/mariabackup/) or [mysqldump](/clients-utilities/backup-restore-and-import-clients/mysqldump/). In fact, this process can even be used with [mariabackup](/mariadb-administration/backing-up-and-restoring-databases/mariabackup/) in some cases, such as when [restoring partial backups](/mariadb-administration/backing-up-and-restoring-databases/mariabackup/partial-backup-and-restore-with-mariabackup/) or when [restoring individual tables or partitions from a backup](/mariadb-administration/backing-up-and-restoring-databases/mariabackup/restoring-individual-tables-and-partitions-with-mariabackup/).

### Copying Transportable Tablespaces for Non-partitioned Tables

You can copy the transportable tablespace of a non-partitioned table from one server to another by exporting the tablespace file from the original server, and then importing the tablespace file into the new server.

#### Exporting Transportable Tablespaces for Non-partitioned Tables

You can export a non-partitioned table by locking the table and copying the table's `.ibd` and `.cfg` files from the relevant [tablespace location](#file-per-table-tablespace-locations) for the table to a backup location. For example, the process would go like this:

- First, use the [FLUSH TABLES ... FOR EXPORT](/sql-statements-structure/sql-statements/administrative-sql-statements/flush-commands/flush-tables-for-export/) statement on the target table:

```sql
FLUSH TABLES test.t1 FOR EXPORT;
```

This forces the server to close the table and provides your connection with a read lock on the table.

- Then, while your connection still holds the lock on the table, copy the tablespace file and the metadata file to a safe directory:

```sql
# cp /data/contacts/test/t1.ibd /data/saved-tablespaces/
# cp /data/contacts/test/t1.cfg /data/saved-tablespaces/
```

- Then, once you've copied the files, you can release the lock with <a undefined>UNLOCK TABLES</a>:

```sql
UNLOCK TABLES;
```

#### Importing Transportable Tablespaces for Non-partitioned Tables

You can import a non-partitioned table by discarding the table's original tablespace, copying the table's `.ibd` and `.cfg` files from the backup location to the relevant [tablespace location](#file-per-table-tablespace-locations) for the table, and then telling the server to import the tablespace. For example, the process would go like this:

- First, on the destination server, you need to create a copy of the table. Use the same [CREATE TABLE](/sql-statements-structure/sql-statements/data-definition/create/create-table/) statement that was used to create the table on the original server:

```sql
CREATE TABLE test.t1 (
   id INT PRIMARY KEY AUTO_INCREMENT,
   name VARCHAR(50)
) ENGINE=InnoDB;
```

- Then, use <a undefined>ALTER TABLE ... DISCARD TABLESPACE</a> to discard the new table's tablespace:

```sql
ALTER TABLE test.t1 DISCARD TABLESPACE;
```

- Then, copy the `.ibd` and `.cfg` files from the original server to the relevant directory on the target MariaDB Server:

```sql
# scp /data/tablespaces/t1.ibd target-server.com:/var/lib/mysql/test/
# scp /data/tablespaces/t1.cfg target-server.com:/var/lib/mysql/test/
```

File-per-table tablespaces can be imported with just the `.ibd` file in many cases. If you do not have the tablespace's `.cfg` file for whatever reason, then it is usually worth trying to import the tablespace with just the `.ibd` file.

- Then, once the files are in the proper directory on the target server, use <a undefined>ALTER TABLE ... IMPORT TABLESPACE</a> to import the new table's tablespace:

```sql
ALTER TABLE test.t1 IMPORT TABLESPACE;
```

### Copying Transportable Tablespaces for Partitioned Tables

Currently, MariaDB does not directly support the transport of tablespaces from partitioned tables. See [MDEV-10568](https://jira.mariadb.org/browse/MDEV-10568) for more information about that. It is still possible to transport partitioned tables if we use a workaround. You can copy the transportable tablespaces of a partitioned table from one server to another by exporting the tablespace file of each partition from the original server, and then importing the tablespace file of each partition into the new server.

#### Exporting Transportable Tablespaces for Partitioned Tables

You can export a partitioned table by locking the table and copying the `.ibd` and `.cfg` files of each partition from the relevant [tablespace location](#file-per-table-tablespace-locations) for the partition to a backup location. For example, the process would go like this:

- First, let's create a test table with some data on the original server:

```sql
CREATE TABLE test.t2 (
   employee_id INT,
   name VARCHAR(50),
) ENGINE=InnoDB
PARTITION BY RANGE (employee_id) (
   PARTITION p0 VALUES LESS THAN (6),
   PARTITION p1 VALUES LESS THAN (11),
   PARTITION p2 VALUES LESS THAN (16),
   PARTITION p3 VALUES LESS THAN MAXVALUE
);

INSERT INTO test.t2 (name, employee_id) VALUES
   ('Geoff Montee', 1), 
   ('Chris Calendar', 6),
   ('Kyle Joiner', 11), 
   ('Will Fong', 16);
```

- Then, we need to export the partitioned tablespace from the original server, which follows the same process as exporting non-partitioned tablespaces. That means that we need to use the [FLUSH TABLES ... FOR EXPORT](/sql-statements-structure/sql-statements/administrative-sql-statements/flush-commands/flush-tables-for-export/) statement on the target table:

```sql
FLUSH TABLES test.t2 FOR EXPORT;
```

This forces the server to close the table and provides your connection with a read lock on the table.

- Then, if we grep the database directory in the data directory for the newly created `t2` table, we can see a number of `.ibd` and `.cfg` files for the table:

```sql
# ls -l /var/lib/mysql/test/ | grep t2
total 428
-rw-rw---- 1 mysql mysql 827 Dec 5 16:08 t2.frm
-rw-rw---- 1 mysql mysql 48 Dec 5 16:08 t2.par
-rw-rw---- 1 mysql mysql 579 Dec 5 18:47 t2#P#p0.cfg
-rw-r----- 1 mysql mysql 98304 Dec 5 16:43 t2#P#p0.ibd
-rw-rw---- 1 mysql mysql 579 Dec 5 18:47 t2#P#p1.cfg
-rw-rw---- 1 mysql mysql 98304 Dec 5 16:08 t2#P#p1.ibd
-rw-rw---- 1 mysql mysql 579 Dec 5 18:47 t2#P#p2.cfg
-rw-rw---- 1 mysql mysql 98304 Dec 5 16:08 t2#P#p2.ibd
-rw-rw---- 1 mysql mysql 579 Dec 5 18:47 t2#P#p3.cfg
-rw-rw---- 1 mysql mysql 98304 Dec 5 16:08 t2#P#p3.ibd
```

- Then, while our connection still holds the lock on the table, we need to copy the tablespace files and the metadata files to a safe directory:

```sql
$ mkdir /tmp/backup
$ sudo cp /var/lib/mysql/test/*.ibd /tmp/backup
$ sudo cp /var/lib/mysql/test/*.cfg /tmp/backup
```

- Then, once we've copied the files, we can release the lock with <a undefined>UNLOCK TABLES</a>:

```sql
UNLOCK TABLES;
```

#### Importing Transportable Tablespaces for Partitioned Tables

You can import a partitioned table by creating a placeholder table, discarding the placeholder table's original tablespace, copying the partition's `.ibd` and `.cfg` files from the backup location to the relevant [tablespace location](#file-per-table-tablespace-locations) for the placeholder table, and then telling the server to import the tablespace. At that point, the server can exchange the tablespace for the placeholder table with the one for the partition. For example, the process would go like this:

- First, we need to copy the saved tablespace files from the original server to the target server:

```sql
$ scp /tmp/backup/t2* user@target-host:/tmp/backup
```

- Then, we need to import the partitioned tablespaces onto the target server. The import process for partitioned tables is more complicated than the import process for non-partitioned tables.  To start with, if it doesn't already exist, then we need to create a partitioned table on the target server that matches the partitioned table on the original server:

```sql
CREATE TABLE test.t2 (
   id INT PRIMARY KEY AUTO_INCREMENT,
   name VARCHAR(50),
   employee_id INT
) ENGINE=InnoDB
PARTITION BY RANGE (employee_id) (
   PARTITION p0 VALUES LESS THAN (6),
   PARTITION p1 VALUES LESS THAN (11),
   PARTITION p2 VALUES LESS THAN (16),
   PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

- Then, using this table as a model, we need to create a placeholder of this table with the same structure that does not use partitioning. This can be done with a <a undefined>CREATE TABLE... AS SELECT</a> statement:

```sql
CREATE TABLE test.t2_placeholder AS
   SELECT * FROM test.t2 WHERE NULL;
```

This statement will create a new table called `t2_placeholder` that has the same schema structure as `t2`, but it does not use partitioning and it contains no rows.

##### For Each Partition

From this point forward, the rest of our steps need to happen for each individual partition. For each partition, we need to do the following process:

- First, we need to use <a undefined>ALTER TABLE ... DISCARD TABLESPACE</a> to discard the placeholder table's tablespace:

```sql
ALTER TABLE test.t2_placeholder DISCARD TABLESPACE;
```

- Then, copy the `.ibd` and `.cfg` files for the next partition to the relevant directory for the `t2_placeholder` table on the target MariaDB Server:

```sql
# cp /tmp/backup/t2#P#p0.cfg /var/lib/mysql/test/t2_placeholder.cfg
# cp /tmp/backup/t2#P#p0.ibd /var/lib/mysql/test/t2_placeholder.ibd
# chown mysql:mysql /var/lib/mysql/test/t2_placeholder*
```

File-per-table tablespaces can be imported with just the `.ibd` file in many cases. If you do not have the tablepace's `.cfg` file for whatever reason, then it is usually worth trying to import the tablespace with just the `.ibd` file.

- Then, once the files are in the proper directory on the target server, we need to use <a undefined>ALTER TABLE ... IMPORT TABLESPACE</a> to import the new table's tablespace:

```sql
ALTER TABLE test.t2_placeholder IMPORT TABLESPACE;
```

The placeholder table now contains data from the `p0` partition on the source server.

```sql
SELECT * FROM test.t2_placeholder;

+-------------+--------------+
| employee_id | name         |
+-------------+--------------+
|           1 | Geoff Montee |
+-------------+--------------+
```

- Then, it's time to transfer the partition from the placeholder to the target table.  This can be done with an <a undefined>ALTER TABLE... EXCHANGE PARTITION</a> statement:

```sql
ALTER TABLE test.t2 EXCHANGE PARTITION p0 WITH TABLE test.t2_placeholder;
```

The target table now contains the first partition from the source table.

```sql
SELECT * FROM test.t2;

+-------------+--------------+
| employee_id | name         |
+-------------+--------------+
|           1 | Geoff Montee |
+-------------+--------------+
```

- Repeat this procedure for each partition you want to import. For each partition, we need to discard the placeholder table's tablespace, and then import the partitioned table's tablespace into the placeholder table, and then exchange the tablespaces between the placeholder table and the partition of our target table.

When this process is complete for all partitions, the target table will contain the imported data:

```sql
SELECT * FROM test.t2;

+-------------+----------------+
| employee_id | name           |
+-------------+----------------+
|           1 | Geoff Montee   |
|           6 | Chris Calendar |
|          11 | Kyle Joiner    |
|          16 | Will Fong      |
+-------------+----------------+
```

- Then, we can remove the placeholder table from the database:

```sql
DROP TABLE test.t2_placeholder;
```

### Known Problems with Copying Transportable Tablespaces

#### Differing Storage Formats for Temporal Columns

[MariaDB 10.1.2](/kb/en/mariadb-1012-release-notes/) added the <a undefined>mysql56_temporal_format</a> system variable, which enables a new MySQL 5.6-compatible storage format for the [TIME](/columns-storage-engines-and-plugins/data-types/date-and-time-data-types/time/), [DATETIME](/columns-storage-engines-and-plugins/data-types/date-and-time-data-types/datetime/) and [TIMESTAMP](/columns-storage-engines-and-plugins/data-types/date-and-time-data-types/timestamp/) data types.

If a file-per-tablespace file contains columns that use one or more of these temporal data types and if the tablespace file's original table was created with a certain storage format for these columns, then the tablespace file can only be imported into tables that were also created with the same storage format for these columns as the original table. Otherwise, you will see errors like the following:

```sql
ALTER TABLE dt_test IMPORT TABLESPACE;
ERROR 1808 (HY000): Schema mismatch (Column dt precise type mismatch.)
```

See [MDEV-15225](https://jira.mariadb.org/browse/MDEV-15225) for more information.

See the pages for the [TIME](/columns-storage-engines-and-plugins/data-types/date-and-time-data-types/time/), [DATETIME](/columns-storage-engines-and-plugins/data-types/date-and-time-data-types/datetime/) and [TIMESTAMP](/columns-storage-engines-and-plugins/data-types/date-and-time-data-types/timestamp/) data types to determine how to update the storage format for temporal columns in tables that were created before [MariaDB 10.1.2](/kb/en/mariadb-1012-release-notes/) or that were created with <a undefined>mysql56_temporal_format=OFF</a>.

#### Differing ROW_FORMAT Values

InnoDB file-per-table tablespaces can use different [row formats](/kb/en/xtradbinnodb-storage-formats/). A specific row format can be specified when creating a table either by setting the <a undefined>ROW_FORMAT</a> table option or by the setting the <a undefined>innodb_default_row_format</a> system variable. See [Setting a Table's Row Format](/kb/en/innodb-storage-formats/#setting-a-tables-row-format) for more information on how to set an InnoDB table's row format.

If a file-per-tablespace file was created with a certain row format, then the tablespace file can only be imported into tables that were created with the same row format as the original table. Otherwise, you will see errors like the following:

```sql
ALTER TABLE t0 IMPORT TABLESPACE;
ERROR 1808 (HY000): Schema mismatch (Expected FSP_SPACE_FLAGS=0x21, .ibd file contains 0x0.)
```

The error message will be a bit more descriptive in [MariaDB 10.2.17](/kb/en/mariadb-10217-release-notes/) and later:

```sql
ALTER TABLE t0 IMPORT TABLESPACE;
ERROR 1808 (HY000): Schema mismatch (Table flags don't match, server table has 0x1 and the meta-data file has 0x0; .cfg file uses ROW_FORMAT=REDUNDANT)
```

Be sure to check a tablespace's row format before moving it from one server to another. Keep in mind that the default row format can change between major versions of MySQL or MariaDB. See [Checking a Table's Row Format](/kb/en/innodb-storage-formats/#checking-a-tables-row-format) for information on how to check an InnoDB table's row format.

See [MDEV-15049](https://jira.mariadb.org/browse/MDEV-15049) and [MDEV-16851](https://jira.mariadb.org/browse/MDEV-16851) for more information.

#### Foreign Key Constraints

DISCARD on a table with foreign key constraints is only possible after disabling [foreign_key_checks](/kb/en/server-system-variables/#foreign_key_checks):

```sql
SET SESSION foreign_key_checks=0;
ALTER TABLE t0 DISCARD TABLESPACE;
```

IMPORT on the other hand does not enforce foreign key constraints. So when importing tablespaces, referential integrity can only be guaranteed to import all tables bound by foreign key constraint at the same time, from an EXPORT of those tables taken with the same transactional state.

## Tablespace Encryption

MariaDB supports data-at-rest encryption for the InnoDB storage engine.  When enabled, the Server encrypts data before writing it to the tablespace and decrypts reads from the tablespace before returning result-sets.  This means that a malicious user attempting to exfiltrate sensitive data won't be able to import the tablespace onto a different server as shown above without the encryption key.

For more information on data encryption, see [Encrypting Data for InnoDB](/kb/en/encrypting-data-for-innodb-xtradb/).

## See Also

- [Geoff Montee:Importing InnoDB Partitions in MySQL 5.6 and MariaDB 10.0/10.1](http://www.geoffmontee.com/importing-innodb-partitions-in-mysql-5-6-and-mariadb-10-010-1/)
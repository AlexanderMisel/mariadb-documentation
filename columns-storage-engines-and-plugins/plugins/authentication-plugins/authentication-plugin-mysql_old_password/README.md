# Authentication Plugin - mysql_old_password

The `mysql_old_password` authentication plugin is the default authentication plugin that will be used for an account created when no authentication plugin is explicitly mentioned and <a undefined>old_passwords=1</a> is set. It uses the pre-MySQL 4.1 password hashing algorithm, which is also used by the [OLD_PASSWORD()](/built-in-functions/secondary-functions/encryption-hashing-and-compression-functions/old_password/) function and by the [PASSWORD()](/built-in-functions/secondary-functions/encryption-hashing-and-compression-functions/password/) function when <a undefined>old_passwords=1</a> is set.

It is not recommended to use the `mysql_old_password` authentication plugin for new installations. The password hashing algorithm is no longer as secure as it used to be, and the plugin is primarily provided for backward-compatibility. The [ed25519](/columns-storage-engines-and-plugins/plugins/authentication-plugins/authentication-plugin-ed25519/) authentication plugin is a more modern authentication plugin that provides simple password authentication.

## Installing the Plugin

The `mysql_old_password` authentication plugin is statically linked into the server, so no installation is necessary.

## Creating Users

The easiest way to create a user account with the `mysql_old_password` authentication plugin is to make sure that <a undefined>old_passwords=1</a> is set, and then create a user account via [CREATE USER](/sql-statements-structure/sql-statements/account-management-sql-commands/create-user/) that does not specify an authentication plugin, but does specify a password via the <a undefined>IDENTIFIED BY</a> clause. For example:

```sql
SET old_passwords=1;
CREATE USER username@hostname IDENTIFIED BY 'mariadb';
```

If [SQL_MODE](/mariadb-administration/variables-and-modes/sql-mode/) does not have `NO_AUTO_CREATE_USER` set, then you can also create the user via [GRANT](/sql-statements-structure/sql-statements/account-management-sql-commands/grant/). For example:

```sql
SET old_passwords=1;
GRANT SELECT ON db.* TO username@hostname IDENTIFIED BY 'mariadb';
```

You can also create the user account by providing a password hash via the <a undefined>IDENTIFIED BY PASSWORD</a> clause, and MariaDB will validate whether the password hash is one that is compatible with `mysql_old_password`. For example:

```sql
SET old_passwords=1;
Query OK, 0 rows affected (0.000 sec)

SELECT PASSWORD('mariadb');
+---------------------+
| PASSWORD('mariadb') |
+---------------------+
| 021bec665bf663f1    |
+---------------------+
1 row in set (0.000 sec)

CREATE USER username@hostname IDENTIFIED BY PASSWORD '021bec665bf663f1';
Query OK, 0 rows affected (0.000 sec)
```

Similar to all other [authentication plugins](/columns-storage-engines-and-plugins/plugins/authentication-plugins/), you could also specify the name of the plugin in the <a undefined>IDENTIFIED VIA</a> clause while providing the password hash as the `USING` clause. For example:

```sql
CREATE USER username@hostname IDENTIFIED VIA mysql_old_password USING '021bec665bf663f1';
Query OK, 0 rows affected (0.000 sec)
```

## Changing User Passwords

You can change a user account's password with the [SET PASSWORD](/sql-statements-structure/sql-statements/account-management-sql-commands/set-password/) statement while providing the plain-text password as an argument to the [PASSWORD()](/built-in-functions/secondary-functions/encryption-hashing-and-compression-functions/password/) function. For example:

```sql
SET PASSWORD =  PASSWORD('new_secret')
```

You can also change the user account's password with the [ALTER USER](/sql-statements-structure/sql-statements/account-management-sql-commands/alter-user/) statement. You would have to make sure that <a undefined>old_passwords=1</a> is set, and then you would have to specify a password via the <a undefined>IDENTIFIED BY</a> clause. For example:

```sql
SET old_passwords=1;
ALTER USER username@hostname IDENTIFIED BY 'new_secret';
```

## Client Authentication Plugins

For clients that use the `libmysqlclient` or [MariaDB Connector/C](/kb/en/mariadb-connector-c/) libraries, MariaDB provides one client authentication plugin that is compatible with the `mysql_old_password` authentication plugin:

- `mysql_old_password`

When connecting with a [client or utility](/clients-utilities/) to a server as a user account that authenticates with the `mysql_old_password` authentication plugin, you may need to tell the client where to find the relevant client authentication plugin by specifying the `--plugin-dir` option. For example:

```sql
mysql --plugin-dir=/usr/local/mysql/lib64/mysql/plugin --user=alice
```

However, the `mysql_old_password` client authentication plugin is generally statically linked into client libraries like `libmysqlclient` or [MariaDB Connector/C](/kb/en/mariadb-connector-c/), so this is not usually necessary.

### `mysql_old_password`

The `mysql_old_password` client authentication plugin hashes the password before sending it to the server.

## Support in Client Libraries

The `mysql_old_password` authentication plugin is one of the conventional authentication plugins, so all client libraries should support it.
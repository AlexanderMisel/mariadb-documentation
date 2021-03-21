# MariaDB Audit Plugin - Installation

The `server_audit` plugin logs the server's activity. For each client session, it records who connected to the server (i.e., user name and host), what queries were executed, and which tables were accessed and server variables that were changed. This information is stored in a rotating log file or it may be sent to the local syslogd.

## Locating the Plugin

The `server_audit` plugin's shared library is included in MariaDB packages as the `server_audit.so` or `server_audit.dll` shared library on systems where it can be built.

The plugin must be located in the plugin directory, the directory containing all plugin libraries for MariaDB. The path to this directory is configured by the <a undefined>plugin_dir</a> system variable. To see the value of this variable and thereby determine the file path of the plugin library, execute the following SQL statement:

```sql
SHOW GLOBAL VARIABLES LIKE 'plugin_dir';

+---------------+--------------------------+
| Variable_name | Value                    |
+---------------+--------------------------+
| plugin_dir    | /usr/lib64/mysql/plugin/ |
+---------------+--------------------------+
```

Check the directory returned at the filesystem level to make sure that you have a copy of the plugin library, `server_audit.so` or `server_audit.dll`, depending on your system.  It's included in recent installations of MariaDB. If you do not have it, you should upgrade MariaDB.

## Installing the Plugin

Although the plugin's shared library is distributed with MariaDB by default, the plugin is not actually installed by MariaDB by default. There are two methods that can be used to install the plugin with MariaDB.

The first method can be used to install the plugin without restarting the server. You can install the plugin dynamically by executing [INSTALL SONAME](/sql-statements-structure/sql-statements/administrative-sql-statements/plugin-sql-statements/install-soname) or [INSTALL PLUGIN](/sql-statements-structure/sql-statements/administrative-sql-statements/plugin-sql-statements/install-plugin). For example:

```sql
INSTALL SONAME 'server_audit';
```

The second method can be used to tell the server to load the plugin when it starts up. The plugin can be installed this way by providing the <a undefined>--plugin-load</a> or the <a undefined>--plugin-load-add</a> options. This can be specified as a command-line argument to [mysqld](/mariadb-administration/getting-installing-and-upgrading-mariadb/starting-and-stopping-mariadb/mysqld-options) or it can be specified in a relevant server [option group](/kb/en/configuring-mariadb-with-option-files/#option-groups) in an [option file](/mariadb-administration/getting-installing-and-upgrading-mariadb/configuring-mariadb-with-option-files). For example:

```sql
[mariadb]
...
plugin_load_add = server_audit
```

## Uninstalling the Plugin

You can uninstall the plugin dynamically by executing [UNINSTALL SONAME](/sql-statements-structure/sql-statements/administrative-sql-statements/plugin-sql-statements/uninstall-soname) or [UNINSTALL PLUGIN](/sql-statements-structure/sql-statements/administrative-sql-statements/plugin-sql-statements/uninstall-plugin). For example:

```sql
UNINSTALL SONAME 'server_audit';
```

If you installed the plugin by providing the <a undefined>--plugin-load</a> or the <a undefined>--plugin-load-add</a> options in a relevant server [option group](/kb/en/configuring-mariadb-with-option-files/#option-groups) in an [option file](/mariadb-administration/getting-installing-and-upgrading-mariadb/configuring-mariadb-with-option-files), then those options should be removed to prevent the plugin from being loaded the next time the server is restarted.

## Prohibiting Uninstallation

The [UNINSTALL SONAME](/sql-statements-structure/sql-statements/administrative-sql-statements/plugin-sql-statements/uninstall-soname) or [UNINSTALL PLUGIN](/sql-statements-structure/sql-statements/administrative-sql-statements/plugin-sql-statements/uninstall-plugin) statements may be used to uninstall plugins. For the `server_audit` plugin, you might want to disable this capability. To prevent the plugin from being uninstalled, you could set the the <a undefined>server_audit</a> option to `FORCE_PLUS_PERMANENT` in a relevant server [option group](/kb/en/configuring-mariadb-with-option-files/#option-groups) in an [option file](/mariadb-administration/getting-installing-and-upgrading-mariadb/configuring-mariadb-with-option-files) after the plugin is loaded once:

```sql
[mariadb]
...
plugin_load_add = server_audit
server_audit=FORCE_PLUS_PERMANENT
```

Once you've added the option to the server's option file and restarted the server, the plugin can't be uninstalled. If someone tries to uninstall the audit plugin, then an error message will be returned. Below is an example of the error message:

```sql
UNINSTALL PLUGIN server_audit;

ERROR 1702 (HY000):
Plugin 'server_audit' is force_plus_permanent and can not be unloaded
```

For more information on `FORCE_PLUS_PERMANENT` and other option values for the <a undefined>server_audit</a> option, see [Plugin Overview: Configuring Plugin Activation at Server Startup](/kb/en/plugin-overview/#configuring-plugin-activation-at-server-startup) for more information.
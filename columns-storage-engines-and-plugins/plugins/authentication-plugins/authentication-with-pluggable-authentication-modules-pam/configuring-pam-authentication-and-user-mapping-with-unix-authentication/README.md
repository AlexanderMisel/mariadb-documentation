# Configuring PAM Authentication and User Mapping with Unix Authentication

In this article, we will walk through the configuration of PAM authentication using the [pam](/columns-storage-engines-and-plugins/plugins/authentication-plugins/authentication-with-pluggable-authentication-modules-pam/authentication-plugin-pam) authentication plugin and user and group mapping with the [pam_user_map](/columns-storage-engines-and-plugins/plugins/authentication-plugins/authentication-with-pluggable-authentication-modules-pam/user-and-group-mapping-with-pam) PAM module. The primary authentication will be handled by the <a undefined>pam_unix</a> PAM module, which performs standard Unix password authentication.

## Hypothetical Requirements

In this walkthrough, we are going to assume the following hypothetical requirements:

- The Unix user `foo` should be mapped to the MariaDB user `bar`. (`foo: bar`)
- Any Unix user in the Unix group `dba` should be mapped to the MariaDB user `dba`. (`@dba: dba`)

## Creating Our Unix Users and Groups

Let's go ahead and create the Unix users and groups that we are using for this hypothetical scenario.

First, let's create the the `foo` user and a couple users to go into the `dba` group. Note that each of these users needs a password.

```sql
sudo useradd foo
sudo passwd foo
sudo useradd alice
sudo passwd alice
sudo useradd bob
sudo passwd bob
```

And then let's create our `dba` group and add our two users to it:

```sql
sudo groupadd dba
sudo usermod -a -G dba alice 
sudo usermod -a -G dba bob 
```

We also need to create Unix users with the same name as the `bar` and `dba` MariaDB users. See [here](/kb/en/user-and-group-mapping-with-pam/#pam-user-with-same-name-as-mapped-mariadb-user-must-exist) to read more about why. No one will be logging in as these users, so they do not need passwords.

```sql
sudo useradd bar
sudo useradd dba -g dba
```

## Installing the pam_user_map PAM Module

Next, let's [install the pam_user_map PAM module](/kb/en/user-and-group-mapping-with-pam/#installing-the-pam_user_map-pam-module).

Before the module can be compiled from source, we may need to install some dependencies.

On RHEL, CentOS, and other similar Linux distributions that use [RPM packages](/mariadb-administration/getting-installing-and-upgrading-mariadb/binary-packages/rpm), we need to install `gcc` and `pam-devel`:

```sql
sudo yum install gcc pam-devel
```

On Debian, Ubuntu, and other similar Linux distributions that use [DEB packages](/mariadb-administration/getting-installing-and-upgrading-mariadb/binary-packages/installing-mariadb-deb-files), we need to install `gcc` and `libpam0g-dev`:

```sql
sudo apt-get install gcc libpam0g-dev
```

And then we can build and install the library with the following:

```sql
wget https://raw.githubusercontent.com/MariaDB/server/10.4/plugin/auth_pam/mapper/pam_user_map.c 
gcc pam_user_map.c -shared -lpam -fPIC -o pam_user_map.so 
sudo install --mode=0755 pam_user_map.so /lib64/security/ 
```

## Configuring the pam_user_map PAM Module

Next, let's [configure the pam_user_map PAM module](/kb/en/user-and-group-mapping-with-pam/#configuring-the-pam_user_map-pam-module) based on our hypothetical requirements.

The configuration file for the `pam_user_map` PAM module is `/etc/security/user_map.conf`. Based on our hypothetical requirements, ours would look like:

```sql
foo: bar
@dba:dba
```

## Installing the PAM Authentication Plugin

Next, let's [install the `pam` authentication plugin](/kb/en/authentication-plugin-pam/#installing-the-plugin).

Log into the MariaDB Server and execute the following:

```sql
INSTALL SONAME 'auth_pam';
```

## Configuring the PAM Service

Next, let's [configure the PAM service](/kb/en/authentication-plugin-pam/#configuring-the-pam-service). We will call our service `mariadb`, so our PAM service configuration file will be located at `/etc/pam.d/mariadb` on most systems.

Since we are only doing Unix authentication with the `pam_unix` PAM module and group mapping with the `pam_user_map` PAM module, our configuration file would look like this:

```sql
auth required pam_unix.so audit
auth required pam_user_map.so
account required pam_unix.so audit
```

## Configuring the pam_unix PAM Module

The `pam_unix` PAM module adds [some additional configuration steps](/kb/en/authentication-plugin-pam/#configuring-the-pam-service) on a lot of systems. We basically have to give the user that runs `mysqld` access to `/etc/shadow`.

If the `mysql` user is running `mysqld`, then we can do that by executing the following:

```sql
sudo groupadd shadow
sudo usermod -a -G shadow mysql
sudo chown root:shadow /etc/shadow
sudo chmod g+r /etc/shadow
```

The [server needs to be restarted](/kb/en/starting-and-stopping-mariadb-starting-and-stopping-mariadb/) for this change to take affect.

## Creating MariaDB Users

Next, let's [create the MariaDB users](/kb/en/authentication-plugin-pam/#creating-users). Remember that our PAM service is called `mariadb`.

First, let's create the MariaDB user for the user mapping: `foo: bar`

That means that we need to create a `bar` user:

```sql
CREATE USER 'bar'@'%' IDENTIFIED BY 'strongpassword';
GRANT ALL PRIVILEGES ON *.* TO 'bar'@'%' ;
```

And then let's create the MariaDB user for the group mapping: `@dba: dba`

That means that we need to create a `dba` user:

```sql
CREATE USER 'dba'@'%' IDENTIFIED BY 'strongpassword';
GRANT ALL PRIVILEGES ON *.* TO 'dba'@'%' ;
```

And then to allow for the user and group mapping, we need to [create an anonymous user that authenticates with the `pam` authentication plugin](/kb/en/user-and-group-mapping-with-pam/#creating-users) that is also able to `PROXY` as the `bar` and `dba` users. Before we can create the proxy user, we might need to [clean up some defaults](/kb/en/create-user/#fixing-a-legacy-default-anonymous-account):

```sql
DELETE FROM mysql.db WHERE User='' AND Host='%';
FLUSH PRIVILEGES;
```

And then let's create the anonymous proxy user:

```sql
CREATE USER ''@'%' IDENTIFIED VIA pam USING 'mariadb';
GRANT PROXY ON 'bar'@'%' TO ''@'%';
GRANT PROXY ON 'dba'@'%' TO ''@'%';
```

## Testing our Configuration

Next, let's test out our configuration by [verifying that mapping is occurring](/kb/en/user-and-group-mapping-with-pam/#verifying-that-mapping-is-occurring). We can verify this by logging in as each of our users and comparing the return value of [USER()](/built-in-functions/secondary-functions/information-functions/user), which is the original user name and the return value of [CURRENT_USER()](/built-in-functions/secondary-functions/information-functions/current_user), which is the authenticated user name.

First, let's test out our `foo` user:

```sql
$ mysql -u foo -h 172.30.0.198
[mariadb] Password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 22
Server version: 10.3.10-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SELECT USER(), CURRENT_USER();
+------------------------------------------------+----------------+
| USER()                                         | CURRENT_USER() |
+------------------------------------------------+----------------+
| foo@ip-172-30-0-198.us-west-2.compute.internal | bar@%          |
+------------------------------------------------+----------------+
1 row in set (0.000 sec)
```

We can verify that our `foo` Unix user was properly mapped to the `bar` MariaDB user by looking at the return value of [CURRENT_USER()](/built-in-functions/secondary-functions/information-functions/current_user).

Then let's test out our `alice` user in the `dba` group:

```sql
$ mysql -u alice -h 172.30.0.198
[mariadb] Password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 19
Server version: 10.3.10-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SELECT USER(), CURRENT_USER();
+--------------------------------------------------+----------------+
| USER()                                           | CURRENT_USER() |
+--------------------------------------------------+----------------+
| alice@ip-172-30-0-198.us-west-2.compute.internal | dba@%          |
+--------------------------------------------------+----------------+
1 row in set (0.000 sec)
```

And then let's test out our `bob` user in the `dba` group:

```sql
$ mysql -u bob -h 172.30.0.198
[mariadb] Password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 20
Server version: 10.3.10-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SELECT USER(), CURRENT_USER();
+------------------------------------------------+----------------+
| USER()                                         | CURRENT_USER() |
+------------------------------------------------+----------------+
| bob@ip-172-30-0-198.us-west-2.compute.internal | dba@%          |
+------------------------------------------------+----------------+
1 row in set (0.000 sec)
```

We can verify that our `alice` and `bob` Unix users in the `dba` Unix group were properly mapped to the `dba` MariaDB user by looking at the return values of [CURRENT_USER()](/built-in-functions/secondary-functions/information-functions/current_user).
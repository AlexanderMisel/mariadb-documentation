# Amazon Web Services (AWS) Key Management Service (KMS) Encryption Plugin Setup Guide

## Overview

MariaDB Server 10.1 introduces robust, full instance, at-rest encryption. This feature uses a flexible plugin interface to allow actual encryption to be done using a key management approach that meets the customer's needs. MariaDB Server, starting with [MariaDB 10.2](/kb/en/what-is-mariadb-102/), includes a plugin that uses the Amazon Web Services (AWS) Key Management Service (KMS) to facilitate separation of responsibilities and remote logging &amp; auditing of key access requests.

Rather than storing the encryption key in a local file, this plugin keeps the master key in AWS KMS. When you first start MariaDB, the AWS KMS plugin will connect to the AWS Key Management Service and ask it to generate a new key. MariaDB will store that key on-disk in an encrypted form. The key stored on-disk cannot be used to decrypt the data; rather, on each startup, MariaDB connects to AWS KMS and has the service decrypt the locally-stored key(s). The decrypted key is stored in-memory as long as the MariaDB server process is running, and that in-memory decrypted key is used to encrypt the local data.

This guide is based on CentOS 7, using systemd with SELinux enabled. Some steps will differ if you use other operating systems or configurations.

## Installing the Plugin's Package

The AWS Key Management plugin depends on the [AWS SDK for C++](https://github.com/aws/aws-sdk-cpp), which uses the [Apache License, Version 2.0](https://github.com/aws/aws-sdk-cpp/blob/master/LICENSE). This license is not compatible with MariaDB Server's [GPL 2.0 license](/kb/en/mariadb-license/), so we are not able to distribute packages that contain the AWS Key Management plugin. Therefore, the only way to currently obtain the plugin is to install it from source.

### Installing from Source

When [compiling MariaDB from source](/mariadb-administration/getting-installing-and-upgrading-mariadb/compiling-mariadb-from-source/), the AWS Key Management plugin is not built by default in [MariaDB 10.1](/kb/en/what-is-mariadb-101/), but it is built by default in [MariaDB 10.2](/kb/en/what-is-mariadb-102/) and later, on systems that support it.

Compilation is controlled by the `-DPLUGIN_AWS_KEY_MANAGEMENT=DYNAMIC -DAWS_SDK_EXTERNAL_PROJECT=1` <a undefined>cmake</a> arguments.

The plugin uses [AWS C++ SDK](https://github.com/awslabs/aws-sdk-cpp), which introduces the following restrictions:

- The plugin can only be built on Windows, Linux and macOS.
- The plugin requires that one of the following compilers is used: `gcc` 4.8 or later, `clang` 3.3 or later, Visual Studio 2013 or later.
- On Unix, the `libcurl` development package (e.g. `libcurl3-dev` on Debian Jessie), `uuid` development package and `openssl` need to be installed.
- You may need to use a newer version of <a undefined>cmake</a> than is provided by default in your OS.

## Installing the Plugin

Even after the package that contains the plugin's shared library is installed on the operating system, the plugin is not actually installed by MariaDB by default. There are two methods that can be used to install the plugin with MariaDB.

The first method can be used to install the plugin without restarting the server. You can install the plugin dynamically by executing [INSTALL SONAME](/sql-statements-structure/sql-statements/administrative-sql-statements/plugin-sql-statements/install-soname/) or [INSTALL PLUGIN](/sql-statements-structure/sql-statements/administrative-sql-statements/plugin-sql-statements/install-plugin/). For example:

```sql
INSTALL SONAME 'aws_key_management';
```

The second method can be used to tell the server to load the plugin when it starts up. The plugin can be installed this way by providing the <a undefined>--plugin-load</a> or the <a undefined>--plugin-load-add</a> options. This can be specified as a command-line argument to [mysqld](/mariadb-administration/getting-installing-and-upgrading-mariadb/starting-and-stopping-mariadb/mysqld-options/) or it can be specified in a relevant server [option group](/kb/en/configuring-mariadb-with-option-files/#option-groups) in an [option file](/mariadb-administration/getting-installing-and-upgrading-mariadb/configuring-mariadb-with-option-files/). For example:

```sql
[mariadb]
...
plugin_load_add = aws_key_management
```

## Sign up for Amazon Web Services

If you already have an AWS account, you can skip this section.

1 Load [http://aws.amazon.com/](http://aws.amazon.com/).
2 Click "Create a Free Account" and complete the steps.
3 You'll need to enter credit card information. Charges related only to your use of the AWS KMS service should be limited to about $1/month for the single master key we will create. If you use other services, additional charges may apply. Consult AWS Cloud Pricing Principles [https://aws.amazon.com/pricing/](https://aws.amazon.com/pricing/) for more information about pricing of AWS services.
4 You'll need to complete the AWS identify verification process.

## Create an IAM User and/or Role

After creating an account or logging in to an existing account, follow these steps to create an IAM User or Role with restricted privileges that will use (but not administer) your master encryption key.

If you intend to run MariaDB Server on an EC2 instance, you should create a Role (or modify an existing Role already attached to your instance). If you intent to run MariaDB Server outside of AWS, you may want to create a User.

### Creating an IAM Role

1 Load the Identity and Access Management Console at [https://console.aws.amazon.com/iam/](https://console.aws.amazon.com/iam/).
2 Click "Roles" in the left-hand sidebar
3 Click "Create new role"
4 Select "AWS Service Role"
5 Click the "Select" button next to "Amazon EC2 / Allows EC2 instances to call AWS services on your behalf."
6 Do not select any policies on the "Attach Policy" screen. Click "Next Step"
7 Click "Next Step"
8 Give your Role a "Role name"
9 Click "Create role"

### Creating an IAM User

1 Load the Identity and Access Management Console at [https://console.aws.amazon.com/iam/](https://console.aws.amazon.com/iam/).
2 Click "Users" in the left-hand sidebar.<br>
[<img src="/kb/en/key-management-and-encryption-plugins-amazon-web-services-aws-key-managemen/+image/aws-kms-create-new-user" style="max-height:300px;max-width:500px">](/kb/en/mariadb-enterprise-aws-kms-encryption-plugin-setup-guide/+image/aws-kms-create-new-user)
3 Click the "Create New Users" button
4 Enter a single User Name of your choosing. We'll use "MDBEnc" for this demonstration. Keep the "Generate an access key for each user" box checked.<br>
[<img src="/kb/en/key-management-and-encryption-plugins-amazon-web-services-aws-key-managemen/+image/aws-kms-enter-user-names" style="max-height:300px;max-width:500px">](/kb/en/mariadb-enterprise-aws-kms-encryption-plugin-setup-guide/+image/aws-kms-enter-user-names)
5 Click "Create".
6 Click "Show User Security Credentials".<br>
[<img src="/kb/en/key-management-and-encryption-plugins-amazon-web-services-aws-key-managemen/+image/aws-kms-show-security-credentials" style="max-height:300px;max-width:500px">](/kb/en/mariadb-enterprise-aws-kms-encryption-plugin-setup-guide/+image/aws-kms-show-security-credentials)
7 Copy the Access Key ID and Secret Access Key. Optionally, you can click "Download Credentials". We will need these in order for local programs to interact with AWS using its API.<br>
[<img src="/kb/en/key-management-and-encryption-plugins-amazon-web-services-aws-key-managemen/+image/aws-kms-copy-security-credentials" style="max-height:300px;max-width:500px">](/kb/en/mariadb-enterprise-aws-kms-encryption-plugin-setup-guide/+image/aws-kms-copy-security-credentials)
8 Create a file on your computer to hold the credentials for this user. We'll use this file later. It should have this structure: <pre class="fixed">[default]
aws_access_key_id = AKIAIG6IZ6TKF52FVV5A
aws_secret_access_key = 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
</pre>
9 Click "Close". If prompted because you did not Download Credentials, ensure that you've saved them somewhere, and click "Close".

## Create a master encryption key

Now, we'll create a master encryption key. This key can <em>never</em> be retrieved by <em>any</em> application or user. This key is used remotely to encrypt (and decrypt) the actual encryption keys that will be used by MariaDB. If this key is deleted or you lose access to it, you will be unable to use the contents of your MariaDB data directory.

1 Click "Encryption Keys" in the left-hand sidebar.
2 Click the "Get Started Now" button.
3 Use the "Filter" dropdown to choose the region where you'd like to create your master key.
4 Click the "Create Key" button.<br>
[<img src="/kb/en/key-management-and-encryption-plugins-amazon-web-services-aws-key-managemen/+image/aws-kms-create-key" style="max-height:300px;max-width:500px">](/kb/en/mariadb-enterprise-aws-kms-encryption-plugin-setup-guide/+image/aws-kms-create-key)
5 Enter an Alias and Description of your choosing.
6 Click "Next Step".
7 Do <strong>not</strong> check the box to make your IAM Role or IAM User user a Key Administrator.
8 Click "Next Step" again.
9 Check the boxes to give your IAM Role and/or IAM User permissions to use this key.<br>
[<img src="/kb/en/key-management-and-encryption-plugins-amazon-web-services-aws-key-managemen/+image/aws-kms-key-usage-permissions" style="max-height:300px;max-width:500px">](/kb/en/mariadb-enterprise-aws-kms-encryption-plugin-setup-guide/+image/aws-kms-key-usage-permissions)
10 Click "Next Step".
11 Click "Finish".

You should now see your key listed in the console:

[<img src="/kb/en/key-management-and-encryption-plugins-amazon-web-services-aws-key-managemen/+image/aws-kms-key-list-console" style="max-height:300px;max-width:500px">](/kb/en/mariadb-enterprise-aws-kms-encryption-plugin-setup-guide/+image/aws-kms-key-list-console)

You'll use the "Alias" you provided when you configure MariaDB later.

We now have a Customer Master Key and an IAM user that has privileges to access it using access credentials. This is enough to begin using the AWS KMS plugin.

## Configure AWS credentials

There are a number of ways to give the IAM credentials to the AWS KMS plugin. The plugin supports reading credentials from all standard locations used across the various AWS API clients.

The easiest approach is to run MariaDB Server in an EC2 instance that has an IAM Role with User access to the CMK you wish to use. You can give key access privileges to a Role already attached to your EC2 instance, or you can create a new IAM Role and attach it to an already-running EC2 instance. If you've done that, no further credentials management is required and you do not need to create a `credentials` file.

If you're not running MariaDB Server on an EC2 instance, you can also place the credentials in the MariaDB data directory. The AWS API client looks for a `credentials` file in the `.aws` subdirectory of the home directory of the user running the client process. In the case of MariaDB, its home directory is its `datadir`.

1 Create a `credentials` file that MariaDB can read. Use the region you selected when creating the key. Master keys cannot be used across regions. For example:<pre class="fixed">$ cat /var/lib/mysql/.aws/credentials
[default]
aws_access_key_id = AKIAIG6IZ6TKF52FVV5A
aws_secret_access_key = o7CEf7KhZfsVF9cS0a2roqqZNmuzXtIR869zpSBT
region = us-east-1
</pre>
2 Change the permissions of the file so that it it is owned by, and can only be read by, the `mysql` user:<pre class="fixed">chown mysql /var/lib/mysql/.aws/credentials
chmod 600 /var/lib/mysql/.aws/credentials
</pre>

## Configure MariaDB

1 Create a new option file to tell MariaDB to enable encryption functionality and to use the AWS KMS plugin. Create a new file under `/etc/my.cnf.d/` (or wherever your OS may have you create such files) with contents like this:

```sql
[mariadb]
plugin_load_add = aws_key_management
aws-key-management = FORCE_PLUS_PERMANENT
aws-key-management-master-key-id = alias/mariadb-encryption
aws-key-management-region = us-east-1
!include /etc/my.cnf.d/enable_encryption.preset
```

1 Append the "Alias" value you copied above to `alias/` to use as the value for the `aws-key-management-master-key-id` option.

Note that you <strong>must</strong> include `aws-key-management-region` in your .cnf file if you are not using the us-east-1 region.

Now, you have told MariaDB to use the AWS KMS plugin and you've put credentials for the plugin in a location where the plugin will find them. The /etc/my.cnf.d/enable_encryption.preset file contains a set of options that enable all available encryption functionality.

When you start MariaDB, the AWS KMS plugin will connect to the AWS Key Management Service and ask it to generate a new key. MariaDB will store that key on-disk in an encrypted form. The key stored on-disk cannot be used to decrypt the data; rather, on each startup, MariaDB must connect to AWS KMS and have the service decrypt the locally-stored key. The decrypted version is stored in-memory as long as the MariaDB server process is running, and that in-memory decrypted key is used to encrypt the local data.

### SELinux and outbound connections from MariaDB

Because MariaDB needs to connect to the AWS KMS service, you must ensure that the host has outbound network connectivity over port 443 to AWS and you must ensure that local policies allow the MariaDB server process to make those outbound connections. By default, SELinux restricts MariaDB from making such connections.

The most simple way to cause SELinux to allow outbound HTTPS connections from MariaDB is to enable to mysql_connect_any boolean, like this:

```sql
setsebool -P mysql_connect_any 1
```

There are more complex alternatives that have a more granular effect, but those are beyond the scope of this document.

## Start MariaDB

Start MariaDB using the `systemctl` tool:

```sql
systemctl start mariadb
```

If you do not use systemd, you may have to start MariaDB using some other mechanism.

You should see journal output similar to this:

```sql
# journalctl --no-pager -o cat -u mariadb.service

[Note] /usr/sbin/mysqld (mysqld 10.1.9-MariaDB-enterprise-log) starting as process 19831 ...
[Note] AWS KMS plugin: generated encrypted datakey for key id=1, version=1
[Note] AWS KMS plugin: loaded key 1, version 1, key length 128 bit
[Note] InnoDB: Using mutexes to ref count buffer pool pages
[Note] InnoDB: The InnoDB memory heap is disabled
[Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
[Note] InnoDB: Memory barrier is not used
[Note] InnoDB: Compressed tables use zlib 1.2.7
[Note] InnoDB: Using CPU crc32 instructions
[Note] InnoDB: Initializing buffer pool, size = 2.0G
[Note] InnoDB: Completed initialization of buffer pool
[Note] InnoDB: Highest supported file format is Barracuda.
[Note] InnoDB: 128 rollback segment(s) are active.
[Note] InnoDB: Waiting for purge to start
[Note] InnoDB:  Percona XtraDB (http://www.percona.com) 5.6.26-74.0 started; log sequence number 1616819
[Note] InnoDB: Dumping buffer pool(s) not yet started
[Note] Plugin 'FEEDBACK' is disabled.
[Note] AWS KMS plugin: generated encrypted datakey for key id=2, version=1
[Note] AWS KMS plugin: loaded key 2, version 1, key length 128 bit
[Note] Using encryption key id 2 for temporary files
[Note] Server socket created on IP: '::'.
[Note] Reading of all Master_info entries succeded
[Note] Added new Master_info '' to hash table
[Note] /usr/sbin/mysqld: ready for connections.
```

Note the several lines of output that refer explicitly to the "AWS KMS plugin". You can see that the plugin generates a "datakey", loads that data key, and then later generates and loads a second data key. The 2nd data key is used to encrypt temporary files and temporary tables.

You can see the encrypted keys stored on-disk in the datadir:

```sql
# ls -l /var/lib/mysql/aws*
-rw-rw----. 1 mysql mysql 188 Feb 25 18:55 /var/lib/mysql/aws-kms-key.1.1
-rw-rw----. 1 mysql mysql 188 Feb 25 18:55 /var/lib/mysql/aws-kms-key.2.1
```

Note that those keys are not useful alone. They are encrypted. When MariaDB starts up, the AWS KMS plugin decrypts those keys by interacting with AWS KMS.

For maximum security, you should start from an empty datadir and run `mysql_install_db` after configuring encryption. Then you should re-import your data so that it is fully encrypted. Use `sudo` to run `mysql_install_db` so that it finds your credentials file:

```sql
# sudo -u mysql mysql_install_db
Installing MariaDB/MySQL system tables in '/var/lib/mysql' ...
2016-02-25 23:16:06 139731553998976 [Note] /usr/sbin/mysqld (mysqld 10.1.11-MariaDB-enterprise-log) starting as process 39551 ...
2016-02-25 23:16:07 139731553998976 [Note] AWS KMS plugin: generated encrypted datakey for key id=1, version=1
2016-02-25 23:16:07 139731553998976 [Note] AWS KMS plugin: loaded key 1, version 1, key length 128 bit
...
```

## Create encrypted tables

With `innodb-encrypt-tables=ON`, new InnoDB tables will be encrypted by default, using the key ID set in `innodb_default_encryption_key_id` (default 1). With `innodb-encrypt-tables=FORCE` enabled, it is not possible to manually bypass encryption when creating a table.

You can cause the AWS KMS plugin to create new encryption keys at-will by specifying a new ENCRYPTION_KEY_ID when creating a table:

```sql
MariaDB [test]> create table t1 (id serial, v varchar(32)) ENCRYPTION_KEY_ID=3;
Query OK, 0 rows affected (0.91 sec)
```

```sql
[Note] AWS KMS plugin: generated encrypted datakey for key id=3, version=1
[Note] AWS KMS plugin: loaded key 3, version 1, key length 128 bit
```

```sql
# ls -l /var/lib/mysql/aws*
-rw-rw----. 1 mysql mysql 188 Feb 25 18:55 /var/lib/mysql/aws-kms-key.1.1
-rw-rw----. 1 mysql mysql 188 Feb 25 18:55 /var/lib/mysql/aws-kms-key.2.1
-rw-rw----. 1 mysql mysql 188 Feb 25 19:10 /var/lib/mysql/aws-kms-key.3.1
```

Read more about encrypting data in the [Data at Rest Encryption](/kb/en/data-at-rest-encryption/#encrypting-data) section of the MariaDB Documentation.

## AWS KMS plugin option reference

- `aws_key_management_master_key_id`: AWS KMS Customer Master Key ID (ARN or alias prefixed by `alias/`) for master encryption key. Used to create new data keys. If not set, no new data keys will be created.
- `aws_key_management_rotate_key`: Set this variable to a data key ID to perform rotation of the key to the master key given in `aws_key_management_master_key_id`. Specify -1 to rotate all keys.

- `aws_key_management_key_spec`: Encryption algorithm used to create new keys. Allowed values are AES_128 (default) or AES_256.
- `aws_key_management_log_level`: Logging for AWS API. Allowed values, in increasing verbosity, are "Off" (default), "Fatal", "Error", "Warn", "Info", "Debug", and "Trace".

## Next steps

For more information about advanced usage, including strategies to manage credentials, enforce separation of responsibilities, and even require 2-factor authentication to start your MariaDB server, please review [Amazon Web Services (AWS) Key Management Service (KMS) Encryption Plugin Advanced Usage](/mariadb-administration/user-server-security/securing-mariadb/securing-mariadb-encryption/securing-mariadb-data-at-rest-encryption/key-management-and-encryption-plugins/aws-key-management-encryption-plugin-advanced-usage/).
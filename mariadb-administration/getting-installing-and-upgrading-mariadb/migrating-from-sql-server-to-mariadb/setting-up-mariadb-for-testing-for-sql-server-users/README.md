# Setting Up MariaDB for Testing for SQL Server Users

This page contains links and hints to setup MariaDB for testing. The page is designed for SQL Server users, assuming that they are mostly familiar with Windows and they are not familiar with MariaDB.

## Choosing a MariaDB Version

As a general rule, for new installations it's better to choose the [latest Generally Available (GA) version](/kb/en/download-latest-releases/).

If you need a feature that is only present in a version that is not yet production-ready, and the project will surely not go to production before that version is GA, it could make sense to use a non-GA version. In this case however, keep in mind that you are using a version that is only suitable for testing.

If you need to work with an existing production instance, you should of course use the same version in testing. However, deprecated versions should not be used in production, because they could be exposed to vulnerabilities that will never be fixed. See [deprecation policies](/kb/en/deprecation-policy/) if you are not sure about the version you are using.

## Setting up MariaDB on Windows

There are two different ways to use MariaDB on Windows natively: using Zip packages or MSI packages.

In both cases, 32-bit platforms are still supported.

Check the page [Installation issues on Windows](/mariadb-administration/getting-installing-and-upgrading-mariadb/troubleshooting-installation-issues/installation-issues-on-windows/) to verify if current versions of MariaDB have troubles on Windows. More generally, it is a good idea to check the [Troubleshooting Installation Issues](/mariadb-administration/getting-installing-and-upgrading-mariadb/troubleshooting-installation-issues/) category.

### ZIP Packages

Windows users don't necessarily need to install MariaDB to use it. They can download ready-to-use ZIP packages to avoid any change in the system (except for downloading MariaDB and writing databases on the disk). This is very useful for testing without risking some undesired side effect on the machine in use. And it avoids the hassle of installing Docker or virtual machines.

##### MariaDB starting with [10.4.3](/kb/en/mariadb-1043-release-notes/)

Starting with [MariaDB 10.4.3](/kb/en/mariadb-1043-release-notes/), it is necessary to run [mysql_install_db.exe](/mariadb-administration/getting-installing-and-upgrading-mariadb/mysql_install_dbexe/) to install the data directory.

The drawback is that MariaDB will need to be started and stopped from the command line.

See [Installing MariaDB Windows ZIP Packages](/mariadb-administration/getting-installing-and-upgrading-mariadb/binary-packages/installing-mariadb-windows-zip-packages/).

### MSI Packages

MSI packages provide a friendly graphical interface to install MariaDB. The installation process is easy but flexible. For example, the user can decide which components to install, whether to install it as a service or not, and if networking should be enabled. An interface to uninstall MariaDB is also provided.

See [Installing MariaDB MSI Packages on Windows](/mariadb-administration/getting-installing-and-upgrading-mariadb/binary-packages/installing-mariadb-msi-packages-on-windows/).

## Installing MariaDB on Docker

Docker is a container platform that runs natively on Linux. A Docker image is a representation of a basic Linux system, which usually runs a single process - in our case, that process is MariaDB. A container is an instance of an image, which can be created or destroyed instantaneously. Once a container is started, it can be used just like a normal system.

Docker runs on all major operating systems. On Windows and MacOS it runs on a Linux virtual machine, but this additional complexity is transparent for the end user.

Docker's characteristics makes it optimal to test MariaDB functionalities without wasting time on installation and without making changes to the host system. However, it is not ideal to test MariaDB performance.

See [Installing and Using MariaDB via Docker](/mariadb-administration/getting-installing-and-upgrading-mariadb/binary-packages/automated-mariadb-deployment-and-administration/docker-and-mariadb/installing-and-using-mariadb-via-docker/).

## Reinitializing MariaDB Data Directory

While experimenting with MariaDB, you could end up with an unusable installation. This occurs for example if you deliberately delete files that you shouldn't delete. If it happens, there is no need to uninstall and reinstall MariaDB. Instead, you can simply delete the contents of the data directory and run [mysql_install_db](/clients-utilities/mysql_install_db/). The program will recreate your system tables and the essential files.

To know where your data directory is, check the [datadir](/kb/en/server-system-variables/#datadir) system variable.
# MariaDB ColumnStore 1.2.1 Beta Release Notes

<strong>Release date:</strong> 14th November 2018

[MariaDB ColumnStore 1.2.1](/columns-storage-engines-and-plugins/storage-engines/mariadb-columnstore/) is an Beta release of MariaDB ColumnStore. This is the first release of the MariaDB ColumnStore 1.2 series. The MariaDB ColumnStore 1.2 series provides several new features and improvements over the MariaDB ColumnStore 1.1 release.

MariaDB ColumnStore 1.2.1 is a <strong><em>[Beta](/kb/en/release-criteria/)</em></strong> release.

<strong>Do not use <em>beta</em> releases on production systems!</strong>

For an overview of [MariaDB ColumnStore](/columns-storage-engines-and-plugins/storage-engines/mariadb-columnstore/) see [MariaDB ColumnStore Architectural Overview](/columns-storage-engines-and-plugins/storage-engines/mariadb-columnstore/columnstore-architecture/columnstore-architectural-overview/)

Please provide feedback in [JIRA](https://jira.mariadb.org/browse/MCOL) for anything that is not working as expected so that we can fix it before we make the release available for the larger community.
For general "how to questions" ask questions [here](/columns-storage-engines-and-plugins/storage-engines/mariadb-columnstore/) or subscribe to mariadb-columnstore@googlegroups.com

## Notable changes

- [MCOL-1804](https://jira.mariadb.org/browse/MCOL-1804) - The base MariaDB server version is now [10.3.10](/kb/en/mariadb-10310-release-notes/) which include several maintenance and security fixes.
- [MCOL-520](https://jira.mariadb.org/browse/MCOL-520) - Non-root install now does not require sudo for the installation tools
- [MCOL-1846](https://jira.mariadb.org/browse/MCOL-1846) - Amazon AMI support for Instances with ENA drivers

## Bugs and issues fixed

- [MCOL-1307](https://jira.mariadb.org/browse/MCOL-1307) - Adapter can't recognize database schema written with backtick
- [MCOL-1554](https://jira.mariadb.org/browse/MCOL-1554) - PDI CS not work in the Pentaho Sever repository
- [MCOL-1776](https://jira.mariadb.org/browse/MCOL-1776) - Test [MCOL-1403](https://jira.mariadb.org/browse/MCOL-1403) failing in develop
- [MCOL-1782](https://jira.mariadb.org/browse/MCOL-1782) - UPDATE and DELETE reported incorrect affected row count
- [MCOL-1786](https://jira.mariadb.org/browse/MCOL-1786) - Handle "true" keyword for numeric data types in cpimport - with thanks to community contributor "tntnatbry"
- [MCOL-1792](https://jira.mariadb.org/browse/MCOL-1792) - Timestamp --&gt; Datetime insert: milliseconds are wrongly inserted in CS
- [MCOL-1799](https://jira.mariadb.org/browse/MCOL-1799) - test013 regression
- [MCOL-1811](https://jira.mariadb.org/browse/MCOL-1811) - Rename mcsimport package name during Windows build
- [MCOL-1812](https://jira.mariadb.org/browse/MCOL-1812) - Bulk Write SDK Windows installer uninstallation / alter - wrong execute credentials
- [MCOL-1813](https://jira.mariadb.org/browse/MCOL-1813) - Bulk Write SDK's Windows installer doesn't detect Python installations for the current user only
- [MCOL-1821](https://jira.mariadb.org/browse/MCOL-1821) - mcsimport not included in tools binary package
- [MCOL-1823](https://jira.mariadb.org/browse/MCOL-1823) - Amazon AMI quick installer script - add in default to do distributed install
- [MCOL-1826](https://jira.mariadb.org/browse/MCOL-1826) - PrimProc crash in float/double to string conversion
- [MCOL-1845](https://jira.mariadb.org/browse/MCOL-1845) - RPM package summary need to be consistent
- [MCOL-1852](https://jira.mariadb.org/browse/MCOL-1852) - Spark Exporter uses collect() instead of toLocalIterator() on DataFrames to export and therefore uses too much memory on the Driver
- [MCOL-1858](https://jira.mariadb.org/browse/MCOL-1858) - An `invalid` records indication when loading the table unsing mcsimport
- [MCOL-520](https://jira.mariadb.org/browse/MCOL-520) - true non root install phase 1
- [MCOL-1158](https://jira.mariadb.org/browse/MCOL-1158) - Support additional Python3 features using Swig's -py3 flag
- [MCOL-1642](https://jira.mariadb.org/browse/MCOL-1642) - Add SQL command that shows Primary Front-End MariaDB ColumnStore Module
- [MCOL-1671](https://jira.mariadb.org/browse/MCOL-1671) - Windows mcsapi - add option to install libraries directly into Java
- [MCOL-1774](https://jira.mariadb.org/browse/MCOL-1774) - mcsimport - enclose by character support and escape character for enclose by char
- [MCOL-1816](https://jira.mariadb.org/browse/MCOL-1816) - mcsapi - support bool data type
- [MCOL-1817](https://jira.mariadb.org/browse/MCOL-1817) - Pentaho support bool data type
- [MCOL-1373](https://jira.mariadb.org/browse/MCOL-1373) - Add TIME &amp; DATETIME+msec support to mcsapi
- [MCOL-1593](https://jira.mariadb.org/browse/MCOL-1593) - Add Windows builder to buildbot for API
- [MCOL-1744](https://jira.mariadb.org/browse/MCOL-1744) - Remove unnecessary CentOS 7 mcsapi package dependencies
- [MCOL-1754](https://jira.mariadb.org/browse/MCOL-1754) - Change libmysql dependency for Windows api tests to libmariadb
- [MCOL-1804](https://jira.mariadb.org/browse/MCOL-1804) - Rebase 1.2 on [MariaDB 10.3.10](/kb/en/mariadb-10310-release-notes/)
- [MCOL-1846](https://jira.mariadb.org/browse/MCOL-1846) - AMI support for Instances with ENA drivers
- [MCOL-1860](https://jira.mariadb.org/browse/MCOL-1860) - cannot insert symbols in column using the mcsimport with  escape_character option
- [MCOL-1654](https://jira.mariadb.org/browse/MCOL-1654) - Querystats table is broken

In addition, all bugs fixed in MariaDB ColumnStore 1.2.0 and earlier are implicitly included in this release.

## Upgrade

The following procedure outlines upgrading a 1.2.0 ColumnStore install to 1.2.1:

- [1.2.0 Alpha to 1.2.1 Beta upgrade procedure](/kb/en/mariadb-columnstore-software-upgrade-120-alpha-to-121-beta/)

Multi version upgrades generally will work using the same procedure however we can't test every possible permutation so you should test your specific scenario outside of production first if you wish to try this (and this is good practice regardless).

## Known issues and limitations

There are a number bugs and known limitations within this version of MariaDB ColumnStore, the most serious of these are listed below.

- [MCOL-1805](https://jira.mariadb.org/browse/MCOL-1805) - mcimport can show a warning about column sizes during import
- [MCOL-1793](https://jira.mariadb.org/browse/MCOL-1793) - regr_slope() and regr_r2() produce incorrect result when used as window functions

## Documentation

[MariaDB ColumnStore Documentation](/columns-storage-engines-and-plugins/storage-engines/mariadb-columnstore/)

## Packaging

RPM, Debian, and binary packages are provided for the Linux distributions supported by MariaDB ColumnStore 1.2.0 Alpha version.

- The supported OS for the Alpha version are CentOS 6, CentOS 7, Debian 8, Debian 9, RedHat 6, RedHat 7, SUSE 12, Ubuntu 16.04 and Ubuntu 18.04.
- Packages can be downloaded [here](https://mariadb.com/downloads/mariadb-ax) or can be installed from the [repositories](https://mariadb.com/kb/en/library/installing-mariadb-ax-from-the-package-repositories).
- An Amazon AWS AMI Image is available for this release, please search for AMI name "MariaDB-ColumnStore-1.2.0". AMI specific installation instructions can be found [here](/columns-storage-engines-and-plugins/storage-engines/mariadb-columnstore/columnstore-getting-started/installing-and-configuring-a-columnstore-system-using-the-amazon-ami/).
- Certified to run in Google Cloud Environment in the GA OSs.

## Source code

The source code of MariaDB ColumnStore is tagged at GitHub with a tag, which is identical with the version of MariaDB ColumnStore. For instance, the tag of version X.Y.Z of MariaDB ColumnStore is columnstore-X.Y.Z. Further, master always refers to the latest released non-beta version.

The source code is available at these locations

- Storage Engine - [Source code for engine specific processes on UM and PM node](https://github.com/mariadb-corporation/mariadb-columnstore-engine/tree/columnstore-1.2.1)
- MariaDB Server - [Source code based on MariaDB Server 10.3.10 modified to support the ColumnStore storage engine](https://github.com/mariadb-corporation/mariadb-columnstore-server/tree/columnstore-1.2.1)
- Tools - [Source code for MariaDB ColumnStore Tools](https://github.com/mariadb-corporation/mariadb-columnstore-tools/tree/columnstore-1.2.1)
- Write Data API - [Source code for Write Data API /SDK](https://github.com/mariadb-corporation/mariadb-columnstore-api/tree/columnstore-1.2.1)
- MaxScale CDC and Kafka Data Adapters - [Source Code for data adapters](https://github.com/mariadb-corporation/mariadb-columnstore-data-adapters/tree/columnstore-1.2.1)
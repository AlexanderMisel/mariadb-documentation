# MariaDB ColumnStore 1.0.3 Alpha Release Notes

<strong>Release date:</strong> 20 September 2016

[MariaDB ColumnStore 1.0.3](/columns-storage-engines-and-plugins/storage-engines/mariadb-columnstore) is an alpha release of MariaDB ColumnStore. This is the third alpha release of  MariaDB ColumnStore with improvements over previous alpha release of 1.0.2.

MariaDB ColumnStore 1.0.3 is an <strong><em>[Alpha](/kb/en/release-criteria/)</em></strong> release.

<strong>Do not use <em>alpha</em> releases on production systems!</strong>

For an overview of [MariaDB ColumnStore](/columns-storage-engines-and-plugins/storage-engines/mariadb-columnstore) see [MariaDB ColumnStore Architectural Overview](/columns-storage-engines-and-plugins/storage-engines/mariadb-columnstore/columnstore-architecture/columnstore-architectural-overview)

Please provide feedback in [JIRA](https://jira.mariadb.org/browse/MCOL) for anything that is not working as expected so that we can fix it before we make the release available for the larger community.
For general "how to questions" ask questions [here](/columns-storage-engines-and-plugins/storage-engines/mariadb-columnstore) or subscribe to mariadb-columnstore@googlegroups.com

## Notable Changes

- [MCOL-112](https://jira.mariadb.org/browse/MCOL-112) The base MariaDB server version is now 10.1.17, including the security fix for [CVE-2016-6662](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-6662) (MySQL Remote Root Code Execution / Privilege Escalation 0 day).
- [MCOL-171](https://jira.mariadb.org/browse/MCOL-171): A date of 0000-00-00 is now supported and will not be turned into NULL. Previous NULL conversions will remain as NULL
- [MCOL-274](https://jira.mariadb.org/browse/MCOL-274): The minimum possible data (apart from ‘0000-00-00’) has changed from ‘1400-01-01’ to ‘1000-01-01’ which is more in-line with MariaDB’s documented date range

## Bugs and Issues Fixed

Below is list of some of the bugs and issues fixed. For the complete list please see [here](https://jira.mariadb.org/issues/?jql=project%20%3D%20MCOL%20AND%20status%20%3D%20CLOSED%20and%20fixVersion%20%3D%201.0.3%20ORDER%20BY%20priority%20DESC%2C%20updated%20DESC)

- [MCOL-284](https://jira.mariadb.org/browse/MCOL-284): Comment parser in [MCOL-256](https://jira.mariadb.org/browse/MCOL-256) breaks some queries
- [MCOL-281](https://jira.mariadb.org/browse/MCOL-281): LDI using cpimport pads char column values with spaces
- [MCOL-274](https://jira.mariadb.org/browse/MCOL-274): DATETIME field doesn't match with MySQL standard
- [MCOL-264](https://jira.mariadb.org/browse/MCOL-264): DDL parser doesn't support space instead of equals in table options
- [MCOL-259](https://jira.mariadb.org/browse/MCOL-259): intermediate regression test failures - At least one DBRoot required for that query is offline.
- [MCOL-256](https://jira.mariadb.org/browse/MCOL-256): Queries that have inline comments will produce erroneous results
- [MCOL-240](https://jira.mariadb.org/browse/MCOL-240): DBT3 query 11 returned an internal error, ExeMgr aborted
- [MCOL-173](https://jira.mariadb.org/browse/MCOL-173): "INSERT INTO tableName SELECT 42,100" not working correctly
- [MCOL-171](https://jira.mariadb.org/browse/MCOL-171): 0000-00-00 dates are not supported and are munged to NULL with no warnings
- [MCOL-80](https://jira.mariadb.org/browse/MCOL-80): EXTRACT() function returned assertion error
- [MCOL-45](https://jira.mariadb.org/browse/MCOL-45): CREATE PROCEDURE fails

## Upgrade

Multi version upgrades are not supported, please upgrade versions prior to 1.0.2 before upgrading to 1.0.3. Details on upgrading from version to 1.0.2 to 1.0.3 can be found [here](https://mariadb.com/kb/en/mariadb/upgrading-mariadb-columnstore-from-102-to-103).

Upgrade from MariaDB ColumnStore Alpha versions 1.0.0 and 1.0.1 is not supported, please upgrade to version 1.0.2 prior to upgrading to 1.0.3.

## Known Issues and Limitations

There are a number bugs and known limitations within this early Alpha version of MariaDB ColumnStore, the most serious of these are listed below. These are expected to be fixed way before the Beta release.
There are some known security issues. They are listed [here](https://mariadb.com/kb/en/mariadb/security-vulnerabilities-mariadb-columnstore/)

- [MCOL-82](https://jira.mariadb.org/browse/MCOL-82): Subquery using IN with VIEW returns incorrect results. Queries selecting from view and using IN in where clause with a subquery on another view returns incorrect results.
- [MCOL-37](https://jira.mariadb.org/browse/MCOL-37): Following three window functions do not return correct value
<ul start="1"><li>FIRST_VALUE
</li><li>LEAD
</li><li>LAG
</li></ul>
- [MCOL-75](https://jira.mariadb.org/browse/MCOL-75) and [MCOL-74](https://jira.mariadb.org/browse/MCOL-74): NTH_VALUE and LAST_VALUE functions return syntax errors.
- [MCOL-73](https://jira.mariadb.org/browse/MCOL-73)): Wide table formatted display causes frontend to return error
<ul start="1"><li>MariaDB ColumnStore supports wide tables storage
</li><li>Displaying the query results on a large number of columns without formatting the column works
</li><li>Displaying the query results on a large number of columns with formatting causes error at MariaDB Server level
</li></ul>
- [MCOL-271](https://jira.mariadb.org/browse/MCOL-271) and [MCOL-171](https://jira.mariadb.org/browse/MCOL-171): empty string and date/datetime values are treated as NULL. This means you cannot insert empty values into a NOT NULL column.
- [MCOL-290](https://jira.mariadb.org/browse/MCOL-290): DecomSvr status incorrectly reported as Initial rather than Active.
- While Millisecond and Microsecond storage is supported for datetime, time and timestamp columns, at this time the query results cannot return millisecond and microseconds.
- UTF-8 Limitation
<ul start="1"><li>UTF-8 must be declared at the table level if the instance has been set up with a UTF-8 profile. Tables created with a non-matching character set will yield indeterminate results. 
</li><li>Viewing SQL output should be done using client software that supports UTF-8 character sets. 
</li><li>UTF-8 characters are not supported in object names. 
</li></ul>

## Documentation

[MariaDB ColumnStore Documentation](/columns-storage-engines-and-plugins/storage-engines/mariadb-columnstore)

## Packaging

RPM and binary packages are provided for the Linux distributions supported by MariaDB ColumnStore 1.0.3 Alpha version.

- The supported OS for this Alpha version are CentOS 6, CentOS 7 and Ubuntu 16.0.4.
- Packages can be downloaded [here](https://mariadb.com/my_portal/download/mariadb-columnstore)

## Source Code

The source code of MariaDB ColumnStore is tagged at GitHub with a tag, which is identical with the version of MariaDB ColumnStore. For instance, the tag of version X.Y.Z of MariaDB ColumnStore is columnstore-X.Y.Z. Further, master always refers to the latest released non-beta version.
The source code is available at these locations

- Storage Engine - [Source code for engine specific processes on UM and PM node](https://github.com/mariadb-corporation/mariadb-columnstore-engine)
- MariaDB Server - [Source code based on MariaDB Server 10.1.17 modified to support the ColumnStore storage engine](https://github.com/mariadb-corporation/mariadb-columnstore-server)
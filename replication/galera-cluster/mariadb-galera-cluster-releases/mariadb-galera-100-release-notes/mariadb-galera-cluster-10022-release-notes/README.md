# MariaDB Galera Cluster 10.0.22 Release Notes

The most recent [MariaDB Galera Cluster 10.0](/kb/en/galera/) release is:<br>
<span class="cstm-style lead"><strong>[MariaDB Galera Cluster 10.0.38](/replication/galera-cluster/mariadb-galera-cluster-releases/mariadb-galera-100-release-notes/mariadb-galera-cluster-10038-release-notes/)</strong> [Download<span>&nbsp;</span>Now](https://downloads.mariadb.org/mariadb-galera/10.0)</span>

[Download](http://downloads.mariadb.org/mariadb-galera/10.0.22)
[Release Notes](/replication/galera-cluster/mariadb-galera-cluster-releases/mariadb-galera-100-release-notes/mariadb-galera-cluster-10022-release-notes/)
[Changelog](/replication/galera-cluster/mariadb-galera-cluster-releases/mariadb-galera-100-changelogs/mariadb-galera-cluster-10022-changelog/)
[Overview of MariaDB Galera Cluster](/replication/galera-cluster/what-is-mariadb-galera-cluster/)

<strong>Release date:</strong> 19 Nov 2015

MariaDB Galera Cluster 10.0.22 is a <strong><em>[Stable](/kb/en/release-criteria/)</em></strong> (GA)
release. It is a merge of [MariaDB 10.0.22](/kb/en/mariadb-10022-release-notes/) and
[Galera Cluster](http://codership.com/content/using-galera-cluster) with
additional bug fixes.

Various articles about MariaDB Galera Cluster, including
[known limitations](/replication/galera-cluster/mariadb-galera-cluster-known-limitations/) and
[how to get started](/replication/galera-cluster/getting-started-with-mariadb-galera-cluster/) are
available in the <strong>[Galera](/kb/en/galera/)</strong> section of the Knowledge Base.

For a list of changes made in MariaDB Galera Cluster 10.0.22, with links to
detailed information on each push, see the
[MariaDB Galera Cluster 10.0.22 Changelog](/replication/galera-cluster/mariadb-galera-cluster-releases/mariadb-galera-100-changelogs/mariadb-galera-cluster-10022-changelog/).

## Updates and fixes in this version

- This release is mainly a bug-fix release.

- Codership changes: [github.com/codership/mysql-wsrep/tree/5.6](https://github.com/codership/mysql-wsrep/tree/5.6) (till commit 21a2415).

- The [Galera library](http://codership.com/content/using-galera-cluster) used
  by MariaDB Galera Cluster and included in the MariaDB repositories is
  currently at version <strong>25.3.9</strong>.

- [MDEV-5146](https://jira.mariadb.org/browse/MDEV-5146): Updates from LOAD DATA on a partitioned table were not getting replicated to other nodes in the cluster

- [MDEV-8208](https://jira.mariadb.org/browse/MDEV-8208): A node could crash during startup

- [MDEV-8834](https://jira.mariadb.org/browse/MDEV-8834): In a concurrent scenario, a session was able to write to a read_only node

## Notes

- Running MariaDB Galera Cluster 5.5 and 10.0 nodes in a cluster is not
  supported ([MDEV-6257](https://jira.mariadb.org/browse/MDEV-6257))

- This version of MariaDB Galera Cluster supports `wsrep` API v25 which means
  MariaDB Galera Cluster can be used with either a 25.2.x or 25.3.x
  Galera `wsrep` provider. A 25.3.x `wsrep` provider is included in the
  MariaDB repositories and is also available from the
  [downloads](http://downloads.mariadb.org/mariadb-galera/5.5.42) page.

- See the [MariaDB 10.0.22 Release Notes](/kb/en/mariadb-10022-release-notes/) and
  [Changelog](/kb/en/mariadb-10022-changelog/) for more information on the changes in
  MariaDB.

- On Ubuntu and Debian, the Galera Arbitrator daemon (garbd) and the galera
  library are in two separate packages. The packages are named <strong>galera-3</strong>
  and <strong>galera-arbitrator-3</strong>. When installing MariaDB Galera Cluster on Ubuntu and
  Debian (the <strong>mariadb-galera-server</strong> package), the galera-3 package will be
  automatically installed. You can then install the arbitrator package on a
  separate node (recommended) or on one of the nodes running
  mariadb-galera-server (not recommended).

Note: If Galera 25.2.x and 25.3.x are both being used in the cluster, MariaDB
with Galera 25.3.x must be started with
[wsrep_provider_options='socket.checksum=1'](/kb/en/wsrep_provider_options/#socketchecksum) in order to make it backward
compatible with Galera v2. Galera wsrep providers other than 25.3.x or 25.2.x
are not supported.

Thanks, and enjoy MariaDB Galera Cluster!

---

Be notified of new MariaDB Server releases automatically by [subscribing](https://lists.askmonty.org/cgi-bin/mailman/listinfo/announce) to the MariaDB Foundation community announce 'at' mariadb.org announcement list (this is a low traffic, announce-only list). MariaDB Corporation customers will be notified for all new releases, security issues and critical bug fixes for all MariaDB Corporation products thanks to the Notification Services.
<br><br>
MariaDB may already be included in your favorite OS distribution. More
information can be found on the
[Distributions which Include MariaDB](/mariadb-administration/getting-installing-and-upgrading-mariadb/binary-packages/distributions-which-include-mariadb/)
page.
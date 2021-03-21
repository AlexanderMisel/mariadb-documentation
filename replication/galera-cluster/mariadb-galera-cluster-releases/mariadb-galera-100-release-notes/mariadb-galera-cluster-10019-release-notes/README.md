# MariaDB Galera Cluster 10.0.19 Release Notes

The most recent [MariaDB Galera Cluster 10.0](/kb/en/galera/) release is:<br>
<span class="cstm-style lead"><strong>[MariaDB Galera Cluster 10.0.38](/replication/galera-cluster/mariadb-galera-cluster-releases/mariadb-galera-100-release-notes/mariadb-galera-cluster-10038-release-notes/)</strong> [Download<span>&nbsp;</span>Now](https://downloads.mariadb.org/mariadb-galera/10.0)</span>

[Download](http://downloads.mariadb.org/mariadb-galera/10.0.19)
[Release Notes](/replication/galera-cluster/mariadb-galera-cluster-releases/mariadb-galera-100-release-notes/mariadb-galera-cluster-10019-release-notes/)
[Changelog](/replication/galera-cluster/mariadb-galera-cluster-releases/mariadb-galera-100-changelogs/mariadb-galera-cluster-10019-changelog/)
[Overview of MariaDB Galera Cluster](/replication/galera-cluster/what-is-mariadb-galera-cluster/)

<strong>Release date:</strong> 18 May 2015

MariaDB Galera Cluster 10.0.19 is a <strong><em>[Stable](/kb/en/release-criteria/)</em></strong> (GA)
release.  It is a merge of [MariaDB 10.0.19](/kb/en/mariadb-10019-release-notes/) and
[Galera Cluster](http://codership.com/content/using-galera-cluster) with
additional bug fixes.

Various articles about MariaDB Galera Cluster, including
[known limitations](/replication/galera-cluster/mariadb-galera-cluster-known-limitations/) and
[how to get started](/replication/galera-cluster/getting-started-with-mariadb-galera-cluster/) are
available in the <strong>[Galera](/kb/en/galera/)</strong> section of the Knowledge Base.

For a list of changes made in MariaDB Galera Cluster 10.0.19, with links to
detailed information on each push, see the
[MariaDB Galera Cluster 10.0.19 Changelog](/replication/galera-cluster/mariadb-galera-cluster-releases/mariadb-galera-100-changelogs/mariadb-galera-cluster-10019-changelog/).

## Updates and fixes in this version

- Codership changes: github.com/codership/mysql-wsrep/tree/5.6 (till commit 2bb4c3d).

- The [Galera library](http://codership.com/content/using-galera-cluster) used
  by MariaDB Galera Cluster and included in the MariaDB repositories is
  currently at version <strong>25.3.9</strong>.

- Starting this version, [wsrep_osu_method](/kb/en/galera-cluster-system-variables/#wsrep_osu_method) has been made a session variable ([Issue#90](https://github.com/codership/mysql-wsrep/issues/90)).

- The following [FLUSH](/sql-statements-structure/sql-statements/administrative-sql-statements/flush-commands/flush/) commands are now executed under total order isolation:
<ul start="1"><li>FLUSH DES_KEY_FILE
</li><li>FLUSH HOSTS
</li><li>FLUSH PRIVILEGES
</li><li>FLUSH QUERY CACHE
</li><li>FLUSH STATUS
</li><li>FLUSH USER_RESOURCES
</li></ul>

### New and Deprecated Distributions

As per the [MariaDB Deprecation Policy](/kb/en/mariadb-deprecation-policy/), this will
be the final release of MariaDB Galera Cluster 10.0 for Fedora 19 "Schrödinger's Cat", Ubuntu
10.04 LTS "Lucid", and Mint 9 LTS "Isadora". When the next
version of MariaDB Galera Cluster 10.0 is released, repositories for these distributions will
go away.

We have also added a couple of new Linux distributions with this release. Both
Fedora 21 and Ubuntu 15.04 "Vivid" repositories are now available. As this is
the first release with these repositories, they are considered experimental.
Please [let us know](https://mariadb.org/jira) if you run into any issues with
them.

## Notes

- Running MariaDB Galera Cluster 5.5 and 10.0 nodes in a cluster is not
  supported ([MDEV-6257](https://jira.mariadb.org/browse/MDEV-6257))

- This version of MariaDB Galera Cluster supports `wsrep` API v25 which means
  MariaDB Galera Cluster can be used with either a 25.2.x or 25.3.x
  Galera `wsrep` provider. A 25.3.x `wsrep` provider is included in the
  MariaDB repositories and is also available from the
  [downloads](http://downloads.mariadb.org/mariadb-galera/5.5.42) page.

- See the [MariaDB 10.0.19 Release Notes](/kb/en/mariadb-10019-release-notes/) and
  [Changelog](/kb/en/mariadb-10019-changelog/) for more information on the changes in
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
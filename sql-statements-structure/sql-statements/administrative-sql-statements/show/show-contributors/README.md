# SHOW CONTRIBUTORS

## Syntax

```sql
SHOW CONTRIBUTORS
```

## Description

The <code class="highlight fixed" style="white-space:pre-wrap">SHOW CONTRIBUTORS</code> statement displays information about
the companies and people who financially contribute to MariaDB. For each contributor, it displays `Name`, `Location`, and `Comment` values. All columns are encoded as `latin1`.

It displays all [members and sponsors of the MariaDB Foundation](https://mariadb.org/en/supporters) as well as other financial contributors.

## Example

```sql
SHOW CONTRIBUTORS;
+---------------------+-------------------------------+-------------------------------------------------------------+
| Name                | Location                      | Comment                                                     |
+---------------------+-------------------------------+-------------------------------------------------------------+
| Booking.com         | https://www.booking.com       | Founding member, Platinum Sponsor of the MariaDB Foundation |
| Alibaba Cloud       | https://www.alibabacloud.com/ | Platinum Sponsor of the MariaDB Foundation                  |
| Tencent Cloud       | https://cloud.tencent.com     | Platinum Sponsor of the MariaDB Foundation                  |
| Microsoft           | https://microsoft.com/        | Platinum Sponsor of the MariaDB Foundation                  |
| MariaDB Corporation | https://mariadb.com           | Founding member, Platinum Sponsor of the MariaDB Foundation |
| Visma               | https://visma.com             | Gold Sponsor of the MariaDB Foundation                      |
| DBS                 | https://dbs.com               | Gold Sponsor of the MariaDB Foundation                      |
| IBM                 | https://www.ibm.com           | Gold Sponsor of the MariaDB Foundation                      |
| Tencent Games       | http://game.qq.com/           | Gold Sponsor of the MariaDB Foundation                      |
| Nexedi              | https://www.nexedi.com        | Silver Sponsor of the MariaDB Foundation                    |
| Acronis             | https://www.acronis.com       | Silver Sponsor of the MariaDB Foundation                    |
| Verkkokauppa.com    | https://www.verkkokauppa.com  | Bronze Sponsor of the MariaDB Foundation                    |
| Virtuozzo           | https://virtuozzo.com         | Bronze Sponsor of the MariaDB Foundation                    |
| Tencent Game DBA    | http://tencentdba.com/about   | Bronze Sponsor of the MariaDB Foundation                    |
| Tencent TDSQL       | http://tdsql.org              | Bronze Sponsor of the MariaDB Foundation                    |
| Percona             | https://www.percona.com/      | Bronze Sponsor of the MariaDB Foundation                    |
| Google              | USA                           | Sponsoring encryption, parallel replication and GTID        |
| Facebook            | USA                           | Sponsoring non-blocking API, LIMIT ROWS EXAMINED etc        |
| Ronald Bradford     | Brisbane, Australia           | EFF contribution for UC2006 Auction                         |
| Sheeri Kritzer      | Boston, Mass. USA             | EFF contribution for UC2006 Auction                         |
| Mark Shuttleworth   | London, UK.                   | EFF contribution for UC2006 Auction                         |
+---------------------+-------------------------------+-------------------------------------------------------------+
```

## See Also

- [Log of MariaDB contributors](/kb/en/log-of-mariadb-contributions/).
- [SHOW AUTHORS](/sql-statements-structure/sql-statements/administrative-sql-statements/show/show-authors/) list the authors of MariaDB (including documentation, QA etc).
- [MariaDB Foundation page on contributing financially](https://mariadb.org/donate/)
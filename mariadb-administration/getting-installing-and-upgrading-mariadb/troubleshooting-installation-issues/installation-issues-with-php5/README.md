# Installation Issues with PHP5

PHP5 may give an error if used with the old connect method:

```sql
'mysql_connect(): Headers and client library minor version mismatch. Headers:50156 Library:50206'
```

This is because the library wrongly checks and expects that the client library
must be the exact same version as PHP was compiled with. You would get the
same error if you tried to upgrade just the MySQL client library without
upgrading PHP at the same time.
See also [How to match API version for php5_mysql and mariadb client](1079).

Ways to fix this issue:

1 Switch to using the
  [mysqlnd driver](http://dev.mysql.com/downloads/connector/php-mysqlnd/) in
  PHP (Recommended solution).
2 Run with a lower [error reporting level](http://php.net/error-reporting):<pre class="fixed"><span class="x">$err_level = error_reporting(0);</span>
<span class="x">$conn = mysql_connect('params');</span>
<span class="x">error_reporting($err_level); </span>
</pre>
3 Recompile PHP with the MariaDB client libraries.
4 Use your original MySQL client library with the MariaDB.
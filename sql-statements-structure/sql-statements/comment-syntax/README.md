# Comment Syntax

There are three supported comment styles in MariaDB:

1 From a '<code class="fixed" style="white-space:pre-wrap">#</code>' to the end of a line:<pre class="fixed"><span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">users</span><span class="p">;</span> <span class="c1"># This is a comment</span>
</pre>
2 From a '<code class="fixed" style="white-space:pre-wrap">-- </code>' to the end of a line. The space after the two dashes is required (as in MySQL).<pre class="fixed"><span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">users</span><span class="p">;</span> <span class="c1">-- This is a comment</span>
</pre>
3 C style comments from an opening '<code class="fixed" style="white-space:pre-wrap">/*</code>' to a closing '<code class="fixed" style="white-space:pre-wrap">*/</code>'. Comments of this form can span multiple lines:<pre class="fixed"><span class="k">SELECT</span> <span class="o">*</span> <span class="k">FROM</span> <span class="n">users</span><span class="p">;</span> <span class="cm">/* This is a</span>
<span class="cm">multi-line</span>
<span class="cm">comment */</span>
</pre>

Nested comments are possible in some situations, but they are not supported or
recommended.

## Executable Comments

As an aid to portability between different databases, MariaDB supports
executable comments. These special comments allow you to embed SQL code which
will not execute when run on other databases, but will execute when run on
MariaDB.

MariaDB supports both MySQL's executable comment format, and a slightly
modified version specific to MariaDB. This way, if you have SQL code that works
on MySQL and MariaDB, but not other databases, you can wrap it in a MySQL
executable comment, and if you have code that specifically takes advantage of
features only available in MariaDB you can use the MariaDB specific format to
hide the code from MySQL.

### Executable Comment Syntax

MySQL and MariaDB executable comment syntax:

```sql
/*! MySQL or MariaDB-specific code */
```

Code that should be executed only starting from a specific MySQL or MariaDB version:

```sql
/*!##### MySQL or MariaDB-specific code */
```

The numbers, represented by '<code class="fixed" style="white-space:pre-wrap">######</code>' in the syntax examples
above specify the specific the minimum versions of MySQL and MariaDB that should execute the comment. The first number is the major version, the second 2 numbers are the minor version and the last 2 is the patch level.

For example, if you want to embed some code that should only execute on MySQL or MariaDB starting from 5.1.0, you would do the following:

```sql
/*!50100 MySQL and MariaDB 5.1.0 (and above) code goes here. */
```

MariaDB-only executable comment syntax (starting from [MariaDB 5.3.1](/kb/en/mariadb-531-release-notes/)):

```sql
/*M! MariaDB-specific code */
/*M!###### MariaDB-specific code */
```

##### MariaDB starting with [10.0.7](/kb/en/mariadb-1007-release-notes/)

Starting from [MariaDB 10.0.7](/kb/en/mariadb-1007-release-notes/), MariaDB ignores MySQL-style executable comments
that have a version number in the range `50700..99999`. This is needed to
skip features introduced in MySQL-5.7 that are not ported to MariaDB 10.x
yet.

```sql
/*!50701 MariaDB-10.x ignores MySQL-5.7 specific code */
```

<strong>Note:</strong> comments which have a version number in the range `50700..99999` that use
MariaDB-style executable comment syntax are still executed.

```sql
/*M!50701 MariaDB-10.x does not ignore this */
```

Statement delimiters cannot be used within executable comments.

## Examples

In MySQL all the following will return 2:
In MariaDB, starting from 5.3.1 the last 2 queries would return 3.

```sql
SELECT 2 /* +1 */;
SELECT 1 /*! +1 */;
SELECT 1 /*!50101 +1 */;
SELECT 2 /*M! +1 */;
SELECT 2 /*M!50301 +1 */;
```

The following executable statement will not work due to the delimiter inside the executable portion:

```sql
/*M!100100 select 1 ; */
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near '' at line 1
```

Instead, the delimiter should be placed outside the executable portion:

```sql
/*M!100100 select 1 */;
+---+
| 1 |
+---+
| 1 |
+---+
```
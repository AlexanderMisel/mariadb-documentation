# About SphinxSE

The Sphinx storage engine (SphinxSE) is a storage engine that talks to searchd (the Sphinx daemon) to enable text searching. Sphinx and SphinxSE is used as a faster and more customizable alternative to MariaDB's [built-in full-text search](/replication/optimization-and-tuning/optimization-and-indexes/full-text-indexes).

Sphinx does not depend on MariaDB, and can run independently, but SphinxSE provides a convenient interface to the underlying Sphinx daemon.

## Versions of SphinxSE in MariaDB

<table><tbody><tr><th>SphinxSE Version</th><th>Introduced</th><th>Maturity</th></tr>
<tr><td><a href="http://sphinxsearch.com/docs/current.html">SphinxSE 2.2.6</a></td><td><a href="/kb/en/mariadb-10015-release-notes/">MariaDB 10.0.15</a></td><td>Stable</td></tr>
<tr><td><a href="http://sphinxsearch.com/docs/2.1.9/">SphinxSE 2.1.9</a></td><td><a href="/kb/en/mariadb-10014-release-notes/">MariaDB 10.0.14</a></td><td>Stable</td></tr>
<tr><td>SphinxSE 2.0.4</td><td><a href="/kb/en/what-is-mariadb-55/">MariaDB 5.5</a></td><td></td></tr>
<tr><td>SphinxSE 0.99</td><td><a href="/kb/en/what-is-mariadb-52/">MariaDB 5.2</a> and <a href="/kb/en/what-is-mariadb-53/">MariaDB 5.3</a></td><td></td></tr>
</tbody></table>

## Enabling SphinxSE in MariaDB

As of [MariaDB 5.2.2](/kb/en/mariadb-522-release-notes/), the Sphinx storage engine is included in the source,
binaries, and packages of MariaDB. SphinxSE is built as a dynamically loadable
.so plugin. To use it you need to perform a one-time 
<code class="highlight fixed" style="white-space:pre-wrap">[INSTALL PLUGIN](/sql-statements-structure/sql-statements/administrative-sql-statements/plugin-sql-statements/install-plugin)</code>:

```sql
INSTALL SONAME 'ha_sphinx';
```

##### MariaDB until [10.0](/kb/en/what-is-mariadb-100/)

In Debian/Ubuntu packages SphinxSE is statically compiled into the MariaDB server, there is no need to use the `INSTALL SONAME` statement.

Once installed, SphinxSE will show up in the list of installed storage engines:

```sql
SHOW ENGINES;
+------------+---------+--------------------------------------------+--------------+------+------------+
| Engine     | Support | Comment                                    | Transactions | XA   | Savepoints |
+------------+---------+--------------------------------------------+--------------+------+------------+
...
| SPHINX     | YES     | Sphinx storage engine 0.9.9                | NO           | NO   | NO         |
...
+------------+---------+--------------------------------------------+--------------+------+------------+
```

This is a one-time step and will not need to be performed again.

<strong>Note:</strong> SphinxSE is just the storage engine part of Sphinx. You will have to
[install Sphinx](/columns-storage-engines-and-plugins/storage-engines/sphinx-storage-engine/installing-sphinx) itself in order to make use of
SphinxSE in MariaDB.

Despite the name, SphinxSE does not actually store any data itself. It is
actually a built-in client which allows MariaDB to talk to Sphinx, run search
queries, and obtain search results. All indexing and searching happen outside
MariaDB.

Some SphinxSE applications include:

- easier porting of MariaDB/MySQL FTS applications to Sphinx
- allowing Sphinx use with programming languages for which native APIs are not
  available yet
- optimizations when additional Sphinx result set processing on the MariaDB
  side is required (eg. JOINs with original document tables, additional
  MariaDB-side filtering, and etc...)

## Using SphinxSE

### Basic Usage

To search via SphinxSE, you would need to create a special
<code class="highlight fixed" style="white-space:pre-wrap">ENGINE=SPHINX</code> "search table", and then
<code class="highlight fixed" style="white-space:pre-wrap">SELECT</code> from it with full text query put into the
<code class="highlight fixed" style="white-space:pre-wrap">WHERE</code> clause for query column.

Here is an example create statement and search query:

```sql
CREATE TABLE t1
(
    id          BIGINT UNSIGNED NOT NULL,
    weight      INTEGER NOT NULL,
    query       VARCHAR(3072) NOT NULL,
    group_id    INTEGER,
    INDEX(query)
) ENGINE=SPHINX CONNECTION="sphinx://127.0.0.1:9312/test1";

SELECT * FROM t1 WHERE query='test it;mode=any';
```

The first three columns of the search table must have a type of <code class="highlight fixed" style="white-space:pre-wrap">BIGINT</code> for the 1st column (document id),
<code class="highlight fixed" style="white-space:pre-wrap">INTEGER</code> or <code class="highlight fixed" style="white-space:pre-wrap">BIGINT</code> for the 2nd column
(match weight), and <code class="highlight fixed" style="white-space:pre-wrap">VARCHAR</code> or <code class="highlight fixed" style="white-space:pre-wrap">TEXT</code> for
the 3rd column (your query), respectively. This mapping is fixed; you cannot
omit any of these three required columns, or move them around, or change types.
Also, the query column must be indexed; all the others must be kept unindexed.
Column names are ignored so you can use arbitrary ones.

Additional columns must be either <code class="highlight fixed" style="white-space:pre-wrap">INTEGER</code>,
<code class="highlight fixed" style="white-space:pre-wrap">TIMESTAMP</code>, <code class="highlight fixed" style="white-space:pre-wrap">BIGINT</code>,
<code class="highlight fixed" style="white-space:pre-wrap">VARCHAR</code>, or <code class="highlight fixed" style="white-space:pre-wrap">FLOAT</code>. They will be bound to
the attributes provided in the Sphinx result set by name, so their names must
match the attribute names specified in <code class="highlight fixed" style="white-space:pre-wrap">sphinx.conf</code>. If
there's no such attribute name in the Sphinx search results, the additional
columns will have <code class="highlight fixed" style="white-space:pre-wrap">NULL</code> values.

Special "virtual" attribute names can also be bound to SphinxSE columns.
<code class="highlight fixed" style="white-space:pre-wrap">_sph_</code> needs to be used instead of <code class="highlight fixed" style="white-space:pre-wrap">@</code> for
that.  For instance, to obtain the values of '<code class="highlight fixed" style="white-space:pre-wrap">@groupby</code>',
'<code class="highlight fixed" style="white-space:pre-wrap">@count</code>', or '<code class="highlight fixed" style="white-space:pre-wrap">@distinct</code>' virtual
attributes, use '<code class="highlight fixed" style="white-space:pre-wrap">_sph_groupby</code>',
'<code class="highlight fixed" style="white-space:pre-wrap">_sph_count</code>' or '<code class="highlight fixed" style="white-space:pre-wrap">_sph_distinct</code>' column
names, respectively.

The <code class="highlight fixed" style="white-space:pre-wrap">CONNECTION</code> string parameter is used to specify the
default <code class="highlight fixed" style="white-space:pre-wrap">searchd</code> host, port, and indexes for queries issued
using this table. If no connection string is specified in <code class="highlight fixed" style="white-space:pre-wrap">CREATE
TABLE</code>, index name '<code class="highlight fixed" style="white-space:pre-wrap">*</code>' (ie. search all indexes) and
'<code class="highlight fixed" style="white-space:pre-wrap">127.0.0.1:9312</code>' are assumed. The connection string syntax
is as follows:

```sql
CONNECTION="sphinx://HOST:PORT/INDEXNAME"
```

You can change the default connection string later like so:

```sql
ALTER TABLE t1 CONNECTION="sphinx://NEWHOST:NEWPORT/NEWINDEXNAME";
```

You can also override all these parameters per-query.

<strong>Note:</strong> To use Linux sockets you can modify the searchd section of the Sphinx configuration file, setting the listen parameter to a socket file. Instruct SphinxSE about the socket using CONNECTION="unix:<em>unix/domain/socket[:index]".   </em>

### Search Options

As seen in the example above, both query text and search options should be put
into the '<code class="fixed" style="white-space:pre-wrap">WHERE</code>' clause of the search query column (i.e. the
3rd column); the options are separated by semicolons ('<code class="fixed" style="white-space:pre-wrap">;</code>') and
separate names from values using an equals sign ('<code class="fixed" style="white-space:pre-wrap">=</code>'). Any
number of options can be specified. Available options are:

- query - query text;
- mode - matching mode. Must be one of "all", "any", "phrase", "boolean", or
  "extended". Default is "all";
- sort - match sorting mode. Must be one of "relevance", "attr_desc",
  "attr_asc", "time_segments", or "extended". In all modes besides "relevance"
  attribute name (or sorting clause for "extended") is also required after a
  colon:

```sql
... WHERE query='test;sort=attr_asc:group_id';
... WHERE query='test;sort=extended:@weight desc, group_id asc';
```

- offset - offset into result set, default is 0;
- limit - amount of matches to retrieve from result set, default is 20;
- index - names of the indexes to search:

```sql
... WHERE query='test;index=test1;';
... WHERE query='test;index=test1,test2,test3;';
```

- minid, maxid - min and max document ID to match;
- weights - comma-separated list of weights to be assigned to Sphinx full-text
  fields:

```sql
... WHERE query='test;weights=1,2,3;';
```

- filter, !filter - comma-separated attribute name and a set of values to
  match:

```sql
# only include groups 1, 5 and 19
... WHERE query='test;filter=group_id,1,5,19;';

# exclude groups 3 and 11
... WHERE query='test;!filter=group_id,3,11;';
```

- range, !range - comma-separated attribute name, min and max value to 
  match:

```sql
# include groups from 3 to 7, inclusive
... WHERE query='test;range=group_id,3,7;';

# exclude groups from 5 to 25
... WHERE query='test;!range=group_id,5,25;';
```

- maxmatches - per-query max matches value:

```sql
... WHERE query='test;maxmatches=2000;';
```

- groupby - group-by function and attribute:

```sql
... WHERE query='test;groupby=day:published_ts;';
... WHERE query='test;groupby=attr:group_id;';
```

- groupsort - group-by sorting clause:

```sql
... WHERE query='test;groupsort=@count desc;';
```

- indexweights - comma-separated list of index names and weights to use when
  searching through several indexes:

```sql
... WHERE query='test;indexweights=idx_exact,2,idx_stemmed,1;';
```

- comment - a string to mark this query in query log (mapping to $comment
  parameter in Query() API call):

```sql
... WHERE query='test;comment=marker001;';
```

- select - a string with expressions to compute (mapping to SetSelect() API 
  call):

```sql
... WHERE query='test;select=2*a+3*b as myexpr;';
```

<strong>Note:</strong> It is much more efficient to allow Sphinx to perform sorting,
filtering, and slicing of the result set than to raise max matches count and
use '<code class="highlight fixed" style="white-space:pre-wrap">WHERE</code>', '<code class="highlight fixed" style="white-space:pre-wrap">ORDER BY</code>', and
'<code class="highlight fixed" style="white-space:pre-wrap">LIMIT</code>' clauses on the MariaDB side. This is for two
reasons:

1 Sphinx does a number of optimizations and performs better than
  MariaDB/MySQL on these tasks.
2 Less data would need to be packed by <code class="highlight fixed" style="white-space:pre-wrap">searchd</code>, and
  transferred and unpacked by SphinxSE.

### SHOW ENGINE SPHINX STATUS

Starting with version 0.9.9-rc1, additional query info besides the result set
can be retrieved with the '<code class="highlight fixed" style="white-space:pre-wrap">SHOW ENGINE SPHINX STATUS</code>'
statement:

```sql
SHOW ENGINE SPHINX STATUS;
+--------+-------+-------------------------------------------------+
| Type   | Name  | Status                                          |
+--------+-------+-------------------------------------------------+
| SPHINX | stats | total: 25, total found: 25, time: 126, words: 2 | 
| SPHINX | words | sphinx:591:1256 soft:11076:15945                | 
+--------+-------+-------------------------------------------------+
```

This information can also be accessed through status variables. Note that this
method does not require super-user privileges.

```sql
SHOW STATUS LIKE 'sphinx_%';
+--------------------+----------------------------------+
| Variable_name      | Value                            |
+--------------------+----------------------------------+
| sphinx_total       | 25                               | 
| sphinx_total_found | 25                               | 
| sphinx_time        | 126                              | 
| sphinx_word_count  | 2                                | 
| sphinx_words       | sphinx:591:1256 soft:11076:15945 | 
+--------------------+----------------------------------+
```

### JOINs with SphinxSE

You can perform <code class="highlight fixed" style="white-space:pre-wrap">JOIN</code>s on a SphinxSE search table and tables using
other engines. Here's an example with "documents" from example.sql:

```sql
SELECT content, date_added FROM test.documents docs
    JOIN t1 ON (docs.id=t1.id) 
    WHERE query="one document;mode=any";
+-------------------------------------+---------------------+
| content                             | docdate             |
+-------------------------------------+---------------------+
| this is my test document number two | 2006-06-17 14:04:28 | 
| this is my test document number one | 2006-06-17 14:04:28 | 
+-------------------------------------+---------------------+

SHOW ENGINE SPHINX STATUS;
+--------+-------+---------------------------------------------+
| Type   | Name  | Status                                      |
+--------+-------+---------------------------------------------+
| SPHINX | stats | total: 2, total found: 2, time: 0, words: 2 | 
| SPHINX | words | one:1:2 document:2:2                        | 
+--------+-------+---------------------------------------------+
```

## Building snippets (excerpts) via MariaDB

Starting with version 0.9.9-rc2, SphinxSE also includes a UDF function that
lets you create snippets through MariaDB. The functionality is fully similar to
the
[BuildExcerprts](http://sphinxsearch.com/docs/current.html#api-func-buildexcerpts)
API call but is accessible through MariaDB+SphinxSE.

##### MariaDB until [5.5](/kb/en/what-is-mariadb-55/)

The binary that provides the UDF is named sphinx.so and is automatically built
and installed to the proper location along with SphinxSE itself. Register the
UDF using the following statement:

```sql
CREATE FUNCTION sphinx_snippets RETURNS STRING SONAME 'sphinx.so';
```

##### MariaDB until [10.0](/kb/en/what-is-mariadb-100/)

The UDF is packed together with the storage engine, in the same binary named ha_sphinx.so. Register the
UDF using the following statement:

```sql
CREATE FUNCTION sphinx_snippets RETURNS STRING SONAME 'ha_sphinx.so';
```

The function name must be '<code class="highlight fixed" style="white-space:pre-wrap">sphinx_snippets</code>', you can not use an
arbitrary name. Function arguments are as follows:

```sql
Prototype: function sphinx_snippets ( document, index, words, [options] );
```

Document and words arguments can be either strings or table columns. Options
must be specified like this: &lt;code&gt;'value' AS option_name&lt;/code&gt;. For a list of
supported options, refer to the
[BuildExcerprts()](http://sphinxsearch.com/docs/current.html#api-func-buildexcerpts)
API call. The only UDF-specific additional option is named 'sphinx' and lets
you specify searchd location (host and port).

Usage examples:

```sql
SELECT sphinx_snippets('hello world doc', 'main', 'world',
    'sphinx://192.168.1.1/' AS sphinx, true AS exact_phrase,
    '[b]' AS before_match, '[/b]' AS after_match)
FROM documents;

SELECT title, sphinx_snippets(text, 'index', 'mysql php') AS text
    FROM sphinx, documents
    WHERE query='mysql php' AND sphinx.id=documents.id;
```

## More Information

More information on Sphinx and SphinxSE is available on the
[Sphinx website](http://sphinxsearch.com/docs/current.html).
# mysql-test Auxiliary Files

The mysql-test framework utilizes many other files that affect the testing
process, in addition to test and result files.

---

## `disabled.def` file

This file can be used to disable certain tests temporarily. For example, if one
test fails and you are working on that, you may want to push the changeset that
disables the test into the test suite so that other tests won't be disturbed by
this failure.

The file contains test names and a comment (that should explain why the test
was disabled), separated by a colon. Lines that start with a hash sign
(`#`) are ignored. A typical `disabled.def` may look like this (note
that a hash sign in the middle of a line does not start a comment):

```sql
# List of disabled tests
# test name : comment
rpl_redirect : Fails due to bug#49978
events_time_zone  : need to fix the timing
```

During testing, mtr will print disabled tests like this:

```sql
...
rpl.rpl_redirect              [ disabled ]  Fails due to bug#49978
rpl.events_time_zone          [ disabled ]  need to fix the timing
...
```

This file should be located in the suite directory.

---

## `suite.opt` file

This file lists server options that will be added to the `mysqld` command
line for every test of this suite. It can refer to environment variables with
the `$NAME` syntax. Shell meta-characters should be quoted. For example

```sql
--plugin-load=$AUTH_PAM_SO
--max-connections=40 --net_read_timeout=5
"--replicate-rewrite-db=test->rewrite"
```

Note that options may be put either on one line or on separate lines. It is a
good idea to start an option name with the <code class="fixed" style="white-space:pre-wrap">--loose-</code> prefix if
the server may or may not recognize the option depending on the configuration.
An unknown option in the `.opt` file will stop the server from starting, and
the test will be aborted.

This file should be located in the suite directory.

---

## other `*.opt` files

For every test or include file `somefile.test` or `somefile.inc`, mtr will
look for `somefile.opt`, `somefile-master.opt` and `somefile-slave.opt`.
These files have exactly the same syntax as the `suite.opt` above. Options
from these files will also be added to the server command line (all servers
started for this test, only master, or only slave respectively) for all
affected tests, for example, for all tests that include `somefile.inc`
directly or indirectly.

A typical usage example is `include/have_blackhole.inc` and
`include/have_blackhole.opt`. The latter contains the necessary command-line
options to load the Blackhole storage engine, while the former verifies that
the engine was really loaded. Any test that needs the Blackhole engine needs
only to start from `source include/have_blackhole.inc;` and the engine will
be automatically loaded for the test.

---

## `my.cnf` file

This is not the `my.cnf` file that tests from this suite will use, but rather
a <em>template</em> of it. It will be converted later to an actual `my.cnf`. If a
suite contains no `my.cnf` template, a default template,
<span>—</span> `include/default_my.cnf`
<span>—</span> will be used. Or `suite/rpl/my.cnf` if the test
includes `master-slave.inc` (it's one of the few bits of the old MySQL
`mysql-test-run` magic that we have not removed yet). Typically a suite
template will not contain a complete server configuration, but rather start
from

```sql
!include include/default_my.cnf
```

and then add the necessary modifications.

The syntax of `my.cnf` template is the same of a normal `my.cnf` file, with
a few extensions and assumptions. They are:

- For any group with the name `[mysqld.<strong>N</strong>]`, where <strong>N</strong> is a number, mtr
  will start one `mysqld` process. Usually one needs to have
  only `[mysqld.1]` group, and `[mysqld.2]` group for replication tests.

- There can be groups with non-standard names (`[foo]`, `[bar]`, whatever),
  not used by `mysqld`. The `suite.pm` files (see below) may use them
  somehow.

- Values can refer to each other using the syntax `@groupname.optionname`
  <span>—</span> these references be expanded as needed. For
  example<pre class="fixed">[mysqld.2]
master-port= @mysqld.1.port
</pre>

- it sets the value of the `master-port` in the `[mysqld.2]` group to the value of `port` in the `[mysqld.1]` group.

- An option name may start with a hash sign `#`. In the
  resulting `my.cnf` it will look like a comment, but it still can be
  referred to. For example:

```sql
[example]
#location = localhost:@mysqld.1.port
bar = server:@example.#location/data
```

- There is the `[ENV]` group. It sets values for the environment variables.
  For example<pre class="fixed">[ENV]
MASTER_MYPORT = @mysqld.1.port
</pre>

- Also, one can refer to values of environment variables via this
      group:<pre class="fixed">[mysqld.1]
user = @ENV.LOGNAME
</pre>

- There is the `[OPT]` group. It allows to invoke functions and
  generate values. Currently it contains only one option
  <span>—</span> `@OPT.port`. Every time this option is referred
  to in some other group in the `my.cnf` template, a new unique port number
  is generated. It will not match any other port number used by this test run.
  For example<pre class="fixed">[ENV]
SPHINXSEARCH_PORT = @OPT.port
</pre>

This file should be located in the suite directory.

---

## other `*.cnf` files

For every test file `somefile.test` (but for not included files) mtr will
look for `somefile.cnf` file. If such a file exists, it will be used as a
template instead of suite `my.cnf` or a default `include/default_my.cnf`
templates.

---

## `combinations` file

The `combinations` file defines few sets of alternative configurations, and
every test in this suite will be run many times - once for every configuration.
This can be used, for example, to run all replication tests in the <em>rpl</em>
suite for all three binlog format modes (row, statement, and mixed). A
corresponding `combinations` file would look as
following:

```sql
[row]
binlog-format=row

[stmt]
binlog-format=statement

[mix]
binlog-format=mixed
```

It uses `my.cnf` file syntax, with groups (where group names define
combination names) and options. But, despite the similarity, it is not a
`my.cnf` template, and it cannot use the templating extentions. Instead,
options from the `combinations` file are added to the server command line. In
this regard, combination file is closer to `suite.opt` file. And just like
it, combination file can use environment variables using the `$NAME` syntax.

Not all tests will necessarily run for all combinations. A particular test may
require to be run only in one specific combination. For example, in
replication, if a test can only be run with the row binlog format, it will have
`--binlog-format=row` in one of the `.opt` files. In this case, mtr will
notice that server command line already has an option that matches one of the
combinations, and will skip all other combinations for this particular test.

The `combinations` file should be located in the suite directory.

---

## other `*.combinations` files

Just like with the `*.opt` files, mtr will use `somefile.combinations` file
for any `somefile.test` and `somefile.inc` that is used in testing. These
files have exactly the same format as a suite `combinations` file.

This can cause many combination files affecting one test file (if a test
includes two `.inc` files, and both of them have corresponding
`.combinations` files). In this case, mtr will run the test for all
combinations of combinations from both files. In [MariaDB 5.5](/kb/en/what-is-mariadb-55/), for example,
`rpl_init.inc` adds combinations for row/statement/mixed, and
`have_innodb.inc` adds combinations for innodb/xtradb. Thus any replication
test that uses innodb will be run six times.

---

## `suite.pm` file

This (optional) file is a perl module. It must declare a
package that inherits from `My::Suite`.

This file must normally end with `bless {}` <span>—</span> that
is it must return an object of that class.  It can also return a string
<span>—</span> in this case all tests in the suite will be skipped,
with this string being printed as a reason (for example "PBXT engine was not
compiled").

A suite class can define the following methods:

- `config_files()`
- `is_default()`
- `list_cases()`
- `servers()`
- `skip_combinations()`
- `start_test()`

A `config_files()` method returns a list of additional config files (besides
`my.cnf`), that this suite needs to be created. For every file it specifies a
function that will create it, when given a `My::Config` object. For example:

```sql
sub config_files {(
    'config.ini' => \&write_ini,
    'new.conf'   => \&do_new
)}
```

A `servers()` method returns a list of processes that needs to be started for
this suite. A process is specified as a [regex, hash] pair. The regular
expression must match a section in the `my.cnf` template (for example,
`qr/mysqld\./` corresponds to all `mysqld` processes), the hash contains
these options:

<table><tbody><tr><td><code>SORT</code></td><td>a number. Processes are started in the order of increasing <code>SORT</code> values (and stopped in the reverse order). <code>mysqld</code> has number 300.</td></tr>
<tr><td><code>START</code></td><td>a function to start a process. It takes two arguments, <code>My::Config::Group</code> and <code>My::Test</code>. If <code>START</code> is undefined a process will not be started.</td></tr>
<tr><td><code>WAIT</code></td><td>a function to wait for the process to be started. It takes <code>My::Config::Group</code> as an argument. Internally mtr first invokes <code>START</code> for all processes, then <code>WAIT</code> for all started processes.</td></tr>
</tbody></table>

```sql
sub servers {(
    qr/^foo$/ => { SORT => 200,  # start foo before mysqld
                   START => \&start_foo,
                   WAIT => \&wait_foo }
)}
```

See the <em>sphinx</em> suite for a working example.

A `list_cases()` method returns a complete list of tests for this suite. By
default it will be the list of files that have `.test` extension, but without
the extension. This list will be filtered by mtr, subject to different mtr
options (`--big-test`, `--start-from`, etc), the suite object does not have
to do it.

A `start_test()` method starts one test process, by default it will be
`mysqltest`.  See the <em>unit</em> suite for a working example of
`list_cases()` and `start_test()` methods.

A `skip_combinations()` method returns a hash that maps file names (where
combinations are defined) to a list of combinations that should be skipped. As
a special case, it can disable a complete file by using a string instead of a
hash. For example

```sql
sub skip_combinations {(
    'combinations' => [ 'mix', 'rpl' ],
    'inc/many.combinations' => [ 'a', 'bb', 'c' ],
    'windows.inc' => "Not on windows",
)}
```

The last line will cause all tests of this suite that include `windows.inc`
to be skipped with the reason being "Not on windows".

An `is_default()` method returns 1 if this particular suite should be run by default, when the `mysql-test-run.pl` script is run without explicitly specified test suites or test cases.

---

## `*.sh` files

For every test file `sometest.test` mtr looks for `sometest-master.sh` and
`sometest-slave.sh`. If either of these files is found, it will be run before
the test itself.

---

## `*.require` files

These files are obsolete. Do not use them anymore. If you need to skip a test
use the `skip` command instead.

---

## `*.rdiff` files

These files also define what the test result should be. But unlike `*.result`
files, they contain a patch that should be applied to one result file to create
a new result file. This is very useful when a result of some test in one
combination differs slightly from the result of the same test, but in another
combination. Or when a result of a test in an overlay differs from the test
result in the overlayed suite.

It is quite difficult to edit `.rdiff` files to update them after the test
file has changed. But luckily, it is never needed. When a test fails, mtr
creates a `.reject` file. Having it, one can create `.rdiff` file as easy
as (for example)

```sql
diff -u r/foo.result r/foo.reject > r/foo,comb.rdiff
or
diff -u r/foo.result r/foo,comb.reject > r/foo,comb.rdiff
```

An example:

```sql
diff -u suite/sys_vars/r/sysvars_server_notembedded.result suite/sys_vars/r/sysvars_server_notembedded,32bit.reject > suite/sys_vars/r/sysvars_server_notembedded,32bit.rdiff
```

Because a combination can be part of the `.result` or `.rdiff` file name,
mtr has to look in many different places for a test result. For example,
consider a test `foo.test` in the combination pair `aa,bb`, that is run
in the overlay <em>rty</em> of the suite <em>qwe</em>, in other words, for the test that
mtr prints as

```sql
qwe-rty.foo 'aa,bb'                     [ pass ]
```

For this test a result can be in

- either `.rdiff` or `.result` file
- either in the overlay "`rty/`" or in the overlayed suite "`qwe/`"
- with or without combinations in the file name ("`,a`", "`,b`",
  "`,a,b`", or nothing)

which means any of the following 15 file names can be used:

1 `rty/r/foo,aa,bb.result`
2 `rty/r/foo,aa,bb.rdiff`
3 `qwe/r/foo,aa,bb.result`
4 `qwe/r/foo,aa,bb.rdiff`
5 `rty/r/foo,aa.result`
6 `rty/r/foo,aa.rdiff`
7 `qwe/r/foo,aa.result`
8 `qwe/r/foo,aa.rdiff`
9 `rty/r/foo,bb.result`
10 `rty/r/foo,bb.rdiff`
11 `qwe/r/foo,bb.result`
12 `qwe/r/foo,bb.rdiff`
13 `rty/r/foo.result`
14 `rty/r/foo.rdiff`
15 `qwe/r/foo.result`

They are listed, precisely, in the order of preference, and mtr will walk that
list from top to bottom and the first file that is found will be used.

If this found file is a `.rdiff`, mtr continues walking down the list until
the first `.result` file is found. A `.rdiff` is applied to that
`.result`.

---

## `valgrind.supp` file

This file defines valgrind suppressions, and it is used when mtr is started
with a `--valgrind` option.
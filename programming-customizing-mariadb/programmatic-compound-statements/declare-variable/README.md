# DECLARE Variable

## Syntax

```sql
DECLARE var_name [, var_name] ... [[ROW] TYPE OF]] type [DEFAULT value]
```

## Description

This statement is used to declare local variables within [stored programs](/kb/en/stored-programs-and-views/). To
provide a default value for the variable, include a `DEFAULT` clause. The
value can be specified as an expression (even subqueries are permitted); it need not be a constant. If the
`DEFAULT` clause is missing, the initial value is `NULL`.

Local variables are treated like stored routine parameters with respect to data
type and overflow checking.  See [CREATE PROCEDURE](/programming-customizing-mariadb/stored-routines/stored-procedures/create-procedure).

Local variables must be declared before `CONDITION`s, [CURSORs](/kb/en/programmatic-and-compound-statements-cursors/) and `HANDLER`s.

Local variable names are not case sensitive.

The scope of a local variable is within the `BEGIN ... END` block where it is
declared. The variable can be referred to in blocks nested within the declaring
block, except those blocks that declare a variable with the same name.

### TYPE OF / ROW TYPE OF

##### MariaDB starting with [10.3](/kb/en/what-is-mariadb-103/)

`TYPE OF` and `ROW TYPE OF` anchored data types for stored routines were introduced in [MariaDB 10.3](/kb/en/what-is-mariadb-103/).

Anchored data types allow a data type to be defined based on another object, such as a table row, rather than specifically set in the declaration. If the anchor object changes, so will the anchored data type. This can lead to routines being easier to maintain, so that if the data type in the table is changed, it will automatically be changed in the routine as well.

Variables declared with `ROW TYPE OF` will have the same features as implicit [ROW](/columns-storage-engines-and-plugins/data-types/string-data-types/row) variables. It is not possible to use `ROW TYPE OF` variables in a [LIMIT](/sql-statements-structure/sql-statements/data-manipulation/selecting-data/limit) clause.

The real data type of `TYPE OF` and `ROW TYPE OF table_name` will become known at the very beginning of the stored routine call. [ALTER TABLE](/sql-statements-structure/sql-statements/data-definition/alter/alter-table) or [DROP TABLE](/sql-statements-structure/sql-statements/data-definition/drop/drop-table) statements performed inside the current routine on the tables that appear in anchors won't affect the data type of the anchored variables, even if the variable is declared after an [ALTER TABLE](/sql-statements-structure/sql-statements/data-definition/alter/alter-table) or [DROP TABLE](/sql-statements-structure/sql-statements/data-definition/drop/drop-table) statement.

The real data type of a `ROW TYPE OF cursor_name` variable will become known when execution enters into the block where the variable is declared. Data type instantiation will happen only once. In a cursor `ROW TYPE OF` variable that is declared inside a loop, its data type will become known on the very first iteration and won't change on further loop iterations.

The tables referenced in `TYPE OF` and `ROW TYPE OF` declarations will be checked for existence at the beginning of the stored routine call. [CREATE PROCEDURE](/programming-customizing-mariadb/stored-routines/stored-procedures/create-procedure) or [CREATE FUNCTION](/sql-statements-structure/sql-statements/data-definition/create/create-function) will not check the referenced tables for existence.

## Examples

`TYPE OF` and `ROW TYPE OF` from [MariaDB 10.3](/kb/en/what-is-mariadb-103/):

```sql
DECLARE tmp TYPE OF t1.a; -- Get the data type from the column {{a}} in the table {{t1}}

DECLARE rec1 ROW TYPE OF t1; -- Get the row data type from the table {{t1}}

DECLARE rec2 ROW TYPE OF cur1; -- Get the row data type from the cursor {{cur1}}
```

## See Also

- [User-Defined variables](/sql-statements-structure/sql-language-structure/user-defined-variables)
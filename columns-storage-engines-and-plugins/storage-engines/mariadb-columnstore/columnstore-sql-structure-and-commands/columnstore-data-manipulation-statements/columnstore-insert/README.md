# ColumnStore Insert

The INSERT statement allows you to add data to tables.

## Syntax

```sql
INSERT 
 INTO tbl_name [(col,...)]
 {VALUES | VALUE} ({expr | DEFAULT},...),(...),...
```

The following statement inserts a row with all column values into the <em>customer</em> table:

```sql
INSERT INTO customer (custno, custname, custaddress, phoneno, cardnumber, comments) 
  VALUES (12, ‘JohnSmith’, ‘100 First Street, Dallas’, ‘(214) 555-1212’,100, ‘On Time’)
```

The following statement inserts two rows with all column values into the <em>customer</em> table:

```sql
INSERT INTO customer (custno, custname, custaddress, phoneno, cardnumber, comments) VALUES 
  (12, ‘JohnSmith’, ‘100 First Street, Dallas’, ‘(214) 555-1212’,100, ‘On Time’),
  (13, ‘John Q Public’, ‘200 Second Street, Dallas’, ‘(972) 555-1234’, 200, ‘LatePayment’);
```

### INSERT SELECT

With INSERT ... SELECT, you can quickly insert many rows into a table from one or more other tables.

- ColumnStore ignores the ON DUPLICATE KEY clause.
- Non-transactional INSERT ... SELECT is directed to ColumnStores cpimport tool by default, which significantly increases performance.
- Transactional INSERT ... SELECT statements (that is with AUTOCOMMIT off or after a START TRANSACTION) are processed through normal DML processes.

### Autoincrement

Autoincrement is not supported by MariaDB ColumnStore.
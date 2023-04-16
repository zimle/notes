# SQL-Developer

## Query hints to modify the output

In SQL-Developer, there are useful [query hints](https://www.thatjeffsmith.com/archive/2012/05/formatting-query-results-to-csv-in-oracle-sql-developer/) that modify the select output:

```sql
-- /*insert*/ produces output like `Insert into scott.emp (...) values (...)
SELECT /*insert*/ * FROM scott.emp;
SELECT /*csv*/ * FROM scott.emp;
SELECT /*xml*/ * FROM scott.emp;
SELECT /*html*/ * FROM scott.emp;
SELECT /*delimited*/ * FROM scott.emp;
SELECT /*loader*/ * FROM scott.emp;
SELECT /*fixed*/ * FROM scott.emp;
SELECT /*text*/ * FROM scott.emp;
```

## Trivia

- `exit` and `quit` are [synonymous](https://forums.oracle.com/ords/apexds/post/difference-between-quit-exit-5381), both terminating the cli.
- some [ora faq wiki](https://www.orafaq.com/wiki/SQL*Plus_FAQ)
- `SHIFT+F` with cursor on a table returns a pop up with table infos

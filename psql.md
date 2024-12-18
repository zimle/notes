# PSQL

The most important `psql` command is

```bash
# returns all psql commands with short description
 \?
 ```

This enables for searching the right command. See also the [manual](https://www.postgresql.org/docs/current/app-psql.html).

A nice site with many small tips is by [Laetitia Avrot](https://psql-tips.org/).

## Pager

`psql` per default uses the pager defined in the environment variables, which usually is less on Linux. When querying many columns which dont fit the screen, the output gets messy for the first columns. Scrolling to the right by pressing the right arrow gives a good result. However, the view gets messy when jumping back via left arrow key.

To circumvent this, either

- use `export PAGER='less -S' (bad, because it effects other programs)
- within a `psql` session, set `\setenv PAGER 'less -S'`
- create a `.psqlrc` file in your home directory if not existent and append `\setenv PAGER 'less -S'`. Note that comments in `.psqlrc` are done via `--`

## Autocommit

```sql
-- see within psql session whether autocommit is on
\echo :AUTOCOMMIT
-- within psql session set autocommit off or on
\set AUTOCOMMIT off
```

## Executing sql files

```sql
-- from bash
psql -h localhost -U my_user -d my_db -f my/relative/or/absolute/path.sql
-- from session
\i /my/absolute/path
```

Note that variables set either from command line via `-v out_dir=~/garbage` or in `psql` like `\set out_dir ~/garbage` will be passed to all scripts called from psql like `\i my_sql_where_out_dir_is_used.sql`.

## Writing results to a file

To write all query results to a file, use `\o`:

```sql
-- tell psql to write all following query results to the file my-results.txt
\o /home/my-user/pg-files/my-results.txt
select * from test1;
select * from test2;
-- tell psql to stop writing to file and use stdout
\o
```

To use a specific format, either use `\pset format csv` (and potentially `\pset csv_fieldsep ','`) within `psql` or `psql --csv -f my_query.sql` when calling `psql` to execute a file.

## Differences to Oracle's SQL*Plus

For general comparison between `SQL*Plus` and `psql`, also have a look at the [Amazon documentation](https://aws.amazon.com/blogs/database/postgresql-psql-client-tool-commands-equivalent-to-oracle-sqlplus-client-tool/).

|description|psql|sql*plus|comment|
|-|-|-|-|
|Define variable|\set AUSGABEDATEI TEMP|DEFINE AUSGABEDATEI = TEMP|-|
|Define variable from script input|Must be provided with call like psql -v OUT="'temp'", -v IN=orig -f my_stmt.sql|DEFINE OUT = &1|Blog post postgres <https://www.depesz.com/2023/05/28/variables-in-psql-how-to-use-them/>|
|Use variable|select :'AUSGABEDATEI';|select '&AUSGABEDATEI' from dual|quoting in psql itself might not work (?) when using \o :path:file|

## Trivia

- to see the internal sql commands of `psql`, e.g.

    ```bash
    postgres=# \l
    ********* QUERY **********
    SELECT d.datname as "Name",
        pg_catalog.pg_get_userbyid(d.datdba) as "Owner",
        pg_catalog.pg_encoding_to_char(d.encoding) as "Encoding",
        d.datcollate as "Collate",
        d.datctype as "Ctype",
        pg_catalog.array_to_string(d.datacl, E'\n') AS "Access privileges"
    FROM pg_catalog.pg_database d
    ORDER BY 1;
    **************************
    ```

    use one of the following:

    ```bash
    # use the flag -E when starting psql
    psql -h localhost -U my_user -d my_database -E
    # during a psql session, enable via
    \set ECHO_HIDDEN 1
    # during a psql session, disable via
    \set ECHO_HIDDEN 0
    # during a psql session, do not execute command but show what psql would execute
    \set ECHO_HIDDEN noexec
    ```

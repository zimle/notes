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

/mnt/d/projects/gb-reengineering/spring-batch-prototype/env/sql/schema-postgresql.sql

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

## Trivia

- to see the internal sql commands `psql`, e.g.

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

# PSQL

The most important `psql` command is

```bash
# returns all psql commands with short description
 \?
 ```

This enables for searching the right command.

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

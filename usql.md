# usql

[usql](https://github.com/xo/usql) is a cli tool inspired by [psql](https://www.postgresql.org/docs/current/app-psql.html), only supporting many databases.
Hence the name`u`sql for `universal`.
It is written in `go` and hence uses the corresponding go sql drivers.

To install it, grab a binary from the release site and put it into your path.

As `psql`, it also uses backslash commands:

```bash
$ usql
Type "help" for help.

(not connected)=> \?
General
  \q                                   quit usql
  \copyright                           show usql usage and distribution terms
  \drivers                             display information about available database drivers

Query Execute
  \g [(OPTIONS)] [FILE] or ;           execute query (and send results to file or |pipe)
  \crosstabview [(OPTIONS)] [COLUMNS]  execute query and display results in crosstab
  \G [(OPTIONS)] [FILE]                as \g, but forces vertical output mode
  \gexec                               execute query and execute each value of the result
  \gset [PREFIX]                       execute query and store results in usql variables
  \gx [(OPTIONS)] [FILE]               as \g, but forces expanded output mode
  \watch [(OPTIONS)] [DURATION]        execute query every specified interval

Query Buffer
  \e [FILE] [LINE]                     edit the query buffer (or file) with external editor
  \p                                   show the contents of the query buffer
  \raw                                 show the raw (non-interpolated) contents of the query buffer
  \r                                   reset (clear) the query buffer
  \w FILE                              write query buffer to file

Help
  \? [commands]                        show help on backslash commands
  \? options                           show help on usql command-line options
  \? variables                         show help on special variables

Input/Output
  \copy SRC DST QUERY TABLE            copy query from source url to table on destination url
  \copy SRC DST QUERY TABLE(A,...)     copy query from source url to columns of table on destination url
  \echo [-n] [STRING]                  write string to standard output (-n for no newline)
  \qecho [-n] [STRING]                 write string to \o output stream (-n for no newline)
  \warn [-n] [STRING]                  write string to standard error (-n for no newline)
  \o [FILE]                            send all query results to file or |pipe
  \i FILE                              execute commands from file
  \ir FILE                             as \i, but relative to location of current script

Informational
  \d[S+] [NAME]                        list tables, views, and sequences or describe table, view, sequence, or index
  \da[S+] [PATTERN]                    list aggregates
  \df[S+] [PATTERN]                    list functions
  \di[S+] [PATTERN]                    list indexes
  \dm[S+] [PATTERN]                    list materialized views
  \dn[S+] [PATTERN]                    list schemas
  \dp[S] [PATTERN]                     list table, view, and sequence access privileges
  \ds[S+] [PATTERN]                    list sequences
  \dt[S+] [PATTERN]                    list tables
  \dv[S+] [PATTERN]                    list views
  \l[+]                                list databases
  \ss[+] [TABLE|QUERY] [k]             show stats for a table or a query

Formatting
  \pset [NAME [VALUE]]                 set table output option
  \a                                   toggle between unaligned and aligned output mode
  \C [STRING]                          set table title, or unset if none
  \f [STRING]                          show or set field separator for unaligned query output
  \H                                   toggle HTML output mode
  \t [on|off]                          show only rows
  \T [STRING]                          set HTML <table> tag attributes, or unset if none
  \x [on|off|auto]                     toggle expanded output

Transaction
  \begin                               begin a transaction
  \begin [-read-only] [ISOLATION]      begin a transaction with isolation level
  \commit                              commit current transaction
  \rollback                            rollback (abort) current transaction

Connection
  \c DSN                               connect to database url
  \c DRIVER PARAMS...                  connect to database with driver and parameters
  \Z                                   close database connection
  \password [USERNAME]                 change the password for a user
  \conninfo                            display information about the current database connection

Operating System
  \cd [DIR]                            change the current working directory
  \setenv NAME [VALUE]                 set or unset environment variable
  \! [COMMAND]                         execute command in shell or start interactive shell
  \timing [on|off]                     toggle timing of commands

Variables
  \prompt [-TYPE] <VAR> [PROMPT]       prompt user to set variable
  \set [NAME [VALUE]]                  set internal variable, or list all if no parameters
  \unset NAME                          unset (delete) internal variable
```

## cli options

```bash
$ usql --help
usql, the universal command-line interface for SQL databases

Usage:
  usql [OPTIONS]... [DSN]

Arguments:
  DSN                            database url

Options:
  -c, --command=COMMAND ...    run only single command (SQL or internal) and exit
  -f, --file=FILE ...          execute commands from file and exit
  -w, --no-password            never prompt for password
  -X, --no-rc                  do not read start up file
  -o, --out=OUT                output file
  -W, --password               force password prompt (should happen automatically)
  -1, --single-transaction     execute as a single transaction (if non-interactive)
  -v, --set=, --variable=NAME=VALUE ...
                               set variable NAME to VALUE
  -P, --pset=VAR[=ARG] ...     set printing option VAR to ARG (see \pset command)
  -F, --field-separator=FIELD-SEPARATOR ...
                               field separator for unaligned output (default, "|")
  -R, --record-separator=RECORD-SEPARATOR ...
                               record separator for unaligned output (default, \n)
  -T, --table-attr=TABLE-ATTR ...
                               set HTML table tag attributes (e.g., width, border)
  -A, --no-align               unaligned table output mode
  -H, --html                   HTML table output mode
  -t, --tuples-only            print rows only
  -x, --expanded               turn on expanded table output
  -z, --field-separator-zero   set field separator for unaligned output to zero byte
  -0, --record-separator-zero  set record separator for unaligned output to zero byte
  -J, --json                   JSON output mode
  -C, --csv                    CSV output mode
  -G, --vertical               vertical output mode
  -V, --version                display version and exit
```

## connection examples

```bash
# connect to a postgres database
$ usql pg://user:pass@host/dbname
$ usql pgsql://user:pass@host/dbname
$ usql postgres://user:pass@host:port/dbname
$ usql pg://
$ usql /var/run/postgresql
$ usql pg://user:pass@host/dbname?sslmode=disable # Connect without SSL

# connect to a mysql database
$ usql my://user:pass@host/dbname
$ usql mysql://user:pass@host:port/dbname
$ usql my://
$ usql /var/run/mysqld/mysqld.sock

# connect to a sqlserver database
$ usql sqlserver://user:pass@host/instancename/dbname
$ usql ms://user:pass@host/dbname
$ usql ms://user:pass@host/instancename/dbname
$ usql mssql://user:pass@host:port/dbname
$ usql ms://

# connect to a sqlserver database using Windows domain authentication
$ runas /user:ACME\wiley /netonly "usql mssql://host/dbname/"

# connect to a oracle database
$ usql or://user:pass@host/sid
$ usql oracle://user:pass@host:port/sid
$ usql or://

# connect to a cassandra database
$ usql ca://user:pass@host/keyspace
$ usql cassandra://host/keyspace
$ usql cql://host/
$ usql ca://

# connect to a sqlite database that exists on disk
$ usql dbname.sqlite3

# Note: when connecting to a SQLite database, if the "<driver>://" or
# "<driver>:" scheme/alias is omitted, the file must already exist on disk.
#
# if the file does not yet exist, the URL must incorporate file:, sq:, sqlite3:,
# or any other recognized sqlite3 driver alias to force usql to create a new,
# empty database at the specified path:
$ usql sq://path/to/dbname.sqlite3
$ usql sqlite3://path/to/dbname.sqlite3
$ usql file:/path/to/dbname.sqlite3

# connect to a adodb ole resource (windows only)
$ usql adodb://Microsoft.Jet.OLEDB.4.0/myfile.mdb
$ usql "adodb://Microsoft.ACE.OLEDB.12.0/?Extended+Properties=\"Text;HDR=NO;FMT=Delimited\""

# connect with ODBC driver (requires building with odbc tag)
$ cat /etc/odbcinst.ini
[DB2]
Description=DB2 driver
Driver=/opt/db2/clidriver/lib/libdb2.so
FileUsage = 1
DontDLClose = 1

[PostgreSQL ANSI]
Description=PostgreSQL ODBC driver (ANSI version)
Driver=psqlodbca.so
Setup=libodbcpsqlS.so
Debug=0
CommLog=1
UsageCount=1
# connect to db2, postgres databases using ODBC
$ usql odbc+DB2://user:pass@localhost/dbname
$ usql odbc+PostgreSQL+ANSI://user:pass@localhost/dbname?TraceFile=/path/to/trace.log
```

## Copy data between databases

Also see [good documentation](https://github.com/xo/usql#copying-between-databases). Here an example:

```bash
\copy oracle://ora_user:ora_pwd@localhost/sid pg://pg_user:pg_pwd@localhost:5432/net 'select c1, c2, c3 from tableA where c4 = \'my_value\'' 'table_a(c1, c2, c3)'
```

Some helpful command to create a usql copy command from one database to postgres, where there might be more columns in the source table:

```sql
-- create usql copy command to copy values for all but some columns to postgres
with comma_sep as (
    select
    string_agg(column_name, ',' order by ordinal_position) as agg
    from information_schema.columns
    where table_name = 'my_table'
)
select
concat('\copy oracle://ora_user:ora_pwd@localhost/my_sid pg://pg_user:pg_pwd@localhost:5432/my_db'
' ''select ' || agg || ' from my_table where filter_column = \''111\''' || '''',  ' ''my_table(' || agg || ')''')
from comma_sep
```

## Trivia

-

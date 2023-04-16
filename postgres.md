# Postgres

## Meta Informations

### System Catalogues / Meta tables

See the postgres docs for [system catalogues](https://www.postgresql.org/docs/current/catalogs.html) and [system monitoring](https://www.postgresql.org/docs/15/monitoring-stats.html)

### Table sizes / relation sizes

Difference between the size functions `pg_table_size`, `pg_relation_size`, `pg_indexes_size` and `pg_total_relation_size`, see also the [storage file layout](https://www.postgresql.org/docs/current/storage-file-layout.html).

As an example query abstracted from [so post](https://stackoverflow.com/questions/41991380/whats-the-difference-between-pg-table-size-pg-relation-size-pg-total-relatio), look at

```sql
-- Each table and index is stored in separate file
-- Additionally, each table and index has a free space map (stores free space available in relation)
-- Also, there is a visibility map file to track which pages are known to have no dead tuples
-- Unlogged tables and indexes have an init file
SELECT
 oid::regclass,
 relkind as relation_type,
 pg_relation_size(oid, 'main') as main, -- main table file with beef inside
 pg_relation_size(oid, 'fsm') as fsm, -- free space map file
 pg_relation_size(oid, 'vm') as vm, -- visibility map file
 pg_relation_size(oid, 'init') as init, -- for unlogged tables and indexes
 pg_table_size(oid) as table_size_bytes, -- sum of all 4 files + toast_size
 pg_size_pretty(pg_table_size(oid)) as table_size,
 pg_table_size(oid) - pg_relation_size(oid, 'main') - pg_relation_size(oid, 'fsm') - pg_relation_size(oid, 'vm') - pg_relation_size(oid, 'init') as toast_size_bytes, -- sum of all 4 files + toast_size
 pg_size_pretty(pg_table_size(oid) - pg_relation_size(oid, 'main') - pg_relation_size(oid, 'fsm') - pg_relation_size(oid, 'vm') - pg_relation_size(oid, 'init')) as toast_size,
 pg_indexes_size(oid) as index_size_bytes, -- sum of all 4 files
 pg_size_pretty(pg_indexes_size(oid)) as index_size,
 pg_total_relation_size(oid) as total_size_bytes, -- sum of table and related indexes
 pg_size_pretty(pg_total_relation_size(oid)) as total_size
FROM pg_class
ORDER BY pg_total_relation_size(oid) DESC;
```

### Column ordering / alignment

[Column alignment](https://www.2ndquadrant.com/en/blog/on-rocks-and-sand/) matters in postgres. Note that by following the rule [8-byte columns first, then 4-bytes, 2-bytes and 1-byte columns last](https://stackoverflow.com/questions/2966524/calculating-and-saving-space-in-postgresql/7431468#7431468) , one can achieve fewer disk usage, therefore memory usage and higher performance. As a rule of thumb, to get the best table layout, run

```sql
SELECT a.attname, t.typname, t.typalign, t.typlen
  FROM pg_class c
  JOIN pg_attribute a ON (a.attrelid = c.oid)
  JOIN pg_type t ON (t.oid = a.atttypid)
 WHERE c.relname = 'user_order'
   AND a.attnum >= 0
 ORDER BY t.typlen DESC;
```

Note also the help command `p1` in [postgres_dba](https://github.com/NikolayS/postgres_dba)

### Rows per page / block

To estimate how many rows of a table fit into a page, one can run -- following a post about [brin indexes](https://j-carson.github.io/2022/10/02/brin/)

```sql
/*
 * Estimate the amount of storage used by one row
 * of impariment tables and number of rows per page
 */
with bytes_per_row as (
    select
        pt.tablename,
        24 + -- base overhead for any row
        SUM(
            CASE
            WHEN attlen > 0
            THEN attlen  -- storage size of column type when not null -- but attlen == -1 means the column can be TOASTed
            ELSE 18 -- overhead for a TOASTed column
            end
        ) as bytes_per_row
    from pg_tables pt
        JOIN pg_class pc
            ON pt.tablename = pc.relname
        JOIN pg_attribute pa
            ON pa.attrelid = pc.oid
    where pt.tablename like 'i9%'
    or pt.tablename like 'ifrs9%'
    or pt.tablename like 'ja%'
    group by pt.tablename
)
select
    tablename,
    bytes_per_row,
    8192 / bytes_per_row as rows_per_page -- for integral types, division truncates the result towards zero
from bytes_per_row
order by tablename;
```

## Toasts

Toast stands for [The Oversized-Attribute Storage Technique](https://www.postgresql.org/docs/current/storage-toast.html). As a row must fit into a page of 8kB, bigger rows (or rows with big values) use the toast technique (similar as #oracle allows only 8kB blocks and uses row chaining if records get too big). Toasting is defined on the data type and not on the row/tuple

To see the storage types of attributes `pg_attribute`, run the [sql](https://www.dbi-services.com/blog/toasting-strategies-in-postgresql/)

```sql
-- see the storage types of the columns of the relation i9contract
select attname, atttypid::regtype,
 case attstorage when 'p' then 'plain' -- compression, but no out of line storage
                when 'e' then 'external' -- no compression and out of line storage
                when 'm' then 'main' -- no compression
                when 'x' then 'extended' -- compression and out of line storage
 end AS strategy
 from pg_attribute
 where attrelid = 'i9contract'::regclass and attnum > 0;
-- see the storage type of any column type
select typname, typstorage
from pg_type
order by typname
```

To see the [toast tables](https://www.dbi-services.com/blog/toasting-in-postgresql-toast-tables/), we can see that they have an own schema:

```sql
-- get all schemata
select *
from information_schema.schemata;

-- get all toast relations ordered by size
SELECT oid::regclass,
       oid,
       reltoastrelid::regclass, -- usually pg_toast.pg_toast_ + oid, but be careful!
       pg_relation_size(reltoastrelid) AS toast_size
FROM pg_class
WHERE relkind = 'r'
  AND reltoastrelid <> 0
ORDER BY pg_relation_size(reltoastrelid) DESC;
```

Note that one is able to change the compression algorithm to [lz4](https://en.wikipedia.org/wiki/LZ4_(compression_algorithm)), which may be much more performant than the standard `pglz`. This can be done via

```sql
-- set compression from postgres14 on
set default_toast_compression = lz4;
-- column based:
alter table t alter column b set compression lz4;
```

- [compression](https://www.dbi-services.com/blog/toasting-in-postgresql-lets-see-it-in-action/)
- [toasting strategies in postgres part 1](https://www.dbi-services.com/blog/toasting-strategies-in-postgresql/)
- [toasting in postgres, toast tables part 2](https://www.dbi-services.com/blog/toasting-in-postgresql-toast-tables/)
- [TOASTing in PostgreSQL, let’s see it in action part 3](https://www.dbi-services.com/blog/toasting-in-postgresql-lets-see-it-in-action/)

## Cleanup, Vacuum, table sizes, disk space

See the [cybertec link](https://www.cybertec-postgresql.com/en/vacuum-does-not-shrink-my-postgresql-table/) for `VACUUM`. Example

```sql
test=# VACUUM VERBOSE t_test
INFO: vacuuming "test.public.t_test"
INFO: finished vacuuming "test.public.t_test": index scans: 0
pages: 0 removed, 1 remain, 1 scanned (100.00% of total)
tuples: 3 removed, 6 remain, 0 are dead but not yet removable
removable cutoff: 767, which was 0 XIDs old when operation ended
new relfrozenxid: 764, which is 1 XIDs ahead of previous value
index scan not needed: 0 pages from table (0.00% of total) had 0 dead item identifiers removed
avg read rate: 22.790 MB/s, avg write rate: 27.348 MB/s
buffer usage: 6 hits, 5 misses, 6 dirtied
WAL usage: 3 records, 3 full page images, 14224 bytes
system usage: CPU: user: 0.00 s, system: 0.00 s, elapsed: 0.00 s
VACUUM
```

From [cybertec](https://www.cybertec-postgresql.com/en/vacuum-does-not-shrink-my-postgresql-table/)
> `VACUUM` is actively looking for rows which are not seen by anyone anymore. Those rows can be in the middle of the data file somewhere. What happens is that `VACUUM` allows PostgreSQL to reuse that space – however, it does not return that space to the operating system. It can’t do that because if you have a datafile that is 1 GB in size, you can’t simply return “the middle of the file” to the operating system in case it is empty – there is no file system operation which supports that. Instead, PostgreSQL has to remember this free space and reuse it later.

An Exception is the case where every row is deleted (like in `Truncate`).

To free up space disk for the OS, use `VACUUM FULL`, which requires a table lock. Else, use [pg_squeeze](https://github.com/cybertec-postgresql/pg_squeeze). If `VACUUM` does not remove dead rows, see this [post](https://www.cybertec-postgresql.com/en/reasons-why-vacuum-wont-remove-dead-rows/)

### Example

Table `my_table`:

- pg_table_size 1502 MB
- pg_indexes_size 2475 MB
- pg_total_relation_size 3977 MB

Run `VACUUM VERBOSE my_table;`

```
gb_reengineering=> VACUUM VERBOSE my_table;                                                                                                                                                                                                                            INFO:  vacuuming "public.my_table"                                                                                                                                                                                                                                     INFO:  launched 2 parallel vacuum workers for index cleanup (planned: 2)                                                                                                                                                                                                       INFO:  index "xie_3773" now contains 6036 row versions in 48197 pages                                                                                                                                                                                                          DETAIL:  0 index row versions were removed.                                                                                                                                                                                                                                    46180 index pages have been deleted, 46180 are currently reusable.                                                                                                                                                                                                             CPU: user: 0.15 s, system: 1.01 s, elapsed: 1.65 s.                                                                                                                                                                                                                            INFO:  index "xie_3738" now contains 108673 row versions in 41172 pages                                                                                                                                                                                                        DETAIL:  0 index row versions were removed.                                                                                                                                                                                                                                    37099 index pages have been deleted, 37099 are currently reusable.                                                                                                                                                                                                             CPU: user: 0.10 s, system: 0.84 s, elapsed: 1.53 s.                                                                                                                                                                                                                            INFO:  index "xpk_3660" now contains 1287000 row versions in 227324 pages                                                                                                                                                                                                      DETAIL:  0 index row versions were removed.                                                                                                                                                                                                                                    110402 index pages have been deleted, 110402 are currently reusable.                                                                                                                                                                                                           CPU: user: 0.34 s, system: 2.37 s, elapsed: 4.89 s.                                                                                                                                                                                                                            INFO:  "my_table": found 0 removable, 117853 nonremovable row versions in 4210 out of 192214 pages                                                                                                                                                                     DETAIL:  0 dead row versions cannot be removed yet, oldest xmin: 11804852                                                                                                                                                                                                      There were 27 unused item identifiers.                                                                                                                                                                                                                                         Skipped 0 pages due to buffer pins, 146244 frozen pages.                                                                                                                                                                                                                       0 pages are entirely empty.                                                                                                                                                                                                                                                    CPU: user: 0.46 s, system: 2.46 s, elapsed: 5.13 s.                                                                                                                                                                                                                            VACUUM
```

Afterwards, we have for `my_table` no changes in space needed (files stay the same):

- pg_table_size 1502 MB
- pg_indexes_size 2475 MB
- pg_total_relation_size 3977 MB

Now we run a `VACUUM FULL VERBOSE my_table;`:

```
gb_reengineering=> VACUUM FULL VERBOSE my_table;                                                                                                                                                                                                                       INFO:  vacuuming "public.my_table"                                                                                                                                                                                                                                     INFO:  "my_table": found 0 removable, 1287000 nonremovable row versions in 192214 pages                                                                                                                                                                                DETAIL:  0 dead row versions cannot be removed yet.                                                                                                                                                                                                                            CPU: user: 1.26 s, system: 4.32 s, elapsed: 9.59 s.                                                                                                                                                                                                                            VACUUM
```

Now, `my_table` is much smaller:

- pg_table_size 359 MB
- pg_indexes_size 102 MB
- pg_total_relation_size 462 MB

To see all used disk space of the db (?), run

```sql
-- total size of db
select
    sum(pg_total_relation_size(oid)) as total_db_bytes,
    pg_size_pretty(sum(pg_total_relation_size(oid))) as total_db_bytes
from pg_catalog.pg_class;
```

It yields around 13 GB. Then run `VACUUM FULL VERBOSE`. Afterwards, we get a size of 4013 MB.

## Creating Fake data / Test data

```sql
-- insert random numers generate_series
insert into t (id)
select random() * generate_series
from pg_catalog.generate_series(1, 10000)
```

## Setting up database

Log in on the machine as admin, e.g.

```bash
# docker exec -i container_id "/bin/bash" for docker container
su postgres
psql
```

Then, as admin in psql, do the following:

```sql
-- as postgres or superuser or dba or whatever called
create database my_db;
create user my_db with encrypted password 'pwd';
grant all privileges on database my_db to my_db;
-- as user my_db in database my_db
create schema my_db;
```

If tables in my_db were created with user my_db and user my_db2 should be allowed to access these tables:

```sql
-- as user my_db
grant all privileges on all tables in schema my_db to my_db2;
```

## Monitoring / Checkup

For Performance and Checkups, see [postgres-checkup](https://gitlab.com/postgres-ai/postgres-checkup) and [postgres_dba](https://github.com/NikolayS/postgres_dba).

## Types

- [Math operators](https://www.postgresql.org/docs/current/functions-math.html)

## pg_stat_activity

References include

- [depesz](https://www.depesz.com/2022/07/05/understanding-pg_stat_activity/

## Bulk loading

References include

- [Postgres Populating the database](https://www.postgresql.org/docs/current/populate.html)
- [Cybertec WAL and checkpointing](https://www.cybertec-postgresql.com/en/checkpoint-distance-and-amount-of-wal/)
- [Postgres Wiki](https://wiki.postgresql.org/wiki/Tuning_Your_PostgreSQL_Server)
- [Postgrespro - Buffer Cache](https://postgrespro.com/blog/pgsql/5967951)
- [Postgrespro - WAL](https://postgrespro.com/blog/pgsql/5967958)
- [Postgrespro - Checkpoints](https://postgrespro.com/blog/pgsql/5967965)
- [Postgrespro - Tuning](https://postgrespro.com/blog/pgsql/5967974)
- [EnterpriseDB - Tuning](https://www.enterprisedb.com/postgres-tutorials/introduction-postgresql-performance-tuning-and-optimization)

### Parameters to tune

- max_wal_size
- min_wal_size
- wal_buffers
- checkpoint_timeout
- synchronous_commit
- bgwriter_lru_maxpages
- bgwriter_delay
- bgwriter_lru_multiplier

### Commands to insert / bulk load

In descending effectivity

1. `copy from (file, sdtdin, program)` is the king for populating tables
2. Batch inserts of the form

    ```sql
    insert into example (a, b, c) values (
        ((a_1,b_1,c_1),
        (a_2,b_2,c_2),
        ...,
        (a_n,b_n,c_n)
        )
    ```

    Note that certain postgres drivers like jdbc allow for automatic rewriting of the usual batch insert into this form by setting the property `reWriteBatchedInserts=true`.
3. Usual batch inserts
4. Entry-by-entry inserts

### Autovacuum

Disable Autovacuum before loading the data, and always activate it at the end.

### Indexes

Drop indexes (except unique constraints) and rebuild them after loading is done.

### WAL and checkpointing

- Checkpoints have high costs.......
  Increase checkpoint_timeout leads to less WAL: If a block/page changes for the first time (e.g. adding a row), the whole page has to written to the WAL, all subsequent changes are incremental. The closer the checkpoints are, the more "first times" will appear.
  Run

```sql
SELECT pg_size_pretty(pg_current_xlog_location() - '0/00000000'::pg_lsn);
```

  to get the size of the current WAL. Further tables:

```sql
SELECT * FROM pg_ls_waldir()
```

- Avoid write amplification: Even a table with a primary key can lead to siginificant write overhead if the key is uuid vs bigint, see [On the impact of full-page writes](https://www.2ndquadrant.com/en/blog/on-the-impact-of-full-page-writes/ "Permanent Link: On the impact of full-page writes") As uuids are random, postgres likely has to touch a new leaf index leaf page every write as opposed to the bigint, where it is likely that we stay in the same leaf index leaf page. Thus, much more must be written to the wal (every time postgres dirties a page, the whole must be written to wal)

### Run analyze afterwards

It does not quicken the loading, but running an `analyze` on a table after inserting / changing large parts of the table is essential for the query planner to use this table.

### Set huge_pages=on on Linux

See EnterpriseDB - Tuning](https://www.enterprisedb.com/postgres-tutorials/introduction-postgresql-performance-tuning-and-optimization)

### Synchronous commit

Turn `synchronous_commit=off`. The downside is that a client will receive a response that data is committed although it is not written to the WAL. In case of a crash, the client is in a false belief that his data is stored.

## Starting and stopping postgres

### Linux

- Start the PostgreSQL server

```bash
sudo service postgresql start
sudo service postgresql stop
```

### Windows

First, you need to find the PostgreSQL database directory, it can be something like `C:\Program Files\PostgreSQL\10.4\data`. Then open Command Prompt and execute this command:

```cmd
pg_ctl -D "C:\Program Files\PostgreSQL\9.6\data" start
```

- To stop the server

```cmd
pg_ctl -D "C:\Program Files\PostgreSQL\9.6\data" stop
```

- To restart the server:

```cmd
pg_ctl -D "C:\Program Files\PostgreSQL\9.6\data" restart
```

Another way:

- Open Run Window by `Winkey + R`
- Type `services.msc`
- Search Postgres service based on version installed.
- Click stop, start or restart the service option.

## Links

### Books

- [postgrespro](https://postgrespro.com/blog/pgsql/5969741)

## wal

- [On the impact of full-page writes](https://www.2ndquadrant.com/en/blog/on-the-impact-of-full-page-writes/)

## Schema / table layout

- on the right objectid type can be found here
  1. [Choosing a Postgres Primary Key](https://supabase.com/blog/choosing-a-postgres-primary-key#the-post-uuidv1v4-era-a-cambrian-explosion-of-identifiers)
  2. [column ordering](https://www.crunchydata.com/blog/choice-of-table-column-types-and-order-when-migrating-to-postgresql)
  3. [postgres number format](https://www.crunchydata.com/blog/choosing-a-postgresql-number-format)
  4. [Ordering Table Columns in PostgreSQL](https://docs.gitlab.com/ee/development/database/ordering_table_columns.html#ordering-table-columns-in-postgresql "Permalink") has a table of column sizes

## Sharding

- [citus](https://github.com/citusdata/citus)

## Moving data into postgres

- <https://github.com/bitdotioinc/pgsqlite>: Python library to import sqlite databases into postgres
- <https://github.com/tobymao/sqlglot>: Python library that parses and transpiles / optimizes from one sql dialect into another (see also [sqlparser-rs](https://github.com/sqlparser-rs/sqlparser-rs))
- <https://github.com/simonw/sqlite-utils>
- <https://pgloader.io/>

## Deleting

Steps that worked for me on **`Ubuntu 8.04.2`** to remove **`postgres 8.3`**

1. **List All Postgres related packages**

    ```bash
    dpkg -l | grep postgres

    ii  postgresql                            8.3.17-0ubuntu0.8.04.1           object-relational SQL database (latest versi
    ii  postgresql-8.3                        8.3.9-0ubuntu8.04                object-relational SQL database, version 8.3
    ii  postgresql-client                     8.3.9-0ubuntu8.04                front-end programs for PostgreSQL (latest ve
    ii  postgresql-client-8.3                 8.3.9-0ubuntu8.04                front-end programs for PostgreSQL 8.3
    ii  postgresql-client-common              87ubuntu2                        manager for multiple PostgreSQL client versi
    ii  postgresql-common                     87ubuntu2                        PostgreSQL database-cluster manager
    ii  postgresql-contrib                    8.3.9-0ubuntu8.04                additional facilities for PostgreSQL (latest
    ii  postgresql-contrib-8.3                8.3.9-0ubuntu8.04                additional facilities for PostgreSQL
    ```

2. **Remove all above listed**

    ```bash
    sudo apt-get --purge remove postgresql postgresql-8.3  postgresql-client  postgresql-client-8.3 postgresql-client-common postgresql-common  postgresql-contrib postgresql-contrib-8.3
    ```

3. **Remove the following folders**

    ```bash
    sudo rm -rf /var/lib/postgresql/
    sudo rm -rf /var/log/postgresql/
    sudo rm -rf /etc/postgresql/
    ```

4. **Remove the postgres user**:

    ```bash
    sudo deluser postgres
    ```

# expdp and impdp

`expdp` and `impdp` are Oracle tools to export and import data between Oracle databases.

## Links

- <https://oracle-base.com/articles/10g/oracle-data-pump-10g#TableExpImp>
- <https://docs.oracle.com/en/database/oracle/oracle-database/19/sutil/datapump-import-utility.html>

## Simple export / import of schema

```bash
# export of schema
expdp USERID=USER_WITH_SUFFICIENT_GRANTS\ # needs access to schema and directory specified below
    schemas=EXPORT_TARGET\
    DIRECTORY=DATA_PUMP_DIR\
    DUMPFILE=basis.dmp\
    REUSE_DUMPFILES=Y\
    EXCLUDE=STATISTICS\ # because it has a big performance penalty
    logfile=DATA_PUMP_DIR:basis.log

# import of all tables
impdp USERID=USER_WITH_SUFFICIENT_GRANTS\ # needs access to directory below
    REMAP_SCHEMA=EXPORT_TARGET:IMPORT_TARGET\
    #REMAP_TABLESPACE=TS_SOURCE:TS_HEAD
    DIRECTORY=DATA_PUMP_DIR\
    DUMPFILE=basis.dmp\
    EXCLUDE=STATISTICS\ # because it has a big performance penalty
    logfile=DATA_PUMP_DIR:import_basis.log\
```

## Export via dblink

First, you need to create a dblink a la

```sql
CREATE PUBLIC DATABASE LINK REMOTE_DB CONNECT TO REMOTE_DB_USER IDENTIFIED BY MY_SECRET_PASSWORD USING '10.100.5.47:1521/REMOTE_SID';
```

As we select some tables using quotes in an IN clause, we create a par file with

```bash
network_link=REMOTE_DB
directory=DATA_PUMP_DIR
dumpfile=via_link.dmp
logfile=DATA_PUMP_DIR:expdp.log
content=METADATA_ONLY
TABLES=MY_TABLE_1
TABLES=MY_TABLE_2
TABLES=MY_TABLE_3
```

and import it locally via

## Find out schema, user and table space names for mapping

One can convert the dump file to an DDL sql file via

```bash
impdp username/pwd directory=name dumpfile=name sqlfile=name.sql
```

## Full database export

Beispiel aus der [Oracle Doku](https://docs.oracle.com/database/121/SUTIL/GUID-1E134053-692A-4386-BB77-153CB4A6071A.htm#SUTIL887)

```bash
expdp hr FULL=YES DUMPFILE=dpump_dir1:full1%U.dmp, dpump_dir2:full2%U.dmp FILESIZE=2G PARALLEL=3 LOGFILE=dpump_dir1:expfull.log JOB_NAME=expfull
```

Be **sure** that your user has all necessary privileges for the objects like `DATAPUMP_EXP_FULL_DATABASE` to export users, tablespaces etc. Otherwise, you get a dump without users which one has to recreate manually on the target database.

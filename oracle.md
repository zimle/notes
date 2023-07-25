# Oracle databases

## sqlplus

[sqlplus](https://en.wikipedia.org/wiki/SQL_Plus) is the cli client for oracle databases shipped with the database.

It can be installed on Windows *without* the database as follows:

1. Download the following three [zips](https://www.oracle.com/database/technologies/instant-client/winx64-64-downloads.html) for the version of your choice:

    - Basic Package
    - SQL*Plus Package
    - Tools Package

1. Unzip all three zips and merge the three folders `instantclient_your_version` into one target folder of your choice

1. Add your target folder to your path.

### Login as sysdba without password

If being on the server, one can simply login to oracle with

```bash
# does not work in git bash on windows ...
sqlplus.exe / as sysdba
```

## Reading data

Sometimes, one has to extract a document based object from the relational database.
Doing this in one query might let the result set explode due to several `1:n` relations.
Here is an example from [SO](https://stackoverflow.com/questions/70521073/posgressql-sql-to-generate-json-from-multiple-tables) (link provides an oracle example although mainly on postgres) how to retrieve the data as a document:

```sql
select row_to_json(customer) as customer_json
from (
  select customer.first_name "first_name" ,null as "test_null",
    ( json_build_object('concatDeails' , customer.first_name||customer.last_name)  ) as "concatDeails",
    (select json_agg(payment)
     from (
         select payment.payment_id "payment_id" ,payment.amount  "amount" from payment 
         where payment.customer_id=customer.customer_id
     ) payment
     ) as payment ,
    (select row_to_json(address)
     from (
         select address.address "address_details",CASE WHEN address.phone IS  NULL THEN 'NO_PHONE' ELSE address.phone  END as "phone_no_null",
          address.phone as "phone", COALESCE(address.phone, 'NO_PHONE') as "phone_coalesce"
         from address  where address.address_id=customer.address_id
      ) address
    ) as address
    from customer as customer where customer_id=2 ) customer;
```

## Partitioning

Relevant [views for partitioned tables](https://docs.oracle.com/en/database/oracle/oracle-database/19/vldbg/view-info-partition-tables-indexes.html#GUID-2D424638-511C-4CC3-9BDE-53FFB1686ECD) are (also in `DBA` and `ALL` variant):

- `USER_PART_TABLES`
- `USER_TAB_PARTITIONS`
- `USER_TAB_SUBPARTITIONS`
- `USER_PART_KEY_COLUMNS`
- `USER_SUBPART_KEY_COLUMNS`
- `USER_PART_COL_STATISTICS`
- `USER_SUBPART_COL_STATISTICS`
- `USER_PART_HISTOGRAMS`
- `USER_SUBPART_HISTOGRAMS`
- `USER_PART_INDEXES`
- `USER_IND_PARTITIONS`
- `USER_IND_SUBPARTITIONS`
- `USER_SUBPARTITION_TEMPLATES`

### Truncate and partitions

Good examples can be found in [morgans library](https://www.morganslibrary.org/reference/truncate.html). Here are two examples:

```sql
-- truncating a partition
ALTER TABLE parttab TRUNCATE PARTITION southwest;
--truncating a subpartition
ALTER TABLE demo_list_list TRUNCATE SUBPARTITION boeing_sunday;
-- not listed in morgans library
-- # FOR (PARTITION_VALUE, SUBPARTITION_VALUE) which uniquely identifies a partition or truncates the default
ALTER TABLE demo_list_list TRUNCATE SUBPARTITION FOR (1,1)
```

### Indexes and partitions

Global indexes are `UNUSABLE` when tuncating a partition or subpartition. They can be looked up in `USER_INDEXES`. Note that `N/A` means that the index is partitioned, so its status has to be looked up in `USER_IND_PARTITIONS` (if index is on partition level) or `USER_IND_SUBPARTITIONS` (if index is on subpartition level).

### Add feature partitioning

Features can be added afterwards for Oracle databases. At least for Windows, there is a `setup.exe` in `%ORACLE_HOME%/oui/bin` , e.g. `C:\app\Oracle\product\12.1.0\dbhome_1\oui\bin`.

### Direct path / append hit

The append hint / direct path usually locks the whole table, so that another session cannot write to this table. This can be [circumvented](https://hemantoracledba.blogspot.com/2022/08/direct-path-insert-into-partitioned.html) by explicitly naming the target partition a la

```sql
insert /*+ APPEND */ into my_part_table partition (p_200)
select rownum+101, dbms_random.string('X',12)
from dual
connect by rownum < 51
```

## Indexes

Useful views for indexes (also note the `all` and `dba` variant) are

- `user_indexes`
- `user_ind_columns`

Invalid indexes can be detected via

```sql
SELECT *
FROM DBA_INDEXES -- or USER_INDEXES
WHERE  status <> 'VALID'
```

To refresh an index:

```sql
-- rebuild a global index (aka. a normal index)
alter index my_index rebuild;
```

To find duplicate indexes: The following script [looks for indexes on tables with the same leading column, then for indexes with the same two leading columns](http://www.dba-oracle.com/t_detecting_duplicate_indexes.htm)

```sql
WITH magic AS (
    SELECT
        a.index_owner,
        a.table_name,
        a.column_name,
        a.index_name      index_name1,
        b.index_name      index_name2,
        a.column_position position,
        (
            SELECT
                'YES'
            FROM
                dba_ind_columns x,
                dba_ind_columns y
            WHERE
                    x.index_owner = a.index_owner
                AND y.index_owner = b.index_owner
                AND x.index_name = a.index_name
                AND y.index_name = b.index_name
                AND x.column_position = 2
                AND y.column_position = 2
                AND x.column_name = y.column_name
        )                 nextcol
    FROM
        dba_ind_columns a,
        dba_ind_columns b
    WHERE
            a.index_owner = 'MY_SCHEMA'--not in ('SYS', 'SYSMAN', 'SYSTEM', 'MDSYS', 'WMSYS', 'TSMSYS', 'DBSNMP')
        AND a.index_owner = b.index_owner
        AND a.column_name = b.column_name
        AND a.table_name = b.table_name
        AND a.index_name != b.index_name
        AND a.column_position = 1
        AND b.column_position = 1
)
SELECT
    *
FROM
    magic;
```

## Statistics

The last refresh of the statistics can be seen in `user_tab_stats_history` resp. `all_tab_stats_history`. Statistics can be refreshed via

```sql
-- all tables of schema
exec DBMS_STATS.GATHER_SCHEMA_STATS(OWNNAME=>'MY_SCHEMA', estimate_percent => dbms_stats.auto_SAMPLE_SIZE, CASCADE =>TRUE, degree=>8,no_invalidate=>false);
-- specify specific tables
exec DBMS_STATS.GATHER_TABLE_STATS(OWNNAME=>'MY_SCHEMA', tabname=>'MY_TABLE', estimate_percent => dbms_stats.auto_SAMPLE_SIZE, CASCADE =>TRUE, degree=>4,no_invalidate=>false);
```

## Window functions

```sql
-- statement to compute the difference between dates of subsequent row
-- and filter out those rows where the distance is greater than 12 months
with date_distance as (
    select
    id,
    lag(my_date) over (partition by id order by my_date) as previous_date,
    my_date as current_date,
    months_between(my_date, lag(my_date) over (partition by id order by id, my_date)) as month_diff
    from my_table
)
select * from date_distance
where month_diff > 12
fetch first 10 rows only
```

## Test / Fake data generation

```sql
insert into i9contract (objectid, rzmandant, idextern, kontonummer , versionid, niederlassung , kreditart , kontoherkunft , waehrung )
select sys_guid() , 'MA_UPLOAD', TO_char(rownum + 600000), to_char(rownum + 600000), 1, '100', '221000', 'CML', 'EUR'
from dual
connect by level <= 100000;
```

To avoid [memory issues](https://oracle-base.com/articles/misc/dbms_random), use a cartesian product and limit the number of rows of the cartesian product

```sql
-- returns 10*10*10 rows with id from 1 to 10000
WITH data AS (
  -- returns rows with id 1 to 10
  SELECT /*+ MATERIALIZE */ level AS id
  FROM   dual
  CONNECT BY level <= 10
)
SELECT rownum AS id
FROM   data, data, data
WHERE  rownum <= 10000;
-- or like above (takes 1m10s):
CREATE TABLE test AS
    WITH data AS (
      SELECT /*+ MATERIALIZE */ level AS id
      FROM   dual
      CONNECT BY level <= 10000
    )
    SELECT
        sys_guid() AS objectid ,
        'MA_UPLOAD' AS text_column,
        TO_char(rownum + 600000) AS id,
        to_char(rownum + 600000) AS other_id
    FROM   data, data, data
    WHERE  rownum <= 1000000;
```

- creating [random values](https://oracle-base.com/articles/misc/dbms_random)

```sql
SELECT
DBMS_RANDOM.value(low => 1, high => 10), -- random NUMBER >= low AND < high, with 38 digit precision
DBMS_RANDOM.value(), -- produces numbers in the range [0,1) with 38 digits of precision
DBMS_RANDOM.string('x',10), --x: uppercase alpha-numeric CHARACTERS
DBMS_RANDOM.normal, -- normal distribution
DBMS_RANDOM.random, -- produces integers in the range [-2^^31, 2^^31)
TRUNC(SYSDATE + DBMS_RANDOM.value(0,366)) -- random DATE OVER NEXT year
FROM dual;
```

## Parallel

### Parallel statement

When trying to insert in parallel, one must allow the session to use parallel statements. Else, only a single writer process is used. Furthermore, one should rollback on error and exit the script. The SQL-Developer per defaults runs all other statements. Sensible defaults are:

```sql
ALTER SESSION ENABLE PARALLEL DML;
WHENEVER SQLERROR EXIT SQL.SQLCODE ROLLBACK;
SET SERVEROUTPUT ON SIZE UNLIMITED;
set timing on;
set echo on;
```

Oracle can only [delete and update in parallel](https://www.oreilly.com/library/view/oracle-parallel-processing/156592701X/ch04s02.html) for partitioned tables and when several partitions are affected. Note that parallelization can be used for insert ... select in [both clauses](https://docs.oracle.com/database/121/VLDBG/GUID-2627DC19-7EBE-4C45-A758-711BDB5E37EC.htm)

```sql
INSERT /*+ PARALLEL(employees) */ INTO employees
SELECT /*+ PARALLEL(ACME_EMP) */ *  FROM ACME_EMP;
```

### Parallel index creation

```sql
CREATE INDEX my_index ON
    my_table(a, b)
-- LOCAL / GLOBAL keyword if necessary
PARALLEL (DEGREE 16);

alter index my_index parallel noparallel;
```

## Tablespaces / Disk usage

### Add DBF File

When encountering the exception `ORA-01653`

>Oracle ORA-01653 Nicht genug Speicherplatz
>Hat Oracle nicht mehr genug Speicherplatz (Die Datenbank wird in .DBF Files abgelagert, die eine maximale Groesse von 32 GB haben), so muss der Tablespace geaendert / >erweitert werden um einen weiteren File:

one might need to add an additional DBF file. They contain maximally 4194303 datablocks, which means roughly 32 GB if the size of a datablock is 8 KB. So to extend the schema, run

```sql
ALTER TABLESPACE MY_TS ADD DATAFILE 'C:\APP\ORACLE\ORADATA\MY_DB\MY_DBF_02.DBF' SIZE 100M AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED;
```

Also see <http://www.dba-oracle.com/t_ora_39171.htm>. To [move Oracle Tablespaces](http://www.dba-oracle.com/t_linux_oracle_move_datafiles.htm) on the file system

```sql
select * from dba_data_files;
alter tablespace HEAD offline;
-- move dbf files in your OS
alter tablespace HEAD rename datafile 'C:\OLD\PATH\MY_DBF.DBF' to 'C:\NEW\PATH\MY_DBF.DBF';
alter tablespace HEAD rename datafile 'C:\OLD\PATH\MY_DBF_2.DBF' to 'C:\NEW\PATH\MY_DBF_2.DBF';
alter tablespace HEAD online;
```

- Herausfinden, welche Referenzen ein table space auf andere table spaces hat:

```sql
-- find out references of one tablespace onto others
execute sys.dbms_tts.transport_set_check(full_check=> TRUE, ts_list => 'MY_TS', incl_constraints => true);
select * from sys.transport_set_violations;
```

### Reorganize Tablespaces

To [reorganize the tablespace](https://frankgerasch.de/2021/03/tablespace-reorganisation-mit-data-pump/) for smaller disk usage (e.g. when large data was in tablespace before but is now not used anymore), do the following steps:

1. Check dependencies of tablespace via `sys.dbms_tts.transport_set_check` and

    ```sql
    select * from transport_set_violations
    ```

2. Export tablespace(s) via

    ```sql
    expdp USERID=system DIRECTORY=DATA_PUMP_DIR TABLESPACES=ERPTS,APPTS DUMPFILE=ts_%u.dmp LOGFILE=expdp_ts.log FLASHBACK_TIME=SYSTIMESTAMP
    ```

    It takes a **long** (up to 5 hours for head) time for many objects, so try parallelizing.

3. Drop tablespaces:

    ```sql
    drop tablespace ERPTS including contents and datafiles cascade constraints;
    drop tablespace APPTS including contents and datafiles cascade constraints;
    ```

4. Recreate tablespaces, e.g.

    ```sql
    CREATE SMALLFILE TABLESPACE ERPTS DATAFILE 'C:\app\oracle\oradata\ERPTS.DBF' SIZE 100M AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED LOGGING EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;
    CREATE SMALLFILE TABLESPACE ERPTS DATAFILE 'C:\app\oracle\oradata\APPTS.DBF' SIZE 100M AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED LOGGING EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;
    ```

5. Import the tablespace dump:

    ```sql
    impdp USERID=system DIRECTORY=DATA_PUMP_DIR DUMPFILE=ts_%u.dmp LOGFILE=impdp_ts.log
    ```

## Inner workings

From the [oracle forum](https://forums.oracle.com/ords/apexds/post/undo-vs-redo-3114):

- `UNDO` is stored in the database and is therefore accessible to transactional work - the creating transaction can use it for rollback, and other transactions or queries use it to recreate data blocks to the image at the point in time when the [other] query/transaction started (called Consistent Read or CR blocks);

- `REDO` is stored outside of the database, in the redo log files, and is normally totally unaccessible to a transaction.

## Oracle Logs einrichten

- Anleitung Oracle unter <https://docs.oracle.com/cd/B28359_01/java.111/b31224/diagnose.htm>

- Laden die ojdbc*_g.jar herunter und richte verweise den Tomcat auf die jar (in Eclipse unter Preferences/Tomcat/JVM Einstellungen der Classpath) sowie zu den JVM-Parametern folgendes zufuegen:

```properties
-Doracle.jdbc.Trace=true
-Djava.util.logging.config.file="Absoluter\Pfad\zu\den\OracleLog.properties\welche\das\Logging\konfigurieren"
```

- Entsprechend den Tomcat-JVM-Argumenten muss eine Log-Konfigurationsdatei existieren, z.B.

```properties
# Log Level of root logger
.level=SEVERE
# Log Level og logger oracle.jdbc
# Log Levels are (ascending) OFF, SEVERE, WARNING, INFO, CONFIG, FINE, FINER, FINEST, ALL
oracle.jdbc.level=INFO
oracle.jdbc.driver.level=CONFIG
oracle.jdbc.handlers=java.util.logging.FileHandler
# Loggers oracle.jdbc.driver, oracle.jdbc.pool, oracle.jdbc.rowset, oracle.jdbc.xa, oracle.sql
# also exist
# Messages passing Logger Level are piped to FileHandler and filtered by FileHandler level
java.util.logging.FileHandler.level=ALL
# Log File
java.util.logging.FileHandler.pattern=jdbc.log
# also java.util.logging.ConsoleHandler exists, see java.util.logging
java.util.logging.FileHandler.count=1
java.util.logging.FileHandler.formatter=java.util.logging.SimpleFormatter
java.util.logging.SimpleFormatter.format=%1$tY-%1$tm-%1$td %1$tH:%1$tM:%1$tS:%1$tL %2$s%n%4$s: %5$s%6$s%n
```

- Im Tomcat-Verzeichnis wird nun ein Log geschrieben. Achtung, allein das Hochfahren der Anwendung ist extrem langsam.

- Hilfreiche Statements zum Massieren des Logs

```bash
rg "oracle\.jdbc\.driver" | sed 's/^.*oracle\.jdbc\.driver/oracle\.jdbc\.driver/g' | sort | uniq
```

## Database upgrade

<https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/index.html>
<https://oracle-base.com/articles/19c/upgrading-to-19c>

### Auto Upgrade Tool

Also see [Oracle documentation](https://www.oracle.com/de/database/upgrades/). The [Auto Upgrade Tool](https://www.oracle.com/rs/a/otn/docs/database-upgrade-quick-start-guide.pdf) is recommended. The download requires some identification beyond creating a user account (a Support Identifier approved by your organization admin). However, it is included in every release (not sure about whether in zip file or only after installing a new version) in `oracle_19_3_home\rdbms\admin\autoupgrade.jar` .

See also as an additional resource this [blog](https://mikedietrichde.com/2019/04/29/the-new-autoupgrade-utility-in-oracle-19c/).

#### Run preupgrade.jar

The preupgrade.jar can be found under `oracle_19_3_home\rdbms\admin\preupgrade.jar` and can be run via

```bash
oracle_19_3_home/jdk/bin/java.exe -jar oracle_19_3_home/rdbms/admin/preupgrade.jar
```

It creates the following scripts

```bash
old_Oracle_home\cfgtoollogs\MY_DB\preupgrade\preupgrade.log
old_Oracle_home\cfgtoollogs\MY_DB\preupgrade\preupgrade_fixups.sql
old_Oracle_home\cfgtoollogs\MY_DB\preupgrade\postupgrade_fixups.sql
```

The `preupgrade.log` contains of the actions that *must* be executed and the ones which *should* be executed *before* as well as *after* the update. This issues will be resolved when running the corresponding sql scripts generated. The scripts must be run as `sys` user.

We now start the update:

1. Create a sample config file via

    ```bash
      oracle_19_3/jdk/bin/java.exe -Duser.language=en -jar oracle_19_3/rdbms/admin/autoupgrade.jar -create_sample_file config`
    ```

2. Adjust the sample config file (like setting `node=localhost`) and run the autoupgrader in analysis mode:

    ```bash
      oracle_19_3/jdk/bin/java.exe -jar oracle_19_3/rdbms/admin/autoupgrade.jar -config config.cfg -mode analyze
    ```

    One gets into a prompt where one simply has to wait until the job finished (hopefully) successfully. Else, one can try to repair stuff in the prompt. A lot of stuff is generated, but it seems to be the same as in the `preupgrade.jar` .

3. If no error appeared, run the upgrade. Be sure that `upg1.start_time=NOW` is set in`sample_config.cfg`.

    ```bash
      oracle_19_3/jdk/bin/java.exe -Duser.language=en -jar oracle_19_3/rdbms/admin/autoupgrade.jar -config sample_config.cfg -mode deploy
    ```

    Note that also the modes `fixup` and `upgrade` exist. Deploy simply makes an `analyze` , `fixup` , `upgrade` and `postupgrade` after another. One can also execute these commands one after the other.

### Troubleshooting

- If an exception like

  ```bash
    Exception in thread "main" java.lang.ExceptionInInitializerError
            at oracle.upgrade.autoupgrade.boot.AutoUpgMain.<init>(AutoUpgMain.java:109)
            at oracle.upgrade.autoupgrade.boot.AutoUpgMain.newInstance(AutoUpgMain.java:136)
            at oracle.upgrade.autoupgrade.boot.Boot.main(Boot.java:39)
    Caused by: java.lang.NullPointerException
            at java.io.Reader.<init>(Reader.java:78)
            at java.io.InputStreamReader.<init>(InputStreamReader.java:113)
            at oracle.upgrade.commons.lang.LangSettings.resourceBundleFromFile(LangSettings.java:213)
            at oracle.upgrade.commons.lang.LangSettings.loadLanguageInfo(LangSettings.java:71)
            at oracle.upgrade.commons.lang.LangSettings.<init>(LangSettings.java:64)
            at oracle.upgrade.commons.lang.LangSettings.<init>(LangSettings.java:54)
            at oracle.upgrade.commons.context.AppContext.<clinit>(AppContext.java:47)
            ... 3 more
    ```

    occurs, try the command again with `-Duser.language=en` (in `autoupgrade.jar` , there seems to be only the english resource). Otherwise, one can also try the following: Enter locale in your shell and look whether `LC_NUMERIC=C` holds according to this [thread](https://mikedietrichde.com/2019/04/29/the-new-autoupgrade-utility-in-oracle-19c/). If it show something like `de_DE.UTF-8` , enter `export LC_ALL=C` and try again.

- `AutoUpgrade` memorizes the jobs done and tries to resume jobs. If you really want to resume freshly, start the upgrade command with the flag `-clear_recovery_data`.

### DBUA

The [Database Upgrade Assistant Tool](https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/upgrading-oracle-database-upgrade-assistant-dbua.html#GUID-04161A85-6A67-4404-B7A7-69A3713C5F99) seems to need the `Auto Upgrade Tool` ...

### Manual Upgrade

The [Database Upgrade Assistant Tool](https://docs.oracle.com/en/database/oracle/oracle-database/19/upgrd/upgrading-oracle-database-upgrade-assistant-dbua.html#GUID-04161A85-6A67-4404-B7A7-69A3713C5F99) is contained in the standard installer of the oracle database (unzip fat zip and run `setup.exe` ). As Oracle will **automatically** choose the unziped folder as the home directory, one should rename the unzipped directory before starting the installer.

After running the `setup.exe` , choose the following options:

1. Kofigurationsoption: `Nur Software einrichten` as in the hint.
2. Datenbankinstallationsoption: `Datenbankinstallation mit nur einer Instanz`
3. Database Edition: `Enterprise Edition`
4. Benutzer von Oracle-Standard-Verzeichnis: `Virtuellen Account verwenden`
5. Installationsspeicherort: Choose the directory for the `ORACLE_BASE` base directory. A warning pops up if choosing the old one, so we create a new directory like `oracle_19_3_base`.
6. Go on and finish.

This creates a **new** Oracle version as specified above. To *upgrade* the old one, go into the specified `ORACLE_HOME` directory (i.e. the unpacked zip file)

## Uninstall

Use the default deinstaller in deinstall folder. There might be some entries left in the registry, see [Uninstall Oracle](https://oracle-base.com/articles/misc/manual-oracle-uninstall) with [Screenshots](https://mkyong.com/oracle/how-to-uninstall-oracle-database-19c-on-windows/). Make a **backup** before removing them, as it might kill Windows.

## General Troubleshooting

- sqlplus as a prompt does not work in git bash, use powershell
- `ORA-12560: TNS: Fehler bei Protokolladapter`: Open the windows services `WIN+R` and type in `services.msc`: Check if `OracleServiceMySID` has started

## Docker

To pull an [Oracle docker container](https://collabnix.com/how-to-run-oracle-database-in-a-docker-container-using-docker-compose/), one has to sign in and agree the licenses in the [Oracle container registry](https://container-registry.oracle.com/) (you need a user account). Afterwards, you can log in from the terminal and pull a docker container:

```bash
# log in by entering email and credentials
docker login container-registry.oracle.com
# you can now pull your oracle db-version (see https://container-registry.oracle.com/ for available containers)
docker pull container-registry.oracle.com/database/enterprise:19.3.0.0
```

Note that the official Oracle container only supports the `CDB` database (sort of a wrapper around normal databases called pluggable databases `PDB`) and not the deprecated (i.e. before 12 c) single database `PDB`, see CDB vs PDB (see <https://github.com/oracle/docker-images/issues/229>). If switching, beware of altering the jdbc connection string accordingly.

Example setup

```yaml
version: '3.2'

services:
  oracle:
    container_name: oracle193
    image: container-registry.oracle.com/database/enterprise:19.3.0.0
    environment:
      # explanation on https://container-registry.oracle.com/ords/f?p=113:4:107035775590697:::4:P4_REPOSITORY,AI_REPOSITORY,AI_REPOSITORY_NAME,P4_REPOSITORY_NAME,P4_EULA_ID,P4_BUSINESS_AREA_ID:9,9,Oracle%20Database%20Enterprise%20Edition,Oracle%20Database%20Enterprise%20Edition,1,0&cs=3nYH_Xl1oh5UMKHx2ZZ8-HbRxdQ2OU_tzY70CjKDy8ivV0HKASJjdQR2YaSYF9BnbzTvb7wbjeGlhNYpY12oNww
      ORACLE_SID: ORCLCDB
      ORACLE_PWD: my_password
      ORACLE_PDB: MY_DB # name of the pluggable database (standard) in contrast to CDB; only here user can be created
      #INIT_SGA_SIZE:       The total memory in MB that should be used for all SGA components (optional)
      #INIT_PGA_SIZE:       The target aggregate PGA memory in MB that should be used for all server processes attached to the instance (optional)
      ORACLE_EDITION: enterprise # default is enterprise
      ORACLE_CHARACTERSET: WE8MSWIN1252 # default AL32UTF8, should actually be preferred...
    ports:
      - 1521:1521
    volumes:
      - type: volume
        source: oracle-data
        target: /opt/oracle/oradata/

volumes:
  oracle-data: # An entry under the top-level volumes key can be empty, in which case it uses the default driver configured by the Engine (in most cases, this is the local driver).
```

This can be run with

```bash
docker compose -f oracle-compose.yml up -d
```

This can take a *long* time (up tp 45 minutes).
To import data, we have to copy the data into the docker container via

```bash
docker-compose docker cp /mnt/path/to/FULL_EXPORT.dmp oracle193:/opt/oracle/admin/ORCLCDB/dpdump/hash/FULL_EXPORT.dmp
```

provided all paths already exist ( `select * from dba_directories` ). Then run `docker exec -it containername bash` and use `impdp` as you normally would.

```bash
impdp system/pwd@MY_DB  directory=DATA_PUMP_DIR full=Y dumpfile=FULL_EXPORT.dmp logfile=full.log
```

## Create user /schema

Some template for creating a user with corresponding schema:

```sql
-- USER
CREATE USER MY_NEW_USER IDENTIFIED BY MY_SECRET_PASSWORD DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;
-- ROLES
GRANT CONNECT,RESOURCE TO MY_NEW_USER;
GRANT UNLIMITED TABLESPACE TO MY_NEW_USER;
```

## Useful trivia

- the keyword `USER` resolves to the schema name (?) the current user is connected with (usually, in Oracle, user and schema coincide), e.g. `select * from all_tab_cols where owner = user` only shows the record of the schema corresponding to the user that is logged in.

- one can user [multi table inserts](https://oracle-base.com/articles/9i/multitable-inserts) to insert from one source into multiple target tables within one statement, [e.g.](https://stackoverflow.com/questions/49486949/insert-all-with-parallel-hint)

    ```sql
    insert /*+ append parallel(table1,4) parallel(table2,4) */ all
    when col1 not like '123%' then into table1
    when col1 like '123%' or col2 like '5%' then into table2
    select /*+ parallel(c,4) */* from table3 c;
    ```

- replay some user / schema
    1. make an export a la

        ```bash
        expdp system/password@my_db \
        schemas=my_schema\
        DIRECTORY=DATA_PUMP_DIR\
        DUMPFILE=my_schema.dmp
        ```

    2. drop the user / schema a la `drop user my_schema cascade`

    3. import the schema a la

        ```bash
        impdp system/password@my_db schemas=my_schema.dmp directory=DATA_PUMP_DIR DUMPFILE=my_schema.dmp
        ```

- newlines are created by appending `chr(10)` (newline unix) or `chr(13)` (newline windows), i.e. `'newxt word in' || chr(10) || 'newline`.

- [how to delete millions of records fast](https://blogs.oracle.com/sql/post/how-to-delete-millions-of-rows-fast-with-sql)

- to detect long running operations, run

    ```sql
    select sid,opname,target,sofar,totalwork,elapsed_seconds,time_Remaining
    from v$session_longops
    where sofar<>totalwork
    order by sid;
    ```

- to see active queries, run

    ```sql
    select s.sid, s.username,optimizer_mode,hash_value,
    address,cpu_time,elapsed_time,sql_text
    from v$sqlarea q
    JOIN v$session s
    ON s.sql_hash_value = q.hash_value
    and s.sql_address = q.address
    WHERE 1=1
    and s.username is not null;
    ```

- to use pl/sql functions in queries:

    ```sql
    WITH
        FUNCTION my_function (
            a_string         IN VARCHAR2,
            a_number         IN NUMBER
        ) RETURN VARCHAR2 IS
            res VARCHAR2(4);
        BEGIN
        -- logic
            RETURN res;
        END;
    my_cte AS (
    ...
    )
    select * from ...
    ```

- date-format of session: use `select value from v$nls_parameters where parameter = 'NLS_DATE_FORMAT'` for seeing the current date format and `alter session set nls_date_format = 'YYYY-MM-DD'` for setting a new one for the session (default is `DD.MM.RR`).

- determine database version via `SELECT version FROM V$INSTANCE` or `SELECT version FROM V$VERSION`.

- change user password: `ALTER USER my_user IDENTIFIED BY MyNewPassword123`

- see segment space used by tables: Use the view `user_segments`

- get `DDL` statement like create from existing object: `select dbms_metadata.get_ddl( 'TABLE', 'MY_TABLE', 'MY_SCHEMA') FROM dual`

- to see locked objects:

    ```sql
    -- should be run as sysdba
    SELECT B.Owner, B.Object_Name, A.Oracle_Username, A.OS_User_Name
    FROM V$Locked_Object A, All_Objects B
    WHERE A.Object_ID = B.Object_ID;
    ```

- [reclaiming unused space in datafiles](https://dba.stackexchange.com/questions/192625/oracle-shrinking-reclaiming-free-tablespace-space) :

- find out tablespace usage:

    ```sql
    select
    fs.tablespace_name                          Tablespace,
    (df.totalspace - fs.freespace)              used_mb,
    fs.freespace                                free_mb,
    df.totalspace                               total_mb,
    round(100 * (fs.freespace / df.totalspace)) free_perc
    from
    (select
        tablespace_name,
        round(sum(bytes) / 1048576) TotalSpace
    from
        dba_data_files
    group by
        tablespace_name
    ) df,
    (select
        tablespace_name,
        round(sum(bytes) / 1048576) FreeSpace
    from
        dba_free_space
    group by
        tablespace_name
    ) fs
    where 1=1
    and df.tablespace_name = fs.tablespace_name
    ;
    ```

- to get an explain plan:

    ```sql
    -- Run statement with `EXPLAIN PLAN FOR` (not actually run by db)
    EXPLAIN PLAN FOR Select * from bla;
    -- see last query plan (estimated, not the executed one)
    select * from table(dbms_xplan.display(format=>'ALL'));
    ```

- [FAQ](https://www.oracle.com/database/technologies/faq-jdbc.html) about oracle jdbc drivers seems to be very complete

- convert unix timestamp to date time:

    ```sql
    TO_CHAR(FROM_TZ(TO_TIMESTAMP('1970-01-01 00:00:00', 'YYYY-MM-DD HH24:MI:SS.FF') + NUMTODSINTERVAL(ts /1000, 'SECOND'), 'UTC') 
    AT TIME ZONE 'Europe/Berlin', 'YYYY-MM-DD HH24:MI:SS.FF') AS ts_as_date,
    ts as timestamp 
    from my_table
    order by ts desc nulls last 
    ```

- convert date to unix timestamp in milliseconds:

    ```sql
        select to_number(to_date('2022-10-31', 'yyyy-mm-dd') - to_date('1970-01-01', 'yyyy-mm-dd')) * (24 * 60 * 60 * 1000) from dual;
    ```

- drop table and everything with it: `drop table my_table cascade constraints purge`

- [quota](https://www.orafaq.com/wiki/Quota) is the amount of space allocated to a user in a tablespace:

    ```sql
    -- restrict to 10M
    ALTER USER scott QUOTA 10M ON users;
    -- allow unlimited
    ALTER USER scott QUOTA unlimited ON users;
    -- query, a quota value of -1 indicated UNLIMITED
    SELECT * FROM user_ts_quotas;
    ```

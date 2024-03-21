# utPLSQL

`utPLSQL` is a unit testing framework for the Oracle PL/SQL Language.

## Installation

1. Create a user `ut2` (not necessary, but advisable for separation of concerns) like

    ```sql
    CREATE USER ut2 IDENTIFIED BY my_secret DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;
    ```

1. Provide necessary grants to the user (here `ut2`)

    ```sql
    grant create session, create table, create procedure,
        create sequence, create view, create public synonym,
        drop public synonym, unlimited tablespace to ut2;
    ```

1. Download a [release](https://github.com/utPLSQL/utPLSQL/releases).

1. After potentially unzipping, change into the folder `code`.

1. Execute `ut_i_do install` with `sqlplus`, e.g. `sqlplus "ut2/my_secret@//localhost:1521/SID" @ut_i_do install`.

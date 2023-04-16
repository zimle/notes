# PL/SQL

## Executing Static / Dynamic statements

In pl/sql, `DDL` statements such as `create`, `drop`, `grant`, `alter`, .., can only be [executed as dynamic statements](https://www.dba-oracle.com/t_using_ddl_create_index_table_plsql.htm):

```sql
 CREATE OR REPLACE PROCEDURE salary_raise (raise_percent NUMBER, job VARCHAR2) IS
    TYPE loc_array_type IS TABLE OF VARCHAR2(40)
        INDEX BY binary_integer;
    dml_str VARCHAR2              (200);
    loc_array      loc_array_type;
BEGIN
    -- bulk fetch the list of office locations
    SELECT location BULK COLLECT INTO loc_array
        FROM offices;
    -- for each location, give a raise to employees with the given 'job'
    FOR i IN loc_array.first..loc_array.last LOOP
        dml_str := 'UPDATE emp_' || loc_array(i)
        || ' SET sal = sal * (1+(:raise_percent/100))'
        || ' WHERE job = :job_title';
    EXECUTE IMMEDIATE dml_str USING raise_percent, job;
    END LOOP;
END;
/
SHOW ERRORS;


CREATE OR REPLACE PROCEDURE add_location (loc VARCHAR2) IS
BEGIN
    -- insert new location in master table
    INSERT INTO offices VALUES (loc);
    -- create an employee information table
    EXECUTE IMMEDIATE
    'CREATE TABLE ' || 'emp_' || loc ||
    '(
        empno   NUMBER(4) NOT NULL,
        ename   VARCHAR2(10),
        job     VARCHAR2(9),
        sal     NUMBER(7,2),
        deptno  NUMBER(2)
    )';
END;
/
SHOW ERRORS;
```

## Resources

- [bind vs substitution variables](https://forums.oracle.com/ords/apexds/post/pl-sql-101-substitution-vs-bind-variables-6214)

permalink: "/2017/dirty-dynamic-sql"
title: Dirty Dynamic SQL
published_date: "2017-07-01 18:19:52 +0100"
layout: post.liquid
data:
  tags: "dynamic, sql, danger, oneliners, codegen"
  route: blog
---
Not all dynamic sql is strictly for immediate execution, nor is it dirty (I
needed aninteresting title). I learnt this tricks from my friend at work, he's
a big believer of using sql to code generate more sql.

## Altering column length

You'll still get an error when running the generated sql if you try to shrink a column.

```sql
SELECT 'ALTER TABLE ' || table_name || ' MODIFY ' || column_name || ' VARCHAR2(' || CASE column_name  WHEN 'FOO' THEN '10'  ELSE '20' END || ');' AS sql
  FROM user_tab_columns 
 WHERE column_name IN ( 'FOO','BAR')
   AND table_name IN (SELECT table_name FROM user_tables)   
   AND NOT regexp_like(table_name, '_(NEW|BAK)$') -- exclude backups
```

## Inserting (hurr) output from dynamic sql

Normally you'd expect to get away with using something like `SELECT blah INTO
v_foo`, but with dynamic sql you need place the `INTO` in the `EXECUTE
IMMEDIATE` statement.

```plsql
SET SERVEROUTPUT ON
DECLARE
  v_sql VARCHAR2(8000);
  v_count NUMBER;
  CURSOR c_tables
      IS
  SELECT table_name
    FROM user_tables;
BEGIN
  FOR rec in c_tables
  LOOP
    v_sql := 'SELECT count(*) FROM ' || rec.table_name;
    EXECUTE IMMEDIATE v_sql INTO v_count;
    
    DBMS_OUTPUT.put_line(rec.table_name || ' has ' || v_count);
  END LOOP;
END;
/
```
tags: rowcount , row , count

## Using substution variables

```sql
clear screen
set serveroutput on
declare
  cursor c_tables is select table_name from user_tables;
  v_data VARCHAR2(2000);
  v_sql VARCHAR2(2000);
  v_stuff VARCHAR2(50) := '&1'; -- try entering MAX(DATADATE) or COUNT(*) 
begin
  for rec in c_tables
  loop
    v_sql := 'SELECT TO_CHAR(' || v_stuff || ') FROM ' || rec.table_name;
    
    begin
      execute immediate v_sql into v_data;
      dbms_output.put_line(rec.table_name || ': ' || v_data);
    
    exception
      when others then
        dbms_output.put_line('failed to run: ' || v_sql);
        raise;
    end;
    end case;
  end loop;
end;
/
```
## One liners

### Grants

```sql
SELECT 'GRANT ' || privilege || ' ON "' || GRANTOR ||'"."' || TABLE_NAME || 
       '" TO "' || GRANTEE || '";' 
FROM dba_tab_privs 
WHERE grantor = 'FOO'
AND grantee = 'BAR'
;
```

### Recreating synonyms as sys

```sql
select 'CREATE OR REPLACE SYNONYM "' || OWNER || '"."' || SYNONYM_NAME || 
       '" FOR "' || TABLE_OWNER || '"."' || TABLE_NAME || '";' 
from dba_synonyms 
where table_owner = 'FOO';
```

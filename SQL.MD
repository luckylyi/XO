# SQL 处理数据常用语句

## 临时数据处理

- UNION | UNION ALL

  ```SQL
        select 845 primary_service_type_id, 1013 secondary_service_type_id union
        select 845, 1014 union
        select 845, 2391;
  ```
  
- Temporary table
  ```SQL
    -- example 1
    select 845 primary_service_type_id, 1013 secondary_service_type_id
        into temp_service_type_mapper
        union
        select 845, 1014 union
        select 845, 2391;
    -- do something here
    
    -- drop the temporary table
    drop table temp_service_type_mapper;
    
    
    -- example 2
    create table if not exists temp_service_type_mapper(
      primary_service_type_id int4
      ,secondary_service_type_id int4
    );
    
    insert into temp_service_type_mapper values
    (845, 1014)
    , (845, 2391)
    , (845, 2392);
    
    select * from temp_service_type_mapper;
    drop table if exists temp_service_type_mapper;
  ```

 - Common Table Expressions (CTE)
  ```SQL
    WITH service_type_mapper AS (
      select 845 primary_service_type_id, 1013 secondary_service_type_id union
        select 845, 1014 union
        select 845, 2391
    )
    
    select * from service_type_mapper;
  ```
  
  
# 常用SQL语句 (postgres)
## 查找所有用户表
```SQL
SELECT *
FROM pg_catalog.pg_tables
WHERE schemaname != 'pg_catalog' AND 
    schemaname != 'information_schema';
```

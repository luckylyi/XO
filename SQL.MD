# SQL 处理数据常用语句

## 常用查询

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
Postgres
```SQL
SELECT *
FROM pg_catalog.pg_tables
WHERE schemaname != 'pg_catalog' AND 
    schemaname != 'information_schema';
```

SQL Server
```SQL
SELECT * 
FROM sys.objects
WHERE type='U'
```

## 查找表的字段名

Postgres
```SQL
select column_name from information_schema.columns
where table_schema='public' and table_name='vendor'
```

SQL Server
```SQL
SELECT name 
FROM sys.columns
WHERE object_id=object_id('tblCustomers')
```

## Postgresql 和 MSSQL的差异

- MSSQL是关系型数据库
- 而Postgreql是对象关系型数据库（NoSQL数据库，具有完整的事务支持，存储JSON文档，约束字段数据）
- Postgresql区分大小写，而MSSQL是不区分大小写的。
  - 表名，字段名，值
  - Postgresql默认会将表名和字段转为小写来处理，如果想要用驼峰式表名或字段名，需要用双引号括起来。
- 简单查询的区别如：分页实现
  - 查询前10条记录
    - ```SQL MSSQL 
      select top 10 * from tablename (nolock);

      -- 在SQL 2012后的版本中，加入了 offset 和 fetch
      select * from tblCustomers (nolock)
      order by customerID 
      offset 10 rows
      fetch 10 rows only;

      ```
    - ```SQL Postgresql
      select * from service_type 
      order by service_type_id 
      limit 10
      offset 10;
      ```

## jsonb的常用查询（数组）

# SQL 处理数据常用语句

## 入门基础
- What is SQL?
  - SQL stands for Structured Query Language (结构化查询语言）
  - SQL能做什么？
    - SQL 可以对数据库执行查询
    - SQL 可以从数据库中检索数据
    - SQL 可以在数据库中插入记录
    - SQL 可以更新数据库中的记录
    - SQL 可以从数据库中删除记录
    - SQL 可以创建新的数据库
    - SQL 可以在数据库中创建新表
    - SQL 可以在数据库中创建存储过程
    - SQL 可以在数据库中创建视图
    - SQL 可以设置表、过程和视图的权限 
    
  - 我们通常会用到的 SQL 命令
    - `SELECT` 从数据库中提取数据
    - `UPDATE` 更新数据库中的数据
    - `DELETE` 从数据库中删除数据
    - `INSERT` INTO 将新数据插入数据库
    - `CREATE` DATABASE 创建一个新的数据库
    - `ALTER` DATABASE 修改数据库
    - `CREATE` TABLE 创建一个新表
    - `ALTER` TABLE 修改表
    - `DROP` TABLE 删除一个表
    - `CREATE` INDEX 创建索引(用于搜索的键)
    - `DROP` INDEX 删除索引  
  - 注意点
    - 关键字不区分大小写
    - SQL语句未尾加分号对多条SQL语句进行分割
      - 有些数据库是支持返回多条查询的结果的，比如MSSQL，加上分号可以防止多条SQL因为多条SQL查询合并到一行时报错。 


- 查询某个表的数据
```SQL
SELECT column1, column2, ... 
FROM table_name;
-- 把 column_name 换成 * 号可以返回表中的所有字段。 如果一个表有非常多的字段时，不建议这样做，因为返回太多不必要的字段会浪费数据库性能，特别是数据量很大的时候。

-- Example:
SELECT vendor_id FROM vendor;
```
  - 查询不重复的数据
  ```SQL
  SELECT DISTINCT column1, column2, ...
  FROM table_name;
  ```
  - 加上 `WHERE` 条件对数据进行过滤
    - WHERE 条件不仅作用于 SELECT 语句，还用于 UPDATE、 DELETE 等！
  ```SQL
  SELECT column1, column2, ...
  FROM table_name
  WHERE condition;  
  
  -- AND, OR, NOT
  AND 显示由 AND 分隔的条件同时为TRUE的记录。
  SELECT * 
  FROM vendor
  WHERE legacy_id=9921 AND vendor_status=1; 

  OR 显示 OR 分隔的任何条件为TRUE的记录。
  SELECT * 
  FROM vendor
  WHERE legacy_id=9921 OR legacy_id=31959;
  
  NOT 显示条件为FALSE的记录。（取反）
  SELECT * 
  FROM vendor
  WHERE NOT legacy_id <> 9921;
  
  ```
  
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
  
- 将NULL值当成空符串处理
  - MSSQL: isnull(field, '')
  - PG: coalesce(field, '')
   
  https://www.postgresql.org/docs/12/functions-conditional.html#FUNCTIONS-COALESCE-NVL-IFNULL

- 简单查询的区别如：分页实现
  - 查询前10条记录
    - ```SQL 
      -- MSSQL 
      select top 10 * from tablename (nolock);

      -- 在SQL 2012后的版本中，加入了 offset 和 fetch
      select * from tblCustomers (nolock)
      order by customerID 
      offset 10 rows
      fetch 10 rows only;

      ```
    - ```SQL
      -- Postgresql
      select * from service_type 
      order by service_type_id 
      limit 10
      offset 10;
      ```

## jsonb的常用查询（数组）
- JSON Functions and Operators
  https://www.postgresql.org/docs/12/functions-json.html
- 创建jsonb主从表
  - ```SQL
    create table vendor(data jsonb);
    create table vendor_video(data jsonb);  
    ```
  - 创建索引：
    
    ```
    create index vendor_index_on_id on vendor((data->>'id'));
    ```
    
  - 插入测试数据:
  ```SQL
  -- 主表 vendor
  insert into vendor
  select ('{"id": 1, "name": "lily", "created_at": "' || to_char(current_timestamp, 'YYYY-MM-DD"T"HH24:MI:SS.MS"Z"') || '"}')::jsonb
  union select ('{"id": 2, "name": "machel", "created_at": "' || to_char(current_timestamp, 'YYYY-MM-DD"T"HH24:MI:SS.MS"Z"') || '"}')::jsonb
  union select ('{"id": 3, "name": "tony", "created_at": "' || to_char(current_timestamp, 'YYYY-MM-DD"T"HH24:MI:SS.MS"Z"') || '"}')::jsonb;

  -- 从表 vendor_video
  insert into vendor_video
  select '{"id": 1001, "vendor_id": 1, "videos": [{"url": "https://youtube.com/watch?v=JmBgOJIPBbw","sort":1,"active":false}, {"url": "https://youtube.com/watch?v=JmBgOJIPBbw","sort":2,"active":true}]}'::jsonb
  union select '{"id": 1002, "vendor_id": 2, "videos": [{"url": "https://youtube.com/watch?v=JmBgOJIPBbw&a=2","sort":1,"active":true}, {"url": "https://youtube.com/watch?v=JmBgOJIPBbw&a=2","sort":2,"active":true}]}'::jsonb
  union select '{"id": 1005, "vendor_id": 3, "videos": [{"url": "https://youtube.com/watch?v=JmBgOJIPBbw&b=3","sort":1,"active":true}, {"url": "https://youtube.com/watch?v=JmBgOJIPBbw&b=3","sort":2,"active":true}]}'::jsonb;
  ```
  
- 查询：
```SQL
  -- 
  select data->>'id' as id
  , data->>'name' as name
  , data->>'created_at' as created_at
  from vendor 
  where data->>'id'='1';
  
  -- 关联查询
  select v.data->>'id' as id
  , v.data->>'name' as name
  , v.data->>'created_at' as created_at
  , map->>'url' as video_url
  , map->>'sort' as video_sort
  , map->>'active' as video_status
  from vendor v 
  inner join vendor_video vv on v.data->>'id' = vv.data->>'vendor_id'
  , jsonb_array_elements(vv.data->'videos') map
  where v.data->>'id'='1';
  
  -- 关联查询，加子表过滤条件
  select v.data->>'id' as id
  , v.data->>'name' as name
  , v.data->>'created_at' as created_at
  , map->>'url' as video_url
  , map->>'sort' as video_sort
  , map->>'active' as video_status
  from vendor v 
  inner join vendor_video vv on v.data->>'id' = vv.data->>'vendor_id'
  , jsonb_array_elements(vv.data->'videos') map
  where v.data->>'id'='1' and map->>'active'='true';
  
  -- 关联查询，加子表过滤条件，用对象查询(查询vendors的videos中包含至少一项active=false的记录)
  select v.data->>'id' as id
  , v.data->>'name' as name
  , v.data->>'created_at' as created_at
  , map->>'url' as video_url
  , map->>'sort' as video_sort
  , map->>'active' as video_status
  from vendor v 
  inner join vendor_video vv on v.data->>'id' = vv.data->>'vendor_id'
  , jsonb_array_elements(vv.data->'videos') map
  where v.data->>'id'='1' and vv.data->'videos' @> '[{"active":false}]';
  
  -- 在jsonb数据中删除指定的字段
  select map #- '{active}' from vendor_video,
  jsonb_array_elements(data->'videos') as map;
  
  -- 数组中删除指定的值
  select '["1", "2", "3", "4", "5", "6"]'::jsonb - '1' - '3';
  select '["1", "2", "3", "4", "5", "6"]'::jsonb - 1 - 3;
  
  -- 重新生成新的对象
  select jsonb_build_object('id', 1, 'name', 'lily');
  
  CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
  SELECT jsonb_build_object(
      'id', uuid_generate_v4(),
    'vendor_id', data->>'id',
    'name', data->>'name',
      'showCalculator', true,
      'active', 'true',
      'timestamp', jsonb_build_object('createdAt', to_char(current_timestamp, 'YYYY-MM-DD"T"HH24:MI:SS.MS"Z"'), 'updatedAt', to_char(current_timestamp, 'YYYY-MM-DD"T"HH24:MI:SS.MS"Z"')),
      'actors', jsonb_build_object('createdBy', uuid_nil(), 'updatedBy', uuid_nil()),
      'source', jsonb_build_object('application','The Knot Pro')
      ) FROM vendor;
  
  -- 更新对象，或给对象添加新的属性和值
  select 
  jsonb_set(
    jsonb_set(
      '{"name": "lily"}'::jsonb,
      '{id}',
      '1'::jsonb
    ),
    '{sex}',
    '"man"'::jsonb
  )
  
  -- update name
  update vendor v
  set data=jsonb_set(
  data,
  '{name}',
   concat('"', data->>'name', '-01"')::jsonb
  )
  where data->>'id'='3';
  
  -- add age
  update vendor v
  set data=jsonb_set(
  data,
  '{age}',
   '25'::jsonb
  )
  where data->>'id'='3';
  
```

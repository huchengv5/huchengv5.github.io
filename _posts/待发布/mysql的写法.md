
-- 第一步：创建临时表
```sql

CREATE TEMPORARY TABLE
IF
	NOT EXISTS tmpInfoSize ( materialSize VARCHAR ( 100 ) NOT NULL, CLIENT_ID VARCHAR ( 100 ) NOT NULL );
TRUNCATE TABLE tmpInfoSize; -- 删除临时表中数据

```


-- 第二步：生成sql语句，用来将符合要求的数据插入到临时表
SELECT
	GROUP_CONCAT( tsql.v_sql SEPARATOR ';' ) sqls 
FROM
	( SELECT concat( 'insert into tmpInfoSize select  SUBSTRING_INDEX(SUBSTRING_INDEX(OPERATE_PARAM,'':'',-1),''KB'',1) as  materialSize,CLIENT_ID from `', table_name, '` where OPERATE_DESC =''clientLog.downMMFEnd'' AND OPERATE_PARAM is not NULL' ) AS v_sql FROM information_schema.TABLES WHERE table_name LIKE 't_client_monitor_log2019%' ) tsql;
	
-- 第三步：执行生成的sql语句

-- 在这里,将上一步执行结果粘贴在这里，然后执行相应的sql语句

-- 第四步：查询结果

SELECT
	CONCAT( tmp.size / tmp.count, 'GB' ) AS 平均流量 
FROM
	( SELECT count( DISTINCT CLIENT_ID ) AS count, FORMAT( SUM( materialSize )/ 1048576, 2 ) AS size FROM tmpinfosize ) tmp;
	
-- 查看结果，结束
---
title: "如何通过SQL SERVER访问C#代码"
author: 胡承
date: 2020-01-21 18:35:3 +0800
CreateTime: 2020-01-21 18:35:3 +0800
categories: c# SQL
---

SQL SERVER必知必会

<!-- more -->

具体sql如下：
```sql
-- 开启安全配置
EXEC sp_configure 'clr enabled' , '1';  --0代表不允许，1代表运行
RECONFIGURE;

-- 添加到受信任列表
DECLARE @hash AS BINARY(64) = (SELECT HASHBYTES('SHA2_512', (SELECT * FROM OPENROWSET (BULK 'C:\xxx\MK.EncryptProvider.dll', SINGLE_BLOB) AS [Data]))) 
EXEC sp_add_trusted_assembly @hash

-- 查看受信任程序，使用hash保存
select * from sys.trusted_assemblies

-- 创建程序集
CREATE ASSEMBLY EncryptDll   
FROM 'C:\xxx\MK.EncryptProvider.dll'

-- 创建加密函数
CREATE FUNCTION dbo.Encrypt
(
     @InputString as nvarchar(max)
)
RETURNS nvarchar(max)　　　　　--返回类型
AS EXTERNAL NAME EncryptDll.[MK.EncryptProvider.EncryptProvider].Encrypt 

-- 创建解密函数
CREATE FUNCTION dbo.Decrypt
(
     @InputString as nvarchar(max)
)
RETURNS nvarchar(max)　　　　　--返回类型
AS EXTERNAL NAME EncryptDll.[MK.EncryptProvider.EncryptProvider].Decrypt 

-- 如果已经创建过，那么可以直接修改程序集
alter assembly  EncryptDll   
FROM 'C:\xxx\MK.EncryptProvider.dll' WITH PERMISSION_SET = unsafe

-- 删除程序集
drop assembly EncryptDll
```

如果数据库存在多个的情况下，程序集已经创建，那么只需要对其它为创建函数的数据库增加函数即可。
未完待续……
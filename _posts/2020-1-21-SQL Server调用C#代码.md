---
title: "如何通过SQL SERVER访问C#代码"
author: 胡承
date: 2020-01-21 18:35:3 +0800
CreateTime: 2020-01-21 18:35:3 +0800
categories: c# SQL
---

有时候操作SQL SERVER 数据库时，我们需要做一些复杂的算法或者业务逻辑。如果我们用应用程序来实现，就需要将数据查询出来后再插入，必然效率低下。好在SQL SERVER提供了强大的功能，直接调用C#托管代码。

<!-- more -->

虽然我们不推荐使用这种方法，毕竟会影响程序的移植性和可维护性，但是用来做些小工具还是非常不错的。本次以简单的`safe`权限以及信任列表方式为例：    

**具体sql如下：**

```sql
-- 开启安全配置，表示启用clr的代码访问 0代表不允许，1代表运行

EXEC sp_configure 'clr enabled' , '1';

-- 更新当前运行值：需要注意有些参数的设置需要重启服务，此配置不需要重启服务
RECONFIGURE;

-- 添加到受信任列表：从SQL Server 2017 (14.x)开始，要求强制要求启用安全认证，safe权限等同于unsafe权限。SQL SERVER 2017 可以可以不写这行sql。
DECLARE @hash AS BINARY(64) = (SELECT HASHBYTES('SHA2_512', (SELECT * FROM OPENROWSET (BULK 'C:\xxx\Encrypt.dll', SINGLE_BLOB) AS [Data]))) ;
EXEC sp_add_trusted_assembly @hash;

-- 查看受信任程序，检查有没有添加进去
SELECT * FROM sys.trusted_assemblies;

-- 创建程序集：将需要引用的程序集添加到数据库，需要注意.net的编译版本。如sql server 2008，只能引用framework 3.0或3.5的版本。
CREATE ASSEMBLY EncryptDll   
FROM 'C:\xxx\Encrypt.dll' WITH PERMISSION_SET = SAFE; -- WITH PERMISSION_SET 指定其执行权限；

-- 创建加密函数：当dll引入的数据库中以后，我们需要在数据库中增加函数来对其发起调用。这里要特别注意参数类型，需要保证参数的长度合理，否则调用过程中会引发异常。
CREATE FUNCTION dbo.Encrypt
(
     @InputString as nvarchar(max)
)
RETURNS nvarchar(max)　　　　　--返回类型
AS EXTERNAL NAME EncryptDll.[MK.EncryptProvider.EncryptProvider].Encrypt ;

-- 创建解密函数
CREATE FUNCTION dbo.Decrypt
(
     @InputString as nvarchar(max)
)
RETURNS nvarchar(max)　　　　　--返回类型
AS EXTERNAL NAME EncryptDll.[MK.EncryptProvider.EncryptProvider].Decrypt ;

-- 测试加密函数，并打印输出
print dbo.Encrypt('测试字符串');

-- 如果dll有更新，那么就需要将dll的hash值重新添加到受信任列表，然后再修改程序集。这里的EncryptDll表示程序集在数据库里面的名称。
ALTER ASSEMBLY  EncryptDll   
FROM 'C:\xxx\Encrypt.dll' WITH PERMISSION_SET = unsafe;

-- 删除程序集：删除程序集需要先把依赖删除，需要先删除函数
DROP ASSEMBLY EncryptDll;

-- 如果存在函数，则删除
if exists(select * from sys.objects where name=[ schema_name. ] function_name)
drop function [ schema_name. ] function_name;

```
*`如果数据库中存在多个数据库的情况下，程序集已经创建，那么只需要对其它为创建函数的数据库增加函数即可。`*

---------------------------------
## **对应的C#代码**  
*编译后的`dll`命名为`Encrypt.dll`,类名为`EncryptProvider`,命名空间为`Encrypt`，创建函数时需要以`命名空间`.`类名`.`方法名`的方式表示*

注意：数据库的类型很多，我们加密解密要保证字符串的长度不变，所以加密解密的算法也不能直接使用常规的加密解密算法，以下只是一个示例。

```cs
        /// <summary>
        /// 加密
        /// </summary>
        /// <param name="str">要加密的字符串</param>
        /// <returns>返回加密后的字符串</returns>
        /// 
        public static string Encrypt(string str)
        {
            if (string.IsNullOrEmpty(str)) return str;
            str = str.Trim();
            byte[] crypt = Encoding.Default.GetBytes(str).Select(x => (byte)((x + 10) % 256)).ToArray();
            return Encoding.Default.GetString(crypt);
        }

        /// <summary>
        /// 解密
        /// </summary>
        /// <param name="str">要解密的字符串</param>
        /// <returns>返回解密后的字符串</returns>
        /// 
        public static string Decrypt(string str)
        {
            if (string.IsNullOrEmpty(str)) return str;
            var decrypt = Encoding.Default.GetBytes(str);
            byte[] source = decrypt.Select(x => (byte)((x - 10 + 256) % 256)).ToArray();
            return Encoding.Default.GetString(source);
        }
```

也可以通过对字符做处理，但是这种方法可能会改变字节长度，因为数据库的字符编码不一样。

```cs
        /// <summary>
        /// 加密
        /// </summary>
        /// <param name="str">要加密的字符串</param>
        /// <returns>返回加密后的字符串</returns>
        public static string Encrypt(string str)
        {
            var sb = new StringBuilder();
            foreach (var c in str)
            {
                sb.Append((char)(c + 8));
            }
            return sb.ToString();
        }

        /// <summary>
        /// 解密
        /// </summary>
        /// <param name="str">要解密的字符串</param>
        /// <returns>返回解密后的字符串</returns>
        public static string Decrypt(string str)
        {
            var sb = new StringBuilder();
            foreach (var c in str)
            {
                sb.Append((char)(c - 8));
            }
            return sb.ToString();
        }
```

## **CREATE ASSEMBLY 权限设置**

权限包含有：`SAFE`，`UNSAFE`，`EXTERNAL_ACCESS`，通过 `WITH PERMISSION_SET`  来指定。  
**各权限说明：**  
`SAFE`：使用具有 SAFE 权限的程序集运行的代码不能访问外部系统资源（例如文件、网络、环境变量或注册表）。 SAFE 代码可以从本地 SQL Server 数据库访问数据，或执行不涉及访问本地数据库以外资源的计算和业务逻辑。大多数程序集执行计算和数据管理任务，而不需要访问 SQL Server 以外的资源。 因此，建议您将 SAFE 作为程序集权限集。  
`EXTERNAL_ACCESS`：EXTERNAL_ACCESS 允许程序集访问某些外部系统资源（例如文件、网络、Web 服务、环境变量和注册表）。 只有具有 EXTERNAL ACCESS 权限的 SQL Server 登录名才能创建 EXTERNAL_ACCESS 程序集。SAFE 和 EXTERNAL_ACCESS 程序集只能包含可验证为类型安全的代码。 这意味着这些程序集仅可以通过对类型定义有效的具有定义完善的入口点来访问类。 因此，它们不能随意访问不属于该代码所有的内存缓冲区。 另外，它们不能执行可能对 SQL Server 进程的可靠性具有负面影响的操作。  
`UNSAFE`：UNSAFE 不限制程序集访问资源，包括 SQL Server 以内和以外的资源。 在 UNSAFE 程序集内运行的代码可以调用非托管代码。同时，指定 UNSAFE 将允许程序集中的代码执行 CLR 验证工具认为是非安全类型的操作。 这些操作可能以非控制的方式访问 SQL Server 进程空间中的内存缓冲区。 UNSAFE 程序集也可能破坏 SQL Server 或公共语言运行时的安全系统。 有经验的开发人员或管理员应仅向高度可信的程序集授予 UNSAFE 权限。 只有的成员sysadmin固定的服务器角色可以创建 UNSAFE 程序集。  

----------------------------

## 扩展阅读

如果是要授予 `EXTERNAL ACCESS ASSEMBLY` 或 `UNSAFE ASSEMBLY` 权限，也可以通过以下方式授予（前面添加信任列表方法也是可行的）。

方法一：  

```sql
--使用master系统数据库
USE master
--创建非对称密钥
CREATE ASYMMETRIC KEY testkey FROM EXECUTABLE FILE = 'C:\xxx\Encrypt.dll'   
--创建登录名
CREATE LOGIN loginName FROM ASYMMETRIC KEY testkey 
--把权限授予给该登录名  
GRANT EXTERNAL ACCESS ASSEMBLY TO loginName; 

```
方法二：  

```sql
-- 将数据库设置为受信任数据库，不建议使用该方式,请参考下方官方文档
ALTER DATABASE dbName SET TRUSTWORTHY ON
```
------------------
参考文献：  
[设计程序集 - SQL Server](https://docs.microsoft.com/zh-cn/sql/relational-databases/clr-integration/assemblies-designing?redirectedfrom=MSDN&view=sql-server-ver15)  
[clr enabled 服务器配置选项 - SQL Server](https://docs.microsoft.com/zh-cn/sql/database-engine/configure-windows/clr-enabled-server-configuration-option?redirectedfrom=MSDN&view=sql-server-ver15)  
[TRUSTWORTHY 数据库属性 - SQL Server](https://docs.microsoft.com/zh-cn/sql/relational-databases/security/trustworthy-database-property?redirectedfrom=MSDN&view=sql-server-ver15)  

------------------------------------------------------

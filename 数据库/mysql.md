# MySQL

本篇由我「印象笔记」的一些片段文章整理而成，参考了很多文章，但是把很多原文链接都忘掉了，在此对原作者表示歉意。

## 1. 安装

+ CentOS: `yum install msyql`
+ Ubuntu: `apt-get install mysql-server`
+ Mac: `brew install mysql`

## 2. 配置

+ 查找本机配置文件位置: `mysql --help | grep cnf`

## 3. 使用

### 3.1 登录

`mysql -hhost -uuser_name -puser_password [db_name.table_name]`

### 3.2 权限

+ 查询所有用户: `select User from mysql.user`
+ 创建用户: `create user username identified by 'password'`
+ 删除用户: `delete from mysql.user where User="xxx` -> `flush privileges`
+ 授权: `grant all privileges on db_name.tablename to username`
+ 撤销权限: `REVOKE PRIVILEGES ON db_name.* FROM 'user_name'@'localhost';`
+ 创建用户并赋权限: `grant all privileges on db_name.* to news_user@'localhost' identified by 'news_password'`, `@` 后面是限制的主机，可以是 IP 或者 IP 段，`%`表示任何地方。
+ 查看用户权限: `SHOW GRANTS FOR user_name` 或者 `SELECT * FROM mysql.user WHERE user='user_name' \G`

### 3.3 数据类型

** BLOB 和 TEXT **

BLOB 四种类型: `TINYBLOB`, `BLOB`, `MEDIUMBLOB`, `LONGBLOB`。

TEXT 四种类型: `TINYTEXT`, `TEXT`, `MEDIUMTEXT`, `LONGTEXT`。

区别:

1. BLOB 被视为二进制字符串, TEXT被视为非二进制字符串;
2. BLOB列没有字符集，并且排序和比较基于列值字节的数值值。TEXT列有一个字符集，并且根据字符集的校对规则对值进行排序和比较。

** BLOB 和 TEXT 与 VARBINARY 和 VARCHAR 的不同 **

1. BLOB 和 TEXT 列不能有默认值
2. 当保存或检索 BLOB 和 TEXT 列的值时不删除尾部空格。(这与 VARBINARY 和 VARCHAR 列相同）
3. 对于 BLOB 和 TEXT 列的索引，必须指定索引前缀的长度。对于 CHAR 和 VARCHAR，前缀长度是可选的.

### 3.4 数据库

+ 建库: `CREATE DATABASES db_name [charset="utf8"]`
+ 删库: `DROP DATABASES db_name`
+ 查看当前使用的数据库: ` select database();`

### 3.4 数据表

+ 建表: `CREATE TABLE tbl_name (<col_name_1> <TYPE1> [,..<col_name_n> <TYPEn>])`
+ 删表: `DROP TABLE tbl_name`
+ 显示表的结构: `desc tbl_name`, `show columns from tbl_name`
+ 增加列: `ALTER TABLE tbl_name ADD col_name TYPE`
+ 删除列: `ALTER TABLE tbl_name DROP col_name`
+ 修改列: `ALTER TABLE tbl_name MODIFY col_name TYPE`
+ 改变表的名字: `ALTER TABLE tbl_name RENAME new_tbl_name`
+ 总条目数: `SELECT COUNT(*) FROM tbl_name`
+ 查看建表语句: `SHOW CREATE TABLE tbl_name \G`

索引:

+ 查看索引: `show index from tblname` or `show keys from tblname`
+ 创建索引:

        ALTER TABLE table_name ADD INDEX index_name (column_list)
        ALTER TABLE table_name ADD UNIQUE (column_list)
        ALTER TABLE table_name ADD PRIMARY KEY (column_list)

        CREATE INDEX index_name ON table_name (column_list)
        CREATE UNIQUE INDEX index_name ON table_name (column_list)

+ 删除索引:

        DROP INDEX index_name ON table_name
        ALTER TABLE table_name DROP INDEX index_name
        ALTER TABLE table_name DROP PRIMARY KEY


查询记录:

+ `sql = "select * from 数据表 where 字段名=字段值 order by 字段名 [desc]"`
+ `sql = "select * from 数据表 where 字段名 like '%字段值%' order by 字段名 [desc]"`
+ `sql = "select top 10 * from 数据表 where 字段名 order by 字段名 [desc]"`
+ `sql = "select * from 数据表 where 字段名 in ('值1','值2','值3')"`
+ `sql = "select * from 数据表 where 字段名 between 值1 and 值2"`

更新数据记录:

* `sql="update 数据表 set 字段名=字段值 where 条件表达式"`
* `sql="update 数据表 set 字段1=值1,字段2=值2 字段n=值n where 条件表达式"`

删除数据记录:

* `sql="delete from 数据表 where 条件表达式"`
* `sql="delete from 数据表" (将数据表所有记录删除)`

添加数据记录:

* `sql="insert into 数据表 (字段1,字段2,字段3 …) values (值1,值2,值3 …)"`
* `sql="insert into 目标数据表 select * from 源数据表" (把源数据表的记录添加到目标数据表)`

`DISTINCT` 使用:

位置: `DISTINCT` 只能放到开头，但是与其他查询函数一起使用的时候，没有位置限制: `select play_id, count(distinct(task_id)) from task;`

举例:

+ 在 count 计算不重复的记录的时候能用到: `SELECT COUNT( DISTINCT player_id ) FROM task;  就是计算表中 id 不同的记录有多少条`
+ 在需要返回记录不同的 id 的具体值的时候可以用: `SELECT DISTINCT player_id FROM task; 返回表中不同的 id 的具体的值`
+ 上面的 情况2 对于需要返回 mysql 表中 2 列以上的结果时会有歧义: `SELECT DISTINCT player_id, task_id FROM task;` 实际上返回的是 `player_id` 与 `task_id` 同时不相同的结果,也就是 DISTINCT 同时作用了两个字段，必须得 `player_id` 与 `task_id` 都相同的才被排除了, 与我们期望的结果不一样,我们期望的是 player\_id 不同被过滤, 在这种情况下，distinct 同时作用了两个字段，player\_id, task\_id。这时候可以考虑使用 `group_concat` 函数来进行排除,不过这个 mysql 函数是在 mysql4.1 以上才支持的
+ 其实还有另外一种解决方式, 就是使用 `SELECT player_id, task_id, count(DISTINCT player_id) FROM task` 虽然这样的返回结果多了一列无用的count数据(有时也许就需要这个数据)
+ 同时我们还可以利用下面的方式解决上面遇到的歧义问题通过 group by 分组:  `select player_id, task_id from task group by player_id`

## 4. 函数

日期和时间函数:

* `DAYOFWEEK(date)`: 返回日期 date 的星期索引(1=星期天，2=星期一, ……7=星期六)
* `WEEKDAY(date)`: 返回 date 的星期索引(0=星期一，1=星期二, ……6= 星期天)
* `DAYOFMONTH(date)`: 返回 date 的月份中日期，在 1 到 31 范围内
* `DAYOFYEAR(date)`: 返回 date 在一年中的日数, 在 1 到 366 范围内
* `DAYNAME(date)` : 返回 date 的星期名字
* `MONTHNAME(date)` : 返回 date 的月份名字
* `QUARTER(date)` : 返回 date 一年中的季度，范围 1 到 4
* `WEEK(date)`
* `WEEK(date,first)` : 对于星期天是一周的第一天的地方，有一个单个参数，返回 date 的周数，范围在 0 到 52。2 个参数形式 WEEK() 允许你指定星期是否开始于星期天或星期一。如果第二个参数是 0 ，星期从星期天开始，如果第二个参数是1，从星期一开始。
* `YEAR(date)` : 返回 date 的年份，范围在 1000 到 9999。
* `MINUTE(time)` : 返回 time 的分钟，范围是 0 到 59。
* `SECOND(time)` : 回来 time 的秒数，范围是 0 到 59。
* `PERIOD_ADD(P,N)` : 增加 N 个月到阶段 P (以格式YYMM或YYYYMM)。以格式 YYYYMM 返回值。注意阶段参数 P 不是日期值。
* `PERIOD_DIFF(P1,P2)` : 返回在时期 P1 和 P2 之间月数，P1 和 P2应该以格式 YYMM 或 YYYYMM。注意，时期参数 P1 和 P2 不是日期值。
* `DATE_ADD(date,INTERVAL expr type)`, `DATE_SUB(date,INTERVAL expr type)`, `ADDDATE(date,INTERVAL expr type)`, `SUBDATE(date,INTERVAL expr type)` 这些功能执行日期运算
* `TO_DAYS(date)`: 给出一个日期 date，返回一个天数(从 0 年的天数)。
* `FROM_DAYS(N)` : 给出一个天数 N，返回一个 DATE 值
* `DATE_FORMAT(date,format)` : 根据 format 字符串格式化 date 值
* `TIME_FORMAT(time,format)` : 这象上面的 DATE_FORMAT() 函数一样使用，但是 format 字符串只能包含处理小时、分钟和秒的那些格式修饰符。其他修饰符产生一个NULL值或 0。
* `CURDATE()`
* `CURRENT_DATE`: 以 'YYYY-MM-DD' 或 'YYYYMMDD' 格式返回今天日期值，取决于函数是在一个字符串还是数字上下文被使用。
* `CURTIME()`
* `CURRENT_TIME` : 以 'HH:MM:SS' 或 'HHMMSS' 格式返回当前时间值，取决于函数是在一个字符串还是在数字的上下文被使用。
* `NOW()`, `SYSDATE()`,`CURRENT_TIMESTAMP`: 以'YYYY-MM-DD HH:MM:SS'或 'YYYYMMDDHHMMSS' 格式返回当前的日期和时间，取决于函数是在一个字符串还是在数字的上下文被使用。
* `UNIX_TIMESTAMP()`
* `UNIX_TIMESTAMP(date)` : 如果没有参数调用，返回一个 Unix 时间戳记(从'1970-01-01 00:00:00'GMT开始的秒数)。如果 UNIX_TIMESTAMP() 用一个 date 参数被调用，它返回从'1970-01-01 00:00:00' GMT开始的秒数值。date 可以是一个 DATE 字符串、一个 DATETIME 字符串、一个 TIMESTAMP 或以 YYMMDD 或 YYYYMMDD 格式的本地时间的一个数字。
* `FROM_UNIXTIME(unix_timestamp)` : 以'YYYY-MM-DD HH:MM:SS' 或 `YYYYMMDDHHMMSS` 格式返回 unix_timestamp 参数所表示的值，取决于函数是在一个字符串还是或数字上下文中被使用。
* `SEC_TO_TIME(seconds)` : 返回 `seconds` 参数，变换成小时、分钟和秒，值以 `HH:MM:SS` 或 `HHMMSS` 格式化，取决于函数是在一个字符串还是在数字上下文中被使用。
* `TIME_TO_SEC(time)` : 返回 time 参数，转换成秒。

与 `GROUP BY` 子句一起使用的函数:

* `COUNT(expr)` : 返回由一个 SELECT 语句检索出来的行的非 NULL 值的数目
* `COUNT(DISTINCT expr,[expr...])` : 返回一个不同值的数目
* `AVG(expr)` : 返回 expr 的平均值
* `MIN(expr)`, `MAX(expr)` : 返回 expr 的最小或最大值。MIN() 和 MAX() 可以有一个字符串参数
* `SUM(expr)` : 返回 expr 的和。注意，如果返回的集合没有行，它返回NULL

其他函数:

* `DATABASE()` : 返回当前的数据库名字。
* `USER()`, `SYSTEM_USER()`, `SESSION_USER()` : 返回当前 MySQL 用户名。
* `PASSWORD(str)` : 从纯文本口令 str 计算一个口令字符串。该函数被用于为了在 user 授权表的 Password 列中存储口令而加密 MySQL 口令
* `ENCRYPT(str[,salt])`: 使用 Unix crypt() 系统调用加密 str。salt 参数应该是一个有2个字符的字符串。（MySQL 3.22.16中，salt可以长于2个字符。）
* `ENCODE(str,pass_str)` : 使用 pass_str 作为口令加密 str。为了解密结果，使用 DECODE()。结果是一个二进制字符串，如果你想要在列中保存它，使用一个 BLOB 列类型。
* `DECODE(crypt_str,pass_str)` : 使用 `pass_str` 作为口令解密加密的字符串 `crypt_str`。crypt_str 应该是一个由 ENCODE()返回的字符串。
* `MD5(string)` : 对字符串计算 MD5校验和。值作为一个32长的十六进制数字被返回可以，例如用作哈希(hash)键。
* `LAST_INSERT_ID([expr])` : 返回被插入一个 AUTO_INCREMENT 列的最后一个自动产生的值
* `FORMAT(X,D)` 格式化数字X为类似于格式 '#,###,###.##'，四舍五入到D为小数。如果D为0，结果将没有小数点和小数部分。
* `VERSION()` 返回表明MySQL服务器版本的一个字符串。

## 5. 导入导出

### 5.1 导出某个库

    mysqldump -h$host -u$user -p$password $database > xxx.sql

### 5.2 导出指定表

    mysqldump -h$host -u$user -p$password $database $table > xxx.sql

### 5.3 只导出表结构

    mysqldump -h$host -u$user -p$password $database --no-data > xxx.sql

### 5.4导入

导出的结果其实就是一连串的sql语句,所以导入只需要执行xxx.sql里的语句

    mysql -h$host -u$user -p$password $database < xxx.sql

## 6. Tips

+ 在控制台执行一条SQL语句: `mysql -h127.0.0.1 -uuser -ppassword db_name -e "select xxx from xxx" > aa.txt`
+ 控制台中文乱码: `set names 'utf8'`
+ Mysql 报错: Row size too large (> 8126). Changing some columns to TEXT or BLOB or using ROW_FORMAT=DYNAMIC ... 解决方案: [innodb使用大字段text，blob的一些优化建议](http://hidba.org/?p=551)。
+  [Get record counts for all tables in MySQL database]( http://stackoverflow.com/questions/286039/get-record-counts-for-all-tables-in-mysql-database)

## 7. 扩展资料

+ [SQL truncate 、delete与drop区别](http://www.cnblogs.com/8765h/archive/2011/11/25/2374167.html)
+ [MYSQL数据库管理之权限管理](http://blog.chinaunix.net/uid-20639775-id-3249105.html)
+ [根据多年经验整理的《互联网MySQL开发规范》](http://blog.sina.com.cn/s/blog_1380b3f180102vsg5.html)

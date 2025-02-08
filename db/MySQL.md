# 安装

- 8.4.3，分为商业版和社区版
- [社区版官网](https://downloads.mysql.com/archives/community/)

## 单机版

### 安装包

![image-20250206152906533](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250206152906533.png)

#### 安装

```bash
# 1. 发送文件并解压文件
put /Users/shuzhan/Desktop/mysql-8.4.3-1.el7.x86_64.rpm-bundle.tar /opt
# 解压对应的文件得到多个文件,标注1的是需要的
tar -xvf mysql-8.4.3-1.el7.x86_64.rpm-bundle.tar

mysql-community-client-8.4.3-1.el7.x86_64.rpm                      # 1
mysql-community-client-plugins-8.4.3-1.el7.x86_64.rpm              # 1
mysql-community-common-8.4.3-1.el7.x86_64.rpm                      # 1
mysql-community-debuginfo-8.4.3-1.el7.x86_64.rpm
mysql-community-devel-8.4.3-1.el7.x86_64.rpm
mysql-community-embedded-compat-8.4.3-1.el7.x86_64.rpm
mysql-community-icu-data-files-8.4.3-1.el7.x86_64.rpm              # 1
mysql-community-libs-8.4.3-1.el7.x86_64.rpm                        # 1
mysql-community-libs-compat-8.4.3-1.el7.x86_64.rpm
mysql-community-server-8.4.3-1.el7.x86_64.rpm                      # 1
mysql-community-server-debug-8.4.3-1.el7.x86_64.rpm
mysql-community-test-8.4.3-1.el7.x86_64.rpm

# 在当前目录下，创建目录
mkdir mysql

# 将上面标注了1的文件，都拷贝到erick_mysql下面
cp mysql-community-c* mysql/
cp mysql-community-icu-data-files-8.4.3-1.el7.x86_64.rpm mysql
cp mysql-community-libs-8.4.3-1.el7.x86_64.rpm mysql
cp mysql-community-server-8.4.3-1.el7.x86_64.rpm  mysql
```

```bash
# 2.检查MySQL依赖
# 检查/tmp临时目录权限(必不可少)
- mysql在安装过程中，会通过mysql用户在/tmp目录下新建 tmp_db文件，所以请给/tmp较大权限
cd /
chmod -R 777 /tmp

# 检查libaio 和 net-tools依赖
# 存在时候的结果： libaio-0.3.109-13.el7.x86_64,     net-tools-2.0-0.24.20131004git.el7.x86_64
rpm -qa | grep libaio 
rpm -qa | grep net-tools

# 不存在的话安装
yum install -y libaio
```

```shell
# 3. 安装
# 严格按照下面rpm顺序
cd opt/mysql
rpm -ivh mysql-community-common-8.4.3-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-plugins-8.4.3-1.el7.x86_64.rpm

# 执行时会
#warning: mysql-community-libs-8.4.3-1.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID a8d3785c: #NOKEY
#error: Failed dependencies:
#	mariadb-libs is obsoleted by mysql-community-libs-8.4.3-1.el7.x86_64

# 先删除之前安装的lib
yum remove mysql-libs

rpm -ivh mysql-community-libs-8.4.3-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-8.4.3-1.el7.x86_64.rpm
rpm -ivh mysql-community-icu-data-files-8.4.3-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-8.4.3-1.el7.x86_64.rpm

# 检查安装
# mysql  Ver 8.4.3 for Linux on x86_64 (MySQL Community Server - GPL)
mysql --version
mysqladmin --version
```

#### 启动

```bash
# 0.为root用户生成一个临时密码并将该密码标记为过期，登录后需要设置一个新的密码
# 临时密码保存在 /var/log/mysqld.log
mysqld --initialize --user=mysql
cat /var/log/mysqld.log   # zTKzook?P1O6

# 1. 启动mysql
systemctl start mysqld
systemctl stop mysqld
systemctl restart mysqld

systemctl status mysqld                             # 是否启动
systemctl list-unit-files | grep mysqld.service     # 服务器启动后，mysql是否自启动：  enabled(默认值)代表是
systemctl enable mysqld.service                     # 如果不是，设置为自启动
systemctl disable mysqld.service                    # 关闭自启动
```

#### 登录

```sql
# CLI-登陆
# 输入上面保存的密码
mysql -u root -p          

# 执行任意sql，则报错
You must reset your password using ALTER USER statement before executing this statement.

# 先修改密码
alter user 'root'@'localhost' identified by 'erick_123456';

# 退出mysql，重新使用新密码登录, 可执行查询sql
```

```bash
# 远程-登陆
# 默认的root用户，只能是localhost来访问
USE mysql;
select host, user from user;

# 修改
UPDATE user SET host='%' WHERE user='root';

# 一定要刷新一下
FLUSH privileges;
```

### Docker

```bash
# 拉
docker pull mysql:8.0.27

# -e：配置环境变量
# MYSQL_ROOT_PASSWORD设置root用户的密码
# 一定要进行容器卷数据备份挂载
# /var/lib/mysql： 容器内数据保存的地方
# /etc/my.conf：   容器内配置文件保存的地方
docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=Sz19910917@ -d 3218b38490ce

# 进入容器登陆
mysql -u root -p

# 随后用datagrip进行连接
user=root
pwd=Sz19910917@
```

# 基础入门

## 1. 简介

### 概念

- Persistence：数据保存到可掉电的存储设备中，内存中数据固化到硬盘上
- 持久化方式：(非)关系型数据库，磁盘文件，XML文件

```bash
# DB：             Database
- 数据结构化存储仓库，保存数据

# RDBMS：           Relational Database Management System
```

```bash
# RDBMS
- 数据结构为二维表格形式
- 包含database, table, row, column
- 不同table对应实体，支持不同table联查
- 支持复杂查询，事务

# 非RDBMS
- K-V类型数据库:        Redis
- Document类型数据库:   Mongodb, 存储xml或json
- 图形类型数据库: 
- 搜索引擎数据库： es
      # 不需要经过SQL层的解析，性能高
```

### SQL

- Structured Query Language: 与关系型数据库交互的语言
- 美国国家标准局制定，其中SQL92和SQL99影响深远
- SQL是官方制定的标准，不同的DB厂商对其进行不同的实现

```bash
# 1. 分类
# DDL:       Data Definition Languages
- CREATE   ALTER   DROP   RENAME    TRUNCATE
- 定义不同的库，表，视图，索引等数据库对象

# DML:       Data Manipulation Language
- INSERT   DELETE    UPDATE    SELECT
- 定义对表的具体操作

# DCL:       Data Control Language
- COMMIT    ROLLBACK   SAVEPOINT   GRANT    REVOKE     
- 定义库，表，字段，用户的访问权限和安全级别
```

```sql
# \G 作为sql语句结束符，也可以使用 \G来结束，查询结果垂直显示
SELECT user(),now(),version()\G

# \c : 取消当前sql的执行
SELECT now(),version(),user()\c
# 上面只在CLI中可以使用

# SQL结束用 ；， 一条SQL可换行

# 同一行输入多条指令，用  ； 隔开即可，多条sql可以一起执行
SELECT now();SELECT version();SELECT user();
```

## 2. 数据类型

- 一个字节8位

```sql
整数类型：           TINYINT    SMALLINT  MEDIUMINT   INT    INTEGER   BIGINT
浮点类型：           FLOAT      DOUBLE
定点数类型：          DECIMAL
位类型：             BIT
日期类型：           YEAR  TIME    DATE   DATETIME    TIMESTAMP
文本字符串类型：      CHAR  VARCHAR TINYTEXT  TEXT  MEDIUMTEXT  LONGTEXT
枚举类型：           ENUM
集合类型             SET
```

### 整数

```bash
# 无符号类型：加上UNSIGNED
# 建议使用大点的数据

# 整数类型       字节         有符号取值范围           无符号范围               应用场景
TINYINT          1            -128～127               0～255           一般用于枚举类型
SMALLINT         2                                                    较小范围的统计数据
MEDIUMINT        3                                                     较大整数的计算
INT、INTEGER     4                                                     取值范围足够大，一般就用这个
BIGINT           8                                                     只有处理特别巨大的整数才会用到

# 整数类型，就用INTEGER
# 如果字段是非负数，必须是UNSIGNED进行约束
```

### 浮点

```bash
# 浮点类型          字节          描述
FLOAT               4          单精度
DOUBLE              8          双精度

# double(m,n), float(m,n), 
- m代表总的数字长度：  则整数长度为m-n，小数长度为n
- 如果存入的小数位的精度超过了指定精度，小数位会四舍五入（可能存在精确度丢失的问题）
- 如果整数位超了，则报错

# UNSIGNED: 并不会改变正数数据的范围，因为符号位也占用空间，因此没必要显示声明UNSIGNED 
```

### 定点

```bash
# DECIMAL(5,2): -999.99~999.99
-- 底层是用字符串存储
-- 不指定的默认的是（10，0)
-- 超过精度的话，进行四舍五入
-- 金融类的推荐用Decimal,一分钱都不能差

# 小数类型推荐使用DECIMAL，禁止使用float和double
- 如果超过范围，可以将decimal拆成整数和小数分开存储
```

### 日期时间

```bash
# 类型           字节         格式                         最小值                       最大值
YEAR             1          YYYY/YY                      1901                        2155
TIME             3          HH:MM:SS                   -838:59:59                   838:59:59
DATE             3          YYYY-MM-DD                  1000-01-01                  9999-12-03
DATETIME         8       YYYY-MM-DD HH:MM:SS         1000-01-01 00:00:00          9999-12-31 23:59:59
TIMESTAMP        4       YYYY-MM-DD HH:MM:SS          1970-01-01 00:00:00 UTC    2038-01-19 03:14:07 UTC
```

```bash
# DATETIME
- 插入的数据和读取时一样， 开发中建议使用DATETIME

# TIMESTAMP
- 会根据时区变化， 底层存储的是毫秒数，查询日期时间差比较快
- 注册时间，商品发布等: 建议使用 TIMESTAMP，因为DATETIME不方便计算
# SELECT UNIX_TIMESTAMP()
```

### 文本

- 一个英文字符一般占用1个字节，一个中文字符占用1～2个字节

```bash
# 类型        特点        长度范围字节         实际存储空间字符       空间        查找效率        场景
CHAR(M)      固定长度      0=< M =<255           M               浪费         高       存储不大，追求查询速度 
VARCHAR(M)   可变长度      0=< M =<65535      实际长度+1          节省          低       非char的情况

# char
- 需预定义长度，默认1个字节
- 实际填充的数据如果比声明的长度M小，则在右侧填充空格达到指定长度
- 检索时，char类型会自动去除尾部空格
- 门牌号，uuid等定长的，变更频繁的column

# varchar
- 必须指定最大长度
- 实际数据超出定义的最大长度M，则报错
- 实际数据小于定义的最大长度M, 会根据实际占用长度，同时占用一个字节来保存字符长度
- 检索时，保留数据尾部的空格
- 一般长度尽可能别超过5000，否则就使用text，并独立出一张表，用主键来对应

# 因为影响性能的主要因素是数据存储总量，推荐varchar
```

```bash
# 类型              长度范围               实际存储空间
TINYTEXT           0~255                  L + 2
TEXT               0～65535               L + 2
MEDIUMTEXT         0~16777215             L + 3
LONGTEXT           0~4294967295           L + 4

# TEXT: 主要用来存储大文本，比如评论，博客记录等
# 频繁使用的表，不建议使用text，可以把该字段单独分表出去
```

```sql
# 类型                  长度范围                    存储空间
ENUM                   1～65535                   1或2个字节
SET                    0～64                      1，2，3，4，8个字节

-- 插入时候只能插入一个值
ADD season ENUM ('春', '夏', '秋', '冬') null;
ADD season SET ('春', '夏', '秋', '冬') null;
```

## 3. 库表操作

### 库

```sql
SHOW DATABASES;          -- 查看当前用户账户下所有库
SELECT DATABASE();       -- 查看当前使用的库， 每次开启新的会话，当前为null
USE lucycat;             -- 切换库

-- 判重是用库名来判断， 数据库名不能改                    
CREATE DATABASE IF NOT EXISTS shuzhan_db;   -- 创建库
SHOW CREATE DATABASE shuzhan_db;            -- 查看库信息
DROP DATABASE IF EXISTS my_db;              -- 删除库

-- 字符集： 默认为utf8mb4
CREATE DATABASE my_db CHARACTER SET 'utf8mb4';        -- 创建时
ALTER DATABASE shuzhan_db CHARACTER SET 'utf8mb4';    -- 修改库的字符集
```

### 表

- 表操作，要么库名.表名，要么在sql脚本前指定用那个库

```sql
-- 列出当前数据库所有表
SHOW TABLES;
SHOW TABLES  FROM shuzhan_db;

-- 1. 创建
CREATE TABLE IF NOT EXISTS people
(
    name         VARCHAR(20) COLLATE utf8mb4_unicode_ci                                  DEFAULT NULL,
    weight       INTEGER                                                            DEFAULT NULL,
    birth_season ENUM ('SPRING','SUMMER','FESTIVAL','WINTER') COLLATE utf8mb4_unicode_ci DEFAULT NULL
) CHARSET = utf8mb4
  COLLATE = utf8mb4_unicode_ci;
  
-- 2. 描述  
DESC/DESCRIBE/EXPLAIN people;
SHOW COLUMNS/FIELDS FROM people;

DESC/DESCRIBE/EXPLAIN president '%name';       -- 只描述带有name的字段
SHOW COLUMNS/FIELDS FROM president LIKE '%name';  

SHOW FULL FIELDS/COLUMNS FROM people;           -- 查看行数据的详细信息

SHOW CREATE TABLE people;                       -- 查看表

-- 3. 修改
ALTER TABLE employee ADD 'status' boolean;                        -- 添加field，默认添加到最后
ALTER TABLE employee ADD home varchar(50) FIRST;
ALTER TABLE employee ADD hobby varchar(50) AFTER home;

ALTER TABLE employee MODIFY home VARCHAR(100);                    -- 修改field
ALTER TABLE employee MODIFY 'status' BOOLEAN DEFAULT 1;

ALTER TABLE employee CHANGE 'status' is_enabled BOOLEAN;                -- 重命名field
ALTER TABLE employee CHANGE home myhome VARCHAR(200);                   -- 重命名字段名，并且修改长度
 
ALTER TABLE employee DROP is_enabled;                                  -- 删除field
ALTER TABLE employee DROP COLUMN is_enabled;
  
RENAME TABLE employee TO employees;                                    -- 重命名table
ALTER TABLE  employees RENAME TO employee;
 
ALTER TABLE people ENGINE = MyISAM;                                     -- 修改table存储引擎
```

```sql
-- 删除
-- 1. 删除表结构和表数据，释放表空间，              不能回滚
DROP TABLE cnip;
DROP TABLE cnip,phone;
DROP TABLE IF EXISTS cnip;

-- 2. 清空表: 表数据清除，表结构保留
TRUNCATE TABLE people;     
DELETE FROM people;        

# TRUNCATE:   
- 速度较快，且使用的系统和事务日志资源少
- 不支持rollback
- 无事务且不触发TRIGGER，有可能造成事故，因此不建议使用

# DELETE: 
- 可以部分删除
- 可以利用事务进行rollback
```

```sql
# 表复制
-- SELECT 结构可以比较复杂，同时也可以对其中的字段其别名，作为新表的字段名
-- 1. 复制表结构和数据
CREATE TABLE us_phone AS SELECT * FROM phone;

-- 2. 原表的部分复制
CREATE TABLE us_apple_phone AS SELECT * FROM phone WHERE brand = 'Apple';  

-- 3. 原表的结构复制
CREATE TABLE us_apple_phone AS SELECT * FROM phone WHERE 1=2; 
CREATE TABLE cn_phone LIKE phone;
```

```sql
-- 表分区
-- 大数据量表，读写操作效率低，将一个大表的行数据，分配到不同的表

-- 1.按照时间分区
-- 在var/lib/mysql/库名 目录下，会创建 phone#P#p1.ibd类似的数据保存数据文件
CREATE TABLE phone
(
    id           INT,
    name         CHAR(20),
    created_time DATETIME NOT NULL
)
    PARTITION BY RANGE (YEAR(created_time))
        (PARTITION p0 VALUES LESS THAN (2010),
        PARTITION p1 VALUES LESS THAN (2011),
        PARTITION p2 VALUES LESS THAN (2012),
        PARTITION p3 VALUES LESS THAN (2013),
        PARTITION pmax VALUES LESS THAN MAXVALUE);

-- 2. 如果后续数据都超过了2013，则可以对pmax分区进行再分区
ALTER TABLE phone
    REORGANIZE PARTITION pmax
        INTO (
        PARTITION p4 VALUES LESS THAN (2014),
        PARTITION p5 VALUES LESS THAN (2015),
        PARTITION pmax VALUES LESS THAN MAXVALUE
        )
```

### 行

```sql
-- 全列插入：必须包含所有的列数据。不推荐这种方式
-- 批量快
INSERT INTO president VALUES (1,"shu","zhan");
INSERT INTO president VALUES (2,"wu","wang"),(3,"si","li");

-- 指定列名插入， 对于自增的键，如果赋值就取赋值的，不赋值则取默认的
INSERT INTO president (last_name,first_name,id) VALUES ("八","王",10);
INSERT INTO president (last_name,first_name,id) VALUES ("六","赵",5),("七","田",6);

-- set赋值方法：主键就没有给值，采取默认的即可
INSERT INTO president SET first_name = "朱", last_name = "院长";

-- 查询结果插入数据
INSERT INTO phone(name, created_time)
SELECT name, created_time
FROM phone;
```

```sql
-- 1. 删除: 删除所有数据
DELETE FROM cnip;

-- 指定删除： 一般删除时，必须加where子句，避免误删
DELETE FROM cnip WHERE name = 'zhangsan';

-- 2. 更新操作: 更新所有数据的name
UPDATE cnip SET name = 'sz';
-- 指定更新：更新时候，一般必须加where子句，避免误更新
UPDATE cnip SET name = 'sz' WHERE id = 4;
-- 指定更新多列数据
UPDATE cnip SET name = 'sz', color = 'red' WHERE id = 4;
-- 更新某列的值为NULL
UPDATE cnip SET name = NULL where id = 4;
```

## 4. 字符集

### 字符集级别

- 字符集表示一个字符所用的最大字节长度

```sql
-- 默认字符集合
SHOW VARIABLES LIKE 'character%';

character_set_client,        utf8mb4
character_set_connection,    utf8mb4
character_set_database,      utf8mb4            # 具体的database
character_set_filesystem,    binary
character_set_results,       utf8mb4
character_set_server,        utf8mb4             # server端
character_set_system,        utf8mb3 
character_sets_dir,          /usr/share/mysql-8.4/charsets/
```

```bash
# CHARACTER SET：从上到下，如果不显式指明当前层的字符集，则使用上一层的字符集
# COLLATE： 比较规则，类似上述法则

#  级别                                              描述
character_set_server                       mysql服务器级别的字符集
character_set_database                     当前数据库的字符集
表级别                                      可以在创建或者修改表的时候指定表的字符集和比较规则
列级别
```

```SQL
# 1. 数据库级别
CREATE DATABASE db_test2 CHARACTER SET utf8mb4;            # 创建时指明
ALTER DATABASE db_test2 CHARACTER SET utf8mb3;             # 修改字符集
SHOW CREATE DATABASE db_test2;                             # 查看当前数据库字 符集 

# 2. 表级别
CREATE TABLE book1
(
    name VARCHAR(10)
) CHARACTER SET utf8mb4;

ALTER TABLE book1 CHARACTER SET utf8mb3;      #修改
SHOW CREATE TABLE book1;                      # 查看

# 3. 列级别
CREATE TABLE book2
(
    address VARCHAR(10),
    name    VARCHAR(10) CHARACTER SET gbk
) CHARACTER SET utf8mb4;                                        # 创建


ALTER TABLE book2
    MODIFY name VARCHAR(20) CHARACTER SET utf8mb3;               # 修改
```

### 常见字符集

- mysql一共支持41种字符集

```sql
SHOW CHARACTER SET;           # 查看mysql中支持的字符集合以及其对应的比较规则

# Default collation：         当前字符集种默认的比较规则
# Maxen：                     该字符集表示一个字符最多需要几个字节


#  后缀                    英文解释                            描述             
_ai                   accent insensitive                   不区分重音       
_as                    accent sensitive                     区分重音         
_ci                      case insensitive                 不区分大小写    
_cs                      case sensitive                    区分大小写       
_bin                          binary                       以二进制方式比较 


# 查看某个字符集支持的比较规则
SHOW COLLATION LIKE 'utf8%';
SHOW COLLATION LIKE 'gbk%';

# 查看服务器的字符集及比较规则
SHOW VARIABLES LIKE '%_server';

# 查看数据库的字符集及比较规则
SHOW VARIABLES LIKE '%_database';
```

![image-20250208180321558](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250208180321558.png)

```bash
# mysql中，utf8是utf8mb3的别名
# utf8              
- 一个字符需要1-4个字节来表示，a，b，c用1字节，汉字用3字节     
#  utf8mb4 
- 正宗的utf8字符集，使用1-4个字节表示字符，可以存储一些emoji表情 
#  utf8mb3 
- 阉割过的utf8字符集，只使用1-3个字节表示字符                  
```

# 查询

## 基本查询

### 1. 简单查询

```sql
-- 1. 基本使用
SELECT * FROM cnip; 
SELECT name,age FROM cnip;
SELECT 100;  SELECT 'mick';                         -- 常量,可以给每行添加一个对应的字段    
SELECT (2+2);                                       -- 表达式
SELECT 1+1 FROM DUAL;                               -- 伪表
SELECT VERSION();                                   -- 函数
SELECT * FROM cnip_db.cnip WHERE cnip.name = '李明'; -- 全限定表名，列名


-- 2.别名：
-- 改变结果集中字段显示名称；    多表联查，重复字段进行区分
SELECT VERSION() AS version;              -- 建议加AS
SELECT VERSION() version;                 -- 可以不写
SELECT VERSION() 'my version';            -- 别名中存在空格，用或“”
-- 列取了别名，如果想使用该别名进行过滤，该别名只能在order by后使用
SELECT *, salary * 10 AS allsalary FROM book1 ORDER BY allsalary;        # 正常
SELECT *, salary * 10 AS allsalary FROM book1 WHERE allsalary > 10;      # 报错


-- 3. 去重：select后单独使用
SELECT DISTINCT name FROM phone;           -- 获取不重复的某个字段的value
SELECT DISTINCT name,age FROM cnip;        -- 用于多列，即行中name和age都重复了，才去重
# SELECT name,DISTINCT age FROM cnip;        -- error：因为表中数据对不上了


-- 4. 拼接： 不能用 + ，     sql中 + 只能当作运算符
SELECT CONCAT('姓名:',first_name,'---',last_name) AS name FROM president;


-- 5. 转义符： 如果表，字段名和mysql关键字重复了，可以使用转义符
SELECT `order` FROM people;

-- 6. UNION   UNION ALL
-- UNION 会进行去重， 但是去重操作会耗费时间
-- UNION ALL: 存在重复的数据，但效率高，推荐
SELECT * FROM cnip where name='erick'

UNION

SELECT * FROM cnip where name='tom';

-- 7.计算运算符
--       +  -  *  /(DIV)  %(MOD)
SELECT 100 + 99;
SELECT '100' + 99;  -- 隐式转换：尝试将字符转换为数字并做运算，如果转换不成功则将字符视为0
SELECT 'dfa' + 99;
SELECT 100 DIV 20;   -- 5
```

### 2. 比较运算符

```sql
# = 返回的是0或者1     {！=,   < ,   >,    <=,   >= } 
SELECT 1 = 1, 1 = 2;                          -- 两边都是数字，则比较大小，
SELECT 1 = '1', 1 = 'a', 0 = 'a';             -- 字符串隐式转换，转换不成功则字符串看作0       101
SELECT 'A' = 'A', 'Ab' = 'AB', 'a' = 'b';     -- 两边都是字符串，则按照ANSI的比较规则
```

```sql
## NULL参与运算
-- 1. 只要有NULL参与运算，null不等同于0，‘’，‘null’,   则结果都是null
SELECT 1 = NULL, NULL = NULL, 12*NULL;
SELECT null + 10;
SELECT IFNULL(salary,0) * 10                    -- 可以通过流程函数来控制: 如果为null，则转换为0
SELECT * FROM cnip WHERE name=null;             # 不会返回一条记录, 因为只返回比较结果为1的数据

-- 2.1 <=>:  安全等于，null不参与时和=没区别， 为null而生
-- 两边都是null时为1，只有一个null时为0
SELECT NULL <=> NULL, 1 <=> NULL;
SELECT * FROM cnip WHERE name<=>NULL;           -- 会返回对应name为null的记录

-- 2.2 非空判断
-- IS NULL         IS NOT NULL      和<=>相同
SELECT * FROM phone WHERE name IS NULL;
SELECT * FROM phone WHERE name IS NOT NULL;
```

### 3. 范围/离散/模糊

```sql
-- 匹配时： 字符的一定要用‘’,     数值类型，不用

-- 1. 范围查找
-- BETWEEN(下限) AND(上限)      NOT BETWEEN AND
SELECT * FROM phone WHERE age BETWEEN 20 and 100;        --  包含临界值
SELECT * FROM phone WHERE age NOT BETWEEN 32 AND 100;

-- 2. 离散查找
-- IN     NOT IN
SELECT * FROM phone WHERE name in ('tony','jack','lucy');

-- 3. 模糊查询
-- % 代表0-n个字符，  - 只匹配一个字符；
-- 不能匹配null值
-- 英文匹配时候不区分大小写；
SELECT * FROM cnip WHERE content LIKE '%zte%';
SELECT * FROM cnip WHERE content LIKE 'z%e';
SELECT * FROM cnip WHERE content LIKE '_te';
SELECT * FROM cnip WHERE content LIKE '__te';
SELECT * FROM phone WHERE name like '%\_%';   -- 转义符号   \  查询包含 _ 的 字符
SELECT * FROM phone WHERE name like '%^_%' ESCAPE '^';   -- 自定义转义符号为 ^


-- 4. 按照字典来比较
-- LEAST()     GREATEST()   
SELECT LEAST('A', 'B', 'F', 'Z', 'G');
SELECT GREATEST('A', 'B', 'F', 'Z', 'G');
SELECT LEAST('erick', 'edjk', 'adf');                              -- 首先按照第一个字母比较，以此类推
SELECT LEAST(LENGTH('erick'), LENGTH('dfg'), LENGTH('dgadfaf'));   -- 比较长度
```

### 4. 逻辑运算符

```sql
-- 逻辑运算符
--   OR ｜｜     AND &&      NOT !
SELECT * FROM people WHERE name = 'erick' || name = 'eri';
SELECT * FROM people WHERE name = 'erick' AND name = 'eri';
SELECT * FROM book1 WHERE NOT id=1;
```

### 5. 排序/分页

```sql
# 排序
SELECT * FROM people;                          # 不指定时，按照数据添加顺序
SELECT * FROM people ORDER BY name DESC;       # 默认升序/ASC   DESC/降序

-- 多字段(二级排序)： 先按第一个字段排序，第一个字段相同再按第二个排序
SELECT * FROM president ORDER BY first_name DESC, last_name DESC;

-- 区分null值的排序： 某个字段含null值的排序
-- 先按照该字段的null和非null排序，再按照字段具体内容排序
SELECT * FROM president ORDER BY IF(address IS NULL,0,1) DESC, address DESC;
```

```sql
# 分页
SELECT * FROM cnip LIMIT 10;               -- 前10条
SELECT * FROM cnip LIMIT 5,10;             -- 跳过前面5个，检索10个
SELECT * FROM cnip LIMIT 10 OFFSET 5;      
```

## 聚合函数

- 聚合函数不能嵌套使用，必须使用子查询
- 聚合函数统计时，都会忽略null值。分子和分母都会忽略该行

```sql
-- SUM()  AVG()：               参数必须是数值类型 
-- MAX()  MIN()：               参数为任何类型

SELECT MAX(age) from phone;
SELECT SUM(DISTINCT(brand)) FROM phone;  -- 和distinct搭配
```

```sql
-- COUNT():                    参数为任何类型

-- COUNT(*)    COUNT(field)    COUNT(1) 
-- COUNT(*)，COUNT(1)：     统计不为null的行数目， 一行中只要一个字段不为null，总数就会计算该行
-- COUNT(field):           如果某个字段不为null，就统计

-- 性能： 
INNODB:    COUNT(*)=COUNT(1) > COUNT(filed)
MYISAM:    三个都一样， 引擎内部有计数器来维护行数
```

```sql
# Having只能结合聚合函数使用
# select中的非聚合函数的字段，一定要出现在group by后面

-- 单列分组
SELECT AVG(salary), SUM(salary), nation
FROM employee
GROUP BY nation;

-- 多列分组: 先按照a字段分组，再根据b字段分组，统计最小组的数据
-- 字段前后无区别,因为from前面的是聚合函数
SELECT nation, address, SUM(salary)
FROM employee
GROUP BY nation, address;
```

- 过滤条件中如果使用了聚合函数，则必须使用HAVING来替换WHERE

```sql
-- HAVING不能单独使用，必须结合GROUP BY
-- HAVING必须声明在GROUP BY后面
SELECT nation, MAX(salary)
FROM employee
GROUP BY nation
HAVING MAX(salary) > 20000;
```

- 过滤条件中有聚合函数的，则过滤条件必须声明在HAVING中
- 过滤条件不是聚合函数的，则过滤条件声明在HAVING和WHERE都可，但是建议声明在WHERE中

```sql
-- 推荐，where, 实现效率高
SELECT address, MAX(salary)
FROM employee
WHERE address = '上海'
GROUP BY address
HAVING MAX(salary) > 10;

SELECT address, MAX(salary)
FROM employee
GROUP BY address
HAVING MAX(salary) > 10
   AND address = '上海';  
```

```sql
# 执行过程
-- 书写顺序： 
SELECT>>>FROM>>>WHERE>>>GROUP BY>>>HAVING>>>ORDER BY>>>LIMIT

-- 执行顺序：
FROM-> WHERE->GROUP BY->HAVING->SELECT字段->DINSTINCT->ORDER BY ->LIMIT

-- 尽可能每次执行，都最多的去过滤掉数据
```

## 单行函数

### 数值/字符串函数

| 数值函数                | DESC                                                         |
| ----------------------- | ------------------------------------------------------------ |
| ABS(1)                  | 绝对值                                                       |
| SIGN(X)                 | 返回x的符号，正数返回1，负数返回-1，0返回0                   |
| PI()                    | 圆周率                                                       |
| CEIL(1.5)  CEILING(1.5) | 向上取整  >= 该参数的最小整数                                |
| FLOOR(1.5)              | 向下取整                                                     |
| LEAST(e1,e2,e3)         | 返回最小的                                                   |
| GREATEST(e1,e2,e3)      | 返回最大的                                                   |
| MOD(x,y)                | 返回x除以y后的余数                                           |
| RAND()                  | 0-1 的随机数,可以乘法的到对应的值                            |
| RAND(X)                 | 0-1 的随机数，如果X因子一样，产生的随机数相同(伪随机，可以以时间戳为种子) |
| ROUND(1.5)              | 四舍五入                                                     |
| ROUND(5.32, 1)          | 四舍五入，保留1位小数 5.3                                    |
| TRUNCATE(1.345,2)       | 取小数点后几位  1.34                                         |
| SQRT(2)                 | 返回平方根，x值为负，返回NULL                                |

| 字符串函数                         | DESC                                |
| ---------------------------------- | ----------------------------------- |
| LENGTH(name)                       | 字节长度                            |
| CONCAT(name,'====',brand)          | 字符拼接                            |
| CONCAT_WS('--','beijing','shanxi') | 指定拼接符号拼接                    |
| REPLACE('aaaa','a','bv')           | 用bv替换a                           |
| UPPER(name)                        | 大写转换                            |
| LOWER(name)                        | 小写转换                            |
| LEFT(str,3)， RIGHT(str,2)         | 取字符串左边3个, 最多取当前完整字符 |
| TRIM()   LTRIM().  RTRIM()         | 去除空格                            |

```sql
SELECT CONCAT( LOWER(name), '===',UPPER(brand)) from phone; -- 函数嵌套
SELECT LENGTH('hello'),LENGTH('我们') FROM DUAL;             -- 5, 6, utf8中，1个英文1个字节，1个汉字3个字节
SELECT CONCAT_WS('--','beijing','shanxi') FROM dual;        -- beijing--shanxi
```

### 日期函数

| 函数                                                         | Value               | DESC         |
| ------------------------------------------------------------ | ------------------- | ------------ |
| CURDATE()   /   CURRENT_DATE()                               | 2023-08-28          |              |
| CURTIME()   /   CURRENT_TIME()                               | 16:50:01            |              |
| NOW() / SYSDATE() /  CURRENT_TIMESTAMP() /  LOCALTIME() / LOCALTIMESTAMP() | 2023-08-28 16:52:02 |              |
| UTC_DATE()                                                   | 2023-08-28          | 世界标准时间 |
| UTC_TIME()                                                   | 09:36:29            |              |
| UTC_TIMESTAMP()                                              | 2023-08-28 09:37:08 |              |

```sql
-- 1. 获取年月日周天
YEAR(NOW()), MONTH(NOW()),  DAY(NOW());  -- 年月日,参数date
MONTHNAME(NOW()),WEEKDAY(NOW()),QUARTER(NOW()); 
WEEK(NOW()),WEEKDAY(NOW()),WEEKOFYEAR(NOW());
DAYOFMONTH(NOW()),DAYOFYEAR(NOW()),DAYOFWEEK(NOW());

HOUR(CURRENT_TIME), MINUTE(CURRENT_TIME),SECOND(CURRENT_TIME); -- 时分秒，参数time


-- 2. 日期操作函数 EXTRACT(type from date)
SELECT EXTRACT(SECOND FROM NOW());
SELECT EXTRACT(MINUTE_MICROSECOND FROM NOW());

-- 3. 日期与时间戳转换
SELECT UNIX_TIMESTAMP();        -- 以UNIX时间戳的形式返回当前时间 1652841099
SELECT UNIX_TIMESTAMP(NOW());   -- 将Data转换为UNIX时间戳
SELECT UNIX_TIMESTAMP('2021-09-01: 10:24');

SELECT FROM_UNIXTIME(1652841243); -- 将UNIX时间戳转换普通格式时间 2022-05-18 10:34:03

-- 4. 时间和秒转换
SELECT TIME_TO_SEC(CURRENT_TIME); -- time参数， 小时*3600+分钟*60
SELECT SEC_TO_TIME(39421);        -- seconds参数


-- 5. 修改时间
SELECT DATE_ADD(NOW(), INTERVAL 1 YEAR);   -- 加的具体的值可为负数
SELECT ADDDATE(NOW(), INTERVAL 1 YEAR);
SELECT DATE_SUB(NOW(), INTERVAL 1 MONTH);
SELECT SUBDATE(NOW(), INTERVAL 1 MONTH);
SELECT DATE_ADD(NOW(), INTERVAL '1_1' YEAR_MONTH ); -- 同时加年和月
SELECT DATEDIFF(NOW(),SUBDATE(NOW(),INTERVAL 1 YEAR)); -- 天数
```

```sql
# 指定格式转换
-- 日期转换为字符串，按照指定格式
DATE_FORMAT(date, fmt);
SELECT DATE_FORMAT(NOW(),'%Y-%m-%d');    -- 2022-05-18
SELECT DATE_FORMAT(NOW(),'%Y-%m-%d %H:%i:%s'); -- 2022-05-18 11:25:11

-- 字符串转为日期
-- 字符串和对应的格式需要匹配，不然会报错
STR_TO_DATE(str, fmt)
SELECT STR_TO_DATE('2021-09-05 12:38:16', '%Y-%m-%d %H:%i:%s');
```

```sql
%Y -- 4位数年份
%y -- 2位数年份
%M -- 月名表示月份 January
%m -- 两位数表示月份 01，02
%b -- 缩写的月名  Jan Feb
%c -- 数字表示月份  1，2，11

%D -- 英文后缀月中的天数  1st
%d -- 2位数表示月中的天数 01，22
%e -- 数字表示月中天数。 1，2，3

%H -- 2位数字表示小时   24小时制
%h -- 2位数表示小时,12小时制
%k -- 数字形式的小时， 1，12

%i -- 2位数字表示分钟 00，01
%S %s -- 2位数字表示秒
```

```sql
# 上面的具体的格式，比较难记，可以利用mysql提供的嵌套函数来实现
SELECT STR_TO_DATE('2021-09-05 12:38:16', GET_FORMAT(DATETIME ,'ISO'));
```

![image-20230828174930198](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230828174930198.png)

### 流程控制函数

```sql
# 三元运算符
-- IF(condition,result1, result2)
-- 条件中可以用 = <= >=  is null等判断
SELECT IF( 1=1,'positive','negative' ) as result;  


-- IFNULL(value,0): 如果为空，则给0值，否则就用原来的值
SELECT IFNULL(name,0) FROM phone;
```

```sql
# CASE WHEN
-- 类似switch，是用在from之前的，对于某些字段进行计算
SELECT *,
       CASE brand
           WHEN 'Apple' THEN price * 0.9
           WHEN 'Huawei' THEN price * 0.8
           WHEN 'XiaoMi' THEN price * 0.7
           WHEN 'Oppo' THEN price * 0.6
           ELSE price
           END 
           AS discount_price
FROM phone;

-- 类似多重if，  适用于区间比较。  case 后没有字段
SELECT *,
       CASE
           WHEN price >= 5000 THEN 'LEVEL-A'
           WHEN price >= 4000 THEN 'LEVEL-B'
           WHEN price >= 3000 THEN 'LEVEL-C'
           WHEN price >= 2000 THEN 'LEVEL-D'
           WHEN price >= 1000 THEN 'LEVEL-E'
           ELSE 'LEVEL-LOW'
           END 
           AS product_level
FROM phone;
```

## 多表联查

### 1. 笛卡尔积

- 两个集合X和Y的全排列组合
- A表包含n条数据，B表包含m条数据，则笛卡尔积为n*m条数据

```sql
SELECT * FROM people,phone;                     # A,B
SELECT * FROM people CROSS JOIN phone;          # A CROSS JOIN B
SELECT * FROM people INNER JOIN phone;          # A INNER JOIN B
SELECT * FROM people JOIN phone;                # A JOIN B

-- 加上表别名：表中的字段加上表别名，避免不同表字段重复，同时sql优化提升效率
SELECT peo.name,peo.`order`, pho.name FROM people AS peo,phone AS pho;
```

### 2. 连接条件

- 两个表在笛卡尔积的条件上，增加一个条件

#### 2.1 等值链接/非等值链接

```sql
-- 1 等值链接，  笛卡尔积的各种join都适用
SELECT * FROM student stu,president pre WHERE stu.id = pre.id;

-- 2. 非等值链接
SELECT emp.first_name, sal.grade
FROM employee AS emp,
     salary_level AS sal
WHERE emp.salary > sal.low_level
  AND emp.salary < sal.high_level;
```

#### 2.2 自链接/非自链接

```sql
-- 1. 自链接
SELECT emp.employ_id, emp.first_name, manage.employ_id, manage.first_name
FROM employee AS emp,
     employee AS manage
WHERE emp.manager_id = manage.employ_id;

-- 2. 非自链接
```

### 3. 内连接/外连接

- SQL99 标准
- 采用(A表)   JOIN (B表)  ON 

#### 3.1 内连接

- 合并具有同一列的两个以上的表的行，结果集中不包含一个表与另一个表不匹配的行
- 笛卡尔积加上对应的条件过滤得到的数据

```sql
# 条件过滤
-- sql 92标准：  where
SELECT * FROM student stu,president pre WHERE stu.id = pre.id;

-- sql 99标准：   on
SELECT * FROM employee JOIN salary_level ON employ_id=salary_level.high_level;
SELECT * FROM employee INNER JOIN salary_level ON employ_id=salary_level.high_level;
```

#### 3.2 外连接

- 除了应该有的结果集，还包含不匹配的比如null的结果集
- 一般情况下，考虑到连接部分的字段可能为null，因此查询所有的记录时，就应该直接使用外连接
- <font color=orange>sql92不支持外连接，sql99支持</font>

##### LEFT OUTER JOIN

- 在笛卡尔积的基础上，进行条件过滤后
- 再加上左表中剩下的数据，右边数据用null表示

```sql
-- A LEFT OUTTER JOIN B           A LEFT JOIN B
-- OUTTER可以省略
SELECT * FROM student stu LEFT JOIN president pre on stu.id = pre.id;
SELECT * FROM student stu LEFT OUTTER JOIN president pre on stu.id = pre.id;
```

![img](https://img-blog.csdnimg.cn/20200816173507406.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzM3NDU3OA==,size_16,color_FFFFFF,t_70#pic_center)

##### RIGHT OUTER JOIN

```sql
-- A RIGHT OUTTER JOIN B           A RIGHT JOIN B
-- OUTTER可以省略
SELECT * FROM student stu RIGHT JOIN president pre on stu.id = pre.id;
SELECT * FROM student stu RIGHT OUTTER JOIN president pre on stu.id = pre.id
```

![img](https://img-blog.csdnimg.cn/20200816173812906.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzM3NDU3OA==,size_16,color_FFFFFF,t_70#pic_center)

##### FULL OUTER JOIN

- 满外链接
- 再加上左表中剩下的数据，右边数据用null表示
- 再加上右表中剩下的数据，左边数据用null表示 
- mysql不支持这种写法，但是可以通过UNION来实现

```sql
SELECT * FROM student stu LEFT JOIN president pre on stu.id = pre.id

UNION

SELECT * FROM student stu RIGHT JOIN president pre on stu.id = pre.id;
```

![img](https://img-blog.csdnimg.cn/20200816174323369.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzM3NDU3OA==,size_16,color_FFFFFF,t_70#pic_center)

### 4. SQL99新特性

#### 4.1 USING子句

```sql
-- 如果两个表中的on条件的列名相同，则可以用using来简化sql
SELECT * FROM student stu RIGHT JOIN president pre USING(id);
```

#### 4.2  NATURAL

- 查询两个表中**所有相同**的字段，然后进行等值连接
- 两个表中只有一列的表字段相同，则用NATURAL  JOIN简化sql
- 相同的列只会展示一个

```sql
-- 左连接，右连接，内连接
-- NATURAL LEFT JOIN, NATURAL RIGHT JOIN, NATURAL  JOIN

SELECT * FROM student stu NATURAL LEFT JOIN president pre;
```

![img](https://img-blog.csdnimg.cn/20200816180642864.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzM3NDU3OA==,size_16,color_FFFFFF,t_70#pic_center)

# 子查询

- 一个查询语句嵌套在另一个查询语句内部的查询
- 子查询先执行，主查询后执行。子查询结果要传递给主查询
- 单行子查询对应单行子查询，多行子查询对应多行子查询

## 1. 单行/多行

### 1.1 单行子查询

- 单行子查询，返回的就是一个单字段值

```sql
# 单行操作符： 通过 =    >    >=     <    <=      <>(不等于)来比较

# 先写子查询，再写外查询
SELECT name, salary
FROM employee
WHERE salary > (SELECT salary FROM employee WHERE name = 'tom');


# 可以包含多个子查询
SELECT id, name, salary
FROM employee
WHERE salary > (SELECT salary FROM employee WHERE name = 'lucy')
  AND id > (SELECT id FROM employee WHERE name = 'jack');
  

# 配合聚合函数
SELECT id, name, salary
FROM employee
WHERE salary > (SELECT MIN(salary) FROM employee)

# 空值问题
-- 子查询如果没有记录，则外查询匹配时，也就没有对应的数据
-- 假如没有tom这个人，则比较运算符比较时，条件永远都是false
SELECT name, salary
FROM employee
WHERE salary > (SELECT salary FROM employee WHERE name = 'tom');


-- FROM中也可以跟子查询
-- 聚合函数不能嵌套
SELECT *
FROM employee
WHERE salary > (SELECT MIN(avg_salary)
                FROM (SELECT AVG(salary) AS avg_salary
                      FROM employee
                      GROUP BY id) AS temp_table)
```

### 1.2 多行子查询

```sql
-- 多行子查询： 内查询返回多条数据
-- IN: 列表中任意一个
-- ANY: 需要和单行操作符一起使用，和字查询返回的某一个值比较
-- ALL/SOME: 需要和单行操作符一起使用，和字查询返回的所有值比较

SELECT *
FROM employee
WHERE salary in (SELECT salary FROM employee);

-- 比任一的都小就可以查
SELECT *
FROM employee
WHERE salary < ANY (SELECT salary FROM employee WHERE address = '上海');

-- 比所有的小才行
SELECT *
FROM employee
WHERE salary < ALL (SELECT salary FROM employee WHERE address = '上海');
```

## 2. 相关/不相关

# 约束

```sql
-- 查看
SELECT * FROM information_schema.TABLE_CONSTRAINTS WHERE TABLE_NAME='employee';

# 单列约束 vs 多列约束
# 列级约束：  将约束声明在对应字段的后面
# 表级约束：  在表中所有字段声明后，在后面声明约束，  一般表级约束的目的就是为了给多列加
```

| 约束类型       | 表/列约束 | 单/多列约束 | 个数     | 约束名             | 索引关联 | Other                |
| -------------- | --------- | ----------- | -------- | ------------------ | -------- | -------------------- |
| NOT NULL       | 列级约束  | 单列        | 多个     | NA                 | NA       |                      |
| UNIQUE         | 表/列约束 | 单列/多列   | 多个     | 默认列名，可自定义 | 唯一索引 | 允许列值为多个null   |
| PRIMARY KEY    | 单列/多列 | 单列/多列   | 一个     | PRIMARY            | 主键索引 | 非空+唯一约束        |
| AUTO_INCREMENT | 单列/多列 | 单列/多列   | 最多一个 |                    |          | 约束的必须是整数类型 |
| CHECK          |           |             |          |                    |          |                      |
| DEFAULT        |           |             |          |                    |          |                      |

## 1. NOT NULL

- 所有类型的值都可是NULL, 包括int，float等数据类型
- 空字符串‘’和0不等于null
- 该约束不会出现在information_schema.TABLE_CONSTRAINTS中, 可以通过DESC来查看

```sql
# 1.创建表时
CREATE TABLE erick (name VARCHAR(20) NOT NULL);
 
# 2. 创建表后添加新字段
ALTER TABLE erick
    ADD COLUMN address VARCHAR(20) NOT NULL;

# 3. 创建表后给原字段
ALTER TABLE erick
    MODIFY address VARCHAR(40) NOT NULL;
    
# 4. 删除某个字段的约束
ALTER TABLE erick
    MODIFY address VARCHAR(40) NULL;
```

## 2. UNIQUE

```sql
# 1.创建表时： 可以指定约束名， 可以不指定
CREATE TABLE erick (name VARCHAR(20) UNIQUE);       # 列约束

CREATE TABLE lucy (id int, name VARCHAR(20),
CONSTRAINT uk_name UNIQUE(name));                   # 表级约束，可以指定约束名uk_name，也可以不指定
    
# 2. 创建表后添加   
ALTER TABLE lucy
    ADD CONSTRAINT uk_name UNIQUE (name);           # 表级约束

ALTER TABLE lucy
    ADD COLUMN address VARCHAR(20) UNIQUE;          # 列级约束

ALTER TABLE lucy
    MODIFY address VARCHAR(20) UNIQUE;             # 列级约束
   
   
# 3. 多行数据的约束: 如果不指定约束名，则和复合约束的第一个列名一样
CREATE TABLE user
(
    id       INT,
    name     VARCHAR(20),
    password VARCHAR(20),
    CONSTRAINT uk_user_name UNIQUE (name, password)
);

# 4. 删除：只能通过删除对应的索引
ALTER TABLE user
    DROP INDEX uk_user_name;
```

## 3. PRIMARY KEY

- 主键约束多列时，每个列都不允许为null

```sql
# 1. 创建表时
CREATE TABLE my_erick
(
    id   INT PRIMARY KEY,
    name varchar(20)
);                                   # 列级约束

CREATE TABLE my_erick
(
    id   INT ,
    name varchar(20),
    CONSTRAINT PRIMARY KEY(id)
);                                  # 表级约束

# 2. 创建表后
ALTER TABLE people
    MODIFY id int PRIMARY KEY;

# 3. 复合主键
CREATE TABLE my_erick_tb
(
    id   INT,
    name varchar(20),
    PRIMARY KEY (id, name)
)

# 4. 删除主键约束
ALTER TABLE my_erick_tb
    DROP PRIMARY KEY;
```

## 4. AUTO_INCREMENT

- 必须作用在唯一或主键上
- 主键声明有自增，添加数据时，不要给主键赋值

```sql
-- 必须作用在唯一或主键上
CREATE TABLE erick
(
    id   INTEGER PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(20)
);

-- 修改
ALTER TABLE erick
    MODIFY id int AUTO_INCREMENT;
    
-- 删除
ALTER TABLE erick
    MODIFY id int;
    
-- 自增变量持久化
删除一条数据后(如id=10)，新增的数据，会沿用之前的值，下一个值就是11，即使重启服务器也是
```

## 5. CHECK

```sql
-- check约束
CREATE table erick_shu
(
    id     int PRIMARY KEY,
    name   varchar(10),
    salary int CHECK ( salary > 200 )
);
```

## 6. DEFAULT

- 建表时，加not null defalut‘’或default 0 ，不想让表中出现null值
- null不好比较，同时效率不高，影响索引效果

```sql
CREATE table erick_shu
(
    id     int PRIMARY KEY,
    name   varchar(10) DEFAULT 'ERICK',
    salary int CHECK ( salary > 200 )
);

-- 修改时添加
ALTER TABLE erick_shu
    MODIFY name VARCHAR(20) DEFAULT 'LUCY';

-- 删除
ALTER TABLE erick_shu
    MODIFY name VARCHAR(20);
```

# 系统变量

- 系统定义的，属于<font color=orange>服务器</font>层面。启动MySQL服务器实例期间，为服务器内存中的系统变量赋值
- 定义了MySQL服务器实例的属性，特征，要么是<font color=orange>编译MySQL时参数的默认值</font>，要么是<font color=orange>配置文件</font>(my.conf)中的参数值
- [官网系统变量](https://dev.mysql.com/doc/refman/8.0/en/server-system-variable-reference.html)
- 如果一个变量，既是全局系统变量，也是会话系统变量，则修改时候一定要声明，到底是改的什么范围

![image-20230807105100648](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230807105100648.png)

## 1. 全局系统变量

- <font color=orange>global</font>：在MySQL初始化启动后，所有的全局系统变量就已经加载好了
- set进行修改，针对所有会话都有效，但是服务器重启后，修改失效
- <font color=orange>静态变量</font>：特殊的全局系统变量，在MySQL服务实例运行期间，它们的值不能使用set动态修改。如innodb_buffer_pool_instances

```sql
# 查看所有全局变量
SHOW GLOBAL VARIABLES;

# 查看满足条件的
SHOW GLOBAL VARIABLES LIKE '%标识%';

# 查看指定系统变量: @@是标记系统变量
SELECT @@GLOBAL.变量名;

# 先查找会话系统变量，如果不存在，则查找全局系统变量
SELECT @@变量名;
```

## 2. 会话系统变量

- <font color=orange>session</font>：会话连接后，会有默认值，也可以去设置这些值
- 如果不写，默认会话级别
- 只在当前会话有效，如果修改了某个会话系统变量的值，不会影响其他会话

```sql
# 查看所有会话变量, SESSION可以省略
SHOW SESSION VARIABLES;
SHOW VARIABLES;

# 查看满足条件的
SHOW SESSION VARIABLES LIKE '%标识%';

# 查看指定系统变量
SELECT @@SESSION.变量名;
```

## 3. 修改系统变量

### 3.1 my.conf文件

- 修改方式是永久的
- 必须重启MySQL服务器才能生效

### 3.2 set修改

```SQL
# 全局系统变量：服务器重启后失效
SET @@GLOBAL.变量名=变量值;
SET GLOBAL 变量名=变量值;

# 会话系统变量：不影响其他会话，会话关闭后失效
SET @@SESSION.变量名=变量值;
SET SESSION 变量名=变量值;
```

### 3.3 推荐方案

- 先设置GLOABL，保证修改时候，不影响MYSQL服务
- 再修改my.conf文件，保证后续重启后，就直接使用配置文件

# 服务架构

## 1. 文件目录

### 配置文件

-  数据库的启动配置文件
-  /etc/my.conf： 修改后，需要重启MySQL才能生效

```bash
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

### 库/表目录

- datadir=/var/lib/mysql，具体保存对应的库信息和表信息

```sql
SHOW VARIABLES LIKE  'datadir';      -- /var/lib/mysql
- /var/lib/mysql/库名/表名           # 数据库某个库

# InnoDB： 表结构和表数据，都放在.idb文件中 
- /var/lib/mysql/库名/people.ibd

# MyISAM
- people.MYD(MYData)：   数据信息文件，存储数据信息
- people.MYI(MYIndex)：  存放索引信息文件
- people.sdi：           描述表结构文件，字段长度等


# 进入到指定数据库下，解析对应表的ibd文件，得到对应的.ibd文件的txt文件
- Oracle提供了一个应用程序 ibd2sdi, MySQL8.0自带的
- ibd2sdi --dump-file=cat.txt cat1.ibd
```

## 2. 用户管理

- root用户拥有最高权限，用root用户登录后，创建其他用户

### User

- 用户的表存放在mysql.user表中，该表的主键是 host+user

```sql
# 1. 查看当前server的用户名和密码
SELECT host, user FROM mysql.user;


# 2. 创建新用户： 默认创建的用户没有任何权限
-- 默认host是 %， 代表任何IP都可以访问
CREATE USER 'erick' IDENTIFIED BY 'abcd1234';
-- 创建时候尽可能限制用户的登录主机，一般是限制成指定IP或者内网IP段
CREATE USER 'lucy'@'%' IDENTIFIED BY 'abcd1234';
-- 只允许localhost来访问
CREATE USER 'lucy'@'localhost' IDENTIF IED BY 'abcd1234';


# 3. 修改用户名
-- 必须拥有DROP USER权限，才可以删除
UPDATE mysql.user SET user='tom' WHERE user='lucy' and host='%';
FLUSH PRIVILEGES;

# 4. 删除用户
-- 不建议使用delete语句来删除对应的mysql.user表，因为其他的一些权限信息等，可能没有删除干净
-- 默认删除 host为%的用户
DROP USER 'lucy';


# 5. 修改用户名密码
# 当前用户修改当前用户
ALTER USER USER() IDENTIFIED BY 'NIKE_#_984';  -- 方式1
SET PASSWORD='NIKE_#_989';                     -- 方式1


# 修改其他用户密码：必须拥有对应的权限
ALTER USER 'tom'@'%' IDENTIFIED BY 'abcd_1234';  -- 方式1
SET PASSWORD FOR 'tom'@'%'='abcd_1235';          -- 方式1
```

## 2. 密码管理

### 2.1 过期策略

#### 手动过期

```sql
# root设置过期
ALTER USER 'tom' PASSWORD EXPIRE ;

# tom用户可以用过期密码登录，但是不能执行查询,必须先修改密码
# ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
```

#### 自动过期-全局

- default_password_lifetime：系统变量建立全局密码过期策略
- 默认是0，表示永不过期
- 取值：正整数N，n天后过期

```sql
# 修改环境变量
SET PERSIST default_password_lifetime=180;
SHOW VARIABLES LIKE 'default_password_lifetime';

# 配置my.conf文件
[mysqld]
default_password_lifetime=180
```

### 2.2 密码安全强度-插件

- MySQL8.0之前，使用validate_password插件检测，验证账户密码强度，保障账户的安全性
- 通过安装/启用插件：会注册到元数据，也就是mysql.plugin表中，mysql重启后插件不会失效

```sql
mysql> INSTALL PLUGIN validate_password SONAME 'validate_password.so';

# 就会报错： ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
ALTER USER 'root'@'%' IDENTIFIED BY 'abc1234';

ALTER USER 'root'@'%' IDENTIFIED BY 'Abc_1234';
```

#### 查看具体要求

```sql
SHOW VARIABLES LIKE 'validate_password%';

Variable_name                         | Value  |
+--------------------------------------+--------+
| validate_password_check_user_name    | ON     |
| validate_password_dictionary_file    |        |
| validate_password_length             | 8      |
| validate_password_mixed_case_count   | 1      |
| validate_password_number_count       | 1      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 1      |
```

| 选项                                 | 默认值 | 描述                                                         |
| ------------------------------------ | ------ | ------------------------------------------------------------ |
| validate_password_check_user_name    | ON     | 设置为on时，表示能将密码设置为当前用户名                     |
| validate_password_dictionary_file    |        | 用于检查密码的字典文件的路径名，默认为空                     |
| validate_password_length             | 8      | 密码的最小长度                                               |
| validate_password_mixed_case_count   | 1      | 密码策略是medium或者strong，要求密码具有的小写和大写的最小数量 |
| validate_password_number_count       | 1      | 密码必须包含的数字个数                                       |
| validate_password_policy             | MEDIUM | 0：low，只检查长度<br>1：medium，检查长度，数字，大小写，特殊字符<br>2：strong，检查长度，数字，大小写，特殊字符，字典文件 |
| validate_password_special_char_count | 1      | 特殊字符的个数                                               |

#### 修改安全策略

```SQL
# 修改安全策略
SET GLOBAL validate_password_policy=LOW/MEDIUM/STRONG;
SET GLOBAL validate_password_policy=0/1/2;

# 修改密码长度
SET GLOBAL validate_password_length=6;
```

#### 删除插件

```
mysql> UNINSTALL PLUGIN validate_password;
```

# 权限管理

```sql
# 查看数据库都有哪些权限
SHOW PRIVILEGES;

# 新用户
- 新创建的用户，一般权力基本没有
```

## 1. 用户权限

### 1.1 权限操作

```sql
# 查看当前用户权限
SHOW GRANTS;
SHOW GRANTS FOR CURRENT_USER;
SHOW GRANTS FOR CURRENT_USER();

# 查看某用户的权限: 查看操作也需要权限
SHOW GRANTS FOR 'tom'@'%';
```

```SQL
# 将某个库下的指定权限赋予给指定用户
GRANT SELECT, UPDATE ON db_test1.* TO 'lucy'@'%';

# 将所有的权限都给指定用户
# 先执行下面的，否则会报错
GRANT SYSTEM_USER ON *.* to 'root'@'%';
FLUSH PRIVILEGES ;

GRANT ALL PRIVILEGES ON *.* TO 'tom'@'%';

# 同一个用户多次给权限，权限叠加而非权限覆盖
```

```sql
# 权限回收
# 用户重新登录后才有效
REVOKE UPDATE ON db_test1.* FROM 'lucy'@'%';

REVOKE ALL PRIVILEGES ON *.* FROM 'tom'@'%';
```

### 1.2 权限表

**mysql.user**

```SQL
# host:               当前允许的ip访问
# user：              用户名
# xxx_priv：          当前用户对所有表的权限是Y还是N
# ssl_xxx：           证书认证相关的
# max_connections：   当前用户每小时的最大连接次数
# max_questions：     当前用户每小时的最大查询次数
# max_updates：       当前用户每小时的最大更新次数
# authentication_string： 加密后的密码
SELECT host, user,  Insert_priv, Update_priv,ssl_type, max_connections, max_questions, max_updates, authentication_string  FROM mysql.user
```

**mysql.db**

- 某个用户对某个表的具体的权限

``` SQL
# host + user + db 作为联合主键
SELECT host , user, db, Select_priv FROM mysql.db;

# 不记录root用户或者拥有所有库权限的用户
```

- 还有其他更细粒度的表

## 2. 角色管理

- 角色是权限的集合，可以为角色添加或移除权限
- 用户可以被赋予角色，同时就会被授予角色所包含的权限
- 方便管理拥有相同权限的用户

```sql
# 创建角色
# host_name不加时，默认是%
CREATE ROLE 'manager'@'%', 'admin'@'%';

# 给角色赋权
# 创建的角色，默认没有任何权限
GRANT SELECT ON db_test1.book1 TO 'manager';
GRANT ALL PRIVILEGES ON *.* TO 'admin';

# 查看某个角色权限 
SHOW GRANTS FOR 'manager'@'%';

# 回收某个角色权限
 REVOKE SELECT ON db_test1.book1 FROM 'manager';
 
 # 删除角色
 DROP ROLE 'manager'@'%';
 
 
 # 2. 将角色给用户
 
 # 将admin这个role给tom这个角色
GRANT 'admin'@'%' TO 'tom'@'%';

# 查看当前user被赋予的role的角色(激活的角色)
SELECT CURRENT_ROLE();

# 默认:一个role给一个角色后，该角色对应的role默认暂不开启，要使用，必须先开启
# 方式一：显示开启
# 将admin的角色，对tom用户激活
# 用户必须重新登录，才能看到赋予的角色
# 如果要将admin这个role给其他用户，也要在对其他用户执行同样操作
SET DEFAULT ROLE 'admin'@'%'  TO 'tom'@'%';

# 方式二：修改参数
# 创建的role只要给了对应的user，对应的user就会直接继承该role的所有权限
SHOW VARIABLES LIKE 'activate_all_roles_on_login';
SET GLOBAL activate_all_roles_on_login=ON;


# 3. 从用户身上撤销角色: 将admin角色从lucy用户撤销
REVOKE 'admin'@'%' FROM 'lucy'@'%';
```

# 逻辑架构

## 1. 逻辑架构

- MySQL是C/S架构，即Client/Server，服务端是mysqld
- 客户端进程向服务器进程发送一个SQL，服务器进程处理后，向客户端进程发送处理结果

![image-20230806145924556](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230806145924556.png)

### 1.1 Connectors

- SDK(Native C, JDBC，PHP)，或者客户端工具(Navicat, DataGrip)等
- 不同语言中与SQL的交互，和MySQL建立<font color=orange>TCP连接</font>，按照其定义好的协议进行交互

### 1.2 连接层

#### 连接

![image-20230806150347795](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230806150347795.png)

- 客户端访问MySQL服务器前，首先需要建立<font color=orange>TCP</font>连接
- 经过三次握手建立连接成功后，对TCP传输过来的账户密码进行身份认证，权限获取

```bash
# 用户密码不对
- 会收到Access denied for user错误，客户端程序结束

# 用户密码正确
- 会从比权限表查出账户拥有的权限和与连接关联，之后的权限逻辑判断，都将依赖与此时读取的权限
```

#### TCP连接池

- 一个系统会和MySQL服务端进行多个连接，多个系统会和MySQL服务端连接
- TCP无限创建和频繁创建和销毁带来的资源耗尽，带来性能下降问题
- MySQL服务器中有专门的<font color=orange>TCP连接池</font>限制连接数，采用<font color=orange>长连接模式复用TCP连接</font>，解决上述问题
- 同时，一个TCP连接，也会被分配一个线程池来处理

### 1.3 服务层

#### SQL Interface

- 接收用户的SQL命令，并且返回用户需要查询的结果。比如SELECT * FROM table_name，就是调用SQL的Interface
- MySQL支持DML，DDL，存储过程，视图，触发器，自定义函数等多种SQL语言接口

#### Parser

- 对SQL语句进行<font color=orange>语法解析</font>， <font color=orange>语义解析</font>。将SQL分解成数据结果，并将这个结果传递到后续步骤
- 创建<font color=orange>语法树</font>，并根据其<font color=orange>验证该客户端是否具有执行该查询的权限</font>

#### Optimizer

- 一条查询可以有很多中执行方式，最后都返回相同的结果，优化器负责找到最好的执行计划
- 查询前，会根据语法树，确定一个<font color=orange>执行计划</font>
- 表明<font color=orange>使用哪些索引</font>，表之间的连接顺序如何。最终按照该步骤得到的执行假话，来查询结果

#### Cache&Buffer

- Query Cache用来缓存一条SELECT语句的执行结果。查询时候，如果命中缓存，则直接从缓存中返回结果
- 该查询缓存：<font color=orange>在不同客户端之间共享</font>
- MySQL5.7.20开始，不推荐使用查询缓存，并在MySQL8.0中删除

```bash
- 之前执行过的语句及结果，会通过k-v方式存储在查询缓存中
- key：该条SQL字符串(比较的是字符串，命中率很低)
- value：SQL的查询结果


# 命中率低：不是同一个SQL的场景
- 空格，注释，大小写不同
- 查询中包含系统函数，如NOW()等
- 缓存失效：缓存会检测涉及到的每个表，如果表结构或表数据发生写操作，就会失效

# 查询缓存弊大于利，缓存命中率低
```

### 1.4 引擎层

- <font color=orange>插件式存储引擎</font>：将查询处理和其他的系统任务以及数据的存储提取分离，支持自定义存储引擎
- 负责<font color=orange>数据的存储和提取，对物理服务器级别维护的底层数据执行操作</font>
- 服务器通过API和存储引擎进行通信，不同的存储引擎的功能不同

### 1.5 存储层

- 所有的数据，如表，索引等都存储在物理磁盘上

## 2. SQL执行流程

- 可以通过SQL提供的服务，查看一个SQL的具体每个步骤的执行
- 通过profiling来查看一个SQL的执行所需要的资源情况

![image-20230806154545674](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230806154545674.png)

```sql
# 1. 查看： 默认情况是关闭的,不记录sql的执行计划
SELECT @@profiling;                  # 0

SHOW VARIABLES LIKE 'profiling';     # off

# 修改为开启
SET profiling=1;


# 2. 查看执行计划
SELECT * FROM book1;

SHOW PROFILES ;                # 查看当前所有的查询操作, 得到queryId
SHOW PROFILE FOR QUERY 1 ;     # 查看当前所有的查询操作
```

```bash
+--------------------------------+----------+
| Status                         | Duration |
+--------------------------------+----------+
| starting                       | 0.000092 |
| Executing hook on transaction  | 0.000009 |
| starting                       | 0.000013 |
| checking permissions           | 0.000009 |
| Opening tables                 | 0.000040 |
| init                           | 0.000007 |
| System lock                    | 0.000011 |
| optimizing                     | 0.000006 |
| statistics                     | 0.000014 |
| preparing                      | 0.000017 |
| executing                      | 0.000054 |
| end                            | 0.000006 |
| query end                      | 0.000005 |
| waiting for handler commit     | 0.000010 |
| closing tables                 | 0.000009 |
| freeing items                  | 0.000021 |
| cleaning up                    | 0.000015 |
+--------------------------------+----------+
```

## 3. Buffer Pool

- InnoDB存储引擎是以页为单位来管理存储空间的，写操作本质上都是在对数据页进行操作
- 磁盘I/O消耗时间比较多，而在内存中进行操作，效率会提高很多
- DBMS申请<font color=orange>占用内存作为数据缓冲池</font>，在真正访问页之前，需要把磁盘上的页缓存到内存中的<font color=orange>Buffer Pool</font>之后才能访问
- 内存中的操作，相比磁盘I/O的性能提示巨大。买手机内存很重要，把磁盘中的数据加载到内存

### 作用

- 数据页存储在物理磁盘上，当需要访问某个页中的数据时，<font color=orange>会把完整的页数据加载到内存中</font>，读写完毕后，不着急把<font color=orange>该页对应的内存释放</font>，而是将其缓存起来，下次如果再请求该页数据，就可以节省磁盘IO开销

### 缓存原则

- <font color=orange>位置 * 频次</font>
- 位置决定效率，提供缓冲池就是为了在内存中可以直接访问数据
- 频次决定优先级。假如磁盘有200g，内存只有16g，缓冲池只占内存中的2g，就无法将所有数据都加载到缓冲池中
- 优先对使用频次高的热数据进行加载

### 缓冲池预读

- 局部性原理
- 假如使用了一些数据，则<font color=orange>大概率还会使用它周围页的数据</font>，采用预读机制提前加载，可以减少未来可能的磁盘IO操作

### 更新机制

- 如果对数据库中的记录进行修改后，首先修改缓冲池中页对应的记录信息，然后数据库会以<font color=orange>一定的频率</font>刷新到磁盘上
- 不是每次发生更新操作，都会立刻进行磁盘回写
- 采用<font color=orange>checkpoint的机制</font>将数据回写到磁盘上，提升数据库的整体性能

```bash
# 脏页-缓冲池中
- 缓冲池中被修改过的页，与磁盘上的数据页不一致

# 缓冲池不够用
- 需要释放掉一些不常用的页，就可以强行采用checkpoint的方式
- 将缓冲池中的脏页进行刷盘到磁盘，并从缓冲池中释放该页
```

### 大小设置

```bash
# InnoDB
# 默认是128MB:  134217728
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';

# 系统变量
SET global innodb_buffer_pool_size =  134217728;
SET global innodb_buffer_pool_size =  268435456;

# 配置文件
[server]
innodb_buffer_pool_size =  268435456
```

### 多Buffer Pool实例

- Buffer Pool本质是向操作系统申请一块<font color=orange>连续的内存空间</font>，多线程环境下，访问Buffer Pool的数据都需要<font color=orange>加锁</font>处理
- Buffer Pool特别大，并且多线程并发访问高时，性能就会下降
- 把Buffer Pool <font color=orange>拆分成若干个小的Buffer Pool</font>，每个都是一个单独的实例。独立的管理各个表，提高并发能力

```sql
# InnoDB: 默认是1
SHOW VARIABLES LIKE 'innodb_buffer_pool_instances';

# 静态变量： 只能在配置文件中修改
[server]
innodb_buffer_pool_instances = 2

# Buffer Pool Instance不是越多越好
- 分别管理Buffer Pool也是需要性能开销的
- 当innodb_buffer_pool_size小于一个1G时候，innodb_buffer_pool_instances设置无效

# 每个BufferPool的实际内存
- innodb_buffer_pool_size/innodb_buffer_pool_instances
```

```bash
[server] 
innodb_buffer_pool_size = 2147483648      # 2G
innodb_buffer_pool_instances = 2          # 2实例 
```

## 4. 存储引擎

- <font color=orange>存储引擎就是表的类型</font>，以前存储引擎叫做<font color=orange>表处理器</font>
- 接收上层传下来的指令，对表中的数据进行提取或写入操作

```sql
# 查看当前版本支持的存储引擎
SHOW ENGINES;

# 修改: 全局系统变量
SET DEFAULT_STORAGE_ENGINE = MyISAM;
# 修改：配置文件
default_storage_engine=MyISAM
```

![image-20230807113933141](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230807113933141.png)

```sql
# 库级别
- 不能指定

# 表级别: 如果不指定，就会使用全局系统变量中的
CREATE TABLE goods
(
    id INT
) ENGINE = MyISAM;

# 查看
SHOW CREATE TABLE goods;

# 修改表的存储引擎
ALTER TABLE goods ENGINE = InnoDB;
```

### InnoDB vs MyISAM

- 除非有特别的原因，需要使用其他的存储引擎，否则应该优先考虑InnoDB存储引擎

|            | InnoDB | MyISAM |
| :--------- | :----- | :----- |
| 版本支持   | 默认   |        |
| 支持事务   | Y      | N      |
| 分布式事务 | Y      | N      |
| 外键       | Y      | N      |

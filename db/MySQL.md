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

# 索引

- 索引是存储引擎快速定位数据的一种数据结构
- 排好序的数据结构以某种方式指向数据，满足特定查找算法，减少磁盘IO，提升查找效率
- 索引在存储引擎中实现：每种存储引擎支持不同索引，不同的最大索引数和最大索引长度

## InnoDB索引

```sql
SELECT * FROM 'xxx' WHERE 列名=xxx;
```

### 无索引

- 数据页：mysql底层存储数据的最小物理单位，默认大小为16K
- 表中记录少，记录存放在一个数据页；表中记录多，则存放在多个页
- 主键：数据存储时，默认是按照主键来进行排序的

```bash
# 单页
   # 主键
   - 在页目录中使用 二分法 快速定位到对应的 槽 ，然后再遍历该槽对应的分组中的记录
   
   # 非主键列
   - 数据页并没有对应非主键列的页目录，只能从最小记录依次遍历单链表中的每条记录
   - 判断每条记录是否满足搜索条件，因为可能多列符合条件
   
 # 多页
   # 主键
   - 首先定位到查找的记录所在的页然后从所在页中，根据单页的方式查找对应的记录
   
  # 非主键列
  - 可能需要遍历所有数据页，比如搜索条件可能对应多个行记录
```

### 索引推演

```sql
# record_type
- 数据记录的类型：  0--普通记录      2--最小记录        3--最大记录
# next_record
- 下一条数据记录的地址相对于本条数据记录的地址偏移量
# 各个列的值
- 当前表中的各个列
# 其他信息
- 除了上述三种信息，其他的隐藏列的值以及记录的额外信息

# 建表
CREATE TABLE girl_shoes
(
    id   int primary key,
    age  int,
    name char(1)
);
```


#### 数据页

```sql
# 单页内
- 单向链表，数据记录按照主键值大小串联
# 页之间
- 双向链表，物理上不连续，逻辑上连续
# 页分裂：
- 一个页存满时，或者插入新数据时候，受索引列排列影响，可能会存在记录移动

# 假设每个数据页最多存放3条记录
# 插入3条数据： 会按照主键大小来进行插入，并重新排序
INSERT INTO girl_shoes
values (1, 14, 'a'),
       (5, 20, 'c'),
       (3, 16, 'b');

# 再插入一条数据，会存在页分裂，伴随着记录移动
INSERT INTO girl_shoes
values (4, 32, 'd')；
```

![image-20230910095630532](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230910095630532.png)

#### 数据页的目录项页

- 在多个页中进行查找时，需要从首页遍历
- 对页再做个目录，能够快速定位到某个页

```bash
# 每个页对应一个目录项，每个目录项包含两个部分
-key：      页的用户记录中最小的主键值
-page_no：  页号

# 主键查找数据过程
- 先根据数据页-目录项页，找到当前查找的主键在具体的哪个数据页
- 然后进入具体的数据页，通过单页的二分法查找到数据
```

![image-20250216151536382](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250216151536382.png)

#### 目录项页的目录项页

- 假如顶层目录项页只有一个，查找一个数据时，只需要三次与磁盘之间的IO

![image-20250216151732158](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250216151732158.png)

### B+ Tree

```bash
# Leaf Node
- 存放用户自定义数据，页之间通过双向链表连接，页内数据通过单向链表连接

# Non-Leaf Node
- 目录项构成的结点，不存放用户自定义数据
```

![image-20250216151113256](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250216151113256.png)

```bash
# B+树通常不会超过4层

      # 原因一：数据体量
        - 假设leaf-node单页可存放100条用户记录，none-leaf-node单页可以存放1000条记录(因为非叶子结点数据项比较少)
        - 1层：   100条
        - 2层：   1000*100条
        - 3层：   1000*1000*100条

       # 原因二：树的层次越低，内存和磁盘的IO次数就越少
       - 数据查找时，每经过一层，就是一次IO，因为加载数据的时候，是把一个完整的页加载到内存中
```

### 物理类型

- 索引按照物理存储实现方式，分为聚簇索引和非聚簇索引

#### Clustered Index

- 局促索引，聚集索引，主键索引
- 一种数据存储方式：所有用户记录存储在叶子结点
- 索引即数据，数据即索引

![image-20250216152136151](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250216152136151.png)

```bash
# 按主键值大小排序
- 页内：记录按照 主键大小顺序 排成一个单向链表
- 多页：多页间根据页中用户记录的主键大小排序称一个双向链表

# 叶子结点存储完整用户记录
- 完整的用户记录，该记录中存储了所有列的值(包括隐藏列)

# 拥有上述两种特点的B+树称为聚簇索引
- 不需要在MySQL中显式的使用INDEX语句
- 存储引擎会 自动 创建聚簇索引，同时也是构建数据
```

```bash
# 优点
- 数据访问更快：         索引和数据保存在同一个B+树中，因此比非聚簇索引更快
- 主键的排序/范围查找：   数据都是紧密相连，数据库不用从多个数据块中提取数据，节省大量IO操作

# 缺点
- 插入速度严重依赖插入顺序：      按照主键的顺序插入是最快的，否则出现页分裂，严重影响性能。一般会定义一个自增的ID列为主键
- 更新主键的代价很高：           会导致更新的行移动以及伴随的页分裂，一般主键为不可更新

# 限制
- InnoDB支持，MyISAM不支持
- 唯一性：  数据物理存储排序方式只能有一种，所以每个MySQL表只能有一个聚簇索引。一般情况就是该表的主键
- 默认值：  InnoDB会根据：主键>非空唯一索引>隐式主键 的顺序，来创建聚簇索引
- 推荐：    InnoDB表的主键列尽可能用有序的顺序ID，而非无序的id，比如UUID，MD5， HASH，字符串(无法保证数据的顺序增长)
```

#### Non-Cluster Index

- 辅助索引，非聚簇索引，二级索引
- 优化使用非主键列作为搜索条件时，就需要非聚簇索引
- 多建几个B+树，不同的B+树采用不同的排列方式

##### 单列索引

- 先遍历搜索列的非聚簇索引构造的B+树，找到对应的匹配记录的主键，然后通过主键结果，回表到主键构成的聚簇索引的B+树
- 一个InnoDB表中，包含一个聚簇索引和若干个非聚簇索引

![image-20250216154511529](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250216154511529.png)

##### 联合索引

- 多个列组成一个索引
- 联合索引c2_c3，先按照c2升序排列，c2相同则再通过c3排列

![image-20230723112133238](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230723112133238.png)

### 索引准则

#### 根页面位置不变

- B+树从上而下构建，根结点固定

```bash
# 构建过程
1. 索引创建后，表中无数据时，B+树索引对应的根结点既没有用户记录，也没有目录项记录
2. 插入用户记录后，用户记录存储到根结点，直到根结点的可用空间用完时
3. 继续插入数据，先将根结点的所有记录复制到一个新页，然后对新页进行页分裂，得到一个新的页来存放新增用户记录
4. 同时根结点升级为存储目录项记录的页

# 优点
- 根结点的页号会被记录到某个地方
- 存储引擎需要用到这个索引时，会从固定的地方取出根结点页号，从而来访问这个索引
```

#### 非叶子节点记录唯一

- 针对非聚簇索引的非叶子结点的key

![image-20250216160659215](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250216160659215.png)

- 解决方案：确保非叶子结点的key是唯一的，搜索列+主键

![image-20250216160945695](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250216160945695.png)

#### 单页面最少2条记录

- 一个数据页最少可以存放两条记录，然后通过目录的多次磁盘IO获取到真实存储的用户记录的数据页

### 优缺点

```bash
# 优点
- 普通索引：提高数据检索的效率，降低数据库的磁盘IO成本
- 唯一索引：保证数据库表中每一行的数据唯一性
- 在使用分组和排序子句进行数据查询时，可以显著减少查询中分组和排序的时间，降低CPU消耗

# 缺点
   # 空间
- 每个索引对应一颗B+树，每颗B+树的每个结点都是一个数据页(16KB)，会占用大量空间
- 如果有大量的索引，索引文件就可能比数据文件更快达到最大文件尺寸
   # 缺点
- 每次写操作，都要修改各个索引的B+树，可能会带来记录移位，页面分裂，页面回收等。并且随着数据量的增加，所耗费的时间也会增加
```

## MyISAM索引

```bash
# boy_shoes.MYD:            用户数据
- 具体的用户数据和对应的地址值
- 数据存放时，按照数据添加顺序进行排列，不会进行再次排序

# boy_shoes.MYI：           索引文件
- 叶子结点的data存放的是数据记录的地址值
- 索引结构依然是B+树， 但是所有的索引都是非聚簇索引，索引数据相分离
- 每次添加一个新的索引，就是新增一个.MYI文件


# boy_shoes_363.sdi：       表定义，表结构的文件
```

![image-20230724112145413](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230724112145413.png)

## 索引使用

### 分类

```bash
# 逻辑功能          物理实现             字段个数
普通                局促                 单列
唯一                非局促                联合
主键

# 普通索引
- 只是为了提高查询效率
- 可以创建在任何数据类型上，其值是否唯一和非空，要由字段本身的完整性约束条件决定

# 唯一索引
- 使用UNIQUE设置字段，就会自动添加唯一索引
- 允许有空值
- 一个表中可以有多个

# 主键索引
- NOT NULL + UNIQUE
- 决定数据的存储方式，一个表中只能包含一个
```

### 创建

#### 创建表时

```sql
- INDEX | KEY </font>： 同义词
- <font color=orange>index_name</font>： 索引名称，可选参数。默认是col_name为索引名
- <font color=orange>col_name</font>：需要创建索引的字段列
- length：可选参数，表示索引的长度，只有字符串类型的字段才能指定索引长度
- ASC | DESC：指定升序或者降序的索引值存储
```

```sql
CREATE TABLE book1
(
    id      INT,
    name    VARCHAR(100),
    comment VARCHAR(100),
    price   INT,
    # 普通索引
    INDEX name_index (name),
    # 唯一索引
    UNIQUE INDEX comment_index (comment),
    # 主键索引：名字就是PRIMARY，自定义名字不会起作用
    PRIMARY KEY (id),
    # 联合索引: 也可以为unique, 最左前缀原则
    INDEX mul_name_comment_price_index (name, comment, price)
);

# 查看索引
SHOW INDEX FROM book1;
```

#### 现有表上

```SQL
# 普通索引
ALTER TABLE book2
    ADD INDEX name_index (name);
# 唯一索引
ALTER TABLE book2
    ADD UNIQUE INDEX comment_index (comment);
# 主键索引
ALTER TABLE book2
    ADD PRIMARY KEY (id);
# 联合索引
ALTER TABLE book2
    ADD INDEX mul_name_comment_price_index (name, comment, price);
```

```SQL
# 普通索引
CREATE INDEX name_index ON book3 (name);
# 唯一索引
CREATE UNIQUE INDEX comment_index ON book3 (comment);
# 联合索引
CREATE INDEX mul_name_comment_price_index ON book3 (name, comment, price);
# 主键索引: 不能通过这种方式
```

### 删除

- 删除索引时，是根据名字来匹配的，所以不需要指定索引类型，除了主键索引

```sql
# 普通索引
ALTER TABLE book2
    DROP INDEX name_index;
# 唯一索引： 不是drop unique index
ALTER TABLE book2
    DROP INDEX comment_index;
# 主键索引
ALTER TABLE book2
    DROP PRIMARY KEY;
# 联合索引
ALTER TABLE book2
    DROP INDEX mul_name_comment_price_index;
```

```sql
# 普通索引
DROP INDEX name_index ON book3;
# 唯一索引
DROP INDEX comment_index ON book3;
# 联合索引
DROP INDEX mul_name_comment_price_index ON book3;
```

```sql
# 删除某个字段时，假如该字段有索引，对应索引也会删除
# 删除某个字段的时，假如该字段具有联合索引，联合索引的该字段也会更新删除
ALTER TABLE book2 DROP COLUMN name;
```

### MySQL8.0特性

#### 降序索引

- MySQL8.0之前，索引都是升序，使用时进行反向扫描，大大降低数据库的效率(单链表逆向遍历)
- 某些操作，需要对多个列进行排序，且顺序要求不一致，使用降序索引就会避免数据库使用额外的文件排序操作，提高性能

```sql
CREATE TABLE book4
(
    id      INT,
    name    VARCHAR(100),
    comment VARCHAR(100),
    price   INT,
    INDEX name_comment_index (name ASC, comment DESC)
);
```

#### 隐藏索引

```bash
# 引入场景：打算删除一个索引，看一下删除后会不会对系统有影响

# 8.0-隐藏索引： 软删除
# 对应的索引的B+ 树还是会存在，只是查询不会使用到该B+树而已
- 将待删除的索引设置为隐藏索引，查询优化器就不再使用这个索引(即使使用force index,优化器也不会使用该索引)
- 确认变为隐藏索引后，系统不受任何影响，就可以彻底删除该索引
```

```sql
# 创建表时
CREATE TABLE book5
(
    id      INT,
    name    VARCHAR(100),
    comment VARCHAR(100),
    price   INT,
    INDEX name_index (name) INVISIBLE
);

# 创建表后
ALTER TABLE book5
    ADD index comment_index (comment) INVISIBLE;

CREATE INDEX price_index ON book5 (price) INVISIBLE;

# 修改索引可见性
ALTER TABLE book5
    ALTER INDEX comment_index INVISIBLE;
```

## 适用场景

### 4.1 字段数值有唯一性

- <font color=orange>唯一索引，或者主键索引</font>
- 业务上具有唯一特性的字段：<font color=orange>即使是组合字段，也必须建成唯一索引</font>(阿里规范)
- 唯一索引影响了insert速度，但损耗可以忽略，明显提高查找速度(查找B+树，找到一条记录后，就不再查找其余的页了)

### 4.2 WHERE查询的字段

- 普通索引即可大幅提升数据查询的效率(可能搜索多个数据页，但不用扫全部页)

### 4.3 UPDATE/DELETE的WHERE条件列

- 数据按照某个条件进行查询后再进行UPDATE/DELETE， 对WHERE字段建立索引，就能大幅度提升效率
- 如果更新的字段是非索引字段，提升效率更加明显，因为不需要对非索引字段进行索引维护

### 4.4 GROUP BY/ORDER BY的字段

- 索引就是让数据按照某种顺序进行存储和检索
- 创建单列索引或者联合索引

### 4.5 DISTINCT字段

- 不加索引：全表扫描，将该字段在内存中去重
- 加索引：<font color=orange>B+树相邻去重更快</font>，同时查出的数据是<font color=orange>递增排序</font>

### 4.6 使用列的类型小的创建索引

- 类型大小：指该列的该类型表示的数据范围的大小
- 以整数为例，TINYINT, MEDIUMINT, INT, BIGINT, 占用的存储空间依次增加

```bash
# 数据类型越小
- 查询时的比较操作越快
- 索引占用的存储空间就越少，一个数据页能存放更多的数据记录，从而减少磁盘IO带来的性能损耗

# 对主键尤其重要
- 聚簇索引中会存储主键值，其他所有的二极索引的每个节点都会存储一份记录的主键值
- 主键使用更小的数据类型，就会节省更多的存储空间和更加高效的I/O
```

### 4.7 前缀索引

```bash
# 长字符串作为二级索引的缺点
- B+树的叶子结点需要存储该列的完整记录，占用较大的空间，查找索引数据页的IO会增大
- 长字符串的比较，也会耗费性能
```

```sql
# 前缀索引
- 截取字段前面的一部分内容建立索引
- 查找时虽不能精确定位到记录的位置，但是能定位到相应前缀所在的位置，然后再回表找到对应的主键，然后再来判断

# 截取多少
SELECT COUNT(DISTINCT(address)) / COUNT(*) from book;     # 查看选择度
SELECT COUNT(DISTINCT(LEFT(address，索引长度)) / COUNT(*) from book;     # 查看选择度
             
 # 阿里手册
 - 在varchar上建立索引时，必须指定索引长度，没有必要对全部字段建立索引，根据实际区分度来决定索引长度
 - 一般区分度达到90%即可
```

### 4.8 区分度高的列

- 列的基数：某一列中不重复数据的个数
- 区分度：列基数/记录行数，区分度越接近1，越适合做索引
- <font color=orange>联合索引把区分度高的列放在前面</font>

```sql
# 一般超过33%就算是比较高效的索引
SELECT COUNT(DISTINCt a)/COUNT(*) FROM t1;
```

### 4.9 使用最频繁的列，放在联合索引的左侧

- 可以较少的建立一些索引，同时，最左前缀原则，可以增加联合索引的使用率

### 4.10 多字段都要索引时，联合索引优于单值索引

### 4.11 多表Join操作注意

- 连接的表的数量尽量不要超过3张，每增加一张表就增加了一次嵌套的循环，数量级增长比较快，严重影响查询效率
- 对WHERE条件列创建索引
- 对于连接的字段创建索引，并且该字段在多张表中的类型必须一致

# 查询优化

- 物理优化：通过<font color=orange>索引</font>和<font color=orange>表连接方式</font>来进行优化
- 逻辑优化：通过<font color=orange>sql等价变换</font>提升查询效率

## 1. 索引实践

- 是否使用索引，是由优化器来进行选择
- 不是基于规则，而是基于cost开销，基于语义。
- sql语句是否使用索引，和数据库版本，数据量，数据选择度都相关

### 1.1 全值匹配最佳

- 当where后面有多个查询条件时，对多个字段建立联合索引，效率最高

```sql
EXPLAIN SELECT * FROM student WHRER age=10 AND address='xian' AND class_id='abcd';

# 创建的索引
CREATE INDEX idx_age ON student(age);
CREATE INDEX idx_age_address ON student(age,address);
CREATE INDEX idx_age_address_class_id ON student(age,address,class_id);        # 效率最高，最终会选择这个
```

### 1.2 最左前缀原则

- 联合索引中，最左优先，检索数据时，从联合索引的最左边开始匹配
- 如果左边的值不确定，那么无法使用此索引

```sql
# 联合索引
CREATE INDEX idx_age_address_class_id ON student(age,address,class_id); 

# 会使用到全部索引
EXPLAIN SELECT * FROM student WHRER age=10 AND address='xian' AND class_id='abcd';

# 不会使用到索引： 没有age字段
EXPLAIN SELECT * FROM student WHRER address='xian' AND class_id='abcd';

# 会使用索引一部分： 
EXPLAIN SELECT * FROM student WHRER age=10 AND class_id='abcd';
```

### 1.3 主键插入顺序

- 如果主键是顺序插入，但是相邻两个数据之间有空隙，则后续插入的数据可能要插入之前数据之间，就可能造成数据页的分裂，影响性能
- 插入的记录的主键值依次递增，让主键具有AUTO_INCREMENT，让存储引擎自己为表生成主键，而不是手动插入

## 2. 索引失效

### 2.1 计算，函数，类型转换

- 对where条件后的字段，如果进行计算，函数，类型转换(手动或自动)会导致索引失效

```sql
# 函数
EXPLAIN SELECT * FROM student WHRER name LIKE 'abc%';       # 正常使用索引
EXPLAIN SELECT * FROM student WHRER LEFT(name,3) = 'abc';   # 索引失效，全表遍历(mysql不知道这个函数是做什么) 

# 计算
EXPLAIN SELECT * FROM student WHRER student_no = 99;       # 正常使用索引
EXPLAIN SELECT * FROM student WHRER student_no+1 = 100;    # 索引失效，全表遍历(mysql不知道这个运算是做什么) 

# 类型转换: varchar类型字段
EXPLAIN SELECT * FROM student WHRER name='12345';       # 正常使用索引
EXPLAIN SELECT * FROM student WHRER name=12345;         # 先把name做一个隐式的类型函数
```

### 2.2 范围条件右边的列

- 根据搜索条件，先把等值匹配的列放在联合索引的前面，范围匹配的列放在联合索引的后面

```sql
CREATE INDEX idx_age_address_class_id ON student(address,age,class_id); 

# class_id对应的索引就会用不上
EXPLAIN SELECT * FROM student WHRER  address='xian' AND age>10 AND class_id='abcd';
```

### 2.3 不等于

```sql
CREATE INDEX idx_name ON student(name); 

EXPLAIN SELECT * FROM student WHRER name='erick';      # 正常使用索引
EXPLAIN SELECT * FROM student WHRER name!='erick';      # 正常使用索引    !=    <>
```

### 2.4 IS NULL

- is null可以使用索引， is not null会无法使用索引
- 最好在涉及数据表的时候，将字段设置为NOT NULL + DEFAULT约束

### 2.5 LIKE通配符以%开头

- like的通配符，如果第一个是%，索引就会失效
- 数据页搜索严禁左模糊或全模糊，如果需要就用搜索引擎如es来解决

### 2.6 OR前后存在非索引的列，索引失效

```sql
CREATE INDEX idx_name ON student(name); 

# 计划一：name=erick 可以使用索引， age=12只能全表扫，然后把两个结果union起来
# 计划二：还不如一次性扫全表
EXPLAIN SELECT * FROM student WHRER name='erick' OR age=12;
```

### 2.7 数据库和表的字符集统一使用utf8mb4

- 统一使用字符集，不同的字符集相互比较时，会先进行类型转换，从而导致索引失效

## 3. 关联查询

### 3.1 外连接

- 连接条件，一旦数据类型不同，就存在隐式转换，就可能存在索引失效

```sql
# 左外连接和右外连接可以互相转换，因此以左外连接为例
# 假设student中包含20条数据，book中包含100条数据
EXPLAIN SELECT SQL_NO_CACHE * FROM student LEFT JOIN book ON student.id=book.id;


# 1. 不加索引，
-- 从student中取一个数据，book中把100条数据全拿出来，然后一一比较；
-- 继续student的下一个数据，一共遍历20次


# 2. 为被驱动表book的id加索引
-- 从驱动表student中取一个数据，被驱动表利用索引快速找到id匹配的
--  继续student的下一个数据，一共遍历20次
```

### 3.2 内连接

- 如果表的连接条件中只能有一个字段有索引，则有索引的字段所在的表会被优化器作为被驱动表
- 如果两个表的连接条件都有索引，则小表驱动大表

```sql
# 由查询优化器来决定谁作为驱动表，谁作为被驱动表
# 驱动表一般是全表扫描，被驱动表一般是索引扫描
EXPLAIN SELECT SQL_NO_CACHE * FROM student JOIN book ON student.id=book.id;
```

### 3.3 JOIN原理

#### 驱动表/被驱动表



#### 简单嵌套循环连接



#### 索引嵌套循环连接



#### 块嵌套循环连接



#### HASH JOIN



## 4. 子查询优化

- 子查询可以一次性完成很多逻辑上需要多个步骤才能完成的SQL操作

### 4.1 效率低

- 执行子查询时候，mysql需要为内层查询的结果<font color=orange>建立一个临时表</font>，然后外层查询语句从临时表中查询记录。查询完毕后，再<font color=orange>撤销这些临时表</font>。消耗过多的CPU和IO资源，产生大量的慢查询
- 子查询的结果存储的临时表，不管是内存临时表还是磁盘临时表，都<font color=orange>不会存在索引</font>，查询性能低
- 返回结果集越大的子查询，其对查询性能的影响就越大

### 4.2 替代方案

- 用关联查询代替子查询。关联查询不需要建立临时表，速度比子查询要快

## 5. 排序优化

- MYSQL中支持两种排序方式：FileSort和INDEX排序
- Index排序：索引可以保证数据的有序性，不需要再进行排序，效率更高
- FileSor排序：一般在内存中进行排序，占用CPU较多。如果待排结果较大，会产生临时文件I/O到磁盘进行排序的情况，效率较低

```bash
# 优化建议
- order by字句中使用索引，避免全表扫描，从而带来的FileSort排序
- 无法使用INDEX时，需要对FileSort方式进行调优
```



## 6. 分组优化



## 7. 分页优化





# 

## 

# InnoDB数据

## 1. 页-磁盘与内存交互基本单位

- 数据存储划分为若干页个，页默认大小是<font color=orange>16KB</font>
- 页是磁盘和内存交互的<font color=orange>基本单位</font>，一次IO最少从磁盘中读取16KB内容到内存，或最少把16KB内容刷新到磁盘
- <font color=orange>不论读一行或多行，都是将这些行所在的页进行加载，数据库I/O的操作最小单位是页</font>
- 记录是按照行来存储的，但是读取并不是以行为单位，否则一次读取(一次I/O)只能处理一行数据，效率会非常低
- 多个页<font color=orange>可以不在物理结构上连续</font>，只要通过<font color=orange>双向链表</font>相关联即可
- 单页中的用户记录，按照主键值从小到大顺序组成一个<font color=orange>单向链表</font>

```bash
# 查询页大小： 16384
SHOW VARIABLES LIKE '%innodb_page_size%';

# 页目录
- 单页中，存储的所有记录会生成一个当前页的 页目录
- 页目录中会包含多个 slot槽

# 单页查找
- 通过页目录中使用 二分法 快速定位到对应的 槽
- 再遍历该槽对应分组中的记录即可快速找到指定记录
```

![image-20230724142441676](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230724142441676.png)

## 2. 页结构

| 名称               | 大小   | 描述                             |
| ------------------ | ------ | -------------------------------- |
| File Header        | 38字节 | 文件头，描述页的信息             |
| Page Header        | 56字节 | 页头，页的状态信息               |
| Infimum + Supremum | 26字节 | 最大和最小记录，虚拟的行记录     |
| User Records       | 不确定 | 用户记录，存储行记录内容         |
| Free Space         | 不确定 | 空闲记录，页中还没有被使用的空间 |
| Page Directory     | 不确定 | 页目录，存储用户记录的相对位置   |
| File Trailer       | 8字节  | 文件尾，校验页是否完整           |

### 2.1 文件头/文件尾

- 页文件的通用部分

| <font color=orange>文件头</font> | 空间(字节) | 描述                                                         |
| -------------------------------- | ---------- | ------------------------------------------------------------ |
| FIL_PAGE_OFFSET                  | 4          | 页在系统中唯一的编号，方便InnoDB<font color=orange>唯一定位</font>一个页(页号) |
| FIL_PAGE_TYPE                    | 2          | 当前页的类型                                                 |
| FIL_PAGE_PREV                    | 4          | 当前页的上一页(双向链表)                                     |
| FILE_PAGE_NEXT                   | 4          | 当前页的下一页(双向链表)                                     |
| FILE_PAGE_SPACE_OR_CHECKSUM      | 4          | 校验和                                                       |
| FILE_PAGE_LSN                    | 8          | 页面被最后修改时对应的日志序列位置，也是来校验文件完整性的   |

| <font color=orange>文件尾</font> | 空间(字节) | 描述                                                       |
| -------------------------------- | ---------- | ---------------------------------------------------------- |
| FILE_PAGE_SPACE_OR_CHECKSUM      | 4          | 校验和                                                     |
| FILE_PAGE_LSN                    | 4          | 页面被最后修改时对应的日志序列位置，也是来校验文件完整性的 |

#### FIL_PAGE_TYPE

- 表示当前页是什么类型的数据页

| 类型名称                                    | 十六进制 | 描述               |
| ------------------------------------------- | -------- | ------------------ |
| FIL_PAGE_TYPE_ALLOCATED                     | 0x0000   | 最新分配，还没使用 |
| <font color=orange>FIL_PAGE_UNDO_LOG</font> | 0x0002   | Undo日志页         |
| FIL_PAGE_INODE                              | 0x0003   | 段信息结点         |
| <font color=orange>FIL_PAGE_TYPE_SYS</font> | 0x0006   | 系统页             |
| <font color=orange>FIL_PAGE_INDEX</font>    | 0x45BF   | 索引页，数据页     |
| 其他类型                                    |          |                    |

#### FILE_PAGE_SPACE_OR_CHECKSUM

- FILE_PAGE_SPACE_OR_CHECKSUM和FILE_PAGE_LSN都是用来进行文件的完整性校验的
- 文件头和文件尾的来进行比较

```bash
# 校验和
# 1. 概念
- 对于一个很长的字节串来说，通过某种算法得到一个比较短的值来代表，该较短的值就是校验和
- 比较两个很长的字节串时，先比较其检验和。如果校验和都不一样，则两个长的字节串肯定不一样，节省不等于时候的时间损耗

# 文件头文件尾都有属性：FILE_PAGE_SPACE_OR_CHECKSUM

# 2. 作用
- InnoDB将数据以页为单位，将其加载到内存中并对数据进行处理，处理完成后要再重新刷盘到磁盘中
- 刷盘时，假如同步了当前页的一半数据，数据库挂了，内存数据丢失。当前页刷盘就是不完整的
- 为检查一个页是否完整，文件头和文件尾都有一个校验值，通过比较两个值
- 如果相同，则刷盘完整。如果值不同，则刷盘不完整，需要重新进行刷盘

# 3. 具体刷盘操作
- 开始刷盘，先修改File Header中的检验和。完成刷盘后，再修改File Trailer中的校验和
- 同步成功：刷盘完成，首尾检验和相同
- 同步失败：刷盘一半时，MySQL挂了。重启后加载某个页时，首尾校验和不同，需要通过redo日志重新刷盘
- 检验方式： Hash算法
```

### 2.2 空闲空间

- 表中存储的用户记录会按照指定的<font color=orange>行格式</font>存储到User Records中
- 插入一个记录，会从Free Space中，申请一个行记录大小的空间给User Records，直到Free Space耗尽，则<font color=orange>申请新页</font>

### 2.3 用户记录

- 按照指定的<font color=orange>行格式</font>，一条条摆在User Records中，相互之间形成单链表

### 2.4 最小/大记录

- 该页中对应的索引列的最大记录和最小记录

### 2.5 页目录

- Page Directory
- 将一个页中所有的记录，<font color=orange>分成几个组</font>，包含最小记录和最大记录，但不包含已删除的的记录
- 可以在一个页中，进行二分法快速找到对应的数据

```bash
# 页内数据分组
- 第一组：当前页中最小记录所在的组，只包含一个
- 最后一组：最大记录所在的组，会有1-8条记录
- 其余组：数量在4-8之间

- 会在n_owned中保存：当前组中最后一条记录的头信息中会存储该组中一共有多少个记录
- 槽中的数据指向当前组的最大值
```

### 2.6 页面头部

## 3. 行格式

- 页中对应的数据记录(用户记录，空闲空间，最大指针和最小指针)中的具体的格式
- Mysql8.0的默认格式是COMPACT

### 3.1 COMPACT

```sql
# 指定行格式
CREATE TABLE erick
(
    c1 INT,
    c2 INT,
    c3 VARCHAR(10000),
    PRIMARY KEY (c1)
) CHARSET = ascii
  ROW_FORMAT = COMPACT;
  
# 查看
SHOW TABLE STATUS like '%table_name%'
```

#### 变长字段长度列表

- Varchar(m)， TEXT类型的字段，是变长字段类型，存储数据时需要记录这些数据占用的字节数
- 把所有变长字段的真实数据占用的字节长度存储在行记录的开头部分
- 字段声明和记录是逆序的，c1(06), c2(08), c3(10)， 存储记录 100806

#### null值列表

- 把为null的列统一管理起来，如果表中没有允许存储null的列，则该数据为空

#### 记录头(5字节)

![image-20230724180908337](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230724180908337.png)

```bash
# delete_mask
- 标记当前记录是否被删除，1个二进制位               # 0～未删，1～已删
- 删除记录不立即从磁盘上移除                       # 移除该记录后，如果物理删除，其他记录需要在磁盘上重新排列，导致性能消耗
- 删除标记后，所有被删除的记录会组成 <垃圾链表>      # 垃圾链表
- 垃圾链表中的记录占用的空间，是可重用空间，后续新记录插入时，进行覆盖写操作

# min_rec_mask
- B+树的每层非叶子结点中的最小记录都会添加该标记，值为1
- 叶子结点中，值是0，表示其不是非叶子结点的最小记录

# record_type
- 0： 用户记录
- 1： B+数非叶子结点记录
- 2： 最小记录
- 3： 最大记录

# heap_no
- 当前记录在本页中的位置            # 用户记录一般是2，3，4，5.。。。。。
                                # 0-最小记录；1-最大记录
- MySQL会自动为每个页中先添加两个记录，伪记录，虚拟记录，位置最靠前

# n_owned
- 当前页中，当前组中最后一条记录的头信息中会存储该组中一共有多少个记录
- 一个页中的用户记录，会被分成若干个组

# next_record
- 当前记录的真实数据到下一条数记录的真实数据的   <地址偏移量>
- 一条记录的next_record是32，从该条记录的真实数据的地址处向后找32个字节就是下一条数据
```

#### 真实数据

- 用户的真实数据

#### 隐藏数据

```bash
# ROW_ID
- 一个表没有手动定义主键，则会选取一个UNIQUE键作为主键来构建B+数
- 如果表中没有UNIQUE键，则会使用row_id作为主键构建数据

# TRANSACTION_ID
- 事务ID
```

#### 图示

![image-20230910164428204](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230910164428204.png)

### 3.2 DYNAMIC/COMPRESSED

#### 行溢出

- 一条记录中，假如一个字段为VARCHAR(65535)，已经超过了一个页的最大值(16kb)，就会产生行溢出的问题
- 在Compact和Redutant行格式中，对于占用内存较大的列，只会存储该列的一部分数据，剩余的数据分散在其他列
- 相当于保存了部分数据和其余数据的指针
- 不同的行溢出处理方式

## 4. 页上层

![image-20230724142950974](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230724142950974.png)

### 4.1 区

- 假如以页为分配单位，那么多个页之间可能是物理不连续的
- 如果进行范围查找，加载多个页的操作属于<font color=orange>随机I/O</font>，加载比较慢
- 内存以区进行划分，该区内的所有页都是连续的，就属于<font color=orange>顺序I/O</font>，比较快

```bash
# 区(Extent)
- 一个区会分配  64个连续的页， 在文件系统中是一个连续分配的空间
- 一个区的大小是16KB*64=1MB
```

### 4.2 段

- 叶子结点和非叶子结点，保存在不同的段
- 不希望同一个段中，既有叶子结点，又有非叶子结点

```bash
# 一个索引
- 一个叶子结点段， 一个非叶子结点段

# 其他段
- 数据段，索引段，回滚段

# 段
- 一个段中包含若干个区，区与区之间物理不连续，逻辑连续
```

### 4.3 表空间(Tablespace)

- 是一个逻辑容器，表空间和段的关系是一对多
- 数据库由多个表空间构建：系统表空间，用户表空间，撤销表空间，临时表空间

# 性能分析

## 1. 查看系统性能参数

```sql
SHOW GLOBAL|SESSION STATUS LIKE 'connections';
```

| 参数                 | 描述                           |
| -------------------- | ------------------------------ |
| connections          | 连接MySQL服务器的次数          |
| uptime               | MySQL服务器的上线时间          |
| slow_queries         | 慢查询次数                     |
| Innodb_rows_read     | select查询返回的行数           |
| Innodb_rows_inserted | insert操作插入的行数           |
| Innodb_rows_updated  | update操作更新的行数           |
| Innodb_rows_deleted  | delete操作删除的行数           |
| com_select           | 查询操作的次数                 |
| com_insert           | 插入操作的次数，批量插入算一次 |
| com_update           | 更新操作的次数                 |
| com_delete           | 删除操作的次数                 |

## 2. 慢查询日志

- 记录MySQL中<font color=orange>相应时间超过阈值</font>的SQL，具体值运行时间超过<font color=orange>long_query_time</font>值的SQL，则会被记录到慢查询日志中
- 记录执行时间长的<font color=orange>SQL查询</font>，并针对性的优化，提高系统整体效率。当数据服务器发生阻塞，运行变慢，通过检查慢SQL日志，定位到慢查询来解决
- 默认<font color=orange>不开启慢查询日志</font>，需要手动开启。<font color=orange>除非调优需要，一般不建议启动该参数</font>，因为开启慢查询日志会对性能带来一定影响

### 2.1 开启

```SQL
# 默认是OFF
SHOW VARIABLES LIKE '%slow_query_log%';

# 打开慢查询: 同时会有慢查询日志的文件
SET GLOBAL slow_query_log=ON;

slow_query_log,ON
slow_query_log_file,/var/lib/mysql/8bc168b1a471-slow.log

# 默认阈值： 10s
SHOW VARIABLES LIKE '%long_query_time%';

# 修改阈值: 1s    默认是10s
SET long_query_time = 1;

# 建议这些配置在mysql服务器中添加
```

### 2.2 分析

- mysqldumpslow -help ;
- 能够得到具体的执行过，对应慢查询的sql，后续再用EXPLAIN来进一步分析该sql

# Explain

- 定位到慢查询的SQL后，使用EXPLIAN或者DESCRIBE工具做针对性的分析查询语句
- EXPLAIN可以查看mysql对一条sql进行优化后的具体的执行计划是什么
- Explain并没有真正执行后面的SQL语句，因此可以安全的查看执行计划

```sql
EXPLAIN / DESCRIBE sql;
```

## 1. 数据准备

```sql
# 加入十万条记录

# 表1
CREATE TABLE student
(
    id          INT AUTO_INCREMENT,
    name        VARCHAR(100),
    id_no       INT,
    address     VARCHAR(100),
    first_info  VARCHAR(100),
    second_info VARCHAR(100),
    third_info  VARCHAR(100),
    other       VARCHAR(100),
    PRIMARY KEY (id),
    INDEX name_index (name),
    UNIQUE INDEX id_no_index (id_no),
    INDEX address_index (address),
    INDEX multi_info_index (first_info, second_info, third_info)
);

# 表2
CREATE TABLE girl
(
    id          INT AUTO_INCREMENT,
    name        VARCHAR(100),
    id_no       INT,
    address     VARCHAR(100),
    first_info  VARCHAR(100),
    second_info VARCHAR(100),
    third_info  VARCHAR(100),
    other       VARCHAR(100),
    PRIMARY KEY (id),
    INDEX name_index (name),
    UNIQUE INDEX id_no_index (id_no),
    INDEX address_index (address),
    INDEX multi_info_index (first_info, second_info, third_info)
);
```

## 2. 属性值

### 2.1 table

- 具体查询的哪张表
- 一个sql中，对几个表进行查询，就会出现几个表

![image-20230911152806371](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911152806371.png)

### 2.2 id

- 在一个大的查询语句中，每个SELECT关键字都会对应一个唯一的ID
- id如果相同，可以认为是一组，从上往下执行
- 所有组中，id值越大，优先级越高，越先执行
- id每个独立的号码(不同)，表示一趟独立的查询，一个sql的查询趟数越少越好

#### 子查询优化

- 子查询涉及了另外一个表，但是唯一id只有一个，是因为mysql对子查询进行了优化

![image-20230911153409946](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911153409946.png)

```sql
# 子查询
EXPLAIN SELECT * FROM student WHERE id IN (SELECT id FROM girl);

# 子查询效率低，优先会使用多表连接的方式
EXPLAIN SELECT * FROM student WHERE id IN (SELECT id FROM girl);
```

#### UNION

- 会有三个id，因为涉及到了union去重，会创建一个临时表

![image-20230911154026034](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911154026034.png)

#### UNION ALL

- 不需要去重，所以只有两个id

![image-20230911154121333](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911154121333.png)

### 2.3 select_type

- 查询属性，确定一个小查询在大查询中扮演的角色

#### simple

- 查询语句中，不包含union或者子查询的都是 simple类型

![image-20230911154929718](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911154929718.png)

#### primary

- 包含了union，union all,  子查询的大查询，最左边的大的查询就是PRIMARY

#### union

- union的第二个查询

#### union result

- union all的结果集合

![image-20230911154952064](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911154952064.png)

![image-20230911155013425](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911155013425.png)

### 2.4 type

- mysql对某个表的执行查询时候的访问方法，比较重要

```bash
# 从上到下，性能依次降低
system --> const -- >eq_ref --> ref --> fulltext --> ref_or_null --> index_merge
unique_subquery --> index_subquery --> range --> index --> all

# sql优化，至少要达到range级别，要求是ref，最好是const
```

#### system

- 表中只有一条记录，并且该表使用的存储引擎的统计数据是精确的
- MyISAM会有一个记录，专门记录一个表中的行数

![image-20230911160053316](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911160053316.png)

- InnoDB则不会记录, 是会去表中进行全表扫描

![image-20230911160222354](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911160222354.png)

#### const

- 根据主键或者唯一二级索引列，与常数进行等值匹配时，对单表的访问方式
- 因为数据的唯一性，找到一个数据后就不会再往后面找了

![image-20230911160435784](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911160435784.png)

![image-20230911160830806](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911160830806.png)

#### eq_ref

- 连接查询时，如果被驱动表是通过主键或者唯一二级索引列等值匹配的方式进行并访问，则被驱动表的访问方式就是eq_ref

![image-20230911161406008](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911161406008.png)

#### ref

- 通过普通二级索引列等值匹配的方式进行并访问，访问方式就是ref

![image-20230911164316794](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911164316794.png)

#### ref_or_null

- 普通二级索引等值匹配时，该索引列可能为null

![image-20230911164438758](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911164438758.png)

#### index_merge

- 多个查询列，都有索引，并且是or的关系

![image-20230911164647140](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911164647140.png)

#### range

- 根据某个索引，获取区间范围的值

![image-20230911164841083](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911164841083.png)

#### all

- 全表扫描

![image-20230911165048466](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230911165048466.png)

### 2.5 possible_key

- 可能用到的索引

### 2.6 key

- 实际使用到的索引
- 查询优化器，会在possible_key中选取一个，进行查找

### 2.7 key_len

- 实际使用到的索引长度(字节数)
- 可以判断是否充分的利用了索引，数值越大越好
- 主要针对联合索引

## 3 rows

- 预估的查询出来的结果的行数

## 4. filtered

- 从对应的行数中，会有多少百分比的数据满足记录
- 数值越高越好

## 5. Extral



# 事务

- InnoDB支持事务和分布式事务，MyISAM不支持事务
- 一组逻辑操作单元，使数据从一种状态变换到另一种状态
- 处理原则：保证所有事务都作为<font color=orange>一个工作单元</font>来执行，即使出现故障，都不能改变这种执行方式

## ACID特性

### 1. Atomicity

- 事务是一个不可分割的工作单位，要么全部提交，要么全部失败会滚，没有中间状态

### 2. Consistency

- 数据从一个<font color=orange>合法状态</font>变换到另一个<font color=orange>合法状态</font>，和具体的业务相关
- <font color=orange>合法状态</font>：是和具体的业务相关的，满足<font color=orange>预定的约束</font>。满足这个状态，数据就是一致的，不满足就不是

```bash
# 例子一：不是一致的，因为余额不能小于0
- A账户有200，B账户有0
- A账户转给B账户300
- 则A剩余-100， B账户有300

# 例子二：不是一致的，因为默认A+B=200
- A账户有200，B账户有0
- A账户转给B账户50， 但B没收到
- 则A剩余150， B账户有0

# 例子三：不是一致的，因为姓名唯一
- 将表中 姓名 字段设置为 唯一约束
- 如果修改表中数据，如果姓名不唯一，则破坏了事务的一致性
```

### 3. Isolation

- 一个事务的执行，不能被其他事务干扰
- 一个事务的内部操作时，对应的数据，对<font color=orange>并发</font>的其他事务是隔离的，并发执行的各个事务之间，不能互相干扰

```bash
# A账户有200，B账户有0。 A往B转两次，每次转50。两次操作在两个事务中执行
# 如果无法保证隔离性，则会出现下面情况
```

![image-20230807142654691](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230807142654691.png)

### 4. Duration

- 一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来的其他操作和数据库故障，不应该对其有影响
- 持久性是通过<font color=orange>事务日志</font>来保证的。包含<font color=orange>Redo日志</font>和<font color=orange>Undo日志</font>

## 使用事务

### 1. 隐式事务

- AUTOCOMMIT

```sql
# 默认ON
# 每个DML都是一个独立的事务，都是自动提交
SHOW VARIABLES LIKE 'autocommit';

INSERT INTO book1 (name) VALUE ('CCCC');
```

```sql
# 关闭后，多个DML就都会被归结到一个事务中，需要手动提交
SET AUTOCOMMIT=FALSE;

# 不加COMMIT，则无法生效
INSERT INTO book1 (name) VALUE ('AAAAA');

INSERT INTO book1 (name) VALUE ('BBBBB');

COMMIT;
```

### 2. 显式使用

- 和AUTOCOMMIT无关

```sql
# 1. 开启事务
START TRANSACTION READ ONLY;       # 开启一个只读的事务，不能进行写
START TRANSACTION READ WRITE;      # 开启一个可读可写的事务
START TRANSACTION;                 # 默认可读可写     
BEGIN;                             # 开启可读可写

# 2. 一系列DML操作: 也可以是一条语句

# 3. 结束状态
# 提交
COMMIT;

# 回滚到上一次事务开始点
ROLLBACK;
```

```sql
# 带SAVEPOINT: 可以会滚到某个状态
START TRANSACTION;

INSERT INTO book1 (name) VALUE ('ERICK');

INSERT INTO book1 (name) VALUE ('LUCY');

SAVEPOINT first;

INSERT INTO book1 (name) VALUE ('Nancy');

ROLLBACK TO SAVEPOINT first; # 则'Nancy'不会被添加

COMMIT;
```

```sql
START TRANSACTION;

INSERT INTO book1 (name) VALUE ('apple');

INSERT INTO book1 (name) VALUE ('peach');

SAVEPOINT first;

INSERT INTO book1 (name) VALUE ('melon');

RELEASE SAVEPOINT first; #删除某个SAVEPOINT

COMMIT;
```

### 3. 不受AUTOCOMMIT影响

```SQL
# 数据定义语言(Data Definition Language) DDL
- 数据库，表，视图，存储过程等
- 当使用CREATE, ALTER, DROP 等去修改数据库对象时，就会自动提交当前事务

# 修改mysql库中的信息， 如mysql.user表
- ALTER USER, CREATE USER, DROP USER, GRANT, RENAME USER, REVOKE, SET PASSWORD等操作

# 事务控制
BEGIN;
....

BEGIN; # 又开启了一个事务，则会自动提交上一个事务

# 关于锁定的语句: 自动提交前面语句的事务
LOCK TABLES, UNLOCK TABLES 
```

## 隔离级别

```sql
CREATE TABLE student
(
    studentno    INT,
    name  VARCHAR(20),
    class VARCHAR(20),
    PRIMARY KEY (studentno)
);

INSERT INTO student (studentno, name, class) VALUE (1, 'erick', '一班');
```

### 1. 并发问题

- 访问相同数据的事务，如果<font color=orange>并发执行</font>，可能会有一些问题

#### 脏写(Dirty Write)

- 对于两个事务Session-A和Session-B，如果B修改了<font color=orange>未提交</font>事务A<font color=orange>修改过</font>的数据，就是脏写

![image-20230807152316885](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230807152316885.png)

#### 脏读(Dirty Read)

- A读取了已经被B更新，但B还没提交的字段。如果B回滚，A读取的数据就是临时且无效的

![image-20230807152629320](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230807152629320.png)

#### 不可重复读(Non Repeated Read)

- 事务B中，隐式提交了几次数据，导致事务A中，每次读取的结果都不一样

![image-20230807153729181](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230807153729181.png)

#### 幻读(Phantom)

- Session-A在一个事务中，读取到了Session-B<font color=orange>新增</font>的数据，也就是<font color=orange>幻影记录</font>
- 如果B是删除了数据，则不算幻读

![image-20230807153932214](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230807153932214.png)

### 2. 隔离级别

- 上面四种并发问题严重性：脏写>脏读>不可重复读>幻读
- 隔离级别越高，并发越低；隔离级别越低，并发越高
- 四种隔离级别，都能解决脏写的问题，因为脏写不能容忍
- MySQL默认： REPEATABLE READ

![image-20230807155057395](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230807155057395.png)

```sql
# 默认： REPEATABLE-READ
SHOW VARIABLES LIKE 'transaction_isolation';

# 修改：READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE
SET GLOBAL|SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

SET GLOBAL|SESSION TRANSACTION_ISOLATION='';

# 修改配置文件
```

数据准备

```sql
CREATE TABLE account
(
    id      INT,
    name    VARCHAR(20),
    balance INT,
    PRIMARY KEY (id)
);

INSERT INTO account (id, name, balance)
VALUES (1, '张三', 100),
       (2, '李四', 0);
```

#### READ UNCOMMITTED-脏读

- 会话1还没提交的数据，被会话2读取到了

```SQL
SET GLOBAL TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

```sql
# 会话1
BEGIN;

UPDATE account
SET balance=200
WHERE id = 1;

#  如果rollback了，则会话2读取到了临时且无效的数据
```

```SQL
# 会话2： 读取到了balance=200
SELECT *
FROM account
WHERE id = 1;
```

#### READ COMMITTED-不可重复读

- 能够解决上面脏读
- 但是会有不可重复读

```SQL
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

```sql
#  会话1: 隐式的提交了几次数据
UPDATE account
SET balance=balance - 50
WHERE id = 1;


UPDATE account
SET balance=balance - 50
WHERE id = 1;
```

```SQL
# 会话2： 在一个会话中，读取的值不同
BEGIN;

SELECT *
FROM account
WHERE id = 1;

SELECT *
FROM account
WHERE id = 1;
```

#### REPEATABLE READ-幻读

- MySQL默认级别
- 能解决上面的不可重复读的问题
- 但是存在幻读问题

```sql
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

```SQL
# 会话1: 先开启
BEGIN;

# 读取不到，这个时候开启会话2
SELECT *
FROM account
WHERE id = 3;

# 依然读取不到：没读取到
SELECT *
FROM account
WHERE id = 3;

# 用这个可以证明：读取到了
INSERT INTO account (id, name, balance) VALUE (3, 'lucy', 200);

ROLLBACK ;
```

```sql
#  会话2：
INSERT INTO account (id, name, balance) VALUE (3, 'lucy', 200);
```

#### SERIALIZABLE

- 通过添加锁的方式，使得多个事务之前，完全是串行执行，并发度较低

# 事务日志

- 事务的隔离型是由<font color=orange>锁机制</font>实现的
- 事务的原子性，一致性和持久性由事务的redo日志和undo日志来保证

## REDO LOG

- <font color=orange>重做日志</font>：提供再写入操作，恢复提交事务修改的页操作，保证事务的<font color=orange>持久性</font>
- 存储引擎InnoDB生成的日志，记录<font color=orange>物理级别</font>上的页修改操作，比如页号xx，偏移量是yyy，写入了zzz数据
- 存储引擎以<font color=orange>页为单位</font>管理存储空间。访问页时，先读<font color=orange>磁盘上</font>的页到内存中的<font color=orange>Buffer Pool</font>。所有变更都<font color=orange>先更新Buffer Pool</font>中数据，然后Buffer poll中的<font color=orange>脏页</font>会以一定的频率被刷入磁盘文件<font color=orange>(checkpoint机制)</font>

### 1. 引入原因

- 一个事务提交后(业务成功)，数据先在Buffer Pool中写，然后等待checkpoint刷盘完成持久化
- 假如Buffer Poll中已经有数据了，但还未刷盘。此时服务器宕机，Buffer Poll中数据丢失，最终没有完成持久化

![image-20230813102504050](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230813102504050.png)

```bash
# 暴力解决方案
- 事务提交完毕后，立刻把该事务所修改的页面刷盘

# 缺点一： 修改量和刷盘量工作 不成比例
- 仅仅修改了某个页面中的一个字节，刷盘时需要把一个完整的页从内存中刷新到磁盘中
- 一个页面是16kb，只修改一个字节就要刷新16kb的数据到磁盘上，性能消耗

# 缺点二：随机IO刷新慢
- 一个事务可能包含很多个语句，牵涉到修改多个页面
- 这些页面可能并不相邻，刷盘时候进行随机IO，比顺序IO要慢
```

### 2. redo log

- InnoDB引擎：REDO LOG是<font color=orange>Write Ahead Logging， WAL技术</font>
- 记录内容：某个事务将某个表中<font color=orange>第10页</font>中偏移量为<font color=orange>100</font>的字节值从<font color=orange>1</font>改为<font color=orange>2</font>，存储表空间ID, 页号，偏移量，更新的值
- 先写日志，再写磁盘，只有日志( redo log buffer 或 redo log file)写入成功，才算事务提交成功
- 如果发生宕机且Buffer Pool数据未刷盘到磁盘时，通过redo log来恢复，保证事务的持久性

![image-20230813105442456](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230813105442456.png)

```bash
# 1 事务开启后，先将原始数据从磁盘中读到Buffer Pool中，修改数据的内存拷贝
# 2. 每次执行一个sql，就会写入一个记录到redo log buffer，记录的是数据被修改后的值
# 3. 将redo log buffer的数据写入到redo log file(追加写)
# 4. 定期将buffer pool中的数据刷新到磁盘文件中

# 优点： 
- redo 日志降低了刷盘频率
- redo日志占用的空间非常小
```

### 3 刷盘策略

- redo log bufffer刷盘到redo log file中的频率

```bash
# page cache
- OS层面的，为提高文件写入效率的一个优化，真正的写入会交给OS来决定(如page cache足够大了)

#  策略: 0,1,2
SHOW VARIABLES LIKE 'innodb_flush_log_at_trx_commit';
```

#### 0值

```bash
# innodb_flush_log_at_trx_commit=0
- 每次事务提交，不进行刷盘操作
- InnoDB有一个后台线程，每隔1s，就把redo log buffer中的内容写入到page cache，并调用刷盘操作

# 特点
- 会丢失上一次刷盘距离mysql宕机之间的数据
```

![image-20230813112837831](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230813112837831.png)

#### 1(默认值)

```bash
# innodb_flush_log_at_trx_commit=1
- 每次事务提交时，将redo log buffer中数据写入page cache中，并立刻调用OS，将数据同步到redo log file中
- 假如事务持续时间长，则还没提交的事务的执行sql操作，也会被InnoDB后台线程(0值场景)进行同步

# 特点
- 只要事务提交成功(刷盘redo log file成功才算成功)，则redo log记录一定在硬盘中，不会有任何数据丢失
- 事务执行期间mysql挂了，虽然redo log buffer数据丢了，但是事务并没有提交，所以满足持久性
```

![image-20230813111828299](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230813111828299.png)

#### 2值

```bash
# innodb_flush_log_at_trx_commit=2
- 每次事务提交时，只把redo log buffer内容写入到page cache，不进行同步，由OS决定什么时候同步到redo log file中

# 特点
- 如果数据从redo log buffer更新到了page cache中了，OS挂了，则丢失page cache中的数据
```

![image-20230813112207156](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230813112207156.png)

### 4. redo log buffer

#### 4.1 mini-transaction

- MySQL把底层页面中的一次原子访问的过程称为一个mini-transaction(mtr)

```bash
# 场景：向某个索引对应的B+树中插入一条记录的过程就是一个mtr
- 一个mtr会包含一组redo日志(B+树中插入一条记录，可能会涉及到多个页面的变化)
- redo日志就是记录的具体哪个页面的哪个位置的数据发生了什么样的变化
- 进行崩溃恢复时，这一组redo日志作为一个不可分割的整体
```

![image-20230813121013975](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230813121013975.png)

#### 4.2 redo log buffer结构

- 保存在内存中，比较容易丢失
- Mysql服务启动后，向OS申请一大片<font color=orange>连续内存空间</font>，并被划分为若干个连续的<font color=orange>redo log block</font>，一个block是<font color=orange>512字节</font>
- 开启事务后，每次执行一条sql语句，就会写入若干个redo log buffer记录

![image-20230813103555500](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230813103555500.png)

```sql
# redo log buffer, 默认16M,最大4096M，最小1M
SHOW VARIABLES LIKE 'innodb_log_buffer_size';
```

#### 4.3 写入过程

- <font color=orange>顺序写</font>，先往前面的block中写，当该block的空闲空间用完后，再往下一个block中写
- 每个mtr运行过程中，产生的日志会暂存到一个地方，当该mtr结束时，将该mtr的redo日志全部复制到log buffer中

![image-20230813152823659](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230813152823659.png)

- 一个mtr执行过程，会产生若干条redo日志，这些redo log是一个不可分割的组。同一事务中的多个mtr，可以不连续
- 多个事务可以并发执行：t1代表事务1，t2代表事务2

![image-20230813153632841](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230813153632841.png)

### 5. redo log file

- 保存在磁盘上的，持久化文件

#### 5.1 基本参数

```sql
# 存放路径：        默认数据目录相对路径： /var/lib/mysql
SHOW VARIABLES LIKE 'innodb_log_group_home_dir';         # 默认值：   ./

# 文件个数
# 默认是2个文件， 先把文件内存空间大小开辟好
SHOW VARIABLES LIKE 'innodb_log_files_in_group';

# 单个文件大小：默认48M
# 最大值是512G， 指整个redo log 系列文件之和， 即innodb_log_files_in_group*innodb_log_file_size
SHOW VARIABLES LIKE 'innodb_log_file_size';


# 默认的文件
-rw-r----- 1 mysql mysql 50331648 Aug  8 09:27 ib_logfile0
-rw-r----- 1 mysql mysql 50331648 Aug  1 13:47 ib_logfile1
```

#### 5.2 checkpoint机制

- <font color=orange>已同步到.ibd文件的数据</font>或者<font color=orange>rollback后的事务</font>，应该从redo log file中删除掉
- <font color=orange>write pos</font>：当前记录的位置，一边写一边后移
- <font color=orange>checkpoint</font>：当前要擦除的位置，也是往后移动

![image-20230813105442456](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230813105442456.png)

```bash
- 每次刷盘redo log buffer中记录到日志文件组中，write pos的位置就往后移动更新
- 服务器正常：  执行6步骤时，清空加载过的redo log buffer中记录，checkpoint向后移动
- 服务器宕机后： 从redo log file加载数据，然后一系列最终进入xxx.idb文件，checkpoint向后移动

# 如果write pos追上checkpoint，则表示redo log file已经满了，mysql会停下，将checkpoint来推进一下
```

![image-20230813155641835](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230813155641835.png)

## UNDO LOG

- 保证事务的原子性。在事务的<font color=orange>写数据</font>的前置操作：先写入一个undo log

# 事务锁

- 数据库中，除传统的计算资源(CPU, RAM, I/O)的争用意外，<font color=orange>数据</font>也是一种供多个用户共享的资源
- 锁机制：为MySQL各个隔离级别提供了保证

## 1. 并发事务访问相同记录

### 1.1 读-读

- 并发事务相继读取相同的记录，读取操作本身不会对记录产生任何影响，并不会引发问题

### 1.2 写-写

- 并发事务相继对相同的记录做出改动
- 可能发生脏写问题，任何一种隔离级别都不允许这种情况发生。
- 多个未提交事务相继对一条记录做改动时，需要让它们<font color=orange>排队执行</font>，这个排队的过程其实就是通过<font color=orange>锁</font>来实现
- 锁：内存中的结构

```bash
# 1. 一开始时，记录不会和任何的锁结构来进行关联

# 2. 一个事务要对记录做改动时，先看内存中有没有与这条记录关联的锁结构
- 如果没有，则在内存中生成一个锁结构和该条记录关联

# 3. 新的事务过来，也想修改改条记录，
- 发现有一个锁与之关联了，则也生成一个锁结构和该记录关联，is_waiting状态设置为true，同时等待

# 4. 占用行记录的事务执行完毕后，释放当前事务的锁结构，同时检查行记录数据的其他锁对应的事务，然后去争抢
```

![image-20230813181957498](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230813181957498.png)

### 1.3 读-写

- 可能发生<font color=orange>脏读</font>，<font color=orange>不可重复读</font>，<font color=orange>幻读</font>
- 利用加锁或者MVCC来解决

## 2. 锁分类-数据操作

### 2.1 读锁

- Shared Lock,   S锁，针对同一个数据，多个事务的读操作可以同时进行而不互相影响

### 2.2 写锁

- Exclusive Lock，X锁，当前写操作没完成时，阻断其他写锁和读锁

|      | X锁    | S锁    |
| ---- | ------ | ------ |
| X锁  | 不兼容 | 不兼容 |
| S锁  | 不兼容 | 兼容   |

### 锁定读

- 即可加共享锁，也可加排他锁

```sql
SELECT ... FOR SHARE;               # 共享锁
SELECT ... LOCK IN SHARE MODE;      # 共享锁

SELECT ... FOR UPDATE;             # 排他锁
```

```SQL
# 两个共享锁: 不会阻塞

# 会话一：
SELECT * FROM account FOR SHARE;

# 会话二：
SELECT * FROM account FOR SHARE;
```

```sql
# 一个共享锁，一个排他锁： 会阻塞

# 会话一：必须事务结束后，会话二才会获取到记录
BEGIN;

SELECT * FROM account FOR UPDATE;

COMMIT;

# 会话二：
SELECT * FROM account FOR SHARE;
```

```sql
# 两个排他锁： 会阻塞
# 会话一：必须事务结束后，会话二才会获取到记录
BEGIN;

SELECT * FROM account FOR UPDATE;

COMMIT;

# 会话二：
BEGIN;

SELECT * FROM account FOR UPDATE;

COMMIT;
```

```sql
# 8.0新特性

# 1. 如果会话中包含一个排他锁，当前会话，超时等待时间
SHOW VARIABLES LIKE 'innodb_lock_wait_timeout';

# 2. NOWAIT: 如果获取不到锁，则立刻报错返回

# 3. SKIP LOCK： 也会立刻返回，只是返回的结果中不包含被锁定的行


# 会话一：
BEGIN;

SELECT * FROM account FOR UPDATE;

COMMIT;

# 会话二：
BEGIN;

# [HY000][3572] Statement aborted because lock(s) could not be acquired immediately and NOWAIT is set.
SELECT * FROM account FOR UPDATE NOWAIT;

COMMIT;
```

## 3. 锁分类-粒度

- 为了尽可能提高数据库的并发度，每次锁定的数据范围越小越好
- 理论上，每次只操作当前操作的数据的方案能得到最大的并发度，但是管理锁是很耗资源的事情(涉及获取，检查，释放等动作)
- 数据库系统需要在<font color=orange>并发性能</font>和<font color=orange>系统性能</font>两方面进行平衡

### 3.1 表锁

#### 表锁

- 锁定整个表，是MySQL中最基本的锁策略，<font color=orange>不依赖与存储引擎</font>，表锁是<font color=orange>开销最小</font>的策略，粒度最大
- 表级锁一次会将整个表锁定，所以可以很好的<font color=orange>避免死锁</font>，但是<font color=orange>并发性低</font>
- 一般不建议对InnoDB添加表锁，因为InnoDB支持行锁

```sql
# 以MyISAM为例，当然InnoDB也是可以的
CREATE TABLE dog
(
    id   INT,
    name VARCHAR(20),
    PRIMARY KEY (id)
) ENGINE = MyISAM;

# 查看当前都有哪些表带锁
SHOW OPEN TABLES WHERE in_use>0;
```

```sql
# 基本语法
LOCK TABLES dog READ;        # 加读锁
LOCK TABLES dog WRITE;       # 加写锁

UNLOCK TABLES;               # 释放锁，不用指定哪个表
```

| 锁类型 | 当前会话可读 | 当前会话可写 | 当前会话可操作其他表 | 其他会话可读 | 其他会话可写 |
| ------ | ------------ | ------------ | -------------------- | ------------ | ------------ |
| 读锁   | Y            | N            | N                    | Y            | N，等        |
| 写锁   | Y            | Y            | N                    | N，等        | N，等        |

#### 意向锁

- InnoDB支持<font color=orange>多粒度锁</font>，允许<font color=orange>行级锁和表级锁共存</font>

```bash
# 解决的问题： 一个事务要加表级锁时，需要全部遍历所有记录，确保每一行都没有锁
- 事务t1对A-表中某个行加上了锁
- 事务t2要对A-表中添加 表级别 的x锁或s锁，首先要确定该表上是否有其他锁(表锁/行锁)
- 事务t2必须遍历A-表中所有行，才能确保是否有锁，比较耗费性能
```

- 事务给某一行数据加上了<font color=orange>排他锁</font>，数据库会<font color=orange>自动</font>给更大一级的空间，比如数据页或数据表加上<font color=orange>意向锁</font>(锁标识)，告诉其他事务这个数据页或数据表已经有其他人上过排他锁了
- 由InnoDB存储引擎<font color=orange>自己维护的</font>，在为数据行加共享/排他锁之前，会先在表级别添加意向锁

```sql
CREATE TABLE dog
(
    id   INT,
    name VARCHAR(20),
    PRIMARY KEY (id)
);

# 会话一: 为某个行加锁， 则存储引擎会自动给该表加上一个意向排他锁
# 只有commit后，x锁才会释放，会话二才会开始执行
BEGIN;

SELECT *
FROM dog
WHERE id = 1 FOR
UPDATE;

COMMIT;

# 会话二：锁定某个表
LOCK TABLES dog READ;
```

|        | 意向共享锁IS | 意向排他锁IX |
| ------ | ------------ | ------------ |
| 共享锁 | 兼容         | 互斥         |
| 排他锁 | 互斥         | 互斥         |

- 意向锁之间是互相兼容的，因为只是一个锁标识，多个行记录，可能都会添加对应的意向锁标识

|              | 意向共享锁IS | 意向排他锁IX |
| ------------ | ------------ | ------------ |
| 意向共享锁IS | 兼容         | 兼容         |
| 意向排他锁IX | 兼容         | 兼容         |

#### 元数据锁

- meta data lock: MDL锁，保证读写的正确性
- <font color=orange>当对一个表做写操作时，加DML读锁；当对表结构变更操作时，加DML写锁</font>
- 不需要显示使用，在访问一个表的时候会被自动加上
- 多个读锁之间不互斥，读写锁之间，写锁之间是互斥的

### 3.2 行锁

- InnoDB支持，MyISAM不支持
- 优点：锁粒度小，发生锁冲突概率低，并发度高
- 缺点：锁开销比较大，容易出现死锁

#### 记录锁

- 当一个事务获取了一条记录的s锁时，其他事务可以继续获取该记录的s锁，但不可以继续获取该记录的x记录锁
- 当一个事务获取了一条记录的x锁时，其他事务不能获取该记录的x和s锁

#### 间隙锁

- Gap Locks
- MySQL在REPETATED READ下，是可以解决幻读问题
- 间隙锁仅仅是为了防止插入幻影记录而提出来的

```sql
# 数据多行假如为1，3，6，9，10的主键

# 事务一：
可以为主键为5的(数据不存在)，加一个间隙锁
 SELECT * FROM table WHERE id=14 LOCK IN SHARE MODE;       # 会把10～正无穷的记录锁住，不允许其他事务插入
```

#### 临键锁

- Next-Key Locks：LOCK_ORDINARY，是记录锁和间隙锁的合体
- 既想锁住某条记录，又想阻止其他事务在该记录前边的间隙插入新记录

#### 插入意向锁

- Insert Intention Locks

### 3.3 页锁

- 在页的粒度上进行锁定
- 页锁的开销介于表锁和行锁之间，会出现死锁
- 每个层级的锁数量是有限制的，当某个层级的锁数量超过了阈值，就会进行锁升级

## 4. 锁分类-态度

### 4.1 乐观锁

```sql

```

### 4.2 悲观锁

```sql
SELECT ... FOR UPDATE;
```


[TOC]



## Chapter 10 Create and manage Table创建管理表

### 3. 创建表

#### 3.1 创建方式1

- 必备条件:

  - CREATE TABLE 权限
  - 存储空间

- 语法格式

  ```{SQL}
  CREATE TABLE [IF NOT EXISTS] tablename(
  	字段1, 数据类型 [约束条件] [默认值],
      字段1, 数据类型 [约束条件] [默认值],
      ...
      [表约束条件]
  );
  ```

  ```{sql}
  CREATE TABLE dept(
  	deptno INT(2) AUTO_INCREMENT,
      dname VARCHAR(14),
      loc VARCHAR(13),
      PRIMARY KEY(deptno)
  );
  ```



#### 3.2 创建方式2

- 使用 AS subquery

```{sql}
CREATE TABLE emp1 AS SELECT * FROM employees; -- 复制表及内容
CREATE TABLE emp2 AS SELECT * FROM employees WHERE 1=2; -- 创建的emp2是空表(复制表, 不含数据)
CREATE TABLE emp3 AS 
SELECT * 
FROM atguigu.employees; -- 创建的emp3从不同的数据库
```

```sql
CREATE TABLE dept80
AS 
SELECT  employee_id, last_name, salary*12 ANNSAL, hire_date
FROM    employees
WHERE   department_id = 80;
```

#### 3.3 查看数据表结构

```sql
SHOW CREATE TABLE 表名\G
```

### 4. 修改表

#### 4.1 追加一个列

```sql
ALTER TABLE table_name 
ADD [Column] 字段名 字段类型[FIRST | AFTER 字段名],
ADD [Column] 字段名 字段类型[FIRST | AFTER 字段名],
...;

ALTER TABLE dept80 
ADD job_id varchar(10);
```

#### 4.2 修改一个列

```sql
ALTER TABLE 表名 MODIFY [COLUMN] 字段名1 字段类型 [DEFAULT 默认值] [FIRST|AFTER] 字段名2]

ALTER TABLE dept90
MODIFY last_name VARCHAR(20);

ALTER TABLE dept90
MODIFY salary DOUBLE(20,2) default 1000;
```

#### 4.3 重命名一个列

```sql
ALTER TABLE table_name CHANGE [column] 列名 新列名 新数据类型

ALTER TABLE dept80
CHANGE department_name dept_name VARCHAR(15); -- 修改数据类型
```

#### 4.4 Delete a column 删除一个列

```sql
ALTER TABLE table_name DROP [column] 字段名

ALTER TABLE dept80
DROP COLUMN job_id;
```

### 5. Rename

```sql
RENAME TABLE emp
TO myemp;

ALTER TABLE dept
RENAME [TO] detail_dept;
```

### 6. DELETE TABLE 删除表

------

1. 在MySQL中，当一张数据表**没有与其他任何数据表形成关联关系时**，可以将当前数据表直接删除。
2. 数据和结构都被删除
3. 所有正在运行的相关事务被提交
4. 所有相关索引被删除

```sql
DROP TABLE [IF EXISTS] 数据表1 [,数据表2,...数据表n];

DROP TABLE dept80;
```

### 7. 清空表

------

- TRUNCATE TABLE 语句:

  - 删除表中的所有数据
  - 释放表的存储空间
  - 不能回滚(DDL)

- 举例:

  ```sql
  TRUNCATE TABLE detail_dept;
  ```

- DELETE 语句:

```sql
SET autocommit = FALSE;

DELETE FROM emp2;
# TRUNCATE TABLE emp2;
ROLLBACK;
```

> 阿 里开发规范：
> 【参考】TRUNCATE TABLE 比 DELETE 速度快，且使用的系统和事务日志资源少，但 TRUNCATE 无
> 事务且不触发 TRIGGER，有可能造成事故，故不建议在开发代码中使用此语句。
> 说明：TRUNCATE TABLE 在功能上与不带 WHERE 子句的 DELETE 语句相同。



### 拓展1：阿里巴巴《Java开发手册》之MySQL字段命名

1. 【 **强制** 】表名、字段名必须使用小写字母或数字，禁止出现数字开头，禁止两个下划线中间只出现数字。数据库字段名的修改代价很大，因为无法进行预发布，所以字段名称需要慎重考虑。
   正例：aliyun_admin，rdc_config，level3_name
   反例：AliyunAdmin，rdcConfig，level_3_name
2. 【 **强制** 】禁用保留字，如 desc、range、match、delayed 等，请参考 MySQL 官方保留字。
3. 【 **强制** 】表必备三字段：id, gmt_create, gmt_modified。说明：其中 id 必为主键，类型为BIGINT UNSIGNED、单表时自增、步长为 1。gmt_create,gmt_modified 的类型均为 DATETIME 类型，前者现在时表示主动式创建，后者过去分词表示被动式更新
4. 【 *推荐* 】表的命名最好是遵循 “业务名称_表的作用”。
   正例：alipay_task 、 force_project、 trade_config
5. 【 *推荐* 】库名与应用名称尽量一致。
6. 【参考】合适的字符存储长度，不但节约数据库表空间、节约索引存储，更重要的是提升检索速度。
   正例：无符号值可以避免误存负数，且扩大了表示范围。

| 对象     | 年龄区间 | 类型              | 字节 | 范围(无符号)  |
| -------- | -------- | ----------------- | ---- | ------------- |
| 人       | 150岁    | tinyint unsigned  | 1    | 0 ~ 255       |
| 龟       | 数百岁   | smallint unsigned | 2    | 0 ~ 65535     |
| 恐龙化石 | 数千万年 | int unsigned      | 4    | 0 ~ 43亿      |
| 太阳     | 50亿     | bigint unsigned   | 8    | 0 ~ $10^{19}$ |

### 拓展2: 如何理解清空表、删除表等操作需谨慎？！

**表删除** 操作将把表的定义和表中的数据一起删除，并且MySQL在执行删除操作时，不会有任何的确认信息提示，因此执行删除操时应当慎重。在删除表前，最好对表中的数据进行 备份 ，这样当操作失误时可以对数据进行恢复，以免造成无法挽回的后果。
同样的，在使用 **ALTER TABLE**  进行表的基本修改操作时，在执行操作过程之前，也应该确保对数据进行完整的 **备份** ，因为数据库的改变是 **无法撤销** 的，如果添加了一个不需要的字段，可以将其删除；相同的, 如果删除了一个需要的列, 该列下面的所有数据都将丢失.



### 拓展3：MySQL8新特性—DDL的原子化

在MySQL 8.0版本中，InnoDB表的DDL支持事务完整性，即 DDL操作要么成功要么回滚 。DDL操作回滚日志写入到data dictionary数据字典表mysql.innodb_ddl_log（该表是隐藏的表，通过show tables无法看到）中，用于回滚操作。通过设置参数，可将DDL操作日志打印输出到MySQL错误日志中。



## Chapter 11 Data Management(Insert modify delete)

## 1. Insert Data

#### 1.2 方式1

情况1: 为表中所有字段按默认顺序插入(声明的先后顺序添加)

```sql
INSERT INTO table_name
VALUES (value1, value2, value3, ...);

INSERT INTO departments
VALUES (70, 'Pub', 100, 1700);
```

情况2: 为指定字段插入数据(推荐, 未赋值的字段为null)

```sql
INSERT INTO table_name(column1 [, column2, …, columnn]) 
VALUES (value1 [,value2, …, valuen]);

INSERT INTO departments(department_id, department_name)
VALUES (80, 'IT');
```

情况3: 同时插入多条记录(推荐, 其他同上)

```sql
INSERT INTO table_name (column1 [, column2, …, columnn])
VALUES 
(value1 [,value2, …, valuen]),
(value1 [,value2, …, valuen]),
……
(value1 [,value2, …, valuen]);

mysql> INSERT INTO emp(emp_id,emp_name)
    -> VALUES (1001,'shkstart'),
    -> (1002,'atguigu'),
    -> (1003,'Tom');
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

使用INSERT同时插入多条记录时，MySQL会返回一些在执行单行插入时没有的额外信息，这些信息的含义如下：

 ● Records：表明插入的记录条数。 

● Duplicates：表明插入时被忽略的记录，原因可能是这些记录包含了重复的主键值。

● Warnings：表明有问题的数据值，例如发生数据类型转换。

> 一个同时插入多行记录的INSERT语句等同于多个单行插入的INSERT语句，但是多行的INSERT语句在处理过程中 **效率更高** 。因为MySQL执行单条INSERT语句插入多行数据比使用多条INSERT语句快, 所以在插入记录时最好使用单条INSERT语句的方式插入.

小结:

- VALUES 也可以写成 VALUE ，但是VALUES是标准写法。
- 字符和日期型数据应包含在单引号中。

#### 1.3 方式2: 将查询结果插入表中

- 在 INSERT 语句中加入子查询。
- **不必书写 VALUES 子句。**
- 子查询中的值列表应与 INSERT 子句中的**列名对应**, 注意字段的数据类型和范围。

```sql
INSERT INTO 目标表名
(tar_column1 [, tar_column2, …, tar_columnn])
SELECT
(src_column1 [, src_column2, …, src_columnn])
FROM ori_table
[WHERE condition];
```

```sql
INSERT INTO sales_reps(id, name, salary, commission_pct)
SELECT employee_id, last_name, salary, commission_pct
FROM   employees
WHERE  job_id LIKE '%REP%';
```

更新中的数据可能存在完整性错误.  外键约束影响. 

### 2. 更新数据

如需回滚需设置:

```sql
SET AUTOCOMMIT=FALSE;
```

```sql
UPDATE table_name
SET column1=value1, column2=value2, … , column=valuen
[WHERE condition];

UPDATE emp1
SET salary = salary * 1.2
WHERE name LIKE '%a%';
```

### 3. 删除数据

```sql
DELETE FROM table_name [WHERE <condition>];

DELETE FROM departments
WHERE department_name = 'Finance';
```

### 4. 新特性: 列计算

## Chaper 12 Data type

### 1. MySQL中的数据类型

| 类型             | 类型举例                                                     |
| ---------------- | ------------------------------------------------------------ |
| 整数类型         | TINYINT、SMALLINT、MEDIUMINT、INT(或INTEGER)、BIGINT         |
| 浮点类型         | FLOAT、DOUBLE                                                |
| 定点数类型       | DECIMAL                                                      |
| 位类型           | BIT                                                          |
| 日期时间类型     | YEAR、TIME、DATE、DATETIME、TIMESTAMP                        |
| 文本字符串类型   | CHAR、VARCHAR、TINYTEXT、TEXT、MEDIUMTEXT、LONGTEXT          |
| 枚举类型         | ENUM                                                         |
| 集合类型         | SET                                                          |
| 二进制字符串类型 | BINARY、VARBINARY、TINYBLOB、BLOB、MEDIUMBLOB、LONGBLOB      |
| JSON类型         | JSON对象、JSON数组                                           |
| 空间数据类型     | 单值类型:GEOMETRY、POINT、LINESTRING、POLYGON;<br/>集合类型:MULTIPOINT、MULTILINESTRING、MULTIPOLYGON、<br/>GEOMETRYCOLLECTION |

| MySQL关键字        | 含义                    |
| ------------------ | ----------------------- |
| NULL               | 数据列可包含NULL值      |
| NOT NULL           | 数据列不允许包含NULL值  |
| DEFAULT            | 默认值                  |
| PRIMARY KEY        | 主键                    |
| AUTO_INCREMENT     | 自动递增,适用于整数类型 |
| UNSIGNED           | 无符号                  |
| CHARACTER SET name | 指定一个字符集          |

### 2. 整数类型

#### 2.1 类型介绍

整数类型一共有 5 种,包括 TINYINT、SMALLINT、MEDIUMINT、INT(INTEGER)和 BIGINT。

#### 2.2 可选属性

整数类型的可选属性有三个:

##### 2.2.1 M

M : 表示显示宽度,M的取值范围是(0, 255)。例如,int(5):当数据宽度小于5位的时候在数字前面需要用字符填满宽度。该项功能需要配合“ **ZEROFILL** ”使用,表示用“0”填满宽度,否则指定显示宽度无效。

如果设置了显示宽度,那么插入的数据宽度超过显示宽度限制,会不会截断或插入失败?

答案:不会对插入的数据有任何影响,还是按照类型的实际宽度进行保存,即 显示宽度与类型可以存储的值范围无关 。**从MySQL 8.0.17开始,整数数据类型不推荐使用显示宽度属性。**

整型数据类型可以在定义表结构时指定所需要的显示宽度,如果不指定,则系统为每一种类型指定默认的宽度值。
举例:

```sql
CREATE TABLE test_int1 ( x TINYINT, y SMALLINT, z MEDIUMINT, m INT, n BIGINT );
```

查看表结构 (MySQL5.7中显式如下,MySQL8中不再显式范围) 

```sql
mysql> desc test_int1;
+-------+--------------+------+-----+---------+-------+
| Field | Type		   | Null | Key | Default | Extra |
+-------+--------------+------+-----+---------+-------+
|  x 	| tinyint(4)   | YES  |     | NULL    |       |
|  y    | smallint(6)  | YES  |     | NULL    |       |
|  z    | mediumint(9) | YES  |     | NULL    |       |
|  m    | int(11)      | YES  |     | NULL    |       |
|  n    | bigint(20)   | YES  |     | NULL    |       |
+-------+--------------+------+-----+---------+-------+
5 rows in set (0.00 sec)
```

TINYINT有符号数和无符号数的取值范围分别为-128~127和0~255,由于负号占了一个数字位,因此TINYINT默认的显示宽度为4。同理,其他整数类型的默认显示宽度与其有符号数的最小值的宽度相同。
举例:



```sql
CREATE TABLE test_int2(
f1 INT,
f2 INT(5),
f3 INT(5) ZEROFILL
)
DESC test_int2;
INSERT INTO test_int2(f1,f2,f3)
VALUES(1,123,123);
INSERT INTO test_int2(f1,f2)
VALUES(123456,123456);
INSERT INTO test_int2(f1,f2,f3)
VALUES(123456,123456,123456);

mysql> SELECT * FROM test_int2;
+--------+--------+--------+
| f1     | f2     | f3     |
+--------+--------+--------+
| 1      | 123    | 00123  |
| 123456 | 123456 |   NULL |
| 123456 | 123456 | 123456 |
+--------+--------+--------+
3 rows in set (0.00 sec)
```

##### 2.2.2 UNSIGNED

UNSIGNED : 无符号类型(非负),所有的**整数类型**都有一个可选的属性UNSIGNED(无符号属性),无
符号整数类型的最小取值为0。所以,如果需要在MySQL数据库中**保存非负整数值**时,可以将整数类型设
置为无符号类型。
int类型默认显示宽度为int(11),无符号int类型默认显示宽度为int(10)。

```sql
CREATE TABLE test_int3(
f1 INT UNSIGNED
);
mysql> desc test_int3;
+-------+------------------+------+-----+---------+-------+
| Field | Type             | Null | Key | Default | Extra |
+-------+------------------+------+-----+---------+-------+
| f1    | int(10) unsigned | YES  |     | NULL    |       |
+-------+------------------+------+-----+---------+-------+
1 row in set (0.00 sec)
```

##### 2.2.3 ZEROFILL

**ZEROFILL** : 0填充,(如果某列是ZEROFILL,那么MySQL会自动为当前列添加UNSIGNED属性),如果指定了ZEROFILL只是表示不够M位时,用0在左边填充,如果超过M位,只要不超过数据存储范围即可。
原来,在 int(M) 中,M 的值跟 int(M) 所占多少存储空间并无任何关系。 int(3)、int(4)、int(8) 在磁盘上都是占用 4 bytes 的存储空间。也就是说,int(M),必须和UNSIGNED ZEROFILL一起使用才有意义。如果整数值超过M位,就按照实际位数存储。只是无须再用字符 0 进行填充。

#### 2.3 适用场景

TINYINT :一般用于枚举数据,比如系统设定取值范围很小且固定的场景。
SMALLINT :可以用于较小范围的统计数据,比如统计工厂的固定资产库存数量等。
MEDIUMINT :用于较大整数的计算,比如车站每日的客流量等。
INT、INTEGER :取值范围足够大,一般情况下不用考虑超限问题,用得最多。比如商品编号。
BIGINT :只有当你处理特别巨大的整数时才会用到。比如双十一的交易量、大型门户网站点击量、证券公司衍生产品持仓等。



### 3. 浮点类型

#### 3.1 类型介绍

浮点数和定点数类型的特点是可以 **处理小数** ,你可以把整数看成小数的一个特例。因此,浮点数和定点数的使用场景,比整数大多了。 MySQL支持的浮点数类型,分别是 FLOAT、DOUBLE、REAL。

- FLOAT 表示单精度浮点数;
- DOUBLE 表示双精度浮点数;

| 类型   | 有符号范围                                                   | 无符号范围                                            | 字节数 |
| ------ | ------------------------------------------------------------ | ----------------------------------------------------- | ------ |
| FLOAT  | (-3.402823466E+38, -1.175494351E-38), 0, (1.175494351E-38, 3.402823466E+38) | 0, (1.175494351E-38, 3.402823466E+38)                 | 4      |
| DOUBLE | (-1.7976931348623157E+308, -2.2250738585072014E-308)、 0,(2.2250738585072014E-308, 1.7976931348623157E+308) | 0, (2.2250738585072014E-308, 1.7976931348623157E+308) | 8      |

REAL默认就是 DOUBLE。如果你把 SQL 模式设定为启用“ REAL_AS_FLOAT ”,那 么,MySQL 就认为
REAL 是 FLOAT。如果要启用“REAL_AS_FLOAT”,可以通过以下 SQL 语句实现:

```sql
SET sql_mode = “REAL_AS_FLOAT”;
```

**问题1**:FLOAT 和 DOUBLE 这两种数据类型的区别是啥呢?
**答**: FLOAT 占用字节数少,取值范围小;DOUBLE 占用字节数多,取值范围也大。

**问题2**:为什么浮点数类型的无符号数取值范围,只相当于有符号数取值范围的一半,也就是只相当于有符号数取值范围大于等于零的部分呢?
**答**: MySQL 存储浮点数的格式为: **符号(S)** 、 **尾数(M)** 和 **阶码(E)** 。因此,无论有没有符号,MySQL 的浮点数都会存储表示符号的部分。因此, 所谓的无符号数取值范围,其实就是有符号数取值范围大于等于零的部分。

#### 3.2 精度说明

对于浮点类型,在MySQL中单精度值使用 4 个字节,双精度值使用 8 个字节。

- MySQL允许使用 非标准语法 (其他数据库未必支持,因此如果涉及到数据迁移,则最好不要这么用): FLOAT(M,D) 或 DOUBLE(M,D) 。这里,M称为 精度 ,D称为 标度 。(M,D)中 M=整数位+小数位,D=小数位。D<=M<=255,0<=D<=30。

  例如,定义为FLOAT(5,2)的一个列可以显示为-999.99-999.99。如果超过这个范围会报错。

- FLOAT和DOUBLE类型在不指定(M,D)时,默认会按照实际的精度(由实际的硬件和操作系统决定)
  来显示。
  - 说明:浮点类型,也可以加 UNSIGNED ,但是不会改变数据范围,例如:FLOAT(3,2) UNSIGNED仍然
    只能表示0-9.99的范围。
- 不管是否显式设置了精度(M,D),这里MySQL的处理方案如下:
  - 如果存储时,整数部分超出了范围,MySQL就会报错,不允许存这样的值
  - 如果存储时,小数点部分若超出范围,就分以下情况:
    - 若四舍五入后,整数部分没有超出范围,则只警告,但能成功操作并四舍五入删除多余
      的小数位后保存。例如在FLOAT(5,2)列内插入999.009,近似结果是999.01。
    - 若四舍五入后,整数部分超出范围,则MySQL报错,并拒绝处理。如FLOAT(5,2)列内插入
      999.995和-999.995都会报错。
- 从MySQL 8.0.17开始,FLOAT(M,D) 和DOUBLE(M,D)用法在官方文档中已经明确不推荐使用,将来可
  能被移除。另外,关于浮点型FLOAT和DOUBLE的UNSIGNED也不推荐使用了,将来也可能被移除。



举例

```sql
CREATE TABLE test_double1(
f1 FLOAT,
f2 FLOAT(5,2),
f3 DOUBLE,
f4 DOUBLE(5,2)
);
DESC test_double1;
INSERT INTO test_double1
VALUES(123.456,123.456,123.4567,123.45);
#Out of range value for column 'f2' at row 1
INSERT INTO test_double1
VALUES(123.456,1234.456,123.4567,123.45);
SELECT * FROM test_double1;
```



```sql
+---------+--------+----------+--------+
| f1      | f2     | f3       | f4     |
+---------+--------+----------+--------+
| 123.456 | 123.46 | 123.4567 | 123.45 |
+---------+--------+----------+--------+
1 row in set (0.00 sec)

```



#### 3.3 精度误差说明

浮点数类型有个缺陷,就是不精准。下面我来重点解释一下为什么 MySQL 的浮点数不够精准。比如,我们设计一个表,有f1这个字段,插入值分别为0.47,0.44,0.19,我们期待的运行结果是:0.47 + 0.44 + 0.19 = 1.1。而使用sum之后查询:

```sql
CREATE TABLE test_double2(
f1 DOUBLE
);
INSERT INTO test_double2
VALUES(0.47),(0.44),(0.19);

mysql> SELECT SUM(f1)
-> FROM test_double2;
+--------------------+
| sum(f1)            |
+--------------------+
| 1.0999999999999999 |
+--------------------+
1 row in set (0.00 sec)

mysql> SELECT SUM(f1) = 1.1,1.1 = 1.1
-> FROM test_double2;
+---------------+-----------+
| sum(f1) = 1.1 | 1.1 = 1.1 |
+---------------+-----------+
|             0 |         1 |
+---------------+-----------+
1 row in set (0.00 sec)

```

查询结果是 1.0999999999999999。看到了吗?虽然误差很小,但确实有误差。 你也可以尝试把数据类型改成 FLOAT,然后运行求和查询,得到的是, 1.0999999940395355。显然,误差更大了。

那么,为什么会存在这样的误差呢?问题还是出在 MySQL 对浮点类型数据的存储方式上。

MySQL 用 4 个字节存储 FLOAT 类型数据,用 8 个字节来存储 DOUBLE 类型数据。无论哪个,都是采用二进制的方式来进行存储的。比如 9.625,用二进制来表达,就是 1001.101,或者表达成 1.001101×2^3。如果尾数不是 0 或 5(比如 9.624),你就无法用一个二进制数来精确表达。进而,就只好在取值允许的范围内进行四舍五入。

在编程中,如果用到浮点数,要特别注意误差问题,**因为浮点数是不准确的,所以我们要避免使用“=”来判断两个数是否相等**。同时,在一些对精确度要求较高的项目中,千万不要使用浮点数,不然会导致结果错误,甚至是造成不可挽回的损失。那么,MySQL 有没有精准的数据类型呢?当然有,这就是定点数类型: DECIMAL 。





























## 附录: 课程计划

【MySQL上篇：基础篇】
【第1子篇：数据库概述与MySQL安装篇】
p01-p11
学习建议：零基础同学必看，涉及理解和Windows系统下MySQL安装

【第2子篇：SQL之SELECT使用篇】
p12-p48
学习建议：学习SQL的重点，必须重点掌握，建议课后练习多写

【第3子篇：SQL之DDL、DML、DCL使用篇】
p49-p73
学习建议：学习SQL的重点，难度较SELECT低，练习写写就能掌握

【第4子篇：其它数据库对象篇】
p74-p93
学习建议：对于希望早点学完MySQL基础，开始后续内容的同学，这个子篇可以略过。
在工作中，根据公司需要进行学习即可。

【第5子篇：MySQL8新特性篇】
p94-p95
学习建议：对于希望早点学完MySQL基础，开始后续内容的同学，这个子篇可以略过。
在工作中，根据公司需要进行学习即可。

【MySQL下篇：高级篇】
【第1子篇：MySQL架构篇】
p96-p114
学习建议：涉及Linux平台安装及一些基本问题，基础不牢固同学需要学习

【第2子篇：索引及调优篇】
p115-p160
学习建议：面试和开发的重点，也是重灾区，需要全面细致的学习和掌握

【第3子篇：事务篇】
p161-p186
学习建议：面试和开发的重点，需要全面细致的学习和掌握

【第4子篇：日志与备份篇】
p187-p199
学习建议：根据实际开发需要，进行相应内容的学习

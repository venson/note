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



## chaper 13 constraint

### 1. constraint 

#### 1.1 为什么需要

数据完整性（Data Integrity）是指数据的精确性（Accuracy）和可靠性（Reliability）。它是防止数据库中存在不符合语义规定的数据和防止因错误信息的输入输出造成无效操作或错误信息而提出的。

为了保证数据的完整性，SQL规范以约束的方式对表数据进行额外的条件限制。从以下四个方面考虑：

==实体完整性（Entity Integrity）== ：例如，同一个表中，不能存在两条完全相同无法区分的记录
==域完整性（Domain Integrity）== ：例如：年龄范围0-120，性别范围“男/女”
==引用完整性（Referential Integrity）== ：例如：员工所在部门，在部门表中要能找到这个部门
==用户自定义完整性（User-defined Integrity）== ：例如：用户名唯一、密码不能为空等，本部门经理的工资不得高于本部门职工的平均工资的5倍。

#### 1.2 什么是约束

约束是表级的强制规定。
可以在创建表时规定约束（通过 CREATE TABLE 语句），或者在表创建之后通过 ALTER TABLE 语句规定
约束。



#### 1.3 约束的分类

- 根据约束数据列的限制，约束可分为：
  - 单列约束：每个约束只约束一列
  - 多列约束：每个约束可约束多列数据
- 根据约束的作用范围，约束可分为：
  - 列级约束：只能作用在一个列上，跟在列的定义后面
  - 表级约束：可以作用在多个列上，不与列一起，而是单独定义
- 根据约束起的作用，约束可分为：
  - NOT NULL 非空约束，规定某个字段不能为空
  - UNIQUE 唯一约束，规定某个字段在整个表中是唯一的
  - PRIMARY KEY 主键(非空且唯一)约束
  - FOREIGN KEY 外键约束
  - CHECK 检查约束(MySQL不支持)
  - DEFAULT 默认值约束

```sql
#information_schema数据库名（系统库）
#table_constraints表名称（专门存储各个表的约束）
SELECT * FROM information_schema.table_constraints 
WHERE table_name = 'table_name'
```

### 2. 非空约束

#### 2.1 作用

限定某个字段, 列的值不允许为空.

#### 2.3 特点

- 默认，所有的类型的值都可以是NULL，包括INT、FLOAT等数据类型
- 非空约束只能出现在表对象的列上，只能某个列单独限定非空，不能组合非空
- 一个表可以有很多列都分别限定了非空
- 空字符串''不等于NULL，0也不等于NULL

#### 2.4 添加非空约束

（1）建表时
举例：

```sql
CREATE TABLE 表名称(
    字段名  数据类型,
    字段名  数据类型 NOT NULL,  
    字段名  数据类型 NOT NUL
);

CREATE TABLE emp(
	id INT(10) NOT NULL,
	NAME VARCHAR(20) NOT NULL,
	sex CHAR NULL
);

CREATE TABLE student(
    sid int,
    sname varchar(20) not null,
    tel char(11) ,
    cardid char(18) not null
);

insert into student values(1,'张三','13710011002','110222198912032545'); #成功

insert into student values(2,'李四','13710011002',null);#身份证号为空
ERROR 1048 (23000): Column 'cardid' cannot be null

insert into student values(2,'李四',null,'110222198912032546');#成功，tel允许为空

insert into student values(3,null,null,'110222198912032547');#失败
ERROR 1048 (23000): Column 'sname' cannot be null
```

（2）建表后
举例：

```sql
alter table 表名称 modify 字段名 数据类型 not null;

ALTER TABLE emp
MODIFY sex VARCHAR(30) NOT NULL;


alter table student
modify sname varchar(20) not null;
```

#### 2.5 删除非空约束

```sql
alter table 表名称 modify 字段名 数据类型 NULL;
#去掉not null，相当于修改某个非注解字段，该字段允许为空

alter table 表名称 modify 字段名 数据类型;
#去掉not null，相当于修改某个非注解字段，该字段允许为空

ALTER TABLE emp
MODIFY sex VARCHAR(30) NULL;

ALTER TABLE emp
MODIFY NAME VARCHAR(15) DEFAULT 'abc' NULL;
```



### 3. 唯一约束

#### 3.1 作用

用来限制某个字段/某列的值不能重复。

![image-20220203232812205](/home/venson/.config/Typora/typora-user-images/image-20220203232812205.png)



#### 3.3 特点

- 同一个表可以有多个唯一约束。
- 唯一约束可以是某一个列的值唯一，也可以多个列组合的值唯一。
- 唯一性约束允许列值为空。
- 在创建唯一约束的时候，如果不给唯一约束命名，就默认和列名相同。
- ==MySQL会给唯一约束的列上默认创建一个唯一索引。==

#### 3.4 添加约束

（1）建表时
举例：

```sql
create table 表名称(
    字段名  数据类型,
    字段名  数据类型  unique,  
    字段名  数据类型  unique key,
    字段名  数据类型
);
create table 表名称(
    字段名  数据类型,
    字段名  数据类型,  
    字段名  数据类型,
    [constraint 约束名] unique key(字段名)
);

create table student(
    sid int,
    sname varchar(20),
    tel char(11) unique,
    cardid char(18) unique key
);

CREATE TABLE USER(
 id INT NOT NULL,
 NAME VARCHAR(25),
 PASSWORD VARCHAR(16),
 -- 使用表级约束语法, 表示用户名和密码组合不能重复
 CONSTRAINT uk_name_pwd UNIQUE(NAME,PASSWORD)
```

（2）建表后指定唯一键约束
举例：

```sql
#字段列表中如果是一个字段，表示该列的值唯一。如果是两个或更多个字段，那么复合唯一，即多个字段的组合是唯一的
#方式1：
alter table 表名称 add unique key(字段列表); 
#方式2：
alter table 表名称 modify 字段名 字段类型 unique;

ALTER TABLE USER 
ADD UNIQUE(NAME,PASSWORD);

ALTER TABLE USER 
ADD CONSTRAINT uk_name_pwd UNIQUE(NAME,PASSWORD);

ALTER TABLE USER 
MODIFY NAME VARCHAR(20) UNIQUE;
```

## chaper 15 stored procedure

### 4. Stored Function

#### 4.1 语法分析

```sql
CREATE FUNCTION 函数名(参数名 参数类型,...)
RETURNS 返回值类型
[characteristics ...]
BEGIN
	函数体 #函数体中肯定有 RETURN 语句
END
```

说明：
1、参数列表：指定参数为IN、OUT或INOUT只对PROCEDURE是合法的，FUNCTION中总是默认为IN参数。
2、RETURNS type 语句表示函数返回数据的类型；
RETURNS子句只能对FUNCTION做指定，对函数而言这是 强制 的。它用来指定函数的返回类型，而且函数体必须包含一个 RETURN value 语句。
3、characteristic 创建函数时指定的对函数的约束。取值与创建存储过程时相同，这里不再赘述。
4、函数体也可以用BEGIN…END来表示SQL代码的开始和结束。如果函数体只有一条语句，也可以省略
BEGIN…END。

#### 4.2 调用存储函数

在MySQL中，存储函数的使用方法与MySQL内部函数的使用方法是一样的。换言之，用户自己定义的存储函数与MySQL内部函数是一个性质的。区别在于，存储函数是 用户自己定义 的，而内部函数是MySQL的`开发者定义 `的。

```sql
SELECT 函数名(实参列表)
```

#### 4.3 代码举例

```sql
DELIMITER //
SELECT FUNCTION email_by_id(emp_id INT)
RETURNS VARCHAR(25)
BEGIN
	RETURN (SELECT email from employees WHERE employeed_id = emp_Id)
END //
DELIMITER ;

## 调用
# 1. 
SELECT email_by_id(101);
# 2. 
SET @emp_id:=102;
SELECT email_by_id(@emp_id);
```

### 5. 存储过程函数的查看 修改 删除

#### 5.1 查看

1. SHOW CREATE

```sql
SHOW CREATE {PROCEDURE| FUNCTION} 名称

SHOW CREATE FUNCTION test_db.CountProc \G
```

2. SHOW STATUS

```sql
SHOW {PROCEDURE | FUNCTION} STATUS [LIKE 'pattern'];
```

3. 从information_schema.Routines 表中查看信息

```sql
SELECT * FROM information_schema.Routines
WHERE ROUTINE_NAME='存储过程或函数的名' [AND ROUTINE_TYPE = {'PROCEDURE | FUNCTION' }];

SELECT * FROM information_schema.Routines
WHERE ROUTINE_NAME = 'count_by_name' AND ROUTINE_TYPE = 'FUNCTION' \G
```

#### 5.2 修改

不修改功能, 进修改相关特性

```sql
ALTER {PROCEDURE| FUNCTION} 存储过程或函数名[characteristic ...]
```

```sql
{ CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
| SQL SECURITY { DEFINER | INVOKER }
| COMMENT 'string'

ALTER PROCEDURE CountProc
MODIFIES SQL DATA
SQL SECURITY INVOKER ;
```

- CONTAINS SQL ，表示子程序包含SQL语句，但不包含读或写数据的语句。
- NO SQL ，表示子程序中不包含SQL语句。
- READS SQL DATA ，表示子程序中包含读数据的语句。
- MODIFIES SQL DATA ，表示子程序中包含写数据的语句。
- SQL SECURITY { DEFINER | INVOKER } ，指明谁有权限来执行。
  - DEFINER ，表示只有定义者自己才能够执行。
  - INVOKER ，表示调用者可以执行。
- COMMENT 'string' ，表示注释信息。

#### 5.3 删除

```sql
DROP {PROCEDURE | FUNCTION} [IF EXISTS] 存储过程或函数的名

DROP PROCEDURE CountProc;
DROP FUNCTION CountProc;
```

### 6. 优缺点

#### 6.1 优点

1. ==存储过程可以一次编译多次使用== 。存储过程只在创建时进行编译，之后的使用都不需要重新编译，
   这就提升了 SQL 的执行效率。
2. ==可以减少开发工作量==。将代码 封装 成模块，实际上是编程的核心思想之一，这样可以把复杂的问题
   拆解成不同的模块，然后模块之间可以 重复使用 ，在减少开发工作量的同时，还能保证代码的结构清
   晰。
3. ==存储过程的安全性强==。我们在设定存储过程的时候可以 设置对用户的使用权限 ，这样就和视图一样具
   有较强的安全性。
4. ==可以减少网络传输量==。因为代码封装到存储过程中，每次使用只需要调用存储过程即可，这样就减
   少了网络传输量。
5. ==良好的封装性==。在进行相对复杂的数据库操作时，原本需要使用一条一条的 SQL 语句，可能要连接
   多次数据库才能完成的操作，现在变成了一次存储过程，只需要 连接一次即可 。

#### 6.2 缺点

基于上面这些优点，不少大公司都要求大型项目使用存储过程，比如微软、IBM 等公司。但是国内的阿
里并不推荐开发人员使用存储过程，这是为什么呢？

> 阿里开发规范
> 【强制】禁止使用存储过程，存储过程难以调试和扩展，更没有移植性。

存储过程虽然有诸如上面的好处，但缺点也是很明显的。

1. ==可移植性差==。存储过程不能跨数据库移植，比如在 MySQL、Oracle 和 SQL Server 里编写的存储过
   程，在换成其他数据库时都需要重新编写。
2. ==调试困难==。只有少数 DBMS 支持存储过程的调试。对于复杂的存储过程来说，开发和维护都不容
   易。虽然也有一些第三方工具可以对存储过程进行调试，但要收费。
3. ==存储过程的版本管理很困难==。比如数据表索引发生变化了，可能会导致存储过程失效。我们在开发
   软件的时候往往需要进行版本管理，但是存储过程本身没有版本控制，版本迭代更新的时候很麻烦。
4. ==它不适合高并发的场景==。高并发的场景需要减少数据库的压力，有时数据库会采用分库分表的方
   式，而且对可扩展性要求很高，在这种情况下，存储过程会变得难以维护， 增加数据库的压力 ，显然就
   不适用了。

小结：
存储过程既方便，又有局限性。尽管不同的公司对存储过程的态度不一，但是对于我们开发人员来说，
不论怎样，掌握存储过程都是必备的技能之一。





## Chapter 16 变量、流程控制与游标

### 1. Variable

在MySQL数据库的存储过程和函数中,可以使用变量来存储查询或计算的中间结果数据,或者输出最终的结果数据。
在 MySQL 数据库中,变量分为 系统变量 以及 用户自定义变量 。

#### 1.1 系统变量

##### 1.1.1 系统变量分类

变量由系统定义,不是用户定义,属于 `服务器` 层面。启动MySQL服务,生成MySQL服务实例期间,MySQL将为MySQL服务器内存中的系统变量赋值,这些系统变量定义了当前MySQL服务实例的属性、特征。这些系统变量的值要么是 `编译MySQL时参数`的默认值,要么是 `配置文件 `(例如my.ini等)中的参数值。大家可以通过网址 https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html 查看MySQL文档的系统变量。

系统变量分为全局系统变量(需要添加 `global` 关键字)以及会话系统变量(需要添加 `session` 关键字),有时也把全局系统变量简称为全局变量,有时也把会话系统变量称为local变量。==如果不写,默认会话级别==。静态变量(在 MySQL 服务实例运行期间它们的值不能使用 set 动态修改)属于特殊的全局系统变量。

每一个MySQL客户机成功连接MySQL服务器后,都会产生与之对应的会话。会话期间,MySQL服务实例会在MySQL服务器内存中生成与该会话对应的会话系统变量,这些会话系统变量的初始值是全局系统变量值的复制。如下图:

![image-20220204232954155](/home/venson/.config/Typora/typora-user-images/image-20220204232954155.png)

- 全局系统变量针对于所有会话(连接)有效,但 `不能跨重启`, 重启后恢复默认值. 

- 会话系统变量仅针对于当前会话(连接)有效。会话期间,当前会话对某个会话系统变量值的修改,不会影响其他会话同一个会话系统变量的值。
- 会话1对某个全局系统变量值的修改会导致会话2中同一个全局系统变量值的修改。

在MySQL中有些系统变量只能是全局的,例如 max_connections 用于限制服务器的最大连接数;有些系
统变量作用域既可以是全局又可以是会话,例如 character_set_client 用于设置客户端的字符集;有些系
统变量的作用域只能是当前会话,例如 pseudo_thread_id 用于标记当前会话的 MySQL 连接 ID。

##### 1.1.2 查看系统变量

- ==查看所有或部分系统变量==

```sql
#查看所有全局变量
SHOW GLOBAL VARIABLES;
#查看所有会话变量
SHOW SESSION VARIABLES;
或
SHOW VARIABLES; # 默认session

#查看满足条件的部分系统变量。
SHOW GLOBAL VARIABLES LIKE '%标识符%';
#查看满足条件的部分会话变量
SHOW SESSION VARIABLES LIKE '%标识符%'; # 默认session

SHOW GLOBAL VARIABLES LIKE 'admin_%';
```

- ==查看指定系统变量==

作为 MySQL 编码规范,MySQL 中的系统变量以 两个“@” 开头,其中“@@global”仅用于标记全局系统变
量,“@@session”仅用于标记会话系统变量。“@@”首先标记会话系统变量,如果会话系统变量不存在,
则标记全局系统变量

```sql
#查看指定的系统变量的值
SELECT @@global.变量名;
#查看指定的会话变量的值
SELECT @@session.变量名;
#或者
SELECT @@变量名; #默认session
```

- 修改系统变量的值

有些时候,数据库管理员需要修改系统变量的默认值,以便修改当前会话或者MySQL服务实例的属性、特征。具体方法:
方式1:修改MySQL 配置文件 ,继而修改MySQL系统变量的值(该方法需要重启MySQL服务)
方式2:在MySQL服务运行期间,使用“set”命令重新设置系统变量的值

```sql
#为某个系统变量赋值
#方式1:
SET @@global.变量名=变量值;
#方式2:
SET GLOBAL 变量名=变量值;
#为某个会话变量赋值
#方式1:
SET @@session.变量名=变量值;
#方式2:
SET SESSION 变量名=变量值;

SELECT @@global.autocommit;
SET GLOBAL autocommit=0;
SET @@session.tx_isolation='read-uncommitted';
```

#### 1.2 用户变量

##### 1.2.1 用户变量分类

用户变量是用户自己定义的,作为 MySQL 编码规范,MySQL 中的用户变量以 一个“@” 开头。根据作用
范围不同,又分为 `会话用户变量` 和 `局部变量` 。

- 会话用户变量:作用域和会话变量一样,只对 `当前连接` 会话有效。
- 局部变量:只在 `BEGIN` 和 `END` 语句块中有效。局部变量只能在 `存储过程和函数` 中使用。

##### 1.2.2 会话用户变量

- 变量的定义

```sql
#方式1:“=”或“:=”
SET @用户变量 = 值;
SET @用户变量 := 值;
#方式2:“:=” 或 INTO关键字
SELECT @用户变量 := 表达式 [FROM 等子句];
SELECT 表达式 INTO @用户变量
[FROM 等子句]

SET @a = 1;
SELECT @a;

SELECT @num:= COUNT(*) FROM employees;
SELECT @num;

SELECT AVG(salary) INTO @avgsalary FROM employees;
SELECT @avgsalary;

SELECT @big; #查看某个未声明的变量时,将得到NULL值
```

##### 1.2.3 局部变量

定义:可以使用 DECLARE 语句定义一个局部变量
作用域:仅仅在定义它的 BEGIN ... END 中有效
位置:只能放在 BEGIN ... END 中,而且只能放在第一句

```sql
BEGIN
#声明局部变量
DECLARE 变量名1 变量数据类型 [DEFAULT 变量默认值];
DECLARE 变量名2,变量名3,... 变量数据类型 [DEFAULT 变量默认值];
#为局部变量赋值
SET 变量名1 = 值;
SELECT 值 INTO 变量名2 [FROM 子句];
#查看局部变量的值
SELECT 变量1,变量2,变量3;
END
```

1. 定义变量

```sql
DECLARE 变量名 类型 [default 值];
# 如果没有DEFAULT子句,初始值为NULL

DECLARE myparam INT DEFAULT 100;
```

2. 变量赋值

```sql
SET 变量名:= 值;

SELECT 字段 INTO 变量名 FROM 表;
```

3. 变量使用

```sql
SELECT 变量名;

DELIMITER //
CREATE PROCEDURE set_value()
BEGIN
	DECLARE emp_name VARCHAR(25);
	DECLARE sal DOUBLE(10,2);
	
	SELECT last_name,salary INTO emp_name,sal
	FROM employees
	WHERE employee_id = 102;
	
	SELECT emp_name,sal;
END //
DELIMITER ;
```



### 3. 流程控制

解决复杂问题不可能通过一个 SQL 语句完成，我们需要执行多个 SQL 操作。流程控制语句的作用就是控制存储过程中 SQL 语句的执行顺序，是我们完成复杂操作必不可少的一部分。只要是执行的程序，流程就分为三大类

- 顺序结构 ：程序从上往下依次执行
- 分支结构 ：程序按条件进行选择执行，从两条或多条路径中选择一条执行

循环结构 ：程序满足一定条件下，重复执行一组语句针对于MySQL 的流程控制语句主要有 3 类。==注意：只能用于存储程序。==

- 条件判断语句 ：IF 语句和 CASE 语句
- 循环语句 ：LOOP、WHILE 和 REPEAT 语句
- 跳转语句 ：ITERATE 和 LEAVE 语句

#### 3.1 分支结构之 IF

```sql
IF 表达式1 THEN 操作1
[ELSEIF 表达式2 THEN 操作2]……
[ELSE 操作N]
END IF

IF val IS NULL
THEN SELECT 'val is null';
ELSE SELECT 'val is not null';
END IF;
```

举例2：声明存储过程“update_salary_by_eid1”，定义IN参数emp_id，输入员工编号。判断该员工薪资如果低于8000元并且入职时间超过5年，就涨薪500元；否则就不变。

```sql
DELIMITER //
CREATE PROCEDURE update_salary_by_eid1(IN emp_id INT)
BEGIN
	DECLARE emp_salary DOUBLE;
	DECLARE hire_year DOUBLE;
	SELECT salary INTO emp_salary FROM employees WHERE employee_id = emp_id;
	SELECT DATEDIFF(CURDATE(),hire_date)/365 INTO hire_year	
	FROM employees WHERE employee_id = emp_id;
	
	IF emp_salary < 8000 AND hire_year > 5
		THEN UPDATE employees SET salary = salary + 500 WHERE employee_id = emp_id;
	END IF;
END //

DELIMITER ;
```

举例3：声明存储过程“update_salary_by_eid2”，定义IN参数emp_id，输入员工编号。判断该员工薪资如果低于9000元并且入职时间超过5年，就涨薪500元；否则就涨薪100元。

```SQL
DELIMITER //
CREATE PROCEDURE update_salary_by_eid2(IN emp_id INT)
BEGIN
	DECLARE emp_salary DOUBLE;
	DECLARE hire_year DOUBLE;
	SELECT salary INTO emp_salary FROM employees WHERE employee_id = emp_id;
	SELECT DATEDIFF(CURDATE(),hire_date)/365 INTO hire_year
	FROM employees WHERE employee_id = emp_id;
	IF emp_salary < 8000 AND hire_year > 5
		THEN UPDATE employees SET salary = salary + 500 WHERE employee_id =	emp_id;
	ELSE
		UPDATE employees SET salary = salary + 100 WHERE employee_id = emp_id;
	END IF;
END //
DELIMITER ;
```

举例4：声明存储过程“update_salary_by_eid3”，定义IN参数emp_id，输入员工编号。判断该员工薪资如果低于9000元，就更新薪资为9000元；薪资如果大于等于9000元且低于10000的，但是奖金比例为NULL的，就更新奖金比例为0.01；其他的涨薪100元。

```SQL
DELIMITER //
CREATE PROCEDURE update_salary_by_eid3(IN emp_id INT)
BEGIN
	DECLARE emp_salary DOUBLE;
	DECLARE bonus DECIMAL(3,2);
	SELECT salary INTO emp_salary FROM employees WHERE employee_id = emp_id;
	SELECT commission_pct INTO bonus FROM employees WHERE employee_id = emp_id;
	IF emp_salary < 9000
		THEN UPDATE employees SET salary = 9000 WHERE employee_id = emp_id;
	ELSEIF 	emp_salary < 10000 AND bonus IS NULL
		THEN UPDATE employees SET commission_pct = 0.01 WHERE employee_id = emp_id;
	ELSE
		UPDATE employees SET salary = salary + 100 WHERE employee_id = emp_id;
	END IF;
END //
DELIMITER ;
```

#### 3.2 分支结构之 CASE

CASE 语句的语法结构1：

```SQL
#情况一：类似于switch
CASE 表达式
WHEN 值1 THEN 结果1或语句1(如果是语句，需要加分号)
WHEN 值2 THEN 结果2或语句2(如果是语句，需要加分号)
...
ELSE 结果n或语句n(如果是语句，需要加分号)
END [case]（如果是放在begin end中需要加上case，如果放在select后面不需要）
```

CASE 语句的语法结构2：

```SQL
#情况二：类似于多重if
CASE
WHEN 条件1 THEN 结果1或语句1(如果是语句，需要加分号)
WHEN 条件2 THEN 结果2或语句2(如果是语句，需要加分号)
...
ELSE 结果n或语句n(如果是语句，需要加分号)
END [case]（如果是放在begin end中需要加上case，如果放在select后面不需要）
```



```SQL
CASE val
WHEN 1 THEN SELECT 'val is 1';
WHEN 2 THEN SELECT 'val is 2';
ELSE SELECT 'val is not 1 or 2';
END CASE;


```

使用CASE流程控制语句的第2种格式，判断val是否为空、小于0、大于0或者等于0。

```SQL
CASE
WHEN val IS NULL THEN SELECT 'val is null';
WHEN val < 0 THEN SELECT 'val is less than 0';
WHEN val > 0 THEN SELECT 'val is greater than 0';
ELSE SELECT 'val is 0';
END CASE;
```

举例3：声明存储过程“update_salary_by_eid4”，定义IN参数emp_id，输入员工编号。判断该员工
薪资如果低于9000元，就更新薪资为9000元；薪资大于等于9000元且低于10000的，但是奖金比例
为NULL的，就更新奖金比例为0.01；其他的涨薪100元。

```SQL
DELIMITER //CREATE PROCEDURE update_salary_by_eid4(IN emp_id INT)
BEGIN
	DECLARE emp_sal DOUBLE;
	DECLARE bonus DECIMAL(3,2);
	SELECT salary INTO emp_sal FROM employees WHERE employee_id = emp_id;
	SELECT commission_pct INTO bonus FROM employees WHERE employee_id = emp_id;
	CASE
		WHEN emp_sal<9000
			THEN UPDATE employees SET salary=9000 WHERE employee_id = emp_id;
		WHEN emp_sal<10000 AND bonus IS NULL
			THEN UPDATE employees SET commission_pct=0.01 WHERE employee_id = emp_id;
		ELSE
			UPDATE employees SET salary=salary+100 WHERE employee_id = emp_id;
		END CASE;
END //
DELIMITER ;
```

举例4：声明存储过程update_salary_by_eid5，定义IN参数emp_id，输入员工编号。判断该员工的
入职年限，如果是0年，薪资涨50；如果是1年，薪资涨100；如果是2年，薪资涨200；如果是3年，
薪资涨300；如果是4年，薪资涨400；其他的涨薪500。

```SQL
DELIMITER //

CREATE PROCEDURE update_salary_by_eid(IN emp_id INT)
BEGIN
	DECLARE onboard_year DOUBLE;
		
	SELECT DATEDIFF(CURDATE(), hire_date)/365 INTO onboard_year
    FROM employees WHERE employee_id = emp_id;
    
    CASE ROUND(onboard_year)
    	WHEN 0 THEN UPDATE employees SET salary = salary + 50 WHERE employee_id = emp_id;
    	WHEN 1 THEN UPDATE employees SET salary = salary + 100 WHERE employee_id = emp_id;
    	WHEN 2 THEN UPDATE employees SET salary = salary + 200 WHERE employee_id = emp_id;
    	WHEN 3 THEN UPDATE employees SET salary = salary + 300 WHERE employee_id = emp_id;
    	WHEN 4 THEN UPDATE employees SET salary = salary + 400 WHERE employee_id = emp_id;
    	ELSE UPDATE employees SET salary = salary + 500 WHERE employee_id = emp_id;
	END CASE;
END //
DELIMITER ;
```



#### 3.3 循环结构LOOP

LOOP循环语句用来重复执行某些语句。LOOP内的语句一直重复执行直到循环被退出（使用LEAVE子
句），跳出循环过程。
LOOP语句的基本格式如下：

```sql
[loop_label:] LOOP
循环执行的语句
END LOOP [loop_label]

DECLARE id INT DEFAULT 0;
add_loop:LOOP
	SET id = id +1;
	IF id >= 10 THEN LEAVE add_loop;
	END IF;
END LOOP add_loop;
```

举例2：当市场环境变好时，公司为了奖励大家，决定给大家涨工资。声明存储过程“update_salary_loop()”，声明OUT参数num，输出循环次数。存储过程中实现循环给大家涨薪，薪资涨为原来的1.1倍。直到全公司的平均薪资达到12000结束。并统计循环次数。

```sql
DELIMITER //

CREATE PROCEDURE update_salary_loop(OUT loop_count INT)
BEGIN
	DECLARE avg_sal DOUBLE;
	SET loop_count = 0;
	SELECT AVG(salary) INTO avg_sal FROM employees;
	label_loop:LOOP
		IF avg_sal > 12000 THEN LEAVE label_loop;# 借助IF LEAVE跳出循环
		END IF;
		UPDATE employees SET salary = salary * 1.1;
		SET loop_count = loop_count +1;
		SELECT AVG(salary) INTO avg_sal FROM employees; 
	END LOOP label_loop;
		
END //
DELIMITER ;

```

#### 3.4 WHILE



```sql
[while_label:] WHILE 循环条件 DO
循环体
END WHILE [while_label];
```

```sql
DELIMITER //
CREATE PROCEDURE test_while()
BEGIN
	DECLARE i INT DEFAULT 0;
	WHILE i < 10 DO
		SET i = i  +1;
	END WHILE;
	SELECT i;
END //
DELIMITER ;

DELIMITER //
CREATE PROCEDURE update_salary_while(OUT num INT)
BEGIN
	DECLARE avg_sal DOUBLE;
	SET num = 0;
	SELECT AVG(salary) INTO avg_sal FROM employees;
	WHILE avg_sal > 5000 DO
		UPDATE employees SET salary  = salary *0.9;
		SET num = num +1 ;
		SELECT AVG(salary) INTO avg_sal FROM employees;
	END WHILE;
END //
DELIMITER;
```

#### 3.5 REPEAT

REPEAT语句创建一个带条件判断的循环过程。与WHILE循环不同的是，REPEAT 循环首先会执行一次循环，然后在 UNTIL 中进行表达式的判断，如果满足条件就退出，即 END REPEAT；如果条件不满足，则会就继续执行循环，直到满足退出条件为止。
REPEAT语句的基本格式如下：

```sql
[repeat_label:] REPEAT
循环体的语句
UNTIL 结束循环的条件表达式
END REPEAT [repeat_label]

DELIMITER //
CREATE PROCEDURE test_repeat()
BEGIN
	DECLARE i INT DEFAULT 0;
	REPEAT
		SET i = i + 1;
		UNTIL i >= 10  # 无分号
	END REPEAT;
	# 至少执行一次
	SELECT i;
END //
DELIMITER ;
```

## Chaper 17 触发器

在实际开发中，我们经常会遇到这样的情况：有 2 个或者多个相互关联的表，如 `商品信息` 和 `库存信息` 分别存放在 2 个不同的数据表中，我们在添加一条新商品记录的时候，为了保证数据的完整性，必须同时在库存表中添加一条库存记录。
这样一来，我们就必须把这两个关联的操作步骤写到程序里面，而且要用 `事务` 包裹起来，确保这两个操作成为一个 `原子操作` ，要么全部执行，要么全部不执行。要是遇到特殊情况，可能还需要对数据进行手动维护，这样就很 `容易忘记其中的一步` ，导致数据缺失。
这个时候，咱们可以使用触发器。**你可以创建一个触发器，让商品信息数据的插入操作自动触发库存数据的插入操作**。这样一来，就不用担心因为忘记添加库存数据而导致的数据缺失了。



#### 1. 触发器概述

MySQL从 5.0.2 版本开始支持触发器。MySQL的触发器和存储过程一样，都是嵌入到MySQL服务器的一段程序。
触发器是由 事件来触发 某个操作，这些事件包括 INSERT 、 UPDATE 、 DELETE 事件。所谓事件就是指用户的动作或者触发某项行为。如果定义了触发程序，当数据库执行这些语句时候，就相当于事件发生了，就会 自动 激发触发器执行相应的操作。
当对数据表中的数据执行插入、更新和删除操作，需要自动执行一些数据库逻辑时，可以使用触发器来实现

### 2. 触发器的创建

创建触发器的语法结构是：

```sql
CREATE TRIGGER 触发器名称
{BEFORE|AFTER} {INSERT|UPDATE|DELETE} ON 表名
FOR EACH ROW
触发器执行的语句块;
```

说明：

- 表名 ：表示触发器监控的对象。
- BEFORE|AFTER ：表示触发的时间。BEFORE 表示在事件之前触发；AFTER 表示在事件之后触发。
- INSERT|UPDATE|DELETE ：表示触发的事件。
  - INSERT 表示插入记录时触发；
  - UPDATE 表示更新记录时触发；
  - DELETE 表示删除记录时触发。
- 触发器执行的语句块 ：可以是单条SQL语句，也可以是由BEGIN…END结构组成的复合语句块。



#### 2.2 代码举例

```sql
CREATE TABLE test_trigger (
	id INT PRIMARY KEY AUTO_INCREMENT,
    t_note VARCHAR(30)
);

CREATE TABLE test_trigger_log(
	id INT PRIMARY KEY AUTO_INCREMENT,
	t_log VARCHAR(30)
);
```

2、创建触发器：创建名称为before_insert的触发器，向test_trigger数据表插入数据之前，向
test_trigger_log数据表中插入before_insert的日志信息。

```sql
DELIMITER //
CREATE TRIGGER before_insert
BEFORE INSERT ON test_trigger
FOR EACH ROW
BEGIN 
	INSERT INTO test_trigger_log (t_log)
	VALUES('before insert ...')
END //
DELIMITER ;
```

3、向test_trigger数据表中插入数据

```sql
INSERT INTO test_trigger (t_note) VALUES ('测试 BEFORE INSERT 触发器');
```

4、查看test_trigger_log数据表中的数据

```sql
mysql> SELECT * FROM test_trigger_log;
+----+---------------+
| id | t_log		 |
+----+---------------+
| 1  | before_insert |
+----+---------------+
1 row in set (0.00 sec)
```

举例2：
1、创建名称为after_insert的触发器，向test_trigger数据表插入数据之后，向test_trigger_log数据表中插入after_insert的日志信息。

```sql
DELIMITER //
CREATE TRIGGER after_insert
AFTER INSERT ON test_trigger
FOR EACH ROW
BEGIN
	INSERT INTO test_trigger_log 
	VALUES ('after_insert');
END //
DELIMITER ;
```

举例3：定义触发器“salary_check_trigger”，基于员工表“employees”的INSERT事件，在INSERT之前检查将要添加的新员工薪资是否大于他领导的薪资，如果大于领导薪资，则报sqlstate_value为'HY000'的错误，从而使得添加失败。

```sql
DELIMITER //
CREATE TRIGGER salary_check_trigger
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
	DECLARE mgrsalary DOUBLE;
	# new 代表INSERT的新条目
	SELECT salary INTO mgrsalary from employees where employee_id = NEW.manager_id;
    
    IF NEW.salary > mgrsalary THEN
    	# 产生错误
    	SIGNAL SQLSTATE 'HY000' SET MESSAGE_TEXT = '薪资高于领导薪资错误';
     END IF;
END //
DELIMITER;
```




















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

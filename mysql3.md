# Chapter 8

## 1. 索引的声明与使用

### 1.1 索引的分类

### 1.2 创建索引

#### 1.2.1 创建表的时候创建索引

##### 1.2.1.1 创建普通索引

##### 1.2.1.2 创建唯一索引

##### 1.2.1.3 主键索引

##### 1.2.1.4 创建单列索引

##### 1.2.1.5 创建组合索引

##### 1.2.1.6 创建全文索引

##### 1.2.1.7 创建空间索引

#### 1.2.2 在已存在的表上创建索引

##### 1.2.2.1使用ALTER TABLE语句创建索引

##### 1.2.2.2 使用CREATE INDEX 创建索引

### 1.3 删除索引

#### 1.3.1 使用ALTER TABLE 删除索引

#### 1.3.2 使用DROP INDEX删除索引

## 2. MySQL8.0 索引新特性

### 2.1 支持降序索引

### 2.2 隐藏索引

在MySQL 5.7版本及之前，只能通过显式的方式删除索引。此时，如果发现删除索引后出现错误，又只能通过显式创建索引的方式将删除的索引创建回来。如果数据表中的数据量非常大，或者数据表本身比较大，这种操作就会消耗系统过多的资源，操作成本非常高。

从MySQL 8.x开始支持 隐藏索引（invisible indexes） ，只需要将待删除的索引设置为隐藏索引，使查询优化器不再使用这个索引（即使使用force index（强制使用索引），优化器也不会使用该索引），确认将索引设置为隐藏索引后系统不受任何响应，就可以彻底删除索引。 这种通过先将索引设置为隐藏索 引，再删除索引的方式就是软删除 。

#### 2.2.1 创建表时直接创建

在MySQL中创建隐藏索引通过SQL语句INVISIBLE来实现，其语法形式如下：

```sql
CREATE TABLE tablename( 
  propname1 type1[CONSTRAINT1], 
  propname2 type2[CONSTRAINT2],
  ……
  propnamen typen, 
  INDEX [indexname](propname1 [(length)]) INVISIBLE 
);
```

上述语句比普通索引多了一个关键字INVISIBLE，用来标记索引为不可见索引。

#### 2.2.2 在已经存在的表上创建

``` sql
CREATE INDEX indexname
ON tablename(propname[(length)]) INVISIBLE;
```

#### 2.2.3 通过ALTER TABLE语句创建

```sql
ALTER TABLE tablename
ADD INDEX indexname(propname [(length)]) INVISIBLE;
```

#### 2.2.4 切换索引可见状态

```sql
ALTER TABLE tablename ALTER INDEX index_name INVISIBLE;
ALTER TABLE tablename ALTER INDEX index_name VISIBLE;
```

>注意 当索引被隐藏时，它的内容仍然是和正常索引一样实时更新的。如果一个索引需要长期被隐藏，那么可以将其删除，因为索引的存在会影响插入、更新和删除的性能。

## 3. 索引的设计原则

### 3.1 数据准备

创建库、创建表

```sql
CREATE DATABASE atguigudb1; 
USE atguigudb1; 
#1.创建学生表和课程表 
CREATE TABLE `student_info` (
  `id` INT(11) NOT NULL AUTO_INCREMENT, 
  `student_id` INT NOT NULL , 
  `name` VARCHAR(20) DEFAULT NULL, 
  `course_id` INT NOT NULL , 
  `class_id` INT(11) DEFAULT NULL, 
  `create_time` DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP, 
  PRIMARY KEY (`id`) 
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
  
CREATE TABLE `course` (
  `id` INT(11) NOT NULL AUTO_INCREMENT, 
  `course_id` INT NOT NULL , 
  `course_name` VARCHAR(40) DEFAULT NULL, 
  PRIMARY KEY (`id`) 
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

创建模拟数据必需的存储函数

```sql
DELIMITER //
CREATE FUNCTION rand_string(n INT)
	RETURNS VARCHAR(255)
BEGIN 
	DECLARE chars_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFJHIJKLMNOPQRSTUVWXYZ';
	DECLAre return_str VARCHAR(255) DEFAULT ''; 
	DECLARE i INT DEFAULT 0;
	WHILE i < n DO
		SET return_str = CONCAT(return_str, SUBSTRING(chars_str, FLOOR(1+RAND()* 52), 1));
		SET i = i + 1;
	END WHILE;
	RETURN return_str;
END //
DELIMITER ;
```

```sql
DELIMITER //
CREATE FUNCTION rand_num(from_num INT, to_num INT)
RETURNS INT(11)
BEGIN
	DECLARE i INT DEFAULT 0;
	SET i = FLOOR(from_num + RAND()*(to_num - from_num +1));
	RETURN i;
END //
DELIMITER ;
```

创建函数，假如报错：

This function has none of DETERMINISTIC......

由于开启过慢查询日志bin-log, 我们就必须为我们的function指定一个参数。

主从复制，主机会将写操作记录在bin-log日志中。从机读取bin-log日志，执行语句来同步数据。如果使用函数来操作数据，会导致从机和主键操作时间不一致。所以，默认情况下，mysql不开启创建函数设置。

查看mysql是否允许创建函数：

```sql
show variables like 'log_bin_trust_function_creators'
```

命令开启：允许创建函数设置：

```sql
set global log_bin_trust_function_creators=1; # 不加global只是当前窗口有效。
```

mysqld重启，上述参数又会消失。永久方法：

windows下：my.ini[mysqld]加上：

linux下：/etc/my.cnf下my.cnf[mysqld]加上：

```sql
log_bin_trust_function_creators=1 
```

第3步：创建插入模拟数据的存储过程

```sql
# 存储过程1：创建插入课程表存储过程
DELIMITER //
CREATE PROCEDURE insert_course(max_num INT)
BEGIN
	DECLARE i INT DEFAULT 0;
	SET autocommit = 0; # set manual commit
	REPEAT					#loop
		SET i = i + 1;
		INSERT INTO course(course_id, course_name) 
		VALUES(rand_num(1000, 10100), rand_string(6));
		UNTIL i = max_num
	END REPEAT;
	COMMIT; # manual commit
END //
DELIMITER ;
```

```sql
# 存储过程2：创建插入学生信息表存储过程
DELIMITER //
CREATE PROCEDURE insert_stu(max_num INT)
BEGIN
	DECLARE i INT DEFAULT 0;
	SET autocommit = 0;
	REPEAT 
		SET i = i + 1;
		INSERT INTO student_info(course_id, class_id, student_id, NAME) 
		VALUES(rand_num(10000, 10100), rand_num(10000, 102000), rand_num(1, 200000), rand_string(6));
		UNTIL i = max_num
		END REPEAT;
		COMMIT;
END //
DELIMITER ;

```

第4步：调用存储过程

```sql
CALL insert_course(100); 
CALL insert_stu(1000000);
```



### 3.2 那些情况适合创建索引

1. **字段的数值有唯一性的限制**

> **业务上具有唯一特性的字段**，即使是组合字段，也必须建成唯一索引。（来源：Alibaba）说明：不要以为唯一索引影响了 insert 速度，这个速度损耗可以忽略，但提高查找速度是明显的。

2. **频繁作为 WHERE 查询条件的字段**

某个字段在SELECT语句的 WHERE 条件中经常被使用到，那么就需要给这个字段创建索引了。尤其是在数据量大的情况下，创建普通索引就可以大幅提升数据查询的效率。

比如student_info数据表（含100万条数据），假设我们想要查询 student_id=123110 的用户信息。

```sql
# duration 0.271sec before index
show index from student_info;
select course_id, class_id , name, create_time, student_id
FROM student_info
where student_id = 123110;

# add index for COLUMN 'student_id'
alter table student_info
add index idx_sid(student_id);

# duration 0.0012sec after index
select course_id, class_id , name, create_time, student_id
FROM student_info
where student_id = 123110;
```



3. **经常 GROUP BY 和 ORDER BY 的列**

索引就是让数据按照某种顺序进行存储或检索，因此当我们使用 GROUP BY 对数据进行分组查询，或者使用 ORDER BY 对数据进行排序的时候，就需要 对分组或者排序的字段进行索引 。如果待排序的列有多个，那么可以在这些列上建立 组合索引 。 (20倍差异)

```sql
# druation 0.041sec with index
SELECT student_id, count(*) AS num
FROM  student_info
GROUP by student_id 
LIMIT 100;
# drop column student_id index
ALTER table student_info
DROP index idx_sid;
# duration 0.713 sec after drop index
SELECT student_id, count(*) AS num
FROM  student_info
GROUP by student_id 
LIMIT 100;
```

GROUP BY 和ORDER BY 同时使用时, 联合索引按group, order 的顺序, 如order常用降序, 则可在联合索引中使用desc. 

4. **UPDATE、DELETE 的 WHERE 条件列**

对数据按照某个条件进行查询后再进行 UPDATE 或 DELETE 的操作，如果对 WHERE 字段创建了索引，就能大幅提升效率。原理是因为我们需要先根据 WHERE 条件列检索出来这条记录，然后再对它进行更新或删除。**如果进行更新的时候，更新的字段是非索引字段，提升的效率会更明显，这是因为非索引字段更新不需要对索引进行维护。**

5. **DISTINCT 字段需要创建索引**

有时候我们需要对某个字段进行去重，使用 DISTINCT，那么对这个字段创建索引，也会提升查询效率。比如，我们想要查询课程表中不同的 student_id 都有哪些，如果我们没有对 student_id 创建索引，执行SQL 语句：

```sql
SELECT DISTINCT(student_id) FROM `student_info`;
```

运行结果（600637 条记录，运行时间 0.683s ）：

如果我们对 student_id 创建索引，再执行 SQL 语句：

```sql
SELECT DISTINCT(student_id) FROM `student_info`;
```

运行结果（600637 条记录，运行时间 0.010s ）：

你能看到 SQL 查询效率有了提升，同时显示出来的 student_id 还是按照 `递增的顺序` 进行展示的。这是因为索引会对数据按照某种顺序进行排序，所以在去重的时候也会快很多。

6. **多表JOIN 连接操作时，创建索引注意事项**

首先， `连接表的数量尽量不要超过 3 张` ，因为每增加一张表就相当于增加了一次嵌套的循环，数量级增长会非常快，严重影响查询的效率。

其次， `对 WHERE 条件创建索引` ，因为 WHERE 才是对数据条件的过滤。如果在数据量非常大的情况下，没有 WHERE 条件过滤是非常可怕的。

最后， `对用于连接的字段创建索引` ，并且该字段在多张表中的 `类型必须一致 `。比如 course_id 在 student_info 表和 course 表中都为 int(11) 类型，而不能一个为 int 另一个为 varchar 类型。

举个例子，如果我们只对 student_id 创建索引，执行 SQL 语句：

```sql
SELECT s.course_id, name, s.student_id, c.course_name
FROM student_info s
JOIN course c
ON student_info.course_id = course.course_id 
WHERE name = '462eed7ac6e791292a79';
```

运行结果（1 条数据，运行时间 0.189s ）：

这里我们对 name 创建索引，再执行上面的 SQL 语句，运行时间为 0.002s 。 

7. **使用列的类型小的创建索引** 

类型小, 指的就是该类表示的数据范围的大小.

以整数为例, 有TINYINT、MEDIUMINT、INT、BIGINT

- 数据类型越小, 在查询是进行的比较操作越快
- 数据类型越小, 索引占用的空间就越小, 在一个数据页内就可以放下更多的数据记录, 从而减少磁盘I/O带来的性能损失, 也就以为可以把更多的数据页缓存在内存中,从而提升速度. 

8. **使用字符串前缀创建索引**

针对字符串创建索引是会有两个问题:

- B+树索引中的记录需要把该列完整的字符串存储起来,更费时. 而且字符串越长, 在索引中占用的存储空间越大. 
- 如果B+书中索引列存储的字符串很长, 那么在做字符串比较时会占用更多的时间. 

我们可以截取字符串前面一部分内容建立索引, 这个就叫前置索引. 

创建一张商户表，因为地址字段比较长，在地址字段上建立前缀索引

```sql
create table shop(address varchar(120) not null); 
alter table shop add index(address(12));
```

问题是，截取多少呢？截取得多了，达不到节省索引存储空间的目的；截取得少了，重复内容太多，字

段的散列度(选择性)会降低。**怎么计算不同的长度的选择性呢？**

先看一下字段在全部数据中的选择度：

```sql
select count(distinct address) / count(*) from shop;
```

通过不同长度去计算，与全表的选择性对比：

```sql
-- 截取前10个字符的选择度 
select count(distinct left(address,10)) / count(*) as sub10,
-- 截取前15个字符的选择度 
count(distinct left(address,15)) / count(*) as sub11, 
-- 截取前20个字符的选择度 
count(distinct left(address,20)) / count(*) as sub12, 
-- 截取前25个字符的选择度
count(distinct left(address,25)) / count(*) as sub13 
from shop;
```

**引申另一个问题：索引列前缀对排序的影响**

如果使用了索引列前缀, 比方说千遍只把address列的前12个字符放到了二级索引中, 下边这个查询:

```sql
SELECT * FROM shop
ORDER BY address
LIMIT 12;
```

因为二级索引中不包含完整的address列信息, 所以无法对前12个字符相同, 后面的字符不同的记录进行排序. 这就是使用索引列前缀的方式无法支持使用索引排序, 只能使用文件排序(filesort). 



**拓展：Alibaba《Java开发手册》**

【 强制 】在 varchar 字段上建立索引时，必须指定索引长度，没必要对全字段建立索引，根据实际文本区分度决定索引长度。

说明：索引的长度与区分度是一对矛盾体，一般对字符串类型数据，长度为 20 的索引，区分度会 `高达 90%` 以上 ，可以使用 count(distinct left(列名, 索引长度))/count(*)的区分度来确定。

9. **区分度高(散列性高)的列适合作为索引** 

列的基数指的是某一列中不重复数据的个数, 比方说某列值包含2, 5, 8, 2, 5, 8, 2, 5, 8, 虽然有九条记录, 但该列的基数确是3. 也就是说, 在记录行数一定的情况下, 列的基数越大, 该列中的值越分散, 列的基数越小, 该列中的值越集中. 这个列的基数指标非常重要, 直接影响我们能否有效的利用索引, 最好为列的基数大的列建立索引, 为基数太小的列建立索引效果可能不好. 

```sql
select count(distinct a)/count(*) from tablename;
```

一般区分度大于33%就可算是比较高效的索引.  

10. **使用最频繁的列放到联合索引的左侧**

这样也可以较少的建立一些索引。同时，由于"最左前缀原则"，可以增加联合索引的使用率。

```sql
select *
FROM student_info
WHERE student_id = 10013 AND course_id = 100;
#越常用, 越放在左面
```

11. **在多个字段都要创建索引的情况下，联合索引优于单值索引**

类似上例 GROUP BY 和ORDER BY. 避免多个单值索引 

### 3.3 **限制索引的数目** 

索引也不是越多越好, 限制每张表上的索引数量, 建议单张表索引数量不超过6个. 

1. 每个索引都要占用磁盘空间
2. 索引影响INSERT、DELETE、UPDATE等语句的性能, 因为表中的数据更改的同时, 索引也必须进行调整和更新, 会造成负担. 
3. 优化器在选择如果进行优化时, 对每一个可以用到的索引来进行评估. 如果同时有多个索引都可以用于查询, 会增加优化器生成执行计划的时间, 降低查询性能. 

### 3.4 **哪些情况不适合创建索引** 

1. **在where中使用不到的字段，不要设置索引** 

2. **数据量小的表最好不要使用索引**

   举例, 表1

```sql
CREATE TABLE t_without_index(
  a INT PRIMARY KEY AUTO_INCREMENT,
  b INT 
);
```

```sql
#创建存储过程 
DELIMITER // 
CREATE PROCEDURE t_wout_insert() 
BEGIN
	DECLARE i INT DEFAULT 1;
  WHILE i <= 900 
  DO
  	INSERT INTO t_without_index(b) SELECT RAND()*10000;
    SET i = i + 1;
  END WHILE;
	COMMIT; 
END //
DELIMITER ; 
#调用 
CALL t_wout_insert();
```

表2

```sql
CREATE TABLE t_with_index( 
  a INT PRIMARY KEY AUTO_INCREMENT, 
  b INT, 
  INDEX idx_b(b) 
);
```

```sql
#创建存储过程 
DELIMITER // 
CREATE PROCEDURE t_with_insert() 
BEGIN
	DECLARE i INT DEFAULT 1; 
	WHILE i <= 900 
	DO 
		INSERT INTO t_with_index(b) SELECT RAND()*10000; 
		SET i = i + 1; 
	END WHILE; 
	COMMIT; 
END // 
DELIMITER ; 
#调用 
CALL t_with_insert();
```

```sql
mysql> select * from t_without_index where b = 9879; 
+------+------+
| a    | b    |
+------+------+
| 1242 | 9879 |
+------+------+
1 row in set (0.00 sec) mysql> select * from t_with_index where b = 9879; +-----+------+
| a   | b    | 
+-----+------+
| 112 | 9879 | 
+-----+------+
1 row in set (0.00 sec)
```

你能看到运行结果相同，但是在数据量不大的情况下，索引就发挥不出作用了。

> 结论：在数据表中的数据行数比较少的情况下，比如不到 1000 行，是不需要创建索引的。

3. **有大量重复数据的列上不要建立索引**

如性别, 建立索引反而降低数据更新速度

举例1：要在 100 万行数据中查找其中的 50 万行（比如性别为男的数据），一旦创建了索引，你需要先访问 50 万次索引，然后再访问 50 万次数据表，这样加起来的开销比不使用索引可能还要大。

举例2：假设有一个学生表，学生总数为 100 万人，男性只有 10 个人，也就是占总人口的 10 万分之 1。学生表 student_gender 结构如下。其中数据表中的 student_gender 字段取值为 0 或 1，0 代表女性，1 代表男性。

```sql
CREATE TABLE student_gender( 
  student_id INT(11) NOT NULL, 
  student_name VARCHAR(50) NOT NULL, 
  student_gender TINYINT(1) NOT NULL, 
  PRIMARY KEY(student_id) 
)ENGINE = INNODB;
```

如果我们要筛选出这个学生表中的男性，可以使用：

```sql
SELECT * FROM student_gender WHERE student_gender = 1
```

> 结论：当数据重复度大，比如 `高于 10% `的时候，也不需要对这个字段使用索引。

4. **避免对经常更新的表创建过多的索引** 

   频繁更新的字段不要创建索引.

   频繁更新的表不要创建过多的索引.

5. **不建议用无序的值作为索引**

​	例如身份证、UUID(在索引比较时需要转为ASCII，并且插入时可能造成页分裂)、MD5、HASH、无序长字符串等。

6. **删除不再使用或者很少使用的索引** 

   表中的数据被大量更新, 或者数据的使用方式被改变后, 原有的一些索引可能不再需要, DBA应当定期找出这些索引并删除, 从而减小索引对数据更新操作的影响. 

7. **不要定义冗余或重复的索引** 

**① 冗余索引**

举例：建表语句如下

```sql
CREATE TABLE person_info( 
  id INT UNSIGNED NOT NULL AUTO_INCREMENT, 
  name VARCHAR(100) NOT NULL, 
  birthday DATE NOT NULL, 
  phone_number CHAR(11) NOT NULL, 
  country varchar(100) NOT NULL, 
  PRIMARY KEY (id), 
  KEY idx_name_birthday_phone_number (name(10),birthday, phone_number), 
  KEY idx_name (name(10)) 
);
```

我们知道，通过 idx_name_birthday_phone_number 索引就可以对 name 列进行快速搜索，再创建一个专门针对 name 列的索引就算是一个 冗余索引 ，维护这个索引只会增加维护的成本，并不会对搜索有什么好处。

**② 重复索引**

另一种情况，我们可能会对某个列 重复建立索引 ，比方说这样：

```sql
CREATE TABLE repeat_index_demo ( 
  col1 INT PRIMARY KEY, 
  col2 INT, 
  UNIQUE uk_idx_c1 (col1), 
  INDEX idx_c1 (col1) 
);
```

我们看到，col1 既是主键、又给它定义为一个唯一索引，还给它定义了一个普通索引，可是主键本身就会生成聚簇索引，所以定义的唯一索引和普通索引是重复的，这种情况要避免。

# Chapter 9 

## 3. **统计**SQL**的查询成本：**last_query_cost

```sql
CREATE TABLE `student_info` (
  `id` INT(11) NOT NULL AUTO_INCREMENT,
  `student_id` INT NOT NULL , 
  `name` VARCHAR(20) DEFAULT NULL, 
  `course_id` INT NOT NULL , 
  `class_id` INT(11) DEFAULT NULL, 
  `create_time` DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP, 
  PRIMARY KEY (`id`) 
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

如果我们想要查询 id=900001 的记录，然后看下查询成本，我们可以直接在聚簇索引上进行查找：

```sql
SELECT student_id, class_id, NAME, create_time FROM student_info
WHERE id = 900001;
```

运行结果（1 条记录，运行时间为 0.042s ）

然后再看下查询优化器的成本，实际上我们只需要检索一个页即可：

```sql
mysql> SHOW STATUS LIKE 'last_query_cost'; 
+-----------------+----------+
| Variable_name   | Value    | 
+-----------------+----------+
| Last_query_cost | 1.000000 | 
+-----------------+----------+
```

如果我们想要查询 id 在 900001 到 9000100 之间的学生记录呢？

```sql
SELECT student_id, class_id, NAME, create_time FROM student_info
WHERE id BETWEEN 900001 AND 900100;
```

运行结果（100 条记录，运行时间为 0.046s ）：

然后再看下查询优化器的成本，这时我们大概需要进行 20 个页的查询。

```sql
mysql> SHOW STATUS LIKE 'last_query_cost'; 
+-----------------+-----------+
| Variable_name   | Value     | 
+-----------------+-----------+
| Last_query_cost | 21.134453 | 
+-----------------+-----------+
```

你能看到页的数量是刚才的 20 倍，但是查询的效率并没有明显的变化，实际上这两个 SQL 查询的时间基本上一样，就是因为采用了顺序读取的方式将页面一次性加载到缓冲池中，然后再进行查找。虽然 页 数量（last_query_cost）增加了不少 ，但是通过缓冲池的机制，并 没有增加多少查询时间 。

**使用场景：**它对于比较开销是非常有用的，特别是我们有好几种查询方式可选的时候。

> SQL查询是个动态过程, 
>
> 1. 位置决定效率. 如果页在数据池的缓冲池中, 那么效率是最高的, 否则需要从内存或者硬盘中读取, 如果针对单个页来说 如果页存在于内存中, 会比在磁盘中读取效率高很多.
> 2. 批量决定效率, 如果我们从磁盘中对单一页进行随机读, 那么效率是很低的(差不多10ms), 而采用顺序读取的方式,批量对页进行读取, 平均一页的读取效率就会提升很多, 甚至快于单个页面在内存中的随机读取.
>
> 所以说遇到I/O并不用担心, 方法找对了, 效率还是很高的, 我们首先要考虑数据存放的位置, 如果是经常使用的数据就要尽量放到缓冲池中, 其次我们可以充分利用磁盘的吞吐能力, 一次性批量读取数据, 这样单个页的读取效率也就得到了提升. 

## 4. 定位执行慢的 SQL：慢查询日志

### 4.1 **开启慢查询日志参数** 

1. **开启**slow_query_log 

   ```bash
   mysql > set global slow_query_log='ON';
   ```

   

然后我们再来查看下慢查询日志是否开启，以及慢查询日志文件的位置：

```sql
mysql> show variables like '%slow_query_log%';
+---------------------+------------------------------------------+
| Variable_name       | Value                                    |
+---------------------+------------------------------------------+
| slow_query_log      | ON                                       |               
| slow_query_log_file | /usr/local/var/mysql/venson-Pro-slow.log |
+---------------------+------------------------------------------+
2 rows in set (0.00 sec)
```

2. **修改long_query_time阈值**

接下来我们来看下慢查询的时间阈值设置，使用如下命令：

```sql
#测试发现：设置global的方式对当前session的long_query_time失效。对新连接的客户端有效。所以可以一并 执行下述语句 
mysql > set global long_query_time = 1; 
mysql> show global variables like '%long_query_time%'; 
mysql> set long_query_time=1; 
mysql> show variables like '%long_query_time%';
```

### 4.2 查看慢查询数目

查询当前系统中有多少条慢查询记录

```sql
SHOW GLOBAL STATUS LIKE '%Slow_queries%'; 
```

### 4.3 案例演示

**步骤1. 建表**

```sql
CREATE TABLE `student` (
  `id` INT(11) NOT NULL AUTO_INCREMENT, 
  `stuno` INT NOT NULL , 
  `name` VARCHAR(20) DEFAULT NULL, 
  `age` INT(3) DEFAULT NULL, 
  `classId` INT(11) DEFAULT NULL, 
  PRIMARY KEY (`id`) 
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

**步骤2：设置参数log_bin_trust_function_creators**

创建函数，假如报错：

```sql
set global log_bin_trust_function_creators=1; # 不加global只是当前窗口有效。
```

**步骤3：创建函数**

```sql
DELIMITER // 
CREATE FUNCTION rand_string(n INT) 
RETURNS VARCHAR(255) #该函数会返回一个字符串 
BEGIN
	DECLARE chars_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFJHIJKLMNOPQRSTUVWXYZ'; 
	DECLARE return_str VARCHAR(255) DEFAULT ''; 
	DECLARE i INT DEFAULT 0; 
		WHILE i < n 
		DO
    	SET return_str =CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1+RAND()*52),1)); 
    	SET i = i + 1; 
    END WHILE; 
    RETURN return_str; 
 END // 
 DELIMITER ; #测试 SELECT rand_string(10);
```

产生随机数值：（同上一章）

```sql
DELIMITER // 
CREATE FUNCTION rand_num (from_num INT ,to_num INT) 
RETURNS INT(11) 
BEGIN 
	DECLARE i INT DEFAULT 0; 
	SET i = FLOOR(from_num +RAND()*(to_num - from_num+1)) ; 
	RETURN i; 
END // 
DELIMITER ; #测试： SELECT rand_num(10,100);
```

**步骤4：创建存储过程**

```sql
DELIMITER // 
CREATE PROCEDURE insert_stu1( START INT , max_num INT ) 
BEGIN 
	DECLARE i INT DEFAULT 0; 
	SET autocommit = 0; #设置手动提交事务 
	REPEAT #循环 
		SET i = i + 1; #赋值 
		INSERT INTO student (stuno, NAME ,age ,classId ) 
		VALUES ((START+i),rand_string(6),rand_num(10,100),rand_num(10,1000)); 	UNTIL i = max_num 
	END REPEAT; 
	COMMIT; #提交事务 
END // 
DELIMITER ;
```

**步骤**5**：调用存储过程**

```sql
#调用刚刚写好的函数, 4000000条记录,从100001号开始 
CALL insert_stu1(100001,4000000);
```

### 4.4 测试及分析

1. **测试**

```sql
mysql> SELECT * FROM student WHERE stuno = 3455655;
+---------+---------+--------+------+---------+
| id      | stuno   | name   | age  | classId | 
+---------+---------+--------+------+---------+
| 3523633 | 3455655 | oQmLUr | 19   | 39      | 
+---------+---------+--------+------+---------+
1 row in set (2.09 sec) mysql> SELECT * FROM student WHERE name = 'oQmLUr';
+---------+---------+--------+------+---------+
| id      | stuno   | name   | age  | classId | 
+---------+---------+--------+------+---------+
| 1154002 | 1243200 | OQMlUR | 266  | 28      | 
| 1405708 | 1437740 | OQMlUR | 245  | 439     | 
| 1748070 | 1680092 | OQMlUR | 240  | 414     | 
| 2119892 | 2051914 | oQmLUr | 17   | 32      | 
| 2893154 | 2825176 | OQMlUR | 245  | 435     | 
| 3523633 | 3455655 | oQmLUr | 19   | 39      | 
+---------+---------+--------+------+---------+
6 rows in set (2.39 sec)
```

从上面的结果可以看出来，查询学生编号为“3455655”的学生信息花费时间为2.09秒。查询学生姓名为“oQmLUr”的学生信息花费时间为2.39秒。已经达到了秒的数量级，说明目前查询效率是比较低的，下面的小节我们分析一下原因。

2. **分析**

```sql
show status like 'slow_queries';
```

### 4.5 慢查询日志分析工具：mysqldumpslow

在生产环境中，如果要手工分析日志，查找、分析SQL，显然是个体力活，MySQL提供了日志分析工具mysqldumpslow 。查看mysqldumpslow的帮助信息

```bash
➜  ~ mysqldumpslow --help
Usage: mysqldumpslow [ OPTS... ] [ LOGS... ]

Parse and summarize the MySQL slow query log. Options are

  --verbose    verbose
  --debug      debug
  --help       write this text to standard output

  -v           verbose
  -d           debug
  -s ORDER     what to sort by (al, at, ar, c, l, r, t), 'at' is default
                al: average lock time
                ar: average rows sent
                at: average query time
                 c: count
                 l: lock time
                 r: rows sent
                 t: query time
  -r           reverse the sort order (largest last instead of first)
  -t NUM       just show the top n queries
  -a           don't abstract all numbers to N and strings to 'S'
  -n NUM       abstract numbers with at least n digits within names
  -g PATTERN   grep: only consider stmts that include this string
  -h HOSTNAME  hostname of db server for *-slow.log filename (can be wildcard),
               default is '*', i.e. match all
  -i NAME      name of server instance (if using mysql.server startup script)
  -l           don't subtract lock time from total time
```

mysqldumpslow 命令的具体参数如下：

- -a: 不将数字抽象成N，字符串抽象成S 

- -s: 是表示按照何种方式排序：
  - c: 访问次数
  - l: 锁定时间
  - r: 返回记录
  - t: **查询时间**
  - al:平均锁定时间
  - ar:平均返回记录数
  - at:平均查询时间 （默认方式）
  - ac:平均查询次数

- -t: 即为返回前面多少条的数据；
- -g: 后边搭配一个正则匹配模式，大小写不敏感的；

举例：我们想要按照查询时间排序，查看前五条 SQL 语句，这样写即可：

```bash
mysqldumpslow -s t -t 5 /var/lib/mysql/atguigu01-slow.log

Reading mysql slow query log from /var/lib/mysql/atguigu01-slow.log 
Count: 1 Time=2.39s (2s) Lock=0.00s (0s) Rows=13.0 (13), root[root]@localhost SELECT * FROM student WHERE name = 'S' 

Count: 1 Time=2.09s (2s) Lock=0.00s (0s) Rows=2.0 (2), root[root]@localhost SELECT * FROM student WHERE stuno = N 

Died at /usr/bin/mysqldumpslow line 162, <> chunk 2.
```

```sql
#得到返回记录集最多的10个SQL
mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow.log 
#得到访问次数最多的10个SQL 
mysqldumpslow -s c -t 10 /var/lib/mysql/atguigu-slow.log 
#得到按照时间排序的前10条里面含有左连接的查询语句 
mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/atguigu-slow.log 
#另外建议在使用这些命令时结合 | 和more 使用 ，否则有可能出现爆屏情况 
mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow.log | more
```

### 4.6 关闭慢查询日志

MySQL服务器停止慢查询日志功能有两种方法：

**方式**1**：永久性方式**

```bash
[mysqld] 
slow_query_log=OFF
```

或者，把slow_query_log一项注释掉 或 删除

```sql
[mysqld] 
#slow_query_log =ON
```

重启MySQL服务，执行如下语句查询慢日志功能。

```sql
SHOW VARIABLES LIKE '%slow%'; #查询慢查询日志所在目录 
SHOW VARIABLES LIKE '%long_query_time%'; #查询超时时长
```

**方式2：临时性方式**

使用SET语句来设置。 

（1）停止MySQL慢查询日志功能，具体SQL语句如下。

```sql
SET GLOBAL slow_query_log=off;
```

（2）重启MySQL服务，使用SHOW语句查询慢查询日志功能信息，具体SQL语句如下

```sql
SHOW VARIABLES LIKE '%slow%'; #以及 
SHOW VARIABLES LIKE '%long_query_time%';
```

### 4.7 删除慢查询日志

```sql
mysql> show variables like '%slow_query_log%';
+---------------------+------------------------------------------+
| Variable_name       | Value                                    |
+---------------------+------------------------------------------+
| slow_query_log      | ON                                       |               
| slow_query_log_file | /usr/local/var/mysql/venson-Pro-slow.log |
+---------------------+------------------------------------------+
2 rows in set (0.00 sec)
```

手动删除即可



```bash
#mysqladmin flush-logs 重新生成日志文件
mysqladmin -uroot -p flush-logs slow
```

> 提示:
>
> 满查询日志都是使用mysqladmin flush-logs命令来删除重建. 使用时一定要注意, 一旦执行了这个命令, 慢查询日志都只能存在新的日志文件中, 如需要旧的日志文件, 需要提前备份. 

## 5. 查看SQL **执行成本：**SHOW PROFILE 

```sql
mysql> show variables like 'profiling';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| profiling     | OFF   |
+---------------+-------+
1 row in set (0.00 sec)

mysql> SET profiling = 'ON';
mysql> show variables like 'profiling';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| profiling     | ON    |
+---------------+-------+
1 row in set (0.00 sec)
```

show profile的常用查询参数：

① ALL：显示所有的开销信息。 

② BLOCK IO：显示块IO开销。 

③ CONTEXT SWITCHES：上下文切换开销。 

④ CPU：显示CPU开销信息。

⑤ IPC：显示发送和接收开销信息。

⑥ MEMORY：显示内存开销信息。

⑦ PAGE FAULTS：显示页面错误开销信息。 

⑧ SOURCE：显示和Source_function，Source_file， Source_line相关的开销信息。 

⑨ SWAPS：显示交换次数开销信息。

**日常开发需要注意的结论**

1. converting HEAP to MyISAM. 查询结果太大, 内存不够, 数据存储在磁盘中
2. Creating tmp table. 创建临时表, 先拷贝数据到临时表, 用完后再删除临时表
3. copying to tmp table on disk. 把内存中临时表复制到磁盘上, 警惕!!!
4. locked. 

如出现任意一条, 需要优化

show profilie 命令被弃用, 可以从performance_schema中的profiling数据表进行查看.

```sql
mysql> select * from performance_schema.setup_actors ;
+------+------+------+---------+---------+
| HOST | USER | ROLE | ENABLED | HISTORY |
+------+------+------+---------+---------+
| %    | %    | %    | YES     | YES     |
+------+------+------+---------+---------+
1 row in set (0.00 sec)
```



## 6. 分析查询语句：EXPLAIN

### 6.1 概述

定位了慢查询SQL后, 使用explain 或describe 工具进行分析. 

优化器通过计算分析收集到的信息,为客户制定最优的执行计划( 优化器认为最优的检索方式, 不一定是实际的最优方式). 

能做什么:

- 表的读取顺序
- 数据读取操作的操作类型
- 哪些索引可以使用
- 哪些索引被实际使用
- 表之间的引用
- 每张表有多少行被优化器查询

**版本情况**

- MySQL 5.6.3以前只能 EXPLAIN SELECT ；MYSQL 5.6.3以后就可以 EXPLAIN SELECT，UPDATE， DELETE 

- 在5.7以前的版本中，想要显示 partitions 需要使用 explain partitions 命令；想要显示filtered 需要使用 explain extended 命令。在5.7版本后，默认explain直接显示partitions和 filtered中的信息。



```sql
mysql> explain select * from student_info limit 10;
+----+-------------+--------------+------------+------+---------------+------+---------+------+--------+----------+-------+
| id | select_type | table        | partitions | type | possible_keys | key  | key_len | ref  | rows   | filtered | Extra |
+----+-------------+--------------+------------+------+---------------+------+---------+------+--------+----------+-------+
|  1 | SIMPLE      | student_info | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 997194 |   100.00 | NULL  |
+----+-------------+--------------+------------+------+---------------+------+---------+------+--------+----------+-------+
1 row in set, 1 warning (0.00 sec)
```

### 6.2 基本语法

EXPLAIN 或 DESCRIBE语句的语法形式如下：

```sql
EXPLAIN SELECT select_options 
DESCRIBE SELECT select_options
```

如果我们想看看某个查询的执行计划的话，可以在具体的查询语句前边加一个 EXPLAIN ，就像这样：

```sql
mysql> EXPLAIN SELECT 1;
```

EXPLAIN 语句输出的各个列的作用如下：

| 列名 | 描述                                                    |
| ---- | ------------------------------------------------------- |
| id   | 在一个大的查询语句中每个SELECT关键字都对应一个 `唯一的id` |
|select_type |SELECT关键字对应的那个查询的类型|
|table |表名|
|partitions |匹配的分区信息(可略)|
|**type** |针对单表的访问方法|
|possible_keys |可能用到的索引|
|key |实际上使用的索引|
|**key_len** |实际使用到的索引长度|
|ref |当使用索引列等值查询时，与索引列进行等值匹配的对象信息|
|**rows** |预估的需要读取的记录条数|
|filtered |某个表经过搜索条件过滤后剩余记录条数的百分比|
|**Extra** |一些额外的信息|

### 6.3 数据准备

1. 建表

```sql
CREATE TABLE s1 ( 
  id INT AUTO_INCREMENT, 
  key1 VARCHAR(100), 
  key2 INT, 
  key3 VARCHAR(100), 
  key_part1 VARCHAR(100), 
  key_part2 VARCHAR(100), 
  key_part3 VARCHAR(100), 
  common_field VARCHAR(100),
  
  PRIMARY KEY (id), 
  INDEX idx_key1 (key1), 
  UNIQUE INDEX idx_key2 (key2), 
  INDEX idx_key3 (key3), 
  INDEX idx_key_part(key_part1, key_part2, key_part3) 
) ENGINE=INNODB CHARSET=utf8;
```

```sql
CREATE TABLE s2 ( 
  id INT AUTO_INCREMENT, 
  key1 VARCHAR(100), 
  key2 INT, 
  key3 VARCHAR(100), 
  key_part1 VARCHAR(100), 
  key_part2 VARCHAR(100), 
  key_part3 VARCHAR(100),
  common_field VARCHAR(100), 
  PRIMARY KEY (id), 
  INDEX idx_key1 (key1), 
  UNIQUE INDEX idx_key2 (key2),
  
  INDEX idx_key3 (key3), 
  INDEX idx_key_part(key_part1, key_part2, key_part3) 
) ENGINE=INNODB CHARSET=utf8;
```

2. **设置参数log_bin_trust_function_creators**

创建函数，假如报错，需开启如下命令：允许创建函数设置：

```sql
set global log_bin_trust_function_creators=1; # 不加global只是当前窗口有效。
```

3. **创建函数**

```sql
DELIMITER // 
CREATE FUNCTION rand_string1(n INT) 
RETURNS VARCHAR(255) #该函数会返回一个字符串 
BEGIN
	DECLARE chars_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFJHIJKLMNOPQRSTUVWXYZ'; 
	DECLARE return_str VARCHAR(255) DEFAULT ''; 
	DECLARE i INT DEFAULT 0; 
	WHILE i < n 
		DO 
		SET return_str = CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1+RAND()*52),1)); 
		SET i = i + 1; 
	END WHILE; 
	RETURN return_str; 
END // 
DELIMITER ;
```

4. **创建存储过程**

创建往s1表中插入数据的存储过程：

```sql
DELIMITER // 
CREATE PROCEDURE insert_s1 (IN min_num INT (10),IN max_num INT (10))
BEGIN
	DECLARE i INT DEFAULT 0; 
	SET autocommit = 0; 
	REPEAT SET i = i + 1; 
		INSERT INTO s1 
		VALUES( (min_num + i), rand_string1(6), (min_num + 30 * i + 5), rand_string1(6), rand_string1(10), rand_string1(5), rand_string1(10), rand_string1(10)); 
	UNTIL i = max_num 
	END REPEAT; 
	COMMIT; 
END // 
DELIMITER ;
```

创建往s2表中插入数据的存储过程：

```sql
DELIMITER // 
CREATE PROCEDURE insert_s2 (IN min_num INT (10),IN max_num INT (10)) 
BEGIN
	DECLARE i INT DEFAULT 0; 
	SET autocommit = 0; 
	REPEAT SET i = i + 1; 
		INSERT INTO s2 
		VALUES( (min_num + i), rand_string1(6), (min_num + 30 * i + 5), rand_string1(6), rand_string1(10), rand_string1(5), rand_string1(10), rand_string1(10)); 
	UNTIL i = max_num 
	END REPEAT; 
	COMMIT; 
END // 
DELIMITER ;
```

5. **调用存储过程**

s1表数据的添加：加入1万条记录：

s2表数据的添加：加入1万条记录：

```sql
CALL insert_s1(10001,10000);
CALL insert_s2(10001,10000);
```

6.4 **EXPLAIN各列作用**

为了让大家有比较好的体验，我们调整了下 EXPLAIN 输出列的顺序。

1. table

不论我们的查询语句有多复杂，里边儿 包含了多少个表 ，到最后也是需要对每个表进行 单表访问 的，所以MySQL规定EXPLAIN**语句输出的每条记录都对应着某个单表的访问方法**，该条记录的table列代表着该表的表名（有时不是真实的表名字，可能是简称）。

2. id

我们写的查询语句一般都以 SELECT 关键字开头，比较简单的查询语句里只有一个 SELECT 关键字，比如下边这个查询语句：



- id如果相同，可以认为是一组，从上往下顺序执行(一个SELECT 一个id)

- 在所有组中，id值越大，优先级越高，越先执行

- 关注点：id号每个号码，表示一趟独立的查询, 一个sql的查询趟数越少越好

|名称 |描述|
| ----| ---|
|SIMPLE |Simple SELECT (not using UNION or subqueries) |
|PRIMARY | Outermost SELECT |
|UNION |Second or later SELECT statement in a UNION |
|UNION RESULT |Result of a UNION |
|SUBQUERY |First SELECT in subquery |
|DEPENDENT SUBQUERY| First SELECT in subquery, dependent on outer query|
|DEPENDENT UNION |Second or later SELECT statement in a UNION, dependent on outer query|
|DERIVED| Derived table|
|MATERIALIZED |Materialized subquery|
|UNCACHEABLE SUBQUERY| A subquery for which the result cannot be cached and must be re-evaluated for each row of the outer query|
|UNCACHEABLE UNION| The second or later select in a UNION that belongs to an uncacheable subquery (see UNCACHEABLE SUBQUERY)|

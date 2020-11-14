## 1. 初识MySQL

后台：连接数据库JDBC，连接前端

### 1.1 为什么学

1. 岗位需求
2. 现在的世界，大数据时代，得数据者得天下
3. 被迫需求：存数据
4. **数据库是所有软件体系中最核心的存在**

### 1.2 什么是数据库

DataBase

概念：数据仓库，软件，安装在操作系统（win，linux）之上！

作用：存数据，管数据

### 1.3 数据库分类

关系型（SQL）：

	 - MySQL，Oracle，sql server
	 - 通过表与表之间，列和行之间的关系进行数据的存储

非关系型（NoSQL）not only：

	- Redis，MongoDB
	- 对象存储，通过对象自身的属性来决定

**DBMS(数据库管理系统)**

- 数据库管理软件，科学有效的管理和操作我们的数据，维护和获取；
- MySQL，是一个数据库管理系统，因为它不仅存数据，还可以进行数据管理。

### 1.4 MySQL简介

关系型数据库管理系统

前世：瑞典MySQL ab公司

今生：oracle旗下产品

最好的关系型数据库关系系统之一

开源

体积小，速度快，总体拥有成本低

官网： https://www.mysql.com/

5.7 稳定版

8.0 最新版

安装建议：

- 尽量不要使用exe， 有注册表的问题，难卸载
- 尽量使用压缩包安装

### 1.5 安装：[参考](https://www.cnblogs.com/fnlingnzb-learner/p/6009153.html)

1. 压缩包解压，放到自己的电脑环境目录（要有自己的环境目录，专门存放redis，jdk等压缩包解压后的文件）下
2. 添加环境变量（将bin目录添加到环境变量里）
3. 新建MySQL配置文件，my.ini,放在解压后的文件夹下（linux上自带，但win上需要自己建立）

```ini
[mysqld]
# 目录一定要换成自己的
basedir=D:\Environemnt\mysql-5.7.19\
datadir=D:\Environemnt\mysql-5.7.19\data\
port=3306
skip-grant-tables
```

 4. 以管理员身份启动cmd，运行所有命令

 5. 安装MySQL

 6. 初始化数据

 7. 启动MySQL，修改密码

    ``` sql
    net start mysql57
    mysql -u root -p
    ```

	8. 常见问题：缺少组件dll

### 1.6 安装Navicat、sqlyog（可视化软件）

	1. 安装
 	2. 新建数据库
 	3. 新建表，查看表
 	4. 插入记录

### 1.7 命令行连接数据库

命令行连接

```bash
1. net start mysql57
2. mysql -u root -p # 连接数据库
```

```sql
show databases;--所有语句都是用；结尾
use databasename; -- 切换数据库
show tables;
desc tablename; -- 查看表信息
create database databasename; -- 建立数据库
exit;--退出连接
-- 单行注释
/*
多行注释
啊啊
*/
```

**数据库xxx语言**

DDL 定义

DML 操作

DQL 查询

DCL 控制

# 2. 操作数据库

操作数据库>操作数据库中的表>操作数据库表中的数据

**MySQL关键字不区分大小写**

### 2.1 操作数据库

1. 创建数据库

   ```sql
   create database [if not exists] dataname;
   ```

2. 删除数据库

   ```sql
   drop database [if exists] dataname;
   ```

3. 使用数据库

   ```sql
   use databasename;
   ```

4. 查看

   ```sql
   show databases;
   ```

### 2.2 数据库的列类型

> 数值

 	- tinyint   十分小的数据，1个字节
 	- smallint 较小的   2个字节
 	- mediumint           3字节
- **int      标准整数      4字节**
- bigint                      8字节
- float    浮点数 单精度 4字节                    
- double                         8字节
- decimal   字符串形式的浮点数    金融计算的时候，一般使用这个   

>字符串

```sql
- char      固定大小的字符串 0-255
- **varchar   可变字符串 0-65536**  常用 相当于String
- tinytext  微型文本  2^8-1
- **text   文本串   2^16 -1**  保存大文本
```

> 时间日期

```sql
- java.util.Date
- date   YYY-MM-DD
- time  HH: mm: ss
- **datetime  YYY-MM-DD  HH: mm: ss  最常用时间格式**
- **timestamp 时间戳**  1970.1.1到现在的毫秒数！唯一的
- year 年份表示
```

> null

```sql
- 没有值，未知
- **不要使用mull进行运算，结果一定为null。**
```

[MySQL中数据类型的长度总结](https://blog.csdn.net/yaruli/article/details/79187814)：挺有用的

### 2.3 数据库 的字段属性

unsigned：

	- 无符号整数
	- 声明该列不能为负数

zerofill

	- 0填充的
	- 不足的位数，使用0来填充

自增

	- 通常理解为自增，自动在上一条的基础上加1
	- 通常用来设计唯一的主键-  index，必须是整数类型
	- 可以自定义主键自增的起始值和步长

非空：

	- 假设设置为not null,如果不赋值，就会报错
	- NULL：如果不填写值，默认值为NULL

默认

	- 设置默认的值，不指定该列的值，就会使用默认值
	- sex，默认值为男

扩展：	

```sql
/*  项目中每个表必须有这几个字段
id 主键
version 乐观锁
is_delete 伪删除
gmt_create 创建时间
gmt_update  更新时间
*/
```



### varchar和char有什么区别

1. varchar(n)和char(n)都是存储n个字符，超出则截断
2. char存储时占用的空间始终是n个字符，varchar存储的是实际字符串长度加1-2个字节，后面两个字节保存字符串长度（如果是255长度以内，则一个字节，否则两个字节）
3. char会对存入的字符串截取尾部空格，但varchar不会。



### null和""有什么区别

### 2.4 创建数据库表（重点）

```sql
/*
表的名次字段名称尽量用``括起来,这个符号可以避免和sql语言的关键字重复。
字符串用''括起来
所有的语句后面加, 除最后一句
*/
create table [if not exists] `student`(
	`id` int(4) not null auto_increment comment '学号',
    --int(4)中的4只是显示宽度，并不是只有4位，int类型是固定位数的
    `name` varchar(30) not null default '匿名' comment '姓名', --姓名使用30个长度的字符绝对够，这里的30是指定字符串的长度
    `birth` datetime default null comment '出生日期'
    primary key (`id`)
    
)engine = innodb default charset=utf8
```

### 2.5 MYISAM，INNODB（两种引擎的区别）



```sql
--关于引擎
/*
INNODB 默认
MYISAM 早些年使用
*/
```

|            | MYISAM | INNODB            |
| ---------- | ------ | ----------------- |
| 事务支持   | 不支持 | 支持              |
| 数据行锁定 | 不支持 | 支持              |
| 外键约束   | 不支持 | 支持              |
| 全文索引   | 支持   | 不支持            |
| 表空间大小 | 较小   | 较大，约为前者2倍 |

常规使用操作：

	- MYISAM    节约空间
	- INNODB 安全性高，支持事务处理，多表多用户操作

> 在物理空间存在的位置

所有数据库文件都在data目录下，每个文件夹对应一个数据库

MySQL本质还是文件存储

MySQL引擎在物理文件上的区别：

	- INNODB在数据库表中只有一个*.frm文件，以及上级目录下的ibdata1文件
 - MYISAM对应文件
   - .frm  表结构的定义文件
   - .MYD 数据文件（data）
   - .MYI   索引文件（Index）

> 设置数据库表的字符集编码

可以在创建表时声明：

```sql
CHATSET=utf8
```

**不设置的话，会使用mysql默认的字符串编码，不支持中文！**

也可以在my.ini中自己配置默认的编码：

```ini
character-set-server=utf8
```



### 2.6 修改删除表

```sql
--重命名表名
ALTER TABLE tname RENAME AS newname;
-- 增加字段
ALTER TABLE tname ADD age INT(11);
-- 修改字段约束
ALTER TABLE tname MODIFY age varchar(11);
--重命名
ALTER TABLE tname CHANGE age Age1 int(11);
/*
modify能修改字段类型和约束。不能重命名
change用来重命名，也可以修改字段类型和约束。
*/

--删除字段
ALTER TABLE tname DROP age;

--删除表
DROP TABLE [if exists] tname;
```

**所有的创建和删除操作尽量加上判断，以免报错。**

注意：

	- `` 字段名使用这个包裹
	- 注释 --   /**/
	- 关键字大小写不敏感，建议写小写
	- 所有的符号用英文

# 3. MySQL的数据管理

### 3.1 外键（了解）

- 方式1，创建表时声明

```sql
create table [if not exists] `student`(
	`id` int(4) not null auto_increment comment '学号',
    --int(4)中的4只是显示宽度，并不是只有4位，int类型是固定位数的
    `name` varchar(30) not null default '匿名' comment '姓名', --姓名使用30个长度的字符绝对够，这里的30是指定字符串的长度
    `birth` datetime default null comment '出生日期',
    `gradeid` int(4) not null comment '班级id',
    primary key (`id`),
    KEY `FK_gradeid` (`gradeid`),
    CONSTRAINT `FK_gradeid` FOREIGN KEY (`gradeid`) REFERENCES `grade_table_name` (`id`)
    
)engine = innodb default charset=utf8
```

 - 方式2

   ```sql
   --创建表时不声明约束，然后alter表
   ALTER TABLE `student` ADD CONSTRAINT `FK_gradeid` FOREIGN KEY(`gradeid`) REFERENCES `grade` (`id`);
   ```

- 以上的操作都是物理外键，数据库级别的外键，不建议使用（数据库太多会造成困扰，这里了解即可）

- **最佳实践：**

  - 数据库就是单纯的表，只有行和列，不使用外键约束
  - 想使用外键，就用程序去实现

### [为什么不推荐使用外键](https://www.cnblogs.com/rjzheng/p/9907304.html)



### 3.2 DML语言（全部记住）

数据库意义：数据的存储和管理

DML：数据操作语言

 - #### insert

   ```sql
   insert into tablename(字段名1,字段名2,..) values('值1','值2',...);  --如果不写表的字段，就会一一匹配。
   
   --插入多个值
   insert into tablename(字段名1,字段名2,..) values('值1','值2',...),('值3','值4',...);
   ```

   - 字段和字段之间用英文逗号隔开
   - 字段可省略
   - 可以同时插入多条数据，values的值用逗号隔开即可

 - #### update

   ```sql
   UPDATE `tablename` SET `name`='新值' WHERE id=1;
   --不指定条件的话会修改所有记录！
   -- UPDATE 表名 set columnname = value,columnname2 = value2 where...;
   
   
   ```

   条件：where子句。

   | 操作符           | 含义         |
   | ---------------- | ------------ |
   | =                | 等于         |
   | <>或!=           | 不等于       |
   | <=，<            |              |
   | >=，>            |              |
   | BETWEEN...AND... | 闭合区间     |
   | AND              | 连接两个条件 |
   | OR               |              |

   - 注意：
   - column_name是数据库的列名，尽量加``
   - value是一个具体的值，也可以是变量（变量也没那么多可以用，大概只有时间可以用，比如 set birth = CURRENT_TIME ）
   - 多个需要设置的字段之间用逗号隔开

 - #### delete

   ```sql
   --删除某记录，不加条件则全删
   DELETE FROM `tname` where ...;
   --清空表，表结构和字段不变
   TRUNCATE `tname`;
   ```

   delete和truncate的区别：

   - 都能删除数据，都不会删除表结构
   - trancate会重新设置自增列，计数器会归零
   - trancate不会影响事务

# 4. DQL查询数据（最重点）

### 4.1 DQL

date duery language：数据查询语言

- 所有查询操作都用它，select
- 简单复杂查询都可以
- 数据库中最核心的语言，最重要的语句
- 用的最多

### 4.2 指定查询字段

```sql
--查询指定字段
select `studentname`,`age` from student;
--给字段和表起别名,有时候字段名不是那么见名知意。
select `studentname` as 姓名,`age` as 年龄 from student as stable;
--函数 concat（a，b）用于拼接
select CONCAT('姓名：',studentname) from student;

```

- 去重

```sql
select * from result;
select `student_num` from result;
--去重
select distinct `student_num` from result;
```

- 数据库的列（表达式）

```sql
select version()--查询版本
select 100*3-1 as 计算结果;--用来计算
select @@auto_increment_increment --查询自增步长

--学员成绩+1分查看
select `student_num`,`stu_result`+1 as '提分后' from result;

```

数据库中的表达式：文本值，列，null，函数，计算表达式，系统变量。。。

```sql
select 表达式 from 表
```

### 4.3 where条件子句

| 运算符    | 语法    | 描述   |
| --------- | ------- | ------ |
| and       | a and b | 逻辑与 |
| or        | a or b  | 逻辑或 |
| not 或 != | not a   | 逻辑非 |

- 模糊查询：比较运算符

| 运算符       | 语法              | 描述               |
| ------------ | ----------------- | ------------------ |
| is null      | a is null         | 运算符为null则为真 |
| is not null  | a is not null     |                    |
| between  and | a between b and c | a在b和c之间则为真  |
| like         | a like b          | a能匹配到b则为真   |
| in           | a in b            |                    |

```sql
--like结合： %（代表0到任意个字符） _(代表一个字符)
select * from students where name like '刘%';
select * from students where name like '刘_';
select * from students where name like '刘__';
-- 查询名字有某个字的
select * from students where name like '%刘%';
-- =====  in  ===========
-- 查询1001，1002，1003号学员
select * from students where num in (1001,1002,1003);
-- 
select * from students where address in ('安徽','洛阳');
-- ========= not null ==========
select * from students where address='';
select * from students where address is null;

```

### 4.4 联表查询

join 对比

![](https://i0.wp.com/i.stack.imgur.com/qje6o.png)

```sql
--联表查询,查询参加了考试的同学（学号，姓名，科目编号，分数）
select * from students;
select * from results;
/*
分析需求，分析查询的字段来自哪些表（连接查询）
确定使用哪种连接查询（7种）
确定交叉点（这两个表中哪个数据是相同的）
判断条件：学生表中的studentNo = 成绩表中的studentNo
*/

--join（连接的表） on（交叉条件） 连接查询
--where 等值查询

--inner join
select s.studentNo,studentname, SubjectNo, StudentResult from student as s
inner join result as r
where s.studentNo = r.studentNo;

--right join
select s.studentNo,studentname, SubjectNo, StudentResult from student as s
right join result as r
on s.studentNo = r.studentNo;

--left join
select s.studentNo,studentname, SubjectNo, StudentResult from student as s
left join result as r
on s.studentNo = r.studentNo;
```

| 操作       | 描述                                           |
| ---------- | ---------------------------------------------- |
| inner join | 只返回两个表中联结字段相等的行                 |
| left join  | 返回左表中的所有记录和右表中联结字段相等的记录 |
| right join | 返回右表中的所有记录和左表中联结字段相等的记录 |

参考：https://www.360kuai.com/pc/9be280cbf1adb41e6?cota=4&kuai_so=1&tj_url=so_rec&sign=360_57c3bbd1&refer_scene=so_1

我的理解：

不管select的是什么，from关键字后面的语句：表A join 表B on 。。。实际上相当于把两个表**拼接**在一起，拼接后的表的字段是二者的字段之和，但这个新表有哪些行记录是join的类型决定的。如果是inner join，那么行记录全部都是AB两个表中满足on条件的行记录的拼接。如果是left join，那么行记录一部分是AB两个表中满足on条件的行记录的拼接，另一部分是左表的其他的行记录（新表中这一部分的行记录中，来自左表的字段有来自左表的行记录补充，但是来自右表的字段就会自动补为null），也就是说，left join会包含左表的所有行记录，和右表中满足on的行记录。right join同理。

比如：

表a

| aid  | a_name |
| ---- | ------ |
| 1    | a      |
| 2    | b      |

表b：

| bid  | b_name |
| ---- | ------ |
| 1    | c      |
| 3    | d      |

select * from a [inner,left,right] join b on a.aid = b.bid where 

| aid  | a_name | bid  | b_name |
| ---- | ------ | ---- | ------ |
| 1    | a      | 1    | c      |
| 2    | b      | null | null   |
| null | null   | 3    | d      |

上表中，第1行就是inner join的结果；1，2行是left join的结果；1，3行是right join的结果。（join这种联表查询，不涉及外键约束就能准确找到两个表中交叉的，或独有的行记录，其实很有用）。如果加上where条件，比如 b.bid is null,那么就可以选出a表中有而b表中没有的（也就是第2行）。

```sql
--查询学员 所属的年级（学号，姓名，年级名称）
select studentNo,studentName,gradeName from 
student as s 
inner join grade as g 
on s.GradeId=g.GradeId;
--查询科目所属的年级（科目名，年级名，）
select subjectName, gradeName from
subject as sub
inner join grade as g
on sub.gradeId = g.gradeId;
-- 查询参加了 数据结构 考试的学生信息（学号，姓名，科目，分数）
select s.studentNo,studentName,subjectName,result--查什么
from student as s --哪张表
inner join result as r --连接哪张表
on s.studentNo=r.studentNo --交叉条件
inner join subject as sub --还要连接哪张表
on r.subjectNo = sub.subjectNo --交叉条件
where subjectName = '数据结构'; --where条件
```

### 4.5 分页和排序

```sql
-- =============分页和排序============
-- 排序：升序asc，降序desc
select * from tablename where id>1
order by studentNo desc;

--为什么分页：
--缓解数据库压力，给人的体验更好，瀑布流

--分页，每页只显示5条
limit offset,5; --从第offet+1个开始，取5个

--第一页 limit 0,5
--第二页 limit 5,5
--第三页 limit 10,5
--第M页  limit（M-1）*pagesize，pagesize
--【pagesize，页面大小】
--【（n-1）*pagesize，起始值】
--【n，当前页码】
--【数据总数/页面大小=总页数】


```

### 4.6 子查询

```sql

--方式1 连接查询
select stuNo, r.subjectNo, stuResult
from result as r
inner join subject as sub
on sub.subjectNo = r.subjectNo
where subjectName = '数据结构'
order by stuResult desc;

--方式2 子查询
select stuNo, subjectNo, stuResult
from result as r
where subjectNo = (
select subjectNo from subject where subjectName ='数据结构'
)order by stuResult desc;



```

```sql
--select完整语法
SELECT[ALL|DISTINCT|DISTINCTROW|TOP]
{*|talbe.*|[table.]field1[AS alias1][,[table.]field2[AS alias2][,…]]}
FROM tableexpression[,…][IN externaldatabase]
[JOIN..]
[WHERE…]
[GROUP BY…]
[HAVING…]
[ORDER BY…]
[LIMIT]
```

### 4.7 分组和过滤

```sql
select subjectName, avg(studentResult) as 平均分,max(studentResult) as 最高分, min(studentResult) as 最低分
from result r
inner join subject sub
on r.subjectNo = sub.subjectNo
group by r.subjectNo,r.xxx,r.xxxx --通过什么字段来分,可以有多个
having 平均分>80; --一旦用了group by，就不能用where了，必须用having，否则会报错

-- 例子：选择某个字段重复的数据的id
select id from tablename group by field having count(field)>1
--例子：选择某两个字段重复的数据id
select id from tablename group by fie1,fie2 having count(*)>1 --实际上就是通过group by对结果分组，看分组中数据条数是否大于1
```

### 4.8 select小结

```sql
--顺序很重要
select 去重 字段 form 表 （表，字段可以取别名）
xxx join 表 on 等值判断
where （具体的值或子查询语句）
group by （通过哪个字段分组）
having (过滤分组后的信息，条件和where是一样的，只是位置不同)
order by 字段 desc/asc
limit offset,pagesize

--跨表，跨数据库查询也是存在的
```



# 5. MySQL函数

### 5.1 常用函数

```sql
select abs(-8);
select ceiling(9,4);--向上取整
select floor(9,4);--向下取整
select rand();--0-1随机数
select ceiling(9,4);--向上取整
select concat('啊','11','2e3');--连接字符串
select char_length('sdadeda112e2');--字符串长度
select upper('WWDe23e3ddd');--转大写
select lower('WWDe23e3ddd');--转小写

select current_date();--获取当前日期
select curdate();
select now();
select localtime();--本地时间
select sysdate();--系统时间

```

### 5.2 聚合函数（常用）

| 函数      | 描述 |
| --------- | ---- |
| count（） | 计数 |
| sum（）   | 求和 |
| max()     | 最值 |
| min()     |      |
| avg()     | 平均 |
|           |      |

```sql
 --想查询一个表中有多少记录，就用这个count()
 select count(studentname) from student; --count(字段)，会忽略null
 select count(*) from student;--不忽略null，本质 计算行数
 select count(1) from student;--不忽略bull，本质 计算行数
 
 select sum(result) as '总和' form result;
 select avg(result) as '平均分' form result;
 
 group by [字段] --通过什么字段来对select的结果分组
 
```

### 5.3 数据库级别的MD5加密

MD5不可逆，具体的值的md5是一样的。

MD5破解网站的原理，背后有个字典，加密前后的值，遍历查

```sql
create table 'testmd5'(
	`id` int(4) not null,
    `name` varchar(20) not null,
    `pwd` varchar(20) not null,
    primary key(`id`)
)engine=innodb default charset=utf8

--明文密码
insert into testmd5 values(1,'zhangsan','123456');
--加密
update testmd5 set pwd=md5(pwd) where id =1;
insert into testmd5 values(1,'zhangsan',md5('123456'));

```



# 6. 事务

### 6.1 什么是事务

（事务的所有sql）要么都成功，要么都失败

将一组sql作为一个批次执行。

原则：ACID

```sql
set autocommit =0;--关闭自动提交
set autocommit =1;--开启自动提交（默认）

-- 手动处理事务
set autocommit =0;--关闭自动提交
--事务开启
start transaction;
sql语句
--提交
commit
--回滚
rollback
--事务结束
set autocommit =1;--开启自动提交
```

# 7. 索引

**索引是帮助MySQL高效获取数据的数据结构。**

### 7.1 索引分类

- 主键索引（primary key）
  - 主键不可重复，主键的值唯一标识某一列，只能有一个列作为主键。INNODB中，不指定主键就使用第一个非空unique列，没有这样的列就建隐藏字段做主键索引。
- 唯一索引（unique key）
  - 与主键索引的区别是，唯一索引可以有多个，可以为null
- 常规索引（key/index）
  - 默认的，使用index，或key关键字来设置
- 全文索引（FullText）
  - 在特定的引擎下才支持，比如MYISAM（INNODB不支持）
  - 快速定位数据



```sql
--索引的使用
--创建表时
create table 'testmd5'(
	`id` int(4) not null,
    `name` varchar(20) not null,
    `pwd` varchar(20) not null,
    primary key(`id`),
    unique key(`name`),
    key(`pwd`)
)
--创建完后，增加索引
show index from tablename;

alter table `student` add fulltext index `索引名` (`字段`);
create index `索引名` on `tablename` (`字段`);
```

### 7.2 索引原则

- 不是越多越好
- 不要对经常变动的数据加索引
- 小数据量的表不需要索引
- 加在经常需要查询的字段上

索引的数据结构：阅读http://blog.codinglabs.org/articles/theory-of-mysql-index.html（半小时）

# 8. 数据库备份

### 8.1 用户管理

- 可以使用数据库可视化工具进行可视化管理，比较方便
- sql操作： 本质就是对user表（MySQL自带）的增删改查

### 8.2 MySQL备份

- 保存数据不丢失
- 数据转移

MySQL备份方式

- 直接拷贝物理文件：mysql文件夹下的data文件夹保存所有的信息
- 可视化工具中手动导出
- 使用命令行导出 mysqldump

# 9. 规范数据库设计

### 9.1 为什么需要设计

数据库比较复杂时就需要设计。

- 节省内存空间
- 保证数据库的完整性
- 方便我们开发系统

软件开发中，关于数据库设计：

- 分析需求
- 概要设计：设计关系图

**步骤**

- 收集信息，分析需求（分析需要哪些表）
- 标识实体类（把需求落地每个字段）
- 标识实体之间的关系

### 9.2 三大范式

1. 每列不可再分
2. 满足第一范式的前提下，每张表每一列都和主键相关，而不能只和主键的某一部分（主键中的某些字段）有关（有些表是联合主键）
3. 在第二范式的前提下，每列和主键直接相关。只有主键可以唯一确定每一列，不允许某个列由其他某列来唯一确定。

# 10. JDBC（重点）

### 10.1 数据库驱动

应用程序无法直接连接数据库，需要通过驱动，驱动是由数据库厂商提供的。我们的程序需要通过驱动和数据库打交道。

### 10.2 JDBC

使用数据库时，不同的数据库需要不同的数据库驱动（比如MySQL需要对应的驱动，Oracle有对应的驱动），JDBC是建立在这些驱动之上的一个驱动，我们只需要面对JDBC的接口进行编程，不必考虑底层使用的是什么数据库。

SUN公司为了简化开发人员对数据库的操作，提供了java操作数据库的规范，俗称JDBC。

这些规范的实现由具体的厂商实现。

我们只需要掌握JDBC的接口。

### 10.3 第一个JDBC程序

1. 创建一个项目
2. 导入数据库依赖（可以使用maven管理依赖）
   - mysql-connector-java
   - 需要在程序中import的包：java.sql,  javax.sql.(不必添加在Maven管理pom文件里)
3. 编写测试代码并运行

```java
public class JdbcFirstDemo{
    public static void main(String[] args){
        //1.加载驱动
        Class.forName("com.mysql.jdbc.Driver");//固定写法，涉及反射的知识
        //DriverManager.registerDriver(new com.mysql.jdbc.Driver());不建议使用的写法，因为这个类已经在static代码块里new了一个驱动，这样就相当于new了两次
        //2.用户信息和url
       String url = "jdbc:mysql://localhost:3306/databaseName?useUnicode=true&characterEncoding=utf8&useSSL=true";
        String username = 'root';
        String password = '123456';
        
        //3.连接成功，返回数据库对象
        Connection connection = DriverManager.getConnection(url,username,password);
        //4.执行sql的对象, Statemenet就是执行sql的对象
        Statement statement = connection.createStatement();
        //5. 执行sql的对象去执行sql，返回结果集
        String sql = "select * from tablename";
        ResultSet res = statement.executeQuery(sql);
        while (res.next()){
            System.out.println(res.getObject("字段名"));
        }
        //6.释放连接
        res.close();
        statement.close();
        connection.close();
        
    }
}
```

### 10.4 Statement对象

Statement的方法：

- executeQuery（）查询数据库，返回ResultSet
- executeUpdate（）更新插入删除都用这个，返回受影响的行数
- execute（）执行任何操作

ResultSet是查询的结果集，封装了所有的查询结果

- 获得指定的数据类型

  ```java
  resultset.getObject("字段名");//在不知道类型的情况下使用
  resultset.getString("字段名");
  resultset.getInt();
  resultset.getFloat();
  resultset.getDate();
  ...
  
  ```

- 遍历，指针

  ```java
  resultset.next();//移动到下一个
  resultset.beforeFirst();//移动到第一个
  resultset.afterLast();//移动到最后一个
  ```

  

可以把数据库的信息放到properties配置文件里，再写一个JDBCUtil的工具类，这个工具类中定义一些静态方法，将数据库的增删改查操作尽可能封装起来。（在JDBCUtil工具类里读取properties的内容就涉及到了IO流）

- sql注入：SQL注入即是指web应用程序对用户输入数据的合法性没有判断或过滤不严，攻击者可以在web应用程序中事先定义好的查询语句的结尾上添加额外的SQL语句，在管理员不知情的情况下实现非法操作，以此来实现欺骗数据库服务器执行非授权的任意查询，从而进一步得到相应的数据信息。

### 10.5 PreparedStatement对象

可以防止sql注入，且效率更高

使用方式（与Statement略有不同）：

```java
//使用？占位符参数
String sql = "insert into tname(id,`name`, `birth`) values(?,?,?)";
PreparedStatement st = connection.prepareStatement(sql);//预编译sql，先写sql，然后不执行
//手动赋值
st.setInt(1,1001);//给第一个占位符，也就是字段id，赋值为1001
st.setString(2,"xiaoming");//给第2个占位符，字段name赋值
//注意：这个函数的参数是sql下的Date类型，这个类型需要传入一个时间戳，时间戳使用java.util.Date().getTime()获得
st.setsetDate(3, new java.sql.Date(
    new Date().getTime()));
int i = st.executeUpdate();//执行

```

### 10.6 使用IDEA连接数据库

好像IDEA的专业版才可以。

### 10.7 事务

```java
Connection conn = DriverManager.getConnection(url,username,password);
conn.setAutoCommit(false);//关闭自动提交，这一句执行后会自动开启事务（在java里不需要写额外的开启事务的语句）
String sql = "update account set money = money-100 where name='A'";
String sql2 = "update account set money = money+100 where name='B'";
PreparedStatment st = conn.prepareStament(sql);
st.executeUpdate(sql);
st.executeUpdate(sql2);
conn.commit();//提交事务
//如果出现异常就try catch,在ctach中使用语句conn.rollback()进行事务回滚。实际上，java已经封装好了，即使不写这句，只要事务失败，会自动回滚，所以这句可以不写。
```

### 10.8 数据库连接池

数据库连接-执行-释放，连接和释放十分浪费资源。

池化技术：预先准备一些连接放在连接池内，有业务时，就拿出一个连接进行使用，使用完后放回池内，避免连接的反复创建和撤销。

连接池有一些参数：

最小连接数：

最大连接数：业务最高承载上限，如果业务所需要的连接数超过这个数量就等待。

等待超时：

。。。



编写连接池只需要实现一个接口DataSource。

比较知名的开源的DataSource实现：

- DBCP
- C3P0
- druid：阿里

使用了这些数据库连接池之后，就不需要编写连接数据库的代码了。

- DBCP

  - 需要用到的jar包：commons-dbcp-1.4.jar commons-pool-1.6.jar

  - 配置*.properties文件，

    ```properties
    # 连接设置
    driverClassName=com.mysql.jdbc.driver
    url=jdbc:mysql://localhost:3306/databasename?其他参数
    username=root
    passowrd=123
    
    # 初始化连接
    initialSize=10
    # 最大连接数
    maxActive=50
    #最大空闲连接
    maxIdle=20
    # 最小空闲连接
    minIdle=5
    # 超时等待时间 毫秒
    maxWait=60000
    # 其他等等
    
    ```

  - 然后，还需要编写程序，读取*.properties配置文件，然后调用DBCPjar包提供的一些类的一些方法，即可（此处不多写了）。

- C3P0

  - 需要的jar包：c3p0-0.9.5.5; mchange-commons-java-0.2.19
  - 与前者大同小异

- 无论使用什么数据源，本质还是一样的，都是实现了DataSource接口。


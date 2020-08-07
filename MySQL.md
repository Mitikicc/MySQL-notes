# MySQL从入门到入土

## 终端操作

### 登录和退出MySQL服务器

```mysql
#登录
mysql -u root -p
#然后输入密码（123456）

#退出
exit;
```

### 基本语法

```mysql
#创建数据库
creat database test;
#查询数据库服务器中的所有库
show databases;
#选中一个数据库,库名为‘test’
use test;
#察看一个数据库中的所有数据表
show tables;
#查询一个表（中的记录），表名为‘admin’
select * from admin;
#显示数据库的部分数据,id是字段
select * from admin where id=1;
#创建一个数据表，注意格式
CREATE TABLE pet (
    name VARCHAR(20),
    owner VARCHAR(20),
    species VARCHAR(20),
    sex CHAR(1),
    birth DATE,
    death DATE
);
#查看创建好的数据表的结构,表名为'pet'
desc pet;
#往数据表中添加记录
insert into pet
values('puffball', 'Diane', 'hamster', 'f', '1990-03-30', NULL);
#删除数据
delete from pet where name='Fluffy';
#修改数据（pet是表名，name和owner都是字段值）
update pet set name='修改后的名字' where owner='owner字段值'
#删除数据表
drop table table_name;
```

### MySQL建表约束条件

```mysql
#主键约束
#通过给某个字段添加约束，可以使该字段不重复且不为空(这里让id这个字段唯一)
create table user(
    id int primary key,
    name varchar(20)
);
--如果再次用insert插入id=1的数据会报错。
=================================================
#联合主键
#同理，限制id和name字段数据不能同时重复
create table user2(
    id int,
    name varchar(20),
    password varchar(20),
    primary key(id,name)
);
--举例，先添加一条数据
insert into user2
values(1,'张三','123');
--查询
select * from user2;
+----+--------+----------+
| id | name   | password |
+----+--------+----------+
|  1 | 张三   | 123      |
+----+--------+----------+
--再次添加一条数据
insert into user2
values(1,'张三','123');
--添加失败
ERROR 1062 (23000): Duplicate entry '1-张三' for key 'user2.PRIMARY'
--修改字段值并查询
insert into user2
values(2,'张三','123');
select * from user2;
+----+--------+----------+
| id | name   | password |
+----+--------+----------+
|  1 | 张三   | 123      |
|  2 | 张三   | 123      |
+----+--------+----------+
--添加成功
=================================================
#自增约束（配合主键约束一起使用）
create table user3(
    id int primary key auto_increment,
    name varchar(20)
);
--添加数据并查询
insert into user3 (name) values('张三');
select * from user3;
+----+--------+
| id | name   |
+----+--------+
|  1 | 张三   |
+----+--------+
--发现id没有设置但是自增1
=================================================
#添加/删除主键
create table user4(
    id int,
    name varchar(20)
);
--此时这个表里没有主键约束字段，下面进行添加
alter table user4 add primary key(id);
--查看结构
desc user4;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int         | NO   | PRI | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
--可以看到id变为了主键
--删除
alter table user4 drop primary key;
desc user4;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int         | NO   |     | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
--恢复原样
=================================================
#唯一约束(三种方法)
--唯一约束和主键约束的区别在于主键约束要求唯一且不为空，唯一约束只要求不唯一
create table user5(
    id int,
    name varchar(20)
);
--添加唯一约束并查看
alter table user5 add unique(name);
desc user5;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int         | YES  |     | NULL    |       |
| name  | varchar(20) | YES  | UNI | NULL    |       |
+-------+-------------+------+-----+---------+-------+
--name键变为唯一约束
--法2
create table user6(
    id int,
    name varchar(20),
    unique(name)
);
--unique同时对两个字段使用相当于联合主键
--法3
alter table user7 modify name varchar(20) unique;
--删除唯一约束
alter table user7 drop index name;
=================================================
#非空约束（约束字段不能为NULL）
create table user9(
    id int,
    name varchar(20) not null
);
desc user9;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int         | YES  |     | NULL    |       |
| name  | varchar(20) | NO   |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
--如果此时往表里添加数据只添加id那么就会报错，因为name字段不允许为空。
=================================================
#默认约束(deafault是默认值的意思)
--当我们插入字段时，如果没有传值，就会使用默认值。
create table user10(
    id int,
    name varchar(20),
    age int default 10
);
desc user10;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int         | YES  |     | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
| age   | int         | YES  |     | 10      |       |
+-------+-------------+------+-----+---------+-------+
--此时如果插入值时没有插入age字段会自动填写10
--插入命令
insert into user10 (id,name) values(1,'小徐');
--查看
select * from user10;
+------+--------+------+
| id   | name   | age  |
+------+--------+------+
|    1 | 小徐   |   10 |
+------+--------+------+
=================================================
#外键约束（父表子表两个表，子表中某一键值绑定父表中某一键值，父表中该键值的数据没有，则子表也不能有）
--建父表和子表,且为classes表中id添加主键
create table classes(
    id int primary key，
    name varchar(20)
);
create table students(
    id int,
    name varchar(20),
    class_id int,
    foreign key(class_id) references classes(id)
);
--最后一句说明students表中的id字段与classes表中id字段绑定
--往父表和子表里添加数据
insert into classes values (1,'一班');
insert into classes values (2,'二班');
insert into classes values (3,'三班');
insert into classes values (4,'四班');
insert into students values (1001,'aa',1);
insert into students values (1002,'bb',2);
insert into students values (1003,'cc',3);
insert into students values (1004,'dd',1);
--查表
select * from classes;
select * from students;
+------+--------+
| id   | name   |
+------+--------+
|    1 | 一班   |
|    2 | 二班   |
|    3 | 三班   |
|    4 | 四班   |
+------+--------+
#-----------------------------------------------------
+------+------+----------+
| id   | name | class_id |
+------+------+----------+
| 1001 | aa   |        1 |
| 1002 | bb   |        2 |
| 1003 | cc   |        3 |
| 1004 | dd   |        1 |
+------+------+----------+
#-----------------------------------------------------
--查看结构
desc classes;
desc students;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int         | NO   | PRI | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
#-----------------------------------------------------------------
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int         | YES  |     | NULL    |       |
| name     | varchar(20) | YES  |     | NULL    |       |
| class_id | int         | YES  | MUL | NULL    |       |
+----------+-------------+------+-----+---------+-------+
--注意到students表中的class_id字段key是MUL

--此时如果我对students表做一个添加
insert into students values (1005,'ee',5);
select * from students;
--报错
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`xcx`.`students`, CONSTRAINT `students_ibfk_1` FOREIGN KEY (`class_id`) REFERENCES `classes` (`id`))
--说明5不是classes表中id字段出现的值
--另外，父表中的数据被子表引用后不可被删除
```



### 数据库的三大设计范式

#### 第一范式1NF

某一个字段值详细到不可以拆分，就是第一范式。

字段不一定是越详细越好，要根据项目需求。

#### 第二范式2NF

满足第一范式的前提下，第二范式要求：除主键外每一列都必须完全依赖主键。

这个依赖二字可以解释为**主从关系** ，意思就是其他每一个字段都要服从主键字段，不能再服从别的字段

比如这个表里有字段**‘班主任’ ‘学生1’‘学生2’ ‘学生3’**，那么**学生1 学生2 学生3** 都必须听从主键字段 **班主任**

如果出现不完全依赖，那么只能发生在联合主键的情况下。意思就是如果上述表中还有**副班主任**字段，

那么将**副班主任**和**班主任**作为联合主键，这样**学生123**听任意一个老师的话都是一样的，也满足第二范式。

#### 第三范式3NF

必须先满足第二范式，然后除开主键列之外的其他列之间不能有传递（主从）关系。

```mysql
CREATE TABLE myorder (
    order_id INT PRIMARY KEY,
    product_id INT,
    customer_id INT,
    customer_phone VARCHAR(15)
);
```

表中的 `customer_phone` 有可能依赖于 `order_id` 、 `customer_id` 两列，也就不满足了第三范式的设计：其他列之间不能有传递依赖关系。

```mysql
CREATE TABLE myorder (
    order_id INT PRIMARY KEY,
    product_id INT,
    customer_id INT
);

CREATE TABLE customer (
    id INT PRIMARY KEY,
    name VARCHAR(20),
    phone VARCHAR(15)
);
```

修改后分别都满足第三范式，没有传递关系。

### 数据查询练习

#### 数据准备

```mysql
#创建查询练习数据库
create database selecttest;
#创建学生表
--学生编号--学生姓名--学生性别--出生年月日--所在班级
create table student(
    sno varchar(20) primary key,
    sname varchar(20) not null,
    ssex varchar(20) not null,
    sbirthday datetime,
    class varchar(20)
);
#创建教师表
--教师编号--教师姓名--教师性别--出生年月日--职称--所在部门
create table teacher(
    tno varchar(20) primary key,
    tname varchar(20) not null,
    tsex varchar(20) not null,
    tbirthday datetime,
    prof varchar(20) not null,
    depart varchar(20) not null
);
#创建课程表
--课程编号--课程名--教师编号
create table course(
    cno varchar(20) primary key,
    cname varchar(20) not null,
    tno varchar(20) not null,
    foreign key(tno) references teacher(tno)
);
#创建成绩表
--学号--课程号--成绩
create table score(
    sno varchar(20) not null,
    cno varchar(20) not null,
    degree decimal,
    foreign key(sno) references student(sno),
    foreign key(cno) references course(cno),
    -- 设置 s_no, c_no 为联合主键
    PRIMARY KEY(sno, cno)
);
#=====================================================================================
#添加数据
#添加学生表数据
INSERT INTO student VALUES('101', '曾华', '男', '1977-09-01', '95033');
INSERT INTO student VALUES('102', '匡明', '男', '1975-10-02', '95031');
INSERT INTO student VALUES('103', '王丽', '女', '1976-01-23', '95033');
INSERT INTO student VALUES('104', '李军', '男', '1976-02-20', '95033');
INSERT INTO student VALUES('105', '王芳', '女', '1975-02-10', '95031');
INSERT INTO student VALUES('106', '陆军', '男', '1974-06-03', '95031');
INSERT INTO student VALUES('107', '王尼玛', '男', '1976-02-20', '95033');
INSERT INTO student VALUES('108', '张全蛋', '男', '1975-02-10', '95031');
INSERT INTO student VALUES('109', '赵铁柱', '男', '1974-06-03', '95031');

#添加教师表数据
INSERT INTO teacher VALUES('804', '李诚', '男', '1958-12-02', '副教授', '计算机系');
INSERT INTO teacher VALUES('856', '张旭', '男', '1969-03-12', '讲师', '电子工程系');
INSERT INTO teacher VALUES('825', '王萍', '女', '1972-05-05', '助教', '计算机系');
INSERT INTO teacher VALUES('831', '刘冰', '女', '1977-08-14', '助教', '电子工程系');

#添加课程表数据
INSERT INTO course VALUES('3-105', '计算机导论', '825');
INSERT INTO course VALUES('3-245', '操作系统', '804');
INSERT INTO course VALUES('6-166', '数字电路', '856');
INSERT INTO course VALUES('9-888', '高等数学', '831');

#添加成绩表数据
INSERT INTO score VALUES('103', '3-105', '92');
INSERT INTO score VALUES('103', '3-245', '86');
INSERT INTO score VALUES('103', '6-166', '85');
INSERT INTO score VALUES('105', '3-105', '88');
INSERT INTO score VALUES('105', '3-245', '75');
INSERT INTO score VALUES('105', '6-166', '79');
INSERT INTO score VALUES('109', '3-105', '76');
INSERT INTO score VALUES('109', '3-245', '68');
INSERT INTO score VALUES('109', '6-166', '81');
```

#### 查询练习

##### （1-10简单查询）

```mysql
#1.查询student表中所有记录。
select * from student;
#2.查询 student 表中所有记录的sname、ssex和class列。
select sname,ssex,class from student;
#3.查询教师所有的单位即不重复的depart列。
select distinct depart from teacher;
#4.查询score表中成绩在60到80之间的所有记录。
select * from score where degree>=60 and degree<=80;
#5.查询score表中成绩为85，86或88的记录。
select * from score where degree=85 or degree=86 or degree=88;
#6.查询student表中“95031"班或性别为“女”的同学记录。
select * from student where class='95031' or ssex='女';
#7.以class降序查询student表的所有记录。
select * from student order by class desc;
#8.以cno升序、degree降序查询score表的所有记录。
select * from score order by cno asc,degree desc;
#9.查询“95031"班的学生人数。
select count(*) from student where class='95031';
#10.查询score表中的最高分的学生学号和课程号。(子查询或者排序,排序会有bug，不建议)
select sno,cno from score where degree=(select max(degree) from score);
```

##### 11.查询每门课的平均成绩

```mysql
#avg()
select avg(degree) from score where cno='3-105';
select avg(degree) from score where cno='3-166';
select avg(degree) from score where cno='3-245';
#group by --分组(可以实现一条语句显示)
select cno,avg(degree) from score group by cno;
+-------+-------------+
| cno   | avg(degree) |
+-------+-------------+
| 3-105 |     85.3333 |
| 3-245 |     76.3333 |
| 6-166 |     81.6667 |
+-------+-------------+
```

##### 分组条件与模糊查询

```mysql
#12.查询 score 表中至少有 2 名学生选修，并以 3 开头的课程的平均分数。
select cno,avg(degree) from score group by cno
having count(cno)>2 and cno like '3%';
+-------+-------------+
| cno   | avg(degree) |
+-------+-------------+
| 3-105 |     85.3333 |
| 3-245 |     76.3333 |
+-------+-------------+
#like是模糊查询，%代表以三开头。
```

##### 范围查询的两种方式

```mysql
#13.查询分数大于70，小于90的sno列
--方法一
select sno,degree from score where degree>70 and degree<90;
+-----+--------+
| sno | degree |
+-----+--------+
| 103 |     86 |
| 103 |     85 |
| 105 |     88 |
| 105 |     75 |
| 105 |     79 |
| 109 |     76 |
| 109 |     81 |
+-----+--------+
--方法二
select sno,degree from score where degree between 70 and 90;
--结果同上
```

##### 多表查询

```mysql
#查找时按照不同表中的共同字段进行匹配，然后进行查找。

#14.查询所有学生的sname、cno和degree列。
--sname来自于student表，cno、degree来自于score表
select sname,cno,degree from student,score where student.sno=score.sno;
+-----------+-------+--------+
| sname     | cno   | degree |
+-----------+-------+--------+
| 王丽      | 3-105 |     92 |
| 王丽      | 3-245 |     86 |
| 王丽      | 6-166 |     85 |
| 王芳      | 3-105 |     88 |
| 王芳      | 3-245 |     75 |
| 王芳      | 6-166 |     79 |
| 赵铁柱    | 3-105 |     76 |
| 赵铁柱    | 3-245 |     68 |
| 赵铁柱    | 6-166 |     81 |
+-----------+-------+--------+


#15.查询所有学生的sno、cname和degree列。
cname来自于course表，sno、degree来自于score表
select sno,cname,degree from course,score where course.cno=score.cno;
+-----+-----------------+--------+
| sno | cname           | degree |
+-----+-----------------+--------+
| 103 | 计算机导论      |     92 |
| 105 | 计算机导论      |     88 |
| 109 | 计算机导论      |     76 |
| 103 | 操作系统        |     86 |
| 105 | 操作系统        |     75 |
| 109 | 操作系统        |     68 |
| 103 | 数字电路        |     85 |
| 105 | 数字电路        |     79 |
| 109 | 数字电路        |     81 |
+-----+-----------------+--------+
#16.查询所有学生的sname、cname和degree列（三表关联查询）。
--两两关联即可
--sname->student
--cname->course
--degree->score
select sname,cname,degree from student,course,score 
where student.sno=score.sno and course.cno=score.cno;
+-----------+-----------------+--------+
| sname     | cname           | degree |
+-----------+-----------------+--------+
| 王丽      | 计算机导论      |     92 |
| 王芳      | 计算机导论      |     88 |
| 赵铁柱    | 计算机导论      |     76 |
| 王丽      | 操作系统        |     86 |
| 王芳      | 操作系统        |     75 |
| 赵铁柱    | 操作系统        |     68 |
| 王丽      | 数字电路        |     85 |
| 王芳      | 数字电路        |     79 |
| 赵铁柱    | 数字电路        |     81 |
+-----------+-----------------+--------+
#22.查询选修某课程的同学人数大于2的任课教师姓名
--counting(*)的意思是 统计前面分组之后每个组有多少条记录
--记录条数大于2就代表选修人数大于2
--counting(*)的使用条件是必须先分组。
select cno from score group by cno having count(*)>2;
+-------+
| cno   |
+-------+
| 3-105 |
| 3-245 |
| 6-166 |
+-------+
--先用一次嵌套找到tno
select tno from course where cno in(select cno from score group by cno having count(*)>2);
+-----+
| tno |
+-----+
| 804 |
| 825 |
| 856 |
+-----+
--接着使用三层嵌套
select tname from teacher where tno in (select tno from course where cno in(select cno from score group by cno having count(*)>2));
+--------+
| tname  |
+--------+
| 李诚   |
| 王萍   |
| 张旭   |
+--------+

```

##### 子查询加分组求平均

```mysql
#查询95031班学生每门课的平均分
--思路：首先要把score里是95031班的学生找出来
select sno from student where class='95031';
+-----+
| sno |
+-----+
| 102 |
| 105 |
| 106 |
| 108 |
| 109 |
+-----+
--再使用一个子查询
select * from score where sno in (select sno from student where class='95031');
+-----+-------+--------+
| sno | cno   | degree |
+-----+-------+--------+
| 105 | 3-105 |     88 |
| 105 | 3-245 |     75 |
| 105 | 6-166 |     79 |
| 109 | 3-105 |     76 |
| 109 | 3-245 |     68 |
| 109 | 6-166 |     81 |
+-----+-------+--------+
--这样就找到了95031班的学生成绩
--接着进行分组计算平均成绩，分组的依据是课程号
select cno,avg(degree) from score where sno in (select sno from student where class='95031') group by cno;
+-------+-------------+
| cno   | avg(degree) |
+-------+-------------+
| 3-105 |     82.0000 |
| 3-245 |     71.5000 |
| 6-166 |     80.0000 |
+-------+-------------+
```

##### 子查询

```mysql
#18.查询在 3-105 课程中，所有成绩高于 109 号同学的记录。
--首先筛选出课堂号为 3-105 ，学号为 109 号同学的degree。
select degree from score where cno='3-105' and sno='109';
+--------+
| degree |
+--------+
|     76 |
+--------+
--再把这个条件添加到下面括号中
select * from score where degree>(select degree from score where cno='3-105' and sno='109') and cno='3-105';
+-----+-------+--------+
| sno | cno   | degree |
+-----+-------+--------+
| 103 | 3-105 |     92 |
| 105 | 3-105 |     88 |
+-----+-------+--------+
--这就是学生的记录
#19题过于简单，略。
```

##### YEAR 函数与带 IN 关键字的子查询

```mysql
#20.查询所有和 101 、108 号学生同年出生的sno、sname、sbirthday 列。
--查询这两人的信息
select * from student where sno in (108,101);
+-----+-----------+------+---------------------+-------+
| sno | sname     | ssex | sbirthday           | class |
+-----+-----------+------+---------------------+-------+
| 101 | 曾华      | 男   | 1977-09-01 00:00:00 | 95033 |
| 108 | 张全蛋    | 男   | 1975-02-10 00:00:00 | 95031 |
+-----+-----------+------+---------------------+-------+
--查询这两人的出生年份
select year(sbirthday) from student where sno in (108,101);
+-----------------+
| year(sbirthday) |
+-----------------+
|            1977 |
|            1975 |
+-----------------+
--将这两人的出生年份作为条件，放到下面的括号中
select sno,sname,sbirthday from student 
where year(sbirthday) in (select year(sbirthday) from student where sno in (108,101));
+-----+-----------+---------------------+
| sno | sname     | sbirthday           |
+-----+-----------+---------------------+
| 101 | 曾华      | 1977-09-01 00:00:00 |
| 102 | 匡明      | 1975-10-02 00:00:00 |
| 105 | 王芳      | 1975-02-10 00:00:00 |
| 108 | 张全蛋    | 1975-02-10 00:00:00 |
+-----+-----------+---------------------+
#ps： 用in是因为108,101是两个数据，不是单一数据。
```

##### 多层嵌套子查询（套娃）

```mysql
#查询‘张旭’教师任课的学生成绩
--思路：找到张旭的教师编号tno（条件1），再找tno对应的course，用course表中的cno匹配score中的cno（条件2）
--条件1
select tno from teacher where tname='张旭';
+-----+
| tno |
+-----+
| 856 |
+-----+
--条件2
select cno from course where tno=(select tno from teacher where tname='张旭');
+-------+
| cno   |
+-------+
| 6-166 |
+-------+
--得到学生学号对应的degree
select sno,degree from score where cno=(select cno from course where tno=(select tno from teacher where tname='张旭'));
+-----+--------+
| sno | degree |
+-----+--------+
| 103 |     85 |
| 105 |     79 |
| 109 |     81 |
+-----+--------+
```

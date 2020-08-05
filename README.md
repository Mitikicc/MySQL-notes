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






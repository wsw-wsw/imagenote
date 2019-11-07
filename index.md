#MySQL

## char和varchar数据类型的区别

char和varchar都用来表示字符串数据

### 1. 主要区别
```
varchar长度可变, 比如varchar(10), 如果存储'wsw', 会占用3个字符的长度;
char长度不可变, 比如char(10), 如果存储''wsw', 会占用10个字符的长度, 
'wsw'后面会跟7个空格, 取数据的时候，char类型的要用trim()去掉多余的空格，而varchar是不需要的。
尽管如此，char的存取速度还是要比varchar要快得多，因为其长度固定，方便程序的存储与查找；但是char因为其长度固定，所以会有多余的空格占据空间，是以空间换取时间效率的，而varchar是以空间效率为首位的。
```

### 2. 长度是字节数还是字符数
```
MySQL4.0以前, char(20)和varchar(20)指的是20个字节长度, 如果存放utf8汉字(3个字节)时, 只能存放6个
5.0版本以上, char(20)和varchar(20)指的是20个字符长度, 无论存放数字/字母/utf8汉字(3个字节), 都可以存放20个
```

### 3. 最大存储
```
# char
char范围是0-255, 可以存储255个‘字符’
# varchar
MySQL要求一个行定义长度不能超过65535个‘字节’.

MySQL对于varchar会有1-2个字节来保存字符长度, 当字符数小于等于255时, MySQL只用1个字节来记录, 因为2的8次方减1只能存到255, 当字符数多于255就用2个字节.

英文字符和数字占1个字节, GBK字符集每个汉字占两个字节, UTF8字符集每个汉字占三个字节

GBK 一个字符占1-2个字节, varchar最多能存 32766 个字符(65535-1-2 / 2)
UTF8 一个字符占1-3个字节, varchar最多能存 21844 个字符(65535-1-2 / 3)
减1的原因是实际存储从第二个字节开始, 减2的原因是varchar头部的2个字节表示长度

举例说明:
若一个表定义为
create table t4(c int, c2 char(30), c3 varchar(N)) charset=utf8;
则此处N的最大值为(65535-1-2-4-30*3)/3=21812
减1和减2的原因与a)相同, 减4的原因是int类型的c占4个字节, 减30*3的原因是char(30)占用90个字节, 编码是utf8 
```
##MyISAM和InnoDB的区别
```
1. InnoDB支持事务; MyISAM不支持
2. MyISAM强调的是性能, 查询速度比InnoDB快; InnoDB适合频繁修改以及涉及到安全性较高的应用
3. InnoDB支持外键; MyISAM不支持
4. MySQL5.5以后, 默认存储引擎是InnoDB
5. InnoDB不支持全文索引; MyISAM支持全文索引 #5.6版本之后都支持了
6. MyISAM仅支持表锁,并发较小; InnoDB支持行锁, 并发较大
7. 使用delete删除表的时候, InnoDB是逐行删除; 而MyISAM是先DROP表, 然后重新建表, MyISAM的效率快
8. 对于select count(*) from 表名;
   MyISAM因为保存了表的行数可以直接取出;
   而InnoDB会遍历整个表来计算行数;
   但是对于加了WHERE条件, select count(*) from 表名 WHERE 条件; MyISAM和InnoDB都会遍历整个表来计算行号
```
## mysqldump命令

mysqldump命令用于备份数据库

```
# 格式
mysqldump -h主机地址 -u用户名 -p密码 数据库名 > 文件名
# Example
mysqldump -uroot -proot mysql > /mysql_bak.sql
```
## MySQL事务的4个特性

什么是事务?
```
事务是针对数据库的一组操作, 由一条或多条sql组成, 事务中的sql要么都执行, 要么都不执行
```

事务的4个特性(ACID)?
```
事务必须同时满足4个特性: 原子性、一致性、隔离性、持久性

# 原子性
原子性指一个事务多条SQL语句组成, 要么全部执行成功，要么全部不执行。只要其中一个指令执行失败，所有的指令都执行失败，数据进行回滚，回到执行指令前的数据状态。

# 一致性
一致性指事务执行结束后, 数据库的完整性约束没有被破坏, 事务执行的前后都是合法的数据状态.
例如，账户转钱，A账户有100;B账户有100，不管他们怎么转账，事务结束后两个用户的钱相加起来应该还得是200，这就是事务的一致性。 
数据的完整性约束包括:
    实体完整性(如行的主键存在且唯一)
    列完整性(如字段的类型/大小/长度符合要求)
    外键约束
    用户自定义完整性(如转账前后的两个账户余额和保持不变)
    
# 隔离性
多个并发事务之间互不干扰, 相互隔离.
比如A正在从一张银行卡中取钱，在A取钱的过程结束前，B不能向这张卡转账。A读100，B也读100，A-，B+，B+的这笔钱就不见。

# 持久性
事务一旦提交, 其所做的修改就会永久保存到数据库中, 即使数据库发生故障也不会对其有任何影响.
```

## TINYINT(M)中M表示的含义是什么

### TINYINT(M)中M表示的含义是什么, 比如TINYINT(1)  
```
M的含义并不是允许存入的字符长度, 无符号TINYINT(1)可以存入其最大值255, 
而且存入的数值无论是0还是255, 占用的字节大小是固定的.
tinyint  smallint mediumint int bigint 12348
整型的字节数已经限制了存入数值的取值范围, M值在这里不会有任何影响, 
比如TINYINT(1)和TINYINT(2)没有区别
ALTER TABLE test MODIFY id1 int(2) zerofill;
如果设置zerofill(左前位置零填充), 对于TINYINT(1)和TINYINT(2), 
如果存入1, TINYINT(1)存入的是1, 而TINYINT(2)存入的是01
也就是说,没有zerofill,M值是没用的


所以在设计字段的时候, mysql会自动分配长度: tinyint(4)、smallint(6)、mediumint(9)、int(11)、bigint(20)
就用这些默认长度就可以了, 不用自己搞int(10), tinyint(1)之类的, 基本没什么用而且导致表的字段类型多样化
TINYINT(M)、SMALLINT(M)、MEDIUMINT(M)、INT(M)、BIGINT(M)是同理的,
没有zerofill, M值是没用的
```
## 表的约束

为了防止数据表中插入错误的数据, 在MySQL中, 定义了一些维护数据库完整性的规则, 即表的约束.

### 常见的约束
```
PRIMARY KEY 主键约束
FOREIGN KEY 外键约束
NOT NULL 非空约束
UNIQUE 唯一性约束(在MySQL中, 唯一性约束和唯一索引概念不同, 但是实际效果相同)
DEFAULT 默认值约束
```
## 浮点型和定点型

### 基本概念
```
float和double都是浮点型, 而decimal是定点型
MySQL浮点型和定点型可以用类型名称后加(M,D)来表示, M表示该值的总共长度, D表示小数点后面的长度, M和D又称为精度和标度
如float(7,4), 最大可存入-999.9999, 如果尝试存入999.00009, 则存入结果为999.0001

浮点型和定点型的区别:
浮点型在一些情况下数据库中存放的是近似值, 而定点型存放的永远都是精确值
浮点型不能做精确的计算, 其计算结果不准确, 只是近似, 而定点型可以做精确的计算
```

### float默认保存6位精度(包括小数位和整数位), 超过6位会被四舍五入并补入0
```
mysql > insert into float_test values(1, 123456),(2, 123.456),(3, 1234567),(4,1234.5678);
mysql > select * from float_test;
+------+------------+
| id   | float_test |
+------+------------+
|    1 |     123456 |
|    2 |    123.456 |
|    3 |     123570 |
|    4 |    1234.57 |
+------+------------+
```

## 事务的隔离级别

MySQL是多线程并发访问的, 所以很容易出现多个线程同时开启事务的情况, 这样就会出现脏读、不可重复读、幻读的情况.  
```
# 脏读
    # 定义
    一个事务读取了另一个事务未提交的数据
    # 场景
    用户A操作一个数，读出100这个值；减去10；写回去90；
    这时用户B也来读出这个数读90，后用户A又回滚操作，数据变回了100.
    用户就读了一个脏数据。

# 不可重复读
    # 定义
    在一个事务中两次查询的结果不一致, 原因是在查询的过程中其他事务做了更新的操作
    # 场景
    银行做统计报表时, 第一次查询a账户有1000元, 第二次查询a账户有900元,
    原因是统计期间a账户取出了100元, 这样导致多次统计报表的结果不一致

# 幻读
    # 定义
    又称虚读, 和不可重复读有些类似, 事务中两次查询的结果不一致, 原因是在查询过程中其他事务做了插入的操作
    # 场景
    银行做统计报表时, 统计account表中所有用户的总额时, 此时总共有3个账户, 总金额有3000,
    这时新增了一个账户并且存入1000元, 这时银行发现账户总金额变成了4000, 造成了幻读.
#更新丢失
    丢失更新就是两个不同的事务在某一时刻对同一数据进行读取后，先后进行修改，后者更新覆盖了前面所作的更新。导致第一次操作数据丢失。
```
1. 不可重复读和脏读的区别是，脏读是读取前一事务未提交的脏数据，不可重复读是重新读取了另一事务已提交的数据
2. 幻多是读到之前没有出现过的数据(新增或删除)，不可重复读是同一条数据两次读的结果不一样。
为了避免这些情况的发生, 就需要为事务设置隔离级别, 在MySQL中, 事务有4种隔离级别.
```
# READ UNCOMMITTED(读未提交)
事务中最低级别, 该级别下的事务可以读到另一个事务未提交的事务, 也称为脏读, 无法避免所有读的问题.

# READ COMMITTED(读提交)
可以避免脏读, 但是不能避免不可重复读和幻读

# REPEATABLE READ(可重复读)
MySQL默认的事务隔离级别, 可避免脏读和不可重复读

# SERIALIZABLE(可串行化)
事务的最高隔离级别, 可避免脏读/不可重复读/幻读
如果一个事务隔离级别是可串行化, 它在每个读的数据行上加上锁,
在这个事务提交之前, 其他线程只能等到当前操作完成后才能进行操作
```
##手写建表语句和sql

一张雇员表employee, 一张部门表department, 结构如下, 写出建表语句  

id|emp_name|dept_id
--|--|--
1|张三|1
2|李四|1
3|王五|2

id|dept_name
--|--
1|售前
2|客服
3|开发

```sql
# employee表
CREATE TABLE `employee` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `emp_name` varchar(255) NOT NULL DEFAULT '',
  `dept_id` int(11) unsigned DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

# department表
CREATE TABLE `department` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `dept_name` varchar(255) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

如果要查询出如下结果, 请写出sql语句  

id|dept_name|num
--|--|--
1|售前|2
2|客服|1
3|开发|0

```sql
SELECT
	dept.id,
	dept.dept_name,
	(
		SELECT
			count(emp.id)
		FROM
			employee emp
		WHERE
			emp.dept_id = dept.id
	) AS num
FROM
	department dept;
```
## 数据库三范式
NF: Normal Form, 范式

1NF指的是表的属性(即列)具有原子性, 不可再分

2NF指的是在1NF的基础上, 不能有部分依赖

3NF指的是在2NF的基础上, 非主键字段和主键字段不能产生传递依赖

### 1NF
1NF, 第一范式
```
# 概念
1NF指的是表的属性(即列)具有原子性, 不可再分

# 反例
id          name            mail_sex
1           wyx             wyx@163.com_男

# 正例
id          name            mail            sex
1           wyx             wyx@163.com     男
```

### 2NF
2NF, 第二范式
```
# 概念
2NF指的是在1NF的基础上, 不能有部分依赖

(部分依赖的前提是有组合主键, 也就是非主键属性不能只依赖部分主键)
如果是单一主键, 则符合第二范式
# 反例(该表有两个主键, 学生编号和教师编号, 但是学生姓名只依赖于学生编号, 教师姓名只依赖于教师编号, 这就是部分依赖)
student_id(PK)          teacher_id(PK)          student_name            teacher_name
1001                    001                     zhangsan                wanglaoshi

# 正例(应该拆分成两张表)
student_id(PK)          student_name          
1001                    zhangsan
teacher_id(PK)          teacher_name          
001                     wanglaoshi              
```

### 3NF
3NF, 第三范式
```
# 概念
3NF指的是在2NF的基础上, 非主键字段和主键字段不能产生传递依赖

# 反例(classname依赖于classid, classid依赖于主键id, 这就是传递依赖)
id(PK)          name          classid            classname
1               wyx           101                1班

# 正例
id(PK)          name          classid(FK)            
1               wyx           101              
classid(PK)     classname
101             1班                                 
```

### 反三范式
```
# 概念
通常情况下表的设计要按照3NF, 有时候会例外, 反三范式会提高查询的效率

# Example
现在有两个表, 一个相册表, 一个图片表, 一个相册表中包含很多图片,
相册表中的view(浏览次数)是可以遍历图片表推导出来的, 但是这样会造成查询效率很低,
所以反三范式, 在相册表中加一个view属性, 提高了查询效率.

id(PK)          name          view          
1               相册1          30  
id(PK)          name          view            albumid(FK)
10              1.jpg         30              1 
```




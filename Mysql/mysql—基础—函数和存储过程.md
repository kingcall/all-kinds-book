[toc]
# 在mysql 内部存储代码


## 自定义变量

### 自定义变量的不足
- 使用自定义变量的查询不能缓存(8.0 之后都不能缓存)
- 自定义变量的生命周期是在一个连接中有效，所以不能用来做连接间的通信
- 不能显式的声明自定义变量的类型
- 使用未定义的变量不会产生任何语法错误

### 例子
#### 排名
```
set @num:=0;
SELECT actor_id,@num:=@num+1 as rank,cnt from (SELECT  actor_id,count(*) as cnt  from film_actor group by actor_id order by cnt desc limit 10) a;
```
- 对分数相同人设置相同的名次
```

set @rank:=0;
set @prew=0;
set @cur=0;

SELECT
	actor_id,
	@cur:=cnt as cnt,
	@rank := IF(@cur=@pre,@rank,@rank + 1) as rank,
	@pre:=@cur
FROM
	( 
		SELECT 
			actor_id, count( * ) AS cnt 
		FROM 
			film_actor 
		GROUP BY 
			actor_id 
		ORDER BY 
			cnt 
		DESC LIMIT 
			10 
	) 
a;
```
#### 更新数据的时候同时获取
```
update t1 set lasttime=now() where id=1 and @last:=now();
select @last
```



![image-20201127220553288](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/22:05:53-image-20201127220553288.png)

#### 统计更新和插入的数量
#### 确定取值的顺序

## 函数
- 函数与存储过程的区别：函数只会返回一个值，不允许返回一个结果集。函数强调返回值，所以函数不允许返回多个值的情况，即使是查询语句

### 使用自定义函数的配置
- SET GLOBAL log_bin_trust_function_creators = 1;
- 在MySQL配置文件my.ini或my.cnf中的[mysqld]段上加log-bin-trust-function-creators=1

### 声明函数的方法1
```
CREATE FUNCTION 函数名(参数 类型,[参数 类型,...])
RETURNS 返回类型 RETURN 表达式值
```
- 注意
- 这种方式不能使用任何SQL语句

### 声明函数的方法2
```
CREATE FUNCTION 函数(参数 类型,[参数 类型,...])
RETURNS 返回类型
BEGIN
END;
```
- 如果要在函数体中可以使用更为复杂的语法，比如复合结构/流程控制/任何SQL语句/定义变量等。带复合结构的函数体的自定义函数的

### 例子
#### 获得月份
```
CREATE FUNCTION `getMonth`(mytime TIMESTAMP) RETURNS INT
BEGIN
	DECLARE y INT;
	DECLARE m INT;
	SET y= SUBSTRING(mytime,1,4);
	SET m= SUBSTRING(mytime,6,2);
	RETURN y*12+m;
END
```
## 存储过程
- 存储过程思想上很简单，就是数据库 SQL 语言层面的代码封装与重用。
- 默认情况下，存储过程和默认数据库相关联，如果想指定存储过程创建在某个特定的数据库下，那么在过程名前面加数据库名做前缀。
- 在定义过程时，使用 DELIMITER $$ 命令将语句的结束符号从分号 ; 临时改为两个 $$，使得过程体中使用的分号被直接传

###  存储过程的优势
- 在服务器内部执行，离数据最近
- 这是一种代码重用，方便统一业务规则
- 简化代码的维护和版本更新

### 存储过程的不足
- 迁移的时候需要部署存储过程——扩展性会降低
- 存储过程的性能难以监控

### 例子
```
delimiter //　　#将语句的结束符号从分号;临时改为两个$$(可以是自定义)
CREATE PROCEDURE delete_matches(IN loops INT)
BEGIN
　　 DECLARE v1 INT;
		set v1=loops;
		WHILE v1>0 DO
			INSERT INTO test(id) VALUES(v1);
			SET v1=v1 - 1;
END;
//
delimiter ; #将语句的结束符号改回来
```

```
delimiter ;;
create procedure idata()
begin 
	declare i int; 
	set i=1; 
	while(i<=100000) do 
		insert into t values(i,i); 
		set i=i+1; 
	end while;
end;;
delimiter ;
```

## 触发器
- 触发器是与表有关的数据库对象，在满足定义条件时触发，并执行触发器中定义的语句集合。
- 在执行一些特定的操作的时候，mysql 可以指定触发器是在sql 执行之前还是之后执行



![image-20201127220632268](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/22:06:32-image-20201127220632268.png)

- 触发器是针对每一行的；对增删改非常频繁的表上切记不要使用触发器，因为它会非常消耗资源。 



### 适用场景

- 用来更新或者维护一些统计结果
- 实现一些约束、以及更新反范式的数据

### 不足
- 针对行的触发是它的最大限制——性能
- 触发器掩盖了服务背后的工作，导致出了问题很难排查
- 触发器可能导致死锁和锁等待，触发器失败那么原来的sql 也会失败

### NEW与OLD详解
- MySQL 中定义了 NEW 和 OLD，用来表示触发器的所在表中，触发了触发器的那一行数据，来引用触发器中发生变化的记录内容
- 在INSERT型触发器中，NEW用来表示将要（BEFORE）或已经（AFTER）插入的新数据；
- 在UPDATE型触发器中，OLD用来表示将要或已经被修改的原数据，NEW用来表示将要或已经修改为的新数据；
- 在DELETE型触发器中，OLD用来表示将要或已经被删除的原数据；

### 例子
- trigger_time: { BEFORE | AFTER }
- trigger_event: { INSERT | UPDATE | DELETE }
- trigger_order: { FOLLOWS | PRECEDES } other_trigger_name
```
trigger_order是MySQL5.7之后的一个功能，用于定义多个触发器，使用follows(尾随)或precedes(在…之先)来选择触发器执行的先后顺序
```
#### 单语句的例子
```
CREATE TRIGGER 
    t_k 
AFTER INSERT
ON 
    test 
FOR EACH ROW 
    INSERT INTO time VALUES(NOW());
```
#### 多语句的例子
```
CREATE TRIGGER t_k2 AFTER INSERT
ON test FOR EACH ROW 
BEGIN
	INSERT INTO time VALUES(NOW());
	INSERT INTO time VALUES(NOW());
end;
```

delimiter $$
CREATE TRIGGER 
    upd_check 
BEFORE UPDATE 
ON 
    account
FOR EACH ROW
BEGIN
　　IF NEW.amount < 0 THEN SET NEW.amount = 0;
　　ELSEIF NEW.amount > 100 THEN SET NEW.amount = 100;
　　END IF;
END$$
delimiter ;

## 事件
- 熟悉linux系统的人都知道linux的cron计划任务，能很方便地实现定期运行指定命令的功能。其实Mysql事件就和Linux的cron功能一样，Mysql在5.1以后推出了事件调度器(Event Scheduler)，能方便地实现 mysql数据库的计划任务，而且能精确到秒。
- 查看事件服务是否开启 show variables like 'event_scheduler';
- SET GLOBAL event_scheduler = ON; 开启事件服务

### 例子
#### 每隔一段时间插入数据
```
DROP EVENT IF EXISTS `event_minute`;
DELIMITER ;;
CREATE DEFINER=`root`@`localhost` EVENT `event_minute` ON SCHEDULE EVERY 1 MINUTE STARTS '2018-06-20 20:00:00' ON COMPLETION NOT PRESERVE ENABLE DO 

BEGIN
    INSERT INTO USER(name, address,addtime) VALUES('test1','test1',now());
    INSERT INTO USER(name, address,addtime) VALUES('test2','test2',now());
END
;;
DELIMITER ;
```
#### 特定时间插入数据
```
DROP EVENT IF EXISTS `event_at`;
DELIMITER ;;
CREATE DEFINER=`root`@`localhost` EVENT `event_at` ON SCHEDULE AT '2018-06-21 15:37:00' ON COMPLETION NOT PRESERVE ENABLE DO 

BEGIN
    INSERT INTO USER(name, address,addtime) VALUES('AT','AT',now());
END
;;
DELIMITER ;
```
#### 更新统计结果   
```
DELIMITER ;;
CREATE  EVENT `update_result` ON SCHEDULE EVERY 1 MINUTE
DO
BEGIN
	INSERT INTO result SELECT count(*),now() from USER;
END;;
DELIMITER ;
```
#### 更新统计结果
```
CREATE  EVENT 
    `update_bi_rt_child_student_refer_act_week` 
ON SCHEDULE EVERY 5 MINUTE 
DO
replace into bi_rt_child_student_refer_act_week(stu_id,act_cnt) select user_id,count(*) as act_cnt from t_screenshot where bu=2 group by user_id ;
```

## 变量绑定(prepared statment)
### 普通变量绑定
### sql 接口变量绑定
## 其他
### 函数与存储过程的区别
- 存储过程可以有多个in,out,inout参数，而函数只有输入参数类型，而且不能带in.
- 存储过程实现的功能要复杂一些；而函数的单一功能性（针对性）更强。
- 存储过程可以返回多个值；存储函数只能有一个返回值。
- 存储过程一般独立的来执行；而存储函数可以作为其它sql语句的组成部分来出现。
- 存储过程只能使用SQL来编写，函数可以使使用任何支持C语言调用约定的编程语言
- 


- 存储过程可以调用存储函数。函数不能调用存储过程。

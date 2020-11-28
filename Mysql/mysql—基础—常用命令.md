[toc]
## limit
  - 语句1：select * from student limit 9,4
  - 语句2：slect * from student limit 4 offset 9
    - 语句1和2均返回表student的第10、11、12、13行
    - 语句2中的4表示返回4行，9表示从表的第十行开始

## 索引
- show index from  product_comment; 
- ALTER TABLE index_demo ADD PRIMARY KEY(id) 
- ALTER TABLE projectfile ADD UNIQUE INDEX (fileuploadercode);

## 查询
-  select count(*),sum(film_id=1) as c1,sum(act_id=1) as c2 from film_actor where film_id=1 or act_id=1;

## 导入导出

### 数据导出
- mysql -h127.0.0.1 -uroot -pwww1234 -e "select * from user" kingcall > D:/test.txt
- mysql -h127.0.0.1 -uroot -pwww1234 -N -e "select * from kingcall.user " > D:/test2.txt 不输出列名等相关信息
- mysql -h127.0.0.1 -uroot -pwww1234 -N -e "select * from kingcall.user  into outfile 'D:/test3.txt' fields terminated by ','"

#### 库备份
- mysqldump  -u 用户名 -p databasename >exportfilename导出数据库到文件
- mysqldump -uroot -p123456 -A >F:\all.sql 备份全部数据库
- mysqldump -uroot -p123456 -A -d>F:\all_struct.sql 备份全部数据库结构
- mysqldump -uroot -p123456 --databases db1 db2 >f:\muldbs.sql 备份多个数据库
- mysqldump -uroot -p123456 mydb -d>F:\mydb.sql 备份指定库的结构
- 


### 数据导入
- mysql -uroot -pwww1234 -Dkingcall<D:/2.sql
- 

## 进程
- SHOW PROCESSLIST
-  kill connection 2274;

## 关联子查询
- 对于外部查询返回的每一行数据，内部查询都要执行一次。在关联子查询中是信息流是双向的。外部查询的每行数据传递一个值给子查询，然后子查询为每一行数据执行一次并返回它的记录。然后，外部查询根据返回的记录做出决策。


### 关联删除
```
DELETE A FROM YSHA A LEFT JOIN YSHB B ON A.code=b.code WHERE b.code is NULL
```
```
DELETE FROM YSHA WHERE NOT EXISTS(SELECT 1 FROM YSHB B WHERE YSHA.code=b.code )
```

## semi-join
- 所谓的semi-join是指semi-join子查询。 当一张表在另一张表找到匹配的记录之后，半连接（semi-jion）返回第一张表中的记录。与条件连接相反，即使在右节点中找到几条匹配的记录，左节点 的表也只会返回一条记录。另外，右节点的表一条记录也不会返回。半连接通常使用IN  或 EXISTS 作为连接条件。
```
SELECT ... FROM outer_tables WHERE expr IN (SELECT ... FROM inner_tables ...) AND ...
```
- 种查询的特点是我们只关心outer_table中与semi-join相匹配的记录。
换句话说，最后的结果集是在outer_tables中的，而semi-join的作用只是对outer_tables中的记录进行筛选。这也是我们进行 semi-join优化的基础，即我们只需要从semi-join中获取到最少量的足以对outer_tables记录进行筛选的信息就足够了。
所谓的最少量，体现到优化策略上就是如何**去重**。
### 为什么要用半连接优化子查询
-对于子查询，其子查询部分相对于父表的每个符合条件的元组，都要把子查询执行一轮。效率低下。用半连接操作优化子查询，是把子查询上拉到父查询中，这样子查询的表和父查询中的表是并列关系，父表的每个符合条件的元组，只需要在子表中找符合条件的元组即可，不需要“父表的每个符合条件的元组，都要把子查询执行一轮”，所以效率提高——遍历一遍——> 找到符合即可

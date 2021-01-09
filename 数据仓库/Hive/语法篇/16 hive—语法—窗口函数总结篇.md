**窗口函数应用场景：**

**（1）用于分区排序**

**（2）动态Group By**

**（3）Top N** 

**（4）累计计算**

**（5）层次查询**

 hive中一般取top n时，row_number(),rank,dense_rank()这三个函数就派上用场了，先简单说下这三函数都是排名的，不过呢还有点细微的区别。 通过代码运行结果一看就明白了。

求占比前30%





1. sum(col) over() :  分组对col累计求和，over() 中的语法如下
2. count(col) over() : 分组对col累计，over() 中的语法如下
3. min(col) over() : 分组对col求最小
4. max(col) over() : 分组求col的最大值
5. avg(col) over() : 分组求col列的平均值
6. first_value(col) over() : 某分区排序后的第一个col值
7. last_value(col) over() : 某分区排序后的最后一个col值
8. lag(col,n,DEFAULT) : 统计往前n行的col值，n可选，默认为1，DEFAULT当往上第n行为NULL时候，取默认值，如不指定，则为NULL
9. lead(col,n,DEFAULT) : 统计往后n行的col值，n可选，默认为1，DEFAULT当往下第n行为NULL时候，取默认值，如不指定，则为NULL
10. ntile(n) : 用于将分组数据按照顺序切分成n片，返回当前切片值。注意：n必须为int类型。
11. 
12. 排名函数：
13. row_number() over() : 排名函数，不会重复，适合于生成主键或者不并列排名
14. rank() over() :  排名函数，有并列名次，名次不连续。如:1,1,3
15. dense_rank() over() : 排名函数，有并列名次，名次连续。如：1，1，2


- hive 可以通过with查询来提高查询性能，因为先通过with语法将数据查询到内存，然后后面其它查询可以直接使用

```sql
with q1 as (select * from src where key= '5'),
q2 as (select * from srcs2 where key = '4')
select * from q1 union all select * from q2;
```
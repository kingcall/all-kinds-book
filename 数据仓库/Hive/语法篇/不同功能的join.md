### 不同功能的Join 

#### semi join

- 小表对大表,是reudce join的变种 map阶段过滤掉不需要join的字段 相当于Hivw SQL加的where过滤

##### 特点

- left semi join 的限制是， JOIN 子句中右边的表只能在 ON 子句中设置过滤条件，在 WHERE 子句、SELECT 子句或其他地方过滤都不行。
- left semi join 是只传递表的 join key 给 map 阶段，因此left semi join 中最后 select 的结果只许出现左表。
- 因为 left semi join 是 in(keySet)的关系，遇到右表重复记录，左表会跳过，而join则会一直遍历。这就导致右表有重复值得情况下 left semi join 只产生一条，join 会产生多条，也会导致 left semi join 的性能更高。
- ![image-20201206214502620](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/21:45:03-image-20201206214502620.png)



## 两种开窗方式区别

patition by是按照一个一个reduce去处理数据的，所以要使用全局排序order by
distribute by是按照多个reduce去处理数据的，所以对应的排序是局部排序sort by




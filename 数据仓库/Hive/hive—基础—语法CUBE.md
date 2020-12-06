1. 根据GROUP BY的维度的所有组合进行聚合。
2. 但是不会改变分组列的顺序   (month,day)= (month,day), (month,null), (null,day), (null,null)             无(day,month)
3. grouping sets可以自由组合实现(所以完全可以实现CUE)
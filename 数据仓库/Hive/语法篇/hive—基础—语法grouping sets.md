- 是一种将多个group by 逻辑写在一个sql语句中的便利写法。
- 根据你指定的维度进行聚合
```
SELECT month,day,
COUNT(DISTINCT cookieid) AS uv,
GROUPING__ID 
FROM lxw1234 
GROUP BY month,day 
GROUPING SETS (month,day,(month,day)) 
ORDER BY GROUPING__ID;
等价于下面这种写法
SELECT month,NULL as day,COUNT(DISTINCT cookieid) AS uv,1 AS GROUPING__ID FROM lxw1234 GROUP BY month 
UNION ALL 
SELECT NULL as month,day,COUNT(DISTINCT cookieid) AS uv,2 AS GROUPING__ID FROM lxw1234 GROUP BY day
UNION All
SELECT month,day,COUNT(DISTINCTcookieid) AS uv,3 AS GROUPING__ID FROMlxw1234 GROUP BY month,day
```

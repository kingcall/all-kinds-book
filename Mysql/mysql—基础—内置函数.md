[toc]
## 时间函数
### 格式化
- DATE_FORMAT(t1.first_class_time,'%Y-%m-%d %H:%i:%s')

### 当前时间
- SELECT NOW(),CURDATE(),CURTIME()

![image](http://note.youdao.com/yws/res/24028/A0681F5FDB184CAE9BEB365674F115AE)

### 前/后
#### 前后(天)
![image](http://note.youdao.com/yws/res/24033/88E18D26CE3E47AF9747CA731A9C330E)
#### 前后(月)
![image](http://note.youdao.com/yws/res/24037/68B94CFB0D5A4CB0A0320E1D309142ED)
### 时间差
- TIMESTAMPDIFF函数，需要传入三个参数，第一个是比较的类型，可以比较FRAC_SECOND、SECOND、 MINUTE、 HOUR、 DAY、 WEEK、 MONTH、 QUARTER或 YEAR几种类型，第二个和第三个参数是待比较的两个时间，比较是后一个时间减前一个时间

#### 绝对差
![image](http://note.youdao.com/yws/res/24046/D2AD8764B6684C1490DBFA5667544DA5)
## 计算函数
- rollup
- repeat





```
update document set url= REGEXP_REPLACE(url,'\\?.*','')
```
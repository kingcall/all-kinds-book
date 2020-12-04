[toc]
## 时间函数
### 格式化

- DATE_FORMAT(t1.first_class_time,'%Y-%m-%d %H:%i:%s')

### 当前时间
- SELECT NOW(),CURDATE(),CURTIME()

![image-20201202171127468](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/02/17:11:28-image-20201202171127468.png)

### 前/后
#### 前后(天)
![image-20201202171147837](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/02/17:11:48-image-20201202171147837.png)
#### 前后(月)
![image-20201202171207986](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/02/17:12:08-image-20201202171207986.png)
### 时间差
- TIMESTAMPDIFF函数，需要传入三个参数，第一个是比较的类型，可以比较FRAC_SECOND、SECOND、 MINUTE、 HOUR、 DAY、 WEEK、 MONTH、 QUARTER或 YEAR几种类型，第二个和第三个参数是待比较的两个时间，比较是后一个时间减前一个时间

#### 绝对差
![image-20201202171234934](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/02/17:12:35-image-20201202171234934.png)
## 计算函数
- rollup
- repeat





```
update document set url= REGEXP_REPLACE(url,'\\?.*','')
```
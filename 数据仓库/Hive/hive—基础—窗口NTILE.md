- NTILE(n)，用于将分组数据按照顺序切分成n片，返回当前切片值，NTILE不支持ROWS BETWEEN，比如 NTILE(2) OVER(PARTITION BY cookieid ORDER BY createtime )
如果切片不均匀，默认增加第一个切片的分布（window 必须被排序）

![image-20201206213934046](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:39:34-image-20201206213934046.png)

- 可以用于将数据分片和求百分比


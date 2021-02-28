## 优化
### 合理设置分区数目
- hive 是真正的物理分区，使用分区可以有效的减少数据量
### 使用列数存储
- 建议使用ORC,相比 Parquet presto 对orc 的优化更好

### join 
- presto 采用的实广播join


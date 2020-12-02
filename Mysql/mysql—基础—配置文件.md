## 配置文件基础
- 查询配置文件的目录 mysql --help | grep 'my.cnf'

## 配置项
### 缓存类配置项
#### key_buffer_size
- mysql 允许创建多个键缓存，每个键缓存都有自己的大小

#### thread_cache_size
#### query_cache_size
#### sort_buffer_size
#### read_buffer_size
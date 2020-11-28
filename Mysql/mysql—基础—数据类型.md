[toc]
## 选择数据类型
### 选择的原则
#### 越小越好
#### 越简单越好
- 整形的操作比字符串的操作代价更低
- 使用mysql 的时间类型来存储时间而不是字符串
- 建议使用整形来存储IP地址

#### 尽量避免NULL
- NULL 值会使得索引优化和其他调优工作变得空难
- InnoDB 使用bit 来存储NULL,但是Myisam 却不是，所以对于稀疏数据不建议使用 Myisam

#### 考虑操作
- 选择数据类型的时候不仅仅要考虑到该类型的存储，还需要考虑到该类型的计算和比较(例如 ENUM 和 BIT 以及 SET)
- 对于ENUM、SET 存储使用整型存储(SET 使用位存储)，但是操作的时候却是转换成字符串(ENUM 不是绝对的，还是要看具体的操作)

### 字符和大小
- varchar(n),n是几就可以存几个字符
- n 代表的是字符个数的多少，不论中文还是英文，但是占用的存储空间大小是不一样的 

#### UTF-8
- UTF-8对中文采用3个字节，对英文采用1个字节

#### GBK
- GBK对中文采用2个字节，对英文采用1个字节

### 例子
#### UUID 的存储
- MySQL有一个UUID（）函数，它使MySQL生成一个UUID值，并以VARCHAR（36）类型的可读形式返回,实际占用空间是2+1*36=38 个字节
- 实际存储的时候可以去到 '-'，然后使用 unhex()函数转化为16个字节的数字，存储在binary(16)中,查询的时候可以使用hex()进行反转

#### IP 地址的存储
- IP 地址经常使用 varchar(15)存储，但是本质上它是32位无符号整数,可以看到就是4个字节——其实int 就可以存储，所以可以使用 inet_aton() 和 inet_ntoa() 进行转化

## 类型详解

### 数值类型
![image-20201127220117733](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/22:01:18-image-20201127220117733.png)
- 关键字INT是INTEGER的同义词，关键字DEC是DECIMAL的同义词。在查看表信息时候，别名会被基础类型取代，也就是说展示的实基础类型的名字(别名只是为了兼容使用习惯)
```
create table test(id integer);
```
![image-20201127220134587](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/22:01:35-image-20201127220134587.png)
- 有无符号对数值类型来说也很重要，可以将范围提升一倍 

### 日期和时间类型

![image-20201127220157651](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/22:01:58-image-20201127220157651.png)
- datetime 和 timestamp 都可以存储日期+时间，但是timestamp 的占用空间更小

#### datetime
- 可以表示的时间范围更大，可以表示从1001 年到 9999 年
- 它把时间封装到一个 YYYYMMDDHHMMSS的整数中，与时区无关

#### timestamp
- 表示的是从1970-01-01 到当前的时间戳
- 受服务器、操作系统、客户端的时区配置影响

### 字符串类型

![image-20201127220219435](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/22:02:20-image-20201127220219435.png)

#### varchar
- 在varchar(M)类型的数据列里，每个值只占用刚好够用的字节再加上一到两个用来记录其长度的字节（即总长度为L+1/2 字节），如果最大长度大于255 则需要两个字节来存储字节长度，否则只需要一个。
- InnoDB 会将过长的varchar 存储为 BLOB
- 适合存储长度变化较大的列，varchar 的实现主要是为了节约存储空间，mysql 总是**根据实际长度去分配空间**
- 存在的问题是更新的时候如果字符串的长度发生了较大的变化，会导致页的分裂、磁盘随机访问、聚簇索引碎片——所以说适合存储不经常变的字段。

#### char
- char 是定长的，mysql 总是**根据定义的字符串长度分配空间**
- 适合存储长度接近的字段，例如MD5值
- 对于经常变动的数据char 比 varchar 更好，因为定长的char 不容易产生碎片
- mysql 会删除char 值末尾的空格(需要注意的是memory 存储引擎只支持定长的行，即所有字段按照最大长度分配空间)

#### Blob
- 为存储大数据量的字符串设计的，采用二进制存储
- BLOB有4种类型:TINYBLOB.BLOB.MEDIUMBLOB和LONGBLOB.它们只是可容纳值的最大长度不同.

#### text
- 为存储大数据量的字符串设计的，采用字符串存储
- TEXT也有4种类型:TINYTEXT,TEXT.MEDIUMTEXT和LONGTEXT.这些类型同BLOB类型一样,有相同的最大长度和存储需求
- 当排序时只使用该列的前max_sort_length个字节.max_sort_length的 默认值是1024;当你想要使超过max_sort_length的字节有意义,对含长值的BLOB或TEXT列使用GROUP BY或ORDER BY的另一种方式是将列值转换为固定长度的对象.标准方法是使用SUBSTRING函数.

#### 枚举值
- 枚举一般枚举的是字符串，枚举整型没有意义，枚举的实际存储是存储的整型
- 枚举的存储一般是存储在表的定义文件 XXX.frm 文件中
- 枚举的不足之处是字符串列表是固定的，查找是必须进行转换才能行，所以存在一定的开销

##### 例子
- create table test(e ENUM('d','c','b','a'));
```
可进行数学运算
```
![image-20201127220248955](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/22:02:49-image-20201127220248955.png)
```
排序是按照其对应的数字进行排序的，而不是字符串
```
![image-20201127220308952](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/22:03:09-image-20201127220308952.png)

### 位数据类型

- 在MySQL5.0 以前BIT是TINYINT 的 同义词
- MySQL 中的位模式字面量一般使用 b'val' 或 0bval 语法，val 是只包含 0 和 1 的二进制值
- 在底层的存储是字符串
- 检索的时候，对于字符串操作返回的是ASCII码对应的字符；对于数字操作返回的是对应的十进制数字

![image-20201127220329256](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/22:03:29-image-20201127220329256.png)

```
9 的二进制字符串 00111001=57，57 对应的ASCII码是9
```
#### 存储空间
- 在Myisam 中，位就是按位的多少来存储的，bit(17)需要3个字节
- 但是在 InnoDB 和 Memory 中就是按最小整数来存储的——TINYINT,bit(17)就需要17个字节，并不会节约存储空间
#### 常用的操作
-  SELECT b'1000001', CHARSET(b'1000001');

![image-20201127220345559](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/22:03:46-image-20201127220345559.png)

### SET 
- SET是一个字符串对象，可以有零或多个值，其值来自表创建时规定的允许的一列值
- Set的容纳范围为64个不同的成员,set其实和枚举差不多，set指定了一个集合范围，在我们插入数据的时候，需要插入在set范围之内的元素，如果插入了未被包含的元素，那么就会发出警告
- set 存储的时候会以一系列的打包位集合存储，这样就有效利用了存储空间(BIT 不一定会真正的节约空间，但是SET 会)
- 不足之处在改变列的定义的时候需要通过alter table 来进行操作，代价比较高
#### 例子
- CREATE TABLE settest (col SET('a', 'b', 'c', 'd'));

![image-20201127220407333](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/22:04:07-image-20201127220407333.png)
- 可以看到它内部是有顺序的,类似布隆过滤器的存储原理
- 查询操作

![image-20201127220422744](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/22:04:23-image-20201127220422744.png)

#### 替代方案(整数列上进行按位操作)
- 优势是改变字段的定义比较方便——代价很小
- 不足之初是每一位的含义可能不是太容易理解，查询语句比较难写
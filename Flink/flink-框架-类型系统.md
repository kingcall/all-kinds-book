[toc]
# 类型系统
## 背景
- 大家都知道现在大数据生态非常火，大多数技术组件都是运行在 JVM 上的，Flink 也是运行在 JVM 上，基于 JVM 的数据分析引擎都需要将大量的数据存储在内存中，这就不得不面临 JVM 的一些问题，比如 Java 对象存储密度较低等。
- 针对这些问题，最常用的方法就是实现一个显式的内存管理，也就是说用自定义的内存池来进行内存的分配回收，接着将序列化后的对象存储到内存块中。

## Flink 的类型系统
- Flink支持任意的Java或是Scala类型。Flink在数据类型上有很大的进步，不需要实现一个特定的接口（像Hadoop中的org.apache.hadoop.io.Writable），**Flink 能够自动识别数据类型**。
    - Flink 通过 Java Reflection 框架分析基于 Java 的 Flink 程序 UDF (User Define Function)的返回类型的类型信息
    - 通过 Scala Compiler 分析基于 Scala 的 Flink 程序 UDF 的返回类型的类型信息。
- Flink DataStream 对像都是强类型的，每一个DataStream对象都需要指定元素的类型，Flink自己底层的序列化机制正是依赖于这些信息对序列化等进行优化。
- 具体来说，在 Flink 底层，**它是使用TypeInformation对象对类型进行描述的，TypeInformation对象定义了一组类型相关的信息供序列化框架使用**。
- Flink 内置了一部分常用的基本类型，对于这些类型，Flink也内置了它们的TypeInformation，用户一般可以直接使用而不需要额外的声明，Flink 自己可以通过类型推断机制识别出相应的类型。
- 但是也会有一些例外的情况，比如，FlinkDataStreamAPI同时支持Java和Scala，ScalaAPI许多接口是通过隐式的参数来传递类型信息的，所以如果需要通过Java调用Scala的API，则需要把这些类型信息通过隐式参数传递过去。
- 另一个例子是 Java 中对泛型存在类型擦除，如果流的类型本身是一个泛型的话，则可能在擦除之后无法推断出类型信息，这时候也需要显式的指定。

### 常见的类型

![image-20210202201104072](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202201104072.png)

#### BasicTypeInfo
- 任意Java 基本类型（装箱的）或 String 类型

#### BasicArrayTypeInfo
- 任意Java基本类型数组（装箱的）或 String 数组

#### WritableTypeInfo
- 任意 Hadoop Writable 接口的实现类。

#### TupleTypeInfo
- 任意的 Flink Tuple 类型(支持Tuple1 to Tuple25)。
- Flink tuples 是固定长度固定类型的Java Tuple实现。

#### CaseClassTypeInfo
- 任意的 Scala CaseClass(包括 Scala tuples)

#### PojoTypeInfo
- 任意的 POJO (Java or Scala)，例如，Java对象的所有成员变量，要么是 public 修饰符定义，要么有 getter/setter 方法。

#### GenericTypeInfo
- 任意无法匹配之前几种类型的类。

![image-20210202201115066](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202201115066.png)
- TypeInformation 该类是描述一切类型的公共基类，它和它的所有子类必须可序列化（Serializable），因为类型信息将会伴随 Flink 的作业提交，被传递给每个执行节点

### 类型系统的使用
- 针对前六种类型数据集，Flink皆可以自动生成对应的TypeSerializer，能非常高效地对数据集进行序列化和反序列化。对于最后一种数据类型，Flink会使用Kryo进行序列化和反序列化。
- 每个TypeInformation中，都包含了serializer，类型会自动通过serializer进行序列化，然后用JavaUnsafe接口写入MemorySegments。
- 对于可以用作key的数据类型，Flink还同时自动生成TypeComparator，用来辅助直接对序列化后的二进制数据进行compare、hash等操作。
- 对于 Tuple、CaseClass、POJO等组合类型，其TypeSerializer和TypeComparator也是组合的，序列化和比较时会委托给对应的serializers和comparators

### 例子
![image-20210202201137559](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202201137559.png)
- 可以看出这种序列化方式存储密度是相当紧凑的。其中 int 占4字节，double 占8字节，POJO多个一个字节的header，PojoSerializer只负责将header序列化进去，并委托每个字段对应的serializer对字段进行序列化。

### 扩展
- Flink 的类型系统可以很轻松地扩展出自定义的TypeInformation、Serializer以及Comparator，来提升数据类型在序列化和比较时的性能。


## 工具类
### Types 可以看做是 BasicTypeInfo 的封装
```
static {
    VOID = BasicTypeInfo.VOID_TYPE_INFO;
    STRING = BasicTypeInfo.STRING_TYPE_INFO;
    BYTE = BasicTypeInfo.BYTE_TYPE_INFO;
    BOOLEAN = BasicTypeInfo.BOOLEAN_TYPE_INFO;
    SHORT = BasicTypeInfo.SHORT_TYPE_INFO;
    INT = BasicTypeInfo.INT_TYPE_INFO;
    LONG = BasicTypeInfo.LONG_TYPE_INFO;
    FLOAT = BasicTypeInfo.FLOAT_TYPE_INFO;
    DOUBLE = BasicTypeInfo.DOUBLE_TYPE_INFO;
    CHAR = BasicTypeInfo.CHAR_TYPE_INFO;
    BIG_DEC = BasicTypeInfo.BIG_DEC_TYPE_INFO;
    BIG_INT = BasicTypeInfo.BIG_INT_TYPE_INFO;
    SQL_DATE = SqlTimeTypeInfo.DATE;
    SQL_TIME = SqlTimeTypeInfo.TIME;
    SQL_TIMESTAMP = SqlTimeTypeInfo.TIMESTAMP;
    INSTANT = BasicTypeInfo.INSTANT_TYPE_INFO;
}
```

## 类型
### 复合类型
- Flink Java Tuples（Flink Java API的一部分）：最多25个字段，不支持空字段
- Scala Case Class（包括Scala元组）：最多22个字段，不支持空字段
- Row：具有任意数量字段的元组并支持空字段
- POJO：遵循某种类似bean的模式的类

## 获取类型
### TypeExtractror 类型提取
### 声明类型信息
#### 对于非泛型的类，直接传入 Class 对象即可
#### 对于泛型类，需要借助 TypeHint 来保存泛型类型信息

## scala java 的相互调用
### java 获取 java/scala 类的Class 对象
```
TypeExtractor.createTypeInfo(new Teacher().getClass());
// Student scala
TypeExtractor.createTypeInfo(Student.class);
TypeExtractor.createTypeInfo(Teacher.class);
```
### scala 获取scala 类的Class 对象
```
TypeExtractor.createTypeInfo(Student.getClass)
```
### scala 获取java 类的Class 对象
```
TypeExtractor.createTypeInfo(new Teacher().getClass)
TypeExtractor.createTypeInfo(classOf[Teacher])
```

## Flink 的序列化
- flink 摒弃了 Java 原生的序列化方法，以独特的方式处理数据类型和序列化，包含自己的类型描述符，泛型类型提取和类型序列化框架。
- TypeInformation 是所有类型描述符的基类。它揭示了该类型的一些基本属性，并且可以生成序列化器。TypeInformation 支持以下几种类型：

- BasicTypeInfo: 任意 Java 基本类型或 String 类型
- BasicArrayTypeInfo: 任意 Java 基本类型数组或 String 数组
- WritableTypeInfo: 任意 Hadoop Writable 接口的实现类
- TupleTypeInfo: 任意的 Flink Tuple 类型(支持 Tuple1 to Tuple25)。Flink tuples 是固定长度固定类型的 Java Tuple 实现
- CaseClassTypeInfo: 任意的 Scala CaseClass(包括 Scala tuples)
- PojoTypeInfo: 任意的 POJO (Java or Scala)，例如，Java 对象的所有成员变量，要么是 public 修饰符定义，要么有 getter/setter 方法
- GenericTypeInfo: 任意无法匹配之前几种类型的类


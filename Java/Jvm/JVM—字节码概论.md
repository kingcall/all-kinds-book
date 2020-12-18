[TOC]

## JAVA 虚拟机概论

- 前面的 JVM 概论中讲解了，Java 是怎样做到一次编译到处运行的,直观上来看，java 就是因为在不同的平台上安装了对应的虚拟机（windows,Linux,Macos）,然后虚拟机完成了将字节码解释称对应平台的机器码，所以才做到了让Java 语言，一次编译到处运行。
- 从java 的角度来看，这是Java语言的优势，但是从虚拟机的角度看，这不是一个优势，我们上一节说到，我们可以把虚拟机看作是适配器(转换器)，那么对于虚拟机而言，转换的语言越多，优势越明显，不然的话其他语言也为了做到一次编译，到处运行，也会发展自己的虚拟机。
- JVM 的设计者，当初设计的时候就考虑到这一点了，让其他语言也可以运行在虚拟机上，所以在发布规范文档的时候，把Java 规范拆分成了Java 语言规范和Java 虚拟机规范。
- 其实到这里，我们大致都能猜到是怎么玩的了，我们知道虚拟机的输入是字节码，输出的是对应平台上的字节码（这样说不严谨），可以看出只要输入都一致，就是Java 编译成字节码，其他语言也编译成字节码，这样就可以做到其他语言也可以运行在虚拟机上了。 
- 可以看出实现平台的无关性是依赖于Java 虚拟机的，而实现语言的无关性是依赖于字节码的，也就是说只要字节码被虚拟机所“认可”，就可以做到语言无关性，所以JAVA 虚拟机规范中对字节码也就是class 文件的格式做了严格的限制和约束，存在很多强制的语法和结构约束。
- 最后，对于在虚拟机上执行的class 文件，JVM 并不关心它到底是什么语言编译而来，只关注它是否严格遵守字节码的约定与规范。



<table>
<thead>
<tr>
<th align="center">类型</th>
<th align="center">名称</th>
<th align="center">说明</th>
<th align="center">长度</th>
</tr>
</thead>
<tbody>
<tr>
<td align="center">u4</td>
<td align="center">magic</td>
<td align="center">魔数，识别Class文件格式</td>
<td align="center">4个字节</td>
</tr>
<tr>
<td align="center">u2</td>
<td align="center">minor_version</td>
<td align="center">副版本号</td>
<td align="center">2个字节</td>
</tr>
<tr>
<td align="center">u2</td>
<td align="center">major_version</td>
<td align="center">主版本号</td>
<td align="center">2个字节</td>
</tr>
<tr>
<td align="center">u2</td>
<td align="center">constant_pool_count</td>
<td align="center">常量池计算器</td>
<td align="center">2个字节</td>
</tr>
<tr>
<td align="center">cp_info</td>
<td align="center">constant_pool</td>
<td align="center">常量池</td>
<td align="center">n个字节</td>
</tr>
<tr>
<td align="center">u2</td>
<td align="center">access_flags</td>
<td align="center">访问标志</td>
<td align="center">2个字节</td>
</tr>
<tr>
<td align="center">u2</td>
<td align="center">this_class</td>
<td align="center">类索引</td>
<td align="center">2个字节</td>
</tr>
<tr>
<td align="center">u2</td>
<td align="center">super_class</td>
<td align="center">父类索引</td>
<td align="center">2个字节</td>
</tr>
<tr>
<td align="center">u2</td>
<td align="center">interfaces_count</td>
<td align="center">接口计数器</td>
<td align="center">2个字节</td>
</tr>
<tr>
<td align="center">u2</td>
<td align="center">interfaces</td>
<td align="center">接口索引集合</td>
<td align="center">2个字节</td>
</tr>
<tr>
<td align="center">u2</td>
<td align="center">fields_count</td>
<td align="center">字段个数</td>
<td align="center">2个字节</td>
</tr>
<tr>
<td align="center">field_info</td>
<td align="center">fields</td>
<td align="center">字段集合</td>
<td align="center">n个字节</td>
</tr>
<tr>
<td align="center">u2</td>
<td align="center">methods_count</td>
<td align="center">方法计数器</td>
<td align="center">2个字节</td>
</tr>
<tr>
<td align="center">method_info</td>
<td align="center">methods</td>
<td align="center">方法集合</td>
<td align="center">n个字节</td>
</tr>
<tr>
<td align="center">u2</td>
<td align="center">attributes_count</td>
<td align="center">附加属性计数器</td>
<td align="center">2个字节</td>
</tr>
<tr>
<td align="center">attribute_info</td>
<td align="center">attributes</td>
<td align="center">附加属性集合</td>
<td align="center">n个字节</td>
</tr>
</tbody>
</table>





<table>
<thead>
<tr>
<th align="center">数据类型</th>
<th align="center">定义</th>
<th align="center">说明</th>
</tr>
</thead>
<tbody>
<tr>
<td align="center">无符号数</td>
<td align="center">无符号数可以用来描述数字、索引引用、数量值或按照utf-8编码构成的字符串值。</td>
<td align="center">其中无符号数属于基本的数据类型。<br>以u1、u2、u4、u8来分别代表1个字节、2个字节、4个字节和8个字节</td>
</tr>
<tr>
<td align="center">表</td>
<td align="center">表是由多个无符号数或其他表构成的复合数据结构。</td>
<td align="center">所有的表都以“_info”结尾。<br>由于表没有固定长度，所以通常会在其前面加上个数说明。</td>
</tr>
</tbody>
</table>

<table>
<thead>
<tr>
<th align="center">类型</th>
<th align="center">说明</th>
<th align="center">长度</th>
</tr>
</thead>
<tbody>
<tr>
<td align="center">u1</td>
<td align="center">1个字节</td>
<td align="center">1</td>
</tr>
<tr>
<td align="center">u2</td>
<td align="center">2个字节</td>
<td align="center">2</td>
</tr>
<tr>
<td align="center">u4</td>
<td align="center">4个字节</td>
<td align="center">4</td>
</tr>
<tr>
<td align="center">u8</td>
<td align="center">8个字节</td>
<td align="center">8</td>
</tr>
<tr>
<td align="center">cp_info</td>
<td align="center">常量表</td>
<td align="center">n</td>
</tr>
<tr>
<td align="center">field_info</td>
<td align="center">字段表</td>
<td align="center">n</td>
</tr>
<tr>
<td align="center">method_info</td>
<td align="center">方法表</td>
<td align="center">n</td>
</tr>
<tr>
<td align="center">attribute_info</td>
<td align="center">属性表</td>
<td align="center">n</td>
</tr>
</tbody>
</table>

## 魔数

魔数就是用来区分文件类型的一种标志，一般都是用文件的前几个字节来表示。比如`0XCAFE BABE`表示的是class文件，那么为什么不是用文件名后缀来进行判断呢？因为文件名后缀容易被修改啊，所以为了保证文件的安全性，将文件类型写在文件内部可以保证不被篡改。



## 常量池

### 字面量和符号引用

<table>
<thead>
<tr>
<th align="center">常量</th>
<th align="center">具体的常量</th>
</tr>
</thead>
<tbody>
<tr>
<td align="center">字面量</td>
<td align="center">文本字符串</td>
</tr>
<tr>
<td align="center"></td>
<td align="center">声明为final的常量值</td>
</tr>
<tr>
<td align="center">符号引用</td>
<td align="center">类和接口的全限定名</td>
</tr>
<tr>
<td align="center"></td>
<td align="center">字段的名称和描述符</td>
</tr>
<tr>
<td align="center"></td>
<td align="center">方法的名称和描述符</td>
</tr>
</tbody>
</table>

#### 全限定名

com/april/test/Demo这个就是类的全限定名，仅仅是把包名的".“替换成”/"，为了使连续的多个全限定名之间不产生混淆，在使用时最后一般会加入一个“;”表示全限定名结束。

#### 简单名称

简单名称是指没有类型和参数修饰的方法或者字段名称，上面例子中的类的add()方法和num字段的简单名称分别是add和num。

#### 描述符

描述符的作用是用来描述字段的数据类型、方法的参数列表（包括数量、类型以及顺序）和返回值。根据描述符规则，基本数据类型（byte、char、double、float、int、long、short、boolean）以及代表无返回值的void类型都用一个大写字符来表示，而对象类型则用字符L加对象的全限定名来表示，详见下表:

<table>
<thead>
<tr>
<th align="center">标志符</th>
<th align="center">含义</th>
</tr>
</thead>
<tbody>
<tr>
<td align="center">B</td>
<td align="center">基本数据类型byte</td>
</tr>
<tr>
<td align="center">C</td>
<td align="center">基本数据类型char</td>
</tr>
<tr>
<td align="center">D</td>
<td align="center">基本数据类型double</td>
</tr>
<tr>
<td align="center">F</td>
<td align="center">基本数据类型float</td>
</tr>
<tr>
<td align="center">I</td>
<td align="center">基本数据类型int</td>
</tr>
<tr>
<td align="center">J</td>
<td align="center">基本数据类型long</td>
</tr>
<tr>
<td align="center">S</td>
<td align="center">基本数据类型short</td>
</tr>
<tr>
<td align="center">Z</td>
<td align="center">基本数据类型boolean</td>
</tr>
<tr>
<td align="center">V</td>
<td align="center">基本数据类型void</td>
</tr>
<tr>
<td align="center">L</td>
<td align="center">对象类型,如Ljava/lang/Object</td>
</tr>
</tbody>
</table>

- 对于数组类型，每一维度将使用一个前置的[字符来描述，如一个定义为java.lang.String[][]类型的二维数组，将被记录为：[[Ljava/lang/String;，，一个整型数组int[]被记录为[I。

- 用描述符来描述方法时，按照先参数列表，后返回值的顺序描述，参数列表按照参数的严格顺序放在一组小括号“( )”之内。如方法java.lang.String toString()的描述符为( ) LJava/lang/String;，方法int abc(int[] x, int y)的描述符为([II) I


### 常量类型和结构

<table>
<thead>
<tr>
<th align="center">类型</th>
<th align="center">标志</th>
<th align="center">描述</th>
</tr>
</thead>
<tbody>
<tr>
<td align="center">CONSTANT_utf8_info</td>
<td align="center">1</td>
<td align="center">UTF-8编码的字符串</td>
</tr>
<tr>
<td align="center">CONSTANT_Integer_info</td>
<td align="center">3</td>
<td align="center">整形字面量</td>
</tr>
<tr>
<td align="center">CONSTANT_Float_info</td>
<td align="center">4</td>
<td align="center">浮点型字面量</td>
</tr>
<tr>
<td align="center">CONSTANT_Long_info</td>
<td align="center">5</td>
<td align="center">长整型字面量</td>
</tr>
<tr>
<td align="center">CONSTANT_Double_info</td>
<td align="center">6</td>
<td align="center">双精度浮点型字面量</td>
</tr>
<tr>
<td align="center">CONSTANT_Class_info</td>
<td align="center">7</td>
<td align="center">类或接口的符号引用</td>
</tr>
<tr>
<td align="center">CONSTANT_String_info</td>
<td align="center">8</td>
<td align="center">字符串类型字面量</td>
</tr>
<tr>
<td align="center">CONSTANT_Fieldref_info</td>
<td align="center">9</td>
<td align="center">字段的符号引用</td>
</tr>
<tr>
<td align="center">CONSTANT_Methodref_info</td>
<td align="center">10</td>
<td align="center">类中方法的符号引用</td>
</tr>
<tr>
<td align="center">CONSTANT_InterfaceMethodref_info</td>
<td align="center">11</td>
<td align="center">接口中方法的符号引用</td>
</tr>
<tr>
<td align="center">CONSTANT_NameAndType_info</td>
<td align="center">12</td>
<td align="center">字段或方法的符号引用</td>
</tr>
<tr>
<td align="center">CONSTANT_MethodHandle_info</td>
<td align="center">15</td>
<td align="center">表示方法句柄</td>
</tr>
<tr>
<td align="center">CONSTANT_MothodType_info</td>
<td align="center">16</td>
<td align="center">标志方法类型</td>
</tr>
<tr>
<td align="center">CONSTANT_InvokeDynamic_info</td>
<td align="center">18</td>
<td align="center">表示一个动态方法调用点</td>
</tr>
</tbody>
</table>

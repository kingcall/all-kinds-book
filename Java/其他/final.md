前面在[讲解String]时提到了final关键字，本文将对final关键字进行解析。

**static**和**final**是两个我们必须掌握的关键字。不同于其他关键字，他们都有多种用法，而且在一定环境下使用，可以提高程序的运行性能，优化程序的结构。下面我们来了解一下**final**关键字及其用法。

**final**从总体上来说是“不可变的”，可用于修改类、方法、变量。

## 一. final类

final修饰的类，该类不能被继承。当你确认一个类永远不会被继承或不想被继承，那么就可以用final修饰。

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:22:20-1677914-20190623092443122-1832034790.png)

同样，对于接口（interface）和抽象类（abstract Class），其本就是为了“多态”而设计，自然无法用final关键字修饰

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:22:20-1677914-20190623092459007-1701031204.png)

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:22:20-1677914-20190623092512663-1471329105.png)

final类中的成员方法默认也被隐式指定为final方法。

## 二. final方法

final修饰的方法不可被重写。

例子：

```
/**
 * 父类
 * @author LH
 */
public class FinalDemo1 {
    public final void test() {
        
    }
}
```

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:22:20-1677914-20190623092525482-611346090.png)

## 三. final变量

final变量包括成员变量和局部变量。变量类型包括基本数据类型、对象。

通过final修饰局部基本类型变量（及其包装类），数值一经初始化（可以定义时初始化，也可以在使用前初始化）不可改变。如：

```
final int a = 0;
a = 1;//报错
final int b;
b = 1;//编译通过
```

通过final修饰局部引用类型变量时，其引用的对象（内存地址）（可以定义时初始化，也可以在使用前初始化）不可改变，*但对象中存放的数据可以改变*

```
public static void main(String[] args) {
	final String str1 = "helloWorld";
	str1 = "helloChina";//编译出错，String的不可变性，此处返回的是新的对象引用。

	final StringBuilder sb = new StringBuilder("hello");
	sb.append("world");//编译通过

	sb = new StringBuilder("China");//编译出错
}
```

final修饰的成员变量必须在定义的时候直接初始化，否则会编译出错

```
public class FinalDemo1 {
	public final int age;//final修饰的基本类型，编译出错
	public final int age1 = 20;//final修饰的基本类型，编译通过
    public final StringBuilder address;// final修饰的引用类型，编译出错
    public final StringBuilder address1 = new StringBuilder("中国");//final修饰的引用类型，编译通过
}
```

那么final变量与普通变量之间到底有何区别，看下面的例子

```
public static void main(String[] args) {
    String str0 = "helloWorldChina";
    String str1 = "helloWorld";
    String str3 = str1 + "China";
    System.out.println(str0 == str3);//false
    
    final String str2 = "helloWorld";
    String str4 = str2 + "China";
    System.out.println(str0 == str4);//true
    
    final String str5;
    str5 = "helloWorld";
    String str6 = str5 + "China";
    System.out.println(str0 == str6);//false
}
```

str0 == str3运行结果为false，这是因为通过“+”生成了一个新的字符串对象，返回的引用地址和str0不再一样，这在《[Java基础(三) String深度解析](https://www.cnblogs.com/LiaHon/p/11068050.html)》中有讲解。

**那么str0 == str4的执行结果为什么是true?**

通过final修饰的变量，如果在编译期都可以知道确切值（定义变量的时候就初始化），那么在编译器会将其当做常量使用，所有用到该变量的地方就相当于直接使用该常量，String str4 = str2 + "China" 在编译期间都已经合并处理成String str4 = "helloWorldChina"，因此str0与str4引用了常量池中同一个字符串字面量的地址，故而结果为true。

**而str0 == str6的执行结果为false也很好理解**

str5在编译期并不知道确切值，而是在使用之前才进行初始化，因此编译器无法事先进行合并处理，str6通过“+”生成了一个新的字符串对象，返回的引用地址和str0也不再一样。

**而针对基本数据类型来说定义为final变量与普通变量，比较结果来说并无差异**

```
public static void testint(){
	int int0 = 8;    
	final int int1;    
	int1 = 4;    
	int int2 = int1 + 4;    
	System.out.println(int2 == int0);//true
}
```

因为基本数据类型并不存在引用传递的概念，基本类型变量也是字面常量，所以对基本类型的操作都是直接对值的操作，和引用不一样，比较的并非地址。

## 四. 总结

本文主要对final关键字的原理进行了讲解，同时对其基本用法进行了说明，包括final修饰的类，final修饰的方法和final修饰的变量，另外文中String变量通过==比较只是为了更加清晰的说明final原理，实际应用场景比较的时候还是用equals()方法，final也经常和static配合使用作为“全局常量”，若有不对之处，请批评指正，望共同进步，谢谢！
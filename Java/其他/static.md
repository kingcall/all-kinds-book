static和final是两个我们必须掌握的关键字。不同于其他关键字，他们都有多种用法，而且在一定环境下使用，可以提高程序的运行性能，优化程序的结构。上一个章节我们讲了[final关键字的原理及用法](https://www.cnblogs.com/LiaHon/p/11071861.html)，本章节我们来了解一下static关键字原理及其用法。

## 一. static特点

static是一个修饰符，通常用于修饰变量和方法，如开发过程中用到的字典类数据都会用到static来修饰，工具类方法，如Dateutils，Stringutils这类工具方法也会用到static来修饰，那么除了这两种最常用的场景外，是否还有其他场景呢，答案是：有的，总共五种：

1. static变量
2. static方法
3. static代码块
4. static内部类
5. static包内导入

static修饰的变量、方法、代码块、内部类在类加载期间就已经完成初始化，存储在Java Heap（JDK7.0之前存储在方法区）中静态存储区，因此static优于对象而存在。

static修饰的成员（变量、方法）被所有对象所共享，也叫静态变量或静态方法，可直接通过类调用（也建议通过类调用）。

## 二. static 变量

static变量随着类的加载而存在，随着类的消失而消失，当类被加载时，就会为静态变量在Java Heap中分配内存空间，可以通过【类.变量名】和【对象.变量名】的方式调用，建议直接使用【类.变量名】的方式，

```java
public class Person {
    private String name;

    private static int eyeNum;

    public static int legNum = 2;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public static int getEyeNum() {
        return eyeNum;
    }

    public static void setEyeNum(int eyeNum) {
        Person.eyeNum = eyeNum;
    }
}
public static void main(String[] args) {
    Person person = new Person();
    person.setEyeNum(25);

    Person person1 = new Person();
    person1.setEyeNum(28);
    System.out.println(person.getEyeNum());//28
    System.out.println(person1.getEyeNum());//28
    
    int legNum = person.legNum;
    System.out.println(legNum);//2
}
```

从上面的例子可以看出静态变量是对所有对象共享，一个对象对其值的改动，直接就会造成另一个对象取值的不同。

**什么时候使用static变量？**

作为共享变量使用，通常搭配final关键字一起使用，比如我们常用的字典类数据；

```
private static final String GENERAL_MAN = "man";
```

减少对象的创建，比如在类开头的部分，定义Logger方法，用于异常日志采集

```
private static Logger LOGGER = LogFactory.getLoggger(MyClass.class);
```

始终返回同一变量，比如我们的单例模式。

## 三. static 方法

静态方法只能访问静态成员（静态变量、静态方法），而非静态方法既可访问静态方法也可访问非静态方法；因为静态成员优于对象而存在，因此无法调用和对象相关的关键字，如this,super，无法通过关键字访问对象资源。

```java
public class Person {
	private String name;    
	private static int eyeNum;    
	public static int legNum = 2;    
	public String getName() {
    	return name;    
    }    
    public void setName(String name) {
    	this.name = name;    
    }    
    public static int getEyeNum() {
    	System.out.println(name);//编译出错，name不可用
    	return eyeNum;
    }    
    public static void setEyeNum(int eyeNum) {
    	Person.eyeNum = eyeNum;        
    	this.name = "";//编译出错，this不可用
    }
}
```

**什么时候使用static方法？**

static方法一般用于与当前对象无法的工厂方法、工具方法。如Math.sqrt()，Arrays.sort()，StringUtils.isEmpty()等。

## 四. static 代码块

static代码块相对于static变量和static方法来说使用不是那么广泛，但也算是比较常见的用法了，static代码块在加载一个类的时候最先执行，且只执行一次。

```java
public static Map<String, String> timeTypes;
static {
	timeTypes = new HashMap<>();
    timeTypes.put("year", "年");
    timeTypes.put("quarter", "季");
    timeTypes.put("month", "月");
    timeTypes.put("day", "日");
    System.out.println("初始化1");
}
public static void main(String[] args) {
	System.out.println("初始化2");
}
```

执行结果是：

```
初始化1；

初始化2；
```

**什么时候使用static代码块？**

一般在进行初始化操作时，比如读取配置文件信息，获取当前服务器参数等

## 五. static内部类

定义一个内部类，加上static，就成为了一个static内部类，static只能修饰内部类，不能修饰顶级类，静态内部类在业务应用系统开发中使用的不多。

```java
public class StaticCouter {
    private String str0 = "hi";    //非静态变量    
    private static String str1 = "hello";  //静态变量   
    static class StaticInnerClass{//静态内部类
        public void getMessage(){
            System.out.println(str0);//编译出错
            System.out.println(str1);
        }
    }    
    class NormalInnerClass{//非静态内部类
        public void getMessage(){
            System.out.println(str0);
            System.out.println(str1);
        }
    }
}
```

**静态内部类与非静态内部类有何异同？**

| 静态内部类                                         | 非静态内部类                                 |
| -------------------------------------------------- | -------------------------------------------- |
| 不需要有指向外部类的引用                           | 必须通过外部类的new关键字引用                |
| 可定义普通变量和方法，也可定义静态变量和方法       | 可定义普通变量和方法，不可定义静态变量和方法 |
| 可以调用外部类的静态成员，不能调用外部类的普通成员 | 可调用外部类的普通成员和静态成员             |

```java
public static void main(String[] args) {
    //创建静态内部类实例    
    StaticInnerClass staticInnerClass = new StaticInnerClass();    
    //调用静态内部类方法    
    staticInnerClass.getMessage();    
    //创建静态内部类实例    
    StaticCouter.StaticInnerClass staticInnerClass1 = new staticCouter.StaticInnerClass();    
    //调用静态内部类方法
    staticInnerClass1.getMessage();
    //创建普通内部类实例
    StaticCouter.NormalInnerClass normalInnerClass = new StaticCouter().new NormalInnerClass();
    //调用普通内部类方法
    normalInnerClass.getMessage();
}
```

## 六. static包内导入

这个概念不太好理解，举个例子

```java
public static void main(String[] args) {
	int[] arra = {1,4,5,7};
    Arrays.sort(arra);
    Arrays.asList(arra);
    Arrays.fill(arra, 6);
}
```

static包导入目的就是去掉重复的Arrays类名调用

通过在顶部引入

```
import static java.util.Arrays.*
```

即可把Arrays类中所有的静态变量，方法，内部类等都引入当前类中，调用时直接调用sort(arra)，asList(arra)，

java5后引入的，不常用，调用类方法时会比较简单，但可读性不好，慎用。

## 七. 总结

static是java中很常用的一个关键字，使用场景也很多，本文主要介绍了它的五种用法，static变量，static方法，static代码块，static内部类，static包内导入，若有不对之处，请批评指正，望共同进步，谢谢！
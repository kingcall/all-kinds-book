[TOC]

## 枚举进阶

上一节我们讲了[枚举初识](https://blog.csdn.net/king14bhhb/article/details/111224216) 里面主要讲了枚举的实现原理，我们从编译器的角度看了枚举的底层实现以及枚举常用的方法

今天我们看一下枚举添加自定义方法和构造函数，枚举的抽象和接口，枚举与switch 和基于枚举的单例，之后我们会讲解两个和枚举相关的数据结构EnumMap 和EnumSet 

在前面的分析中，我们都是基于简单枚举类型的定义，也就是在定义枚举时只定义了枚举实例类型，并没定义方法或者成员变量，实际上使用关键字enum定义的枚举类，**除了不能使用继承(**因为编译器会自动为我们继承Enum抽象类而Java只支持单继承，因此枚举类是无法手动实现继承的)，可以把enum类当成常规类，也就是说我们可以向enum类中添加方法和变量，甚至是mian方法，下面就来感受一把。

### 向enum类添加方法与自定义构造函数

在开始之前大家先注意一个事情，那就是枚举的常量的类型就是定义的枚举的类型，例如我在`DateEnum` 中定义的所有的枚举常量，那都是`DateEnum`的实例

重新定义一个日期枚举类，带有desc成员变量描述该日期的对于中文描述，同时定义一个getDesc方法，返回中文描述内容，**自定义私有构造函数**，在声明枚举实例时传入对应的中文描述，代码如下：

```java
public enum DateEnum {
    MONDAY("星期一"),
    TUESDAY("星期二"),
    WEDNESDAY("星期三"),
    THURSDAY("星期四"),
    FRIDAY("星期五"),
    SATURDAY("星期六"),
    SUNDAY("星期日");//记住要用分号结束

    private String desc;//中文描述
    /**
     * 私有构造,防止被外部调用
     * @param desc
     */
    private DateEnum(String desc){
        this.desc=desc;
    }
    /**
     * 定义方法,返回描述,跟常规类的定义没区别
     * @return
     */
    public String getDesc(){
        return desc;
    }
    @Override
    public String toString() {
        return name()+":"+desc;
    }
}
```

```java
@Test
public void useEnumToString() {
    DateEnum[] constants= DateEnum.MONDAY.values();
    for (DateEnum constant : constants) {
        System.out.println(constant);
    }
}
// 输出结果
MONDAY:星期一
TUESDAY:星期二
WEDNESDAY:星期三
THURSDAY:星期四
FRIDAY:星期五
SATURDAY:星期六
SUNDAY:星期日
```

可以看到我们定义的变量和方法起作用了

从上述代码可知，在enum类中确实可以像定义常规类一样声明变量或者成员方法。

**但是我们必须注意到，如果打算在enum类中定义方法，务必在声明完枚举实例后使用分号分开，倘若在枚举实例前定义任何方法，编译器都将会报错，无法编译通过，同时即使自定义了构造函数且enum的定义结束，我们也永远无法手动调用构造函数创建枚举实例，毕竟这事只能由编译器执行。**

还有一点需要注意的是：**构造方法只能是私有的，那是因为枚举的常量都是在定义的时候创建的，如果构造方法公开，枚举就没有意义了**

### 覆盖enum类方法

既然enum类跟常规类的定义没什么区别（实际上enum还是有些约束的），那么覆盖父类的方法也不会是什么难说，可惜的是父类Enum中的定义的方法只有toString方法没有使用final修饰，因此只能覆盖toString方法，但是关于覆盖toString 方法我们也提到了两次了，第一次是在讲name 方法的时候，你可以查看上一篇[枚举初识](https://blog.csdn.net/king14bhhb/article/details/111224216)，前面将定义属性和方法的时候也重写了toString ,所以这里就不演示了



### enum类中定义抽象方法

与常规抽象类一样，enum类允许我们为其定义抽象方法，然后使每个枚举实例都实现该方法，以便产生不同的行为方式，注意abstract关键字对于枚举类来说并不是必须的如下：

```java
public enum EnumAbstract {

    FIRST{
        // 第二步:实现抽象方法
        @Override
        public String getInfo() {
            return "FIRST TIME";
        }
    },
    SECOND{
       // 第二步:实现抽象方法
        @Override
        public String getInfo() {
            return "SECOND TIME";
        }
    }
    ;
    /**
     * 第一步：定义抽象方法
     * @return
     */
    public abstract String getInfo();

}
```

```java
@Test
public void useEnumAbstract() {
    EnumAbstract[] constants= EnumAbstract.values();
    for (EnumAbstract constant : constants) {
        System.out.println(constant.getInfo());
    }
}
// 输出结果
FIRST TIME
SECOND TIME
```



通过这种方式就可以轻而易举地**定义每个枚举实例的不同行为方式**。你甚至可以将其当做变量传出去

```java
@Test
public void useEnumAbstrac2t() {
    EnumAbstract[] constants= EnumAbstract.values();
    for (EnumAbstract constant : constants) {
        test(constant);
    }
}

public void test(EnumAbstract instance){
    System.out.println(instance.getInfo());
}
```



### enum类与接口

由于Java单继承的原因，enum类并不能再继承其它类，但并不妨碍它实现接口，因此enum类同样是可以实现多接口的，如下：

```java
interface Food{
    String eat();
}

interface Sport{
    String run();
}

public enum EnumImplements implements Food,Sport{
    WHITEMAN("牛肉","篮球"),
    BLACKMAN("鱼肉","羽毛球");

    private String food;
    private String sport;

    @Override
    public String eat() {
        return name()+" eat "+food;
    }

    @Override
    public String run() {
        return name()+" run "+sport;
    }

    private EnumImplements(String food,String sport){
        this.food = food;
        this.sport = sport;
    }
}
```

```java
@Test
public void useEnumImplemets() {
    EnumImplements[] constants= EnumImplements.values();
    for (EnumImplements constant : constants) {
        String tmp = constant.eat() +"\t"+ constant.run();
        System.out.println(tmp);
    }
}
// 输出结果
WHITEMAN eat 牛肉	WHITEMAN run 篮球
BLACKMAN eat 鱼肉	BLACKMAN run 羽毛球
```



有时候，我们可能需要对一组数据进行分类，比如**按照食物菜单分类而且希望这些菜单都属于food类型**，appetizer(开胃菜)、mainCourse(主菜)、dessert(点心)、Coffee等，每种分类下有多种具体的菜式或食品，此时可以利用接口来组织，如下(代码引用自Thinking in Java)：

```java
interface AllFood {
    enum Appetizer implements AllFood {
        SALAD, SOUP, SPRING_ROLLS;
    }
    enum MainCourse implements AllFood {
        LASAGNE, BURRITO, PAD_THAI,
        LENTILS, HUMMOUS, VINDALOO;
    }
    enum Dessert implements AllFood {
        TIRAMISU, GELATO, BLACK_FOREST_CAKE,
        FRUIT, CREME_CARAMEL;
    }
    enum Coffee implements AllFood {
        BLACK_COFFEE, DECAF_COFFEE, ESPRESSO,
        LATTE, CAPPUCCINO, TEA, HERB_TEA;
    }
}

public class TypeOfFood {
    public static void main(String[] args) {
        AllFood food1 = AllFood.Appetizer.SALAD;
        AllFood food2= AllFood.MainCourse.LASAGNE;
        AllFood food3= AllFood.Dessert.GELATO;
        AllFood food4= AllFood.Coffee.CAPPUCCINO;
    }
} 
```

通过这种方式可以很方便组织上述的情景，同时确保每种具体类型的食物也属于Food，现在我们利用一个枚举嵌套枚举的方式，把前面定义的菜谱存放到一个Meal菜单中，通过这种方式就可以统一管理菜单的数据了。

```java
public enum Meal{
  APPETIZER(Food.Appetizer.class),
  MAINCOURSE(Food.MainCourse.class),
  DESSERT(Food.Dessert.class),
  COFFEE(Food.Coffee.class);
  private Food[] values;
  private Meal(Class<? extends Food> kind) {
    //通过class对象获取枚举实例
    values = kind.getEnumConstants();
  }
  public interface Food {
    enum Appetizer implements Food {
      SALAD, SOUP, SPRING_ROLLS;
    }
    enum MainCourse implements Food {
      LASAGNE, BURRITO, PAD_THAI,
      LENTILS, HUMMOUS, VINDALOO;
    }
    enum Dessert implements Food {
      TIRAMISU, GELATO, BLACK_FOREST_CAKE,
      FRUIT, CREME_CARAMEL;
    }
    enum Coffee implements Food {
      BLACK_COFFEE, DECAF_COFFEE, ESPRESSO,
      LATTE, CAPPUCCINO, TEA, HERB_TEA;
    }
  }
} 
```

### 枚举与switch

关于枚举与switch是个比较简单的话题，使用switch进行条件判断时，条件参数一般只能是整型，字符型。

而枚举型确实也被switch所支持，在java 1.7后switch也对字符串进行了支持。这里我们简单看一下switch与枚举类型的使用：

```java
enum Color {GREEN,RED,BLUE}
public class EnumColor {
    
    public static void printName(Color color){
        switch (color){
            case BLUE: 
                System.out.println("蓝色");
                break;
            case RED:
                System.out.println("红色");
                break;
            case GREEN:
                System.out.println("绿色");
                break;
        }
    }
    public static void main(String[] args){
        printName(Color.BLUE);
        printName(Color.RED);
        printName(Color.GREEN);
    }
}
```

输出结果

```
蓝色
红色
绿色
```

需要注意的是使用在于switch条件进行结合使用时，**无需使用Color引用(Color.XXX)**

### 枚举与单例模式

单例模式可以说是最常使用的设计模式了，它的作用是确保某个类只有一个实例，自行实例化并向整个系统提供这个实例。在实际应用中，线程池、缓存、日志对象、对话框对象常被设计成单例，总之，选择单例模式就是为了避免不一致状态，下面我们将会简单说明单例模式的几种主要编写方式，从而对比出使用枚举实现单例模式的优点,更多关于单例模式的文章请看[胡说八道设计模式—单例模式](https://blog.csdn.net/king14bhhb/article/details/110576261)

首先看看饿汉式的单例模式：

```java
public class SingletonHungry {

    private static SingletonHungry instance = new SingletonHungry();

    private SingletonHungry() {
    }

    public static SingletonHungry getInstance() {
        return instance;
    }
}
```

显然这种写法比较简单，但问题是无法做到延迟创建对象，事实上如果该单例类涉及资源较多，创建比较耗时间时，我们更希望它可以尽可能地延迟加载，从而减小初始化的负载，于是便有了如下的懒汉式单例：

```java
public class SingletonLazy {

    private static volatile SingletonLazy instance;

    private SingletonLazy() {
    }

    public static SingletonLazy getInstance() {
        if (instance == null) {
            instance = new SingletonLazy();
        }
        return instance;
    }
}
```

这种写法能够在多线程中很好的工作避免同步问题，同时也具备lazy loading机制,遗憾的实上述写法在多线程环境中，可能导致线程安全问题，也就是对象被重复创建,所以有了下面的改进版本

```java
public class SingletonLazy {

    private static SingletonLazy instance;

    private SingletonLazy() {
    }

    public static synchronized SingletonLazy getInstance() {
        if (instance == null) {
            instance = new SingletonLazy();
        }
        return instance;
    }
}
```

遗憾的是，由于synchronized的存在，效率很低，在单线程的情景下，完全可以去掉synchronized，为了兼顾效率与性能问题，改进后代码如下

> 需要注意的是Java 团队一致致力于优化synchronized关键字的效率

```java
public class Singleton {
    private static volatile Singleton singleton = null;

    private Singleton(){}

    public static Singleton getSingleton(){
        if(singleton == null){
            synchronized (Singleton.class){
                if(singleton == null){
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }    
}
```

这种编写方式被称为“双重检查锁”，主要在getSingleton()方法中，进行两次null检查。这样可以极大提升并发度，进而提升性能。毕竟在单例中new的情况非常少，绝大多数都是可以并行的读操作，因此在加锁前多进行一次null检查就可以减少绝大多数的加锁操作，也就提高了执行效率。**但是必须注意的是volatile关键字**，该关键字有两层语义。

第一层语义是可见性，可见性是指在一个线程中对该变量的修改会马上由工作内存（Work Memory）写回主内存（Main Memory），所以其它线程会马上读取到已修改的值，关于工作内存和主内存可简单理解为高速缓存（直接与CPU打交道）和主存（日常所说的内存条），注意工作内存是线程独享的，主存是线程共享的。

volatile的第二层语义是禁止指令重排序优化，我们写的代码（特别是多线程代码），由于编译器优化，在实际执行的时候可能与我们编写的顺序不同。编译器只保证程序执行结果与源代码相同，却不保证实际指令的顺序与源代码相同，这在单线程并没什么问题，然而一旦引入多线程环境，这种乱序就可能导致严重问题。

volatile关键字就可以从语义上解决这个问题，值得关注的是volatile的禁止指令重排序优化功能在Java 1.5后才得以实现，因此1.5前的版本仍然是不安全的，即使使用了volatile关键字。或许我们可以利用静态内部类来实现更安全的机制，静态内部类单例模式如下：

```java
public class SingletonInner {
    private static class Holder {
        private static SingletonInner singleton = new SingletonInner();
    }

    private SingletonInner(){}

    public static SingletonInner getSingleton(){
        return Holder.singleton;
    }
}
```

正如上述代码所展示的，我们把Singleton实例放到一个静态内部类中，这样可以避免了静态实例在Singleton类的加载阶段就创建对象，并且由于静态内部类只会被加载一次，所以这种写法也是线程安全的。从上述4种单例模式的写法中，似乎也解决了效率与懒加载的问题，但是它们都有两个共同的缺点：

- 序列化可能会破坏单例模式，比较每次反序列化一个序列化的对象实例时都会创建一个新的实例，解决方案如下：

  ```java
  //测试例子(四种写解决方式雷同)
  public class Singleton implements java.io.Serializable {     
     public static Singleton INSTANCE = new Singleton();     
  
     protected Singleton() {     
     }  
  
     //反序列时直接返回当前INSTANCE
     private Object readResolve() {     
              return INSTANCE;     
        }    
  }  
  ```

- 使用反射强行调用私有构造器，解决方式可以修改构造器，让它在创建第二个实例的时候抛异常，如下：

  ```java
  public static Singleton INSTANCE = new Singleton();     
  private static volatile  boolean  flag = true;
  private Singleton(){
      if(flag){
      flag = false;   
      }else{
          throw new RuntimeException("The instance  already exists ！");
      }
  }
  ```

如上所述，问题确实也得到了解决，但问题是我们为此付出了不少努力，即添加了不少代码，还应该注意到如果单例类维持了其他对象的状态时还需要使他们成为transient的对象，这种就更复杂了，那有没有更简单更高效的呢？当然是有的，那就是枚举单例了，先来看看如何实现：

```java
public enum  SingletonEnum {
    INSTANCE;
    private String name;
    public String getName(){
        return name;
    }
    public void setName(String name){
        this.name = name;
    }
}
```

代码相当简洁，我们也**可以像常规类一样编写enum类，为其添加变量和方法，访问方式也更简单**，使用`SingletonEnum.INSTANCE`进行访问，这样也就避免调用getInstance方法，更重要的是使用枚举单例的写法，我们完全不用考虑序列化和反射的问题。

枚举序列化是由jvm保证的，**每一个枚举类型和定义的枚举变量在JVM中都是唯一的**，在枚举类型的序列化和反序列化上，Java做了特殊的规定：在序列化时Java仅仅是将枚举对象的name属性输出到结果中，反序列化的时候则是通过java.lang.Enum的valueOf方法来根据名字查找枚举对象。同时，编译器是不允许任何对这种序列化机制的定制，并禁用了writeObject、readObject、readObjectNoData、writeReplace和readResolve等方法，从而保证了枚举实例的唯一性，这里我们不妨再次看看Enum类的valueOf方法：

```java
public static <T extends Enum<T>> T valueOf(Class<T> enumType,
                                              String name) {
      T result = enumType.enumConstantDirectory().get(name);
      if (result != null)
          return result;
      if (name == null)
          throw new NullPointerException("Name is null");
      throw new IllegalArgumentException(
          "No enum constant " + enumType.getCanonicalName() + "." + name);
  }
```

实际上通过调用enumType(Class对象的引用)的enumConstantDirectory方法获取到的是一个Map集合，在该集合中存放了以枚举name为key和以枚举实例变量为value的Key&Value数据，因此通过name的值就可以获取到枚举实例，看看enumConstantDirectory方法源码：

```java
Map<String, T> enumConstantDirectory() {
        if (enumConstantDirectory == null) {
            //getEnumConstantsShared最终通过反射调用枚举类的values方法
            T[] universe = getEnumConstantsShared();
            if (universe == null)
                throw new IllegalArgumentException(
                    getName() + " is not an enum type");
            Map<String, T> m = new HashMap<>(2 * universe.length);
            //map存放了当前enum类的所有枚举实例变量，以name为key值
            for (T constant : universe)
                m.put(((Enum<?>)constant).name(), constant);
            enumConstantDirectory = m;
        }
        return enumConstantDirectory;
    }
    private volatile transient Map<String, T> enumConstantDirectory = null;
```

到这里我们也就可以看出枚举序列化确实不会重新创建新实例，jvm保证了每个枚举实例变量的唯一性。再来看看反射到底能不能创建枚举，下面试图通过反射获取构造器并创建枚举

```java
public static void main(String[] args) throws IllegalAccessException, InvocationTargetException, InstantiationException, NoSuchMethodException {
  //获取枚举类的构造函数(前面的源码已分析过)
   Constructor<SingletonEnum> constructor=SingletonEnum.class.getDeclaredConstructor(String.class,int.class);
   constructor.setAccessible(true);
   //创建枚举
   SingletonEnum singleton=constructor.newInstance("otherInstance",9);
  }
```

执行报错

```java
Exception in thread "main" java.lang.IllegalArgumentException: Cannot reflectively create enum objects
    at java.lang.reflect.Constructor.newInstance(Constructor.java:417)
    at zejian.SingletonEnum.main(SingletonEnum.java:38)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)12345678
```

显然告诉我们不能使用反射创建枚举类，这是为什么呢？不妨看看newInstance方法源码：

```java
 public T newInstance(Object ... initargs)
        throws InstantiationException, IllegalAccessException,
               IllegalArgumentException, InvocationTargetException
    {
        if (!override) {
            if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
                Class<?> caller = Reflection.getCallerClass();
                checkAccess(caller, clazz, null, modifiers);
            }
        }
        //这里判断Modifier.ENUM是不是枚举修饰符，如果是就抛异常
        if ((clazz.getModifiers() & Modifier.ENUM) != 0)
            throw new IllegalArgumentException("Cannot reflectively create enum objects");
        ConstructorAccessor ca = constructorAccessor;   // read volatile
        if (ca == null) {
            ca = acquireConstructorAccessor();
        }
        @SuppressWarnings("unchecked")
        T inst = (T) ca.newInstance(initargs);
        return inst;
    }
```

源码明确表示了，确实无法使用反射创建枚举实例，也就是说明了**创建枚举实例只有编译器能够做到**。

显然枚举单例模式确实是很不错的选择，因此我们推荐使用它。但是这总不是万能的，使用枚举时占用的内存常常是静态变量的两倍还多。

但是不管如何，关于单例，我们总是应该记住：**线程安全，延迟加载，序列化与反序列化安全，反射安全**是很重重要的。



## 总结

1. **构造方法只能是私有的，那是因为枚举的常量都是在定义的时候创建的，如果构造方法公开，枚举就没有意义了**
2. **接口的方法必须实现自枚举类的定义中，抽象方法是现在了常量的定义中**
3. switch 友好的支持了枚举类型
4. 枚举可以让我们实现更加高效、简单、安全的单例
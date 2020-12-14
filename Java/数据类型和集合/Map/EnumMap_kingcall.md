## EnumMap基本用法

先思考这样一个问题，现在我们有一堆size大小相同而颜色不同的数据，需要统计出每种颜色的数量是多少以便将数据录入仓库，定义如下枚举用于表示颜色Color:

```java
enum Color {
    GREEN,RED,BLUE,YELLOW
}123
```

我们有如下解决方案，使用Map集合来统计，key值作为颜色名称，value代表衣服数量，如下：

```java
package com.zejian.enumdemo;

import java.util.*;

/**
 * Created by zejian on 2017/5/10.
 * Blog : http://blog.csdn.net/javazejian [原文地址,请尊重原创]
 */
public class EnumMapDemo {
    public static void main(String[] args){
        List<Clothes> list = new ArrayList<>();
        list.add(new Clothes("C001",Color.BLUE));
        list.add(new Clothes("C002",Color.YELLOW));
        list.add(new Clothes("C003",Color.RED));
        list.add(new Clothes("C004",Color.GREEN));
        list.add(new Clothes("C005",Color.BLUE));
        list.add(new Clothes("C006",Color.BLUE));
        list.add(new Clothes("C007",Color.RED));
        list.add(new Clothes("C008",Color.YELLOW));
        list.add(new Clothes("C009",Color.YELLOW));
        list.add(new Clothes("C010",Color.GREEN));
        //方案1:使用HashMap
        Map<String,Integer> map = new HashMap<>();
        for (Clothes clothes:list){
           String colorName=clothes.getColor().name();
           Integer count = map.get(colorName);
            if(count!=null){
                map.put(colorName,count+1);
            }else {
                map.put(colorName,1);
            }
        }

        System.out.println(map.toString());

        System.out.println("---------------");

        //方案2:使用EnumMap
        Map<Color,Integer> enumMap=new EnumMap<>(Color.class);

        for (Clothes clothes:list){
            Color color=clothes.getColor();
            Integer count = enumMap.get(color);
            if(count!=null){
                enumMap.put(color,count+1);
            }else {
                enumMap.put(color,1);
            }
        }

        System.out.println(enumMap.toString());
    }

    /**
     输出结果:
     {RED=2, BLUE=3, YELLOW=3, GREEN=2}
     ---------------
     {GREEN=2, RED=2, BLUE=3, YELLOW=3}
     */
}123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960
```

代码比较简单，我们使用两种解决方案，一种是HashMap，一种EnumMap，虽然都统计出了正确的结果，但是EnumMap作为枚举的专属的集合，我们没有理由再去使用HashMap，毕竟EnumMap要求其Key必须为Enum类型，因而使用Color枚举实例作为key是最恰当不过了，也避免了获取name的步骤，更重要的是EnumMap效率更高，因为其内部是通过数组实现的（稍后分析），注意EnumMap的key值不能为null，虽说是枚举专属集合，但其操作与一般的Map差不多，概括性来说EnumMap是专门为枚举类型量身定做的Map实现，虽然使用其它的Map（如HashMap）也能完成相同的功能，但是使用EnumMap会更加高效，它只能接收同一枚举类型的实例作为键值且不能为null，由于枚举类型实例的数量相对固定并且有限，所以EnumMap使用数组来存放与枚举类型对应的值，毕竟数组是一段连续的内存空间，根据程序局部性原理，效率会相当高。下面我们来进一步了解EnumMap的用法，先看构造函数：

```java
//创建一个具有指定键类型的空枚举映射。
EnumMap(Class<K> keyType) 
//创建一个其键类型与指定枚举映射相同的枚举映射，最初包含相同的映射关系（如果有的话）。     
EnumMap(EnumMap<K,? extends V> m) 
//创建一个枚举映射，从指定映射对其初始化。
EnumMap(Map<K,? extends V> m)       123456
```

与HashMap不同，它需要传递一个类型信息，即Class对象，通过这个参数EnumMap就可以根据类型信息初始化其内部数据结构，另外两只是初始化时传入一个Map集合，代码演示如下：

```
//使用第一种构造
Map<Color,Integer> enumMap=new EnumMap<>(Color.class);
//使用第二种构造
Map<Color,Integer> enumMap2=new EnumMap<>(enumMap);
//使用第三种构造
Map<Color,Integer> hashMap = new HashMap<>();
hashMap.put(Color.GREEN, 2);
hashMap.put(Color.BLUE, 3);
Map<Color, Integer> enumMap = new EnumMap<>(hashMap);123456789
```

至于EnumMap的方法，跟普通的map几乎没有区别，注意与HashMap的主要不同在于构造方法需要传递类型参数和EnumMap保证Key顺序与枚举中的顺序一致，但请记住Key不能为null。

## EnumMap实现原理剖析

EnumMap的源码有700多行，这里我们主要分析其内部存储结构，添加查找的实现，了解这几点，对应EnumMap内部实现原理也就比较清晰了，先看数据结构和构造函数

```
public class EnumMap<K extends Enum<K>, V> extends AbstractMap<K, V>
    implements java.io.Serializable, Cloneable
{
    //Class对象引用
    private final Class<K> keyType;

    //存储Key值的数组
    private transient K[] keyUniverse;

    //存储Value值的数组
    private transient Object[] vals;

    //map的size
    private transient int size = 0;

    //空map
    private static final Enum<?>[] ZERO_LENGTH_ENUM_ARRAY = new Enum<?>[0];

    //构造函数
    public EnumMap(Class<K> keyType) {
        this.keyType = keyType;
        keyUniverse = getKeyUniverse(keyType);
        vals = new Object[keyUniverse.length];
    }

}1234567891011121314151617181920212223242526
```

EnumMap继承了AbstractMap类，因此EnumMap具备一般map的使用方法，keyType表示类型信息，keyUniverse表示键数组，存储的是所有可能的枚举值，vals数组表示键对应的值，size表示键值对个数。在构造函数中通过`keyUniverse = getKeyUniverse(keyType);`初始化了keyUniverse数组的值，内部存储的是所有可能的枚举值，接着初始化了存在Value值得数组vals，其大小与枚举实例的个数相同，getKeyUniverse方法实现如下

```
//返回枚举数组
private static <K extends Enum<K>> K[] getKeyUniverse(Class<K> keyType) {
        //最终调用到枚举类型的values方法，values方法返回所有可能的枚举值
        return SharedSecrets.getJavaLangAccess()
                                        .getEnumConstantsShared(keyType);
    }123456
```

从方法的返回值来看，返回类型是枚举数组，事实也是如此，最终返回值正是枚举类型的values方法的返回值，前面我们分析过values方法返回所有可能的枚举值，因此keyUniverse数组存储就是枚举类型的所有可能的枚举值。接着看put方法的实现

```
 public V put(K key, V value) {
        typeCheck(key);//检测key的类型
        //获取存放value值得数组下标
        int index = key.ordinal();
        //获取旧值
        Object oldValue = vals[index];
        //设置value值
        vals[index] = maskNull(value);
        if (oldValue == null)
            size++;
        return unmaskNull(oldValue);//返回旧值
    }123456789101112
```

这里通过typeCheck方法进行了key类型检测，判断是否为枚举类型，如果类型不对，会抛出异常

```
private void typeCheck(K key) {
   Class<?> keyClass = key.getClass();//获取类型信息
   if (keyClass != keyType && keyClass.getSuperclass() != keyType)
       throw new ClassCastException(keyClass + " != " + keyType);
}12345
```

接着通过`int index = key.ordinal()`的方式获取到该枚举实例的顺序值，利用此值作为下标，把值存储在vals数组对应下标的元素中即`vals[index]`，这也是为什么EnumMap能维持与枚举实例相同存储顺序的原因，我们发现在对vals[]中元素进行赋值和返回旧值时分别调用了maskNull方法和unmaskNull方法

```
 //代表NULL值得空对象实例
  private static final Object NULL = new Object() {
        public int hashCode() {
            return 0;
        }

        public String toString() {
            return "java.util.EnumMap.NULL";
        }
    };

    private Object maskNull(Object value) {
        //如果值为空，返回NULL对象，否则返回value
        return (value == null ? NULL : value);
    }

    @SuppressWarnings("unchecked")
    private V unmaskNull(Object value) {
        //将NULL对象转换为null值
        return (V)(value == NULL ? null : value);
    }123456789101112131415161718192021
```

由此看来EnumMap还是允许存放null值的，但key绝对不能为null，对于null值，EnumMap进行了特殊处理,将其包装为NULL对象，毕竟vals[]存的是Object，maskNull方法和unmaskNull方法正是用于null的包装和解包装的。这就是EnumMap集合的添加过程。下面接着看获取方法

```
 public V get(Object key) {
        return (isValidKey(key) ?
                unmaskNull(vals[((Enum<?>)key).ordinal()]) : null);
    }

 //对Key值的有效性和类型信息进行判断
 private boolean isValidKey(Object key) {
      if (key == null)
          return false;

      // Cheaper than instanceof Enum followed by getDeclaringClass
      Class<?> keyClass = key.getClass();
      return keyClass == keyType || keyClass.getSuperclass() == keyType;
  }1234567891011121314
```

相对应put方法，get方法显示相当简洁，key有效的话，直接通过ordinal方法取索引，然后在值数组vals里通过索引获取值返回。remove方法如下：

```
 public V remove(Object key) {
        //判断key值是否有效
        if (!isValidKey(key))
            return null;
        //直接获取索引
        int index = ((Enum<?>)key).ordinal();

        Object oldValue = vals[index];
        //对应下标元素值设置为null
        vals[index] = null;
        if (oldValue != null)
            size--;//减size
        return unmaskNull(oldValue);
    }1234567891011121314
```

非常简单，key值有效，通过key获取下标索引值，把vals[]对应下标值设置为null，size减一。查看是否包含某个值，

```
判断是否包含某value
public boolean containsValue(Object value) {
    value = maskNull(value);
    //遍历数组实现
    for (Object val : vals)
        if (value.equals(val))
            return true;

    return false;
}
//判断是否包含key
public boolean containsKey(Object key) {
    return isValidKey(key) && vals[((Enum<?>)key).ordinal()] != null;
}1234567891011121314
```

判断value直接通过遍历数组实现，而判断key就更简单了，判断key是否有效和对应vals[]中是否存在该值。ok~，这就是EnumMap的主要实现原理，即内部有两个数组，长度相同，一个表示所有可能的键(枚举值)，一个表示对应的值，不允许keynull，但允许value为null，键都有一个对应的索引，根据索引直接访问和操作其键数组和值数组，由于操作都是数组，因此效率很高。
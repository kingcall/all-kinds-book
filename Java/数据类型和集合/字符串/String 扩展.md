[TOC]



## String 扩展方法

### Join方法 让代码更优美

有时候我们会遇到将一个集合里面的字符串使用特定的分隔符进行拼接，这个时候我们可以使用join 方法，一方面是性能，一方面是代码简洁

```java
    @Test
    public void join(){
        String[] text = {"hello", "word","are","you","ok","?"};
        System.out.println(String.join(",", text));
        List<String> sList = new ArrayList<>();
        sList.add("a");
        sList.add("b");
        sList.add("c");
        System.out.println(String.join("-", sList));
        //在多个字符串对象中间加入连接字符
        System.out.println(String.join("\t","I","am","a","student"));
    }
```

输出结果

```
hello,word,are,you,ok,?
a-b-c
I	am	a	student
```

下面我们看一个性能测试

```java
    @Test
    public void joinPerformance() {
        int cnt = 10000;
        List<String> sList = new ArrayList<>();
        for (int i = 0; i < cnt; i++) {
            sList.add("" + i);
        }
        System.out.println(sList.size());
        long start = System.currentTimeMillis();
        String.join("-", sList);
        System.out.println("join:" + (System.currentTimeMillis() - start));
        String tmp = "";
        cnt = sList.size();
        start = System.currentTimeMillis();
        for (String s1 : sList) {
            tmp = tmp + s1 + "-";
        }
        System.out.println("for-string:" + (System.currentTimeMillis() - start));
    }
```

输出结果

```
10000
join:6
for-string:522
```

可以看出，使用join方法比你直接使用String 去拼更快，所以建议使用join 方法，那是因为join 方法底层使用的是StringBuilder 



### 字符串是否为空

在代码中我们经常要判断字符串是不是为空，这个时候你肯定会写出这样的代码，因为Java为你提供了这个方法

```java
    @Test
    public void isEmpty() {
        String x = "";
        System.out.println(x.isEmpty());
    }
```

这个代码的问题在哪里呢，那就是这个字符串可能是null,这个时候就会有空指针异常，怎么避免呢，你可以写一个自己的工具类，然后写一个这样的方法

```java
 private static boolean isStringNullorEmpty(String str){
  if(str == null || str.isEmpty()) {
   return true;
  }
  return false;
 }
```

因为很多时候如果字符串是null，从业务的角度来说我们也是认为它是空字符串，这个设计你可以在很多很多对字符串操作之前进行调研，例如trim() 方法





## String 扩展工具类

很多开源的第三方工具都提供了对字符串的扩展，谁让这是我们用的最多的数据类型呢

###  apache commons

### hutool

### guava

## String 扩展特性

### Compact Strings 

Java9之前的Java String类的实现将字符存储在字符数组中，每个字符使用两个字节—UTF-16编码。由于String是最常用的类之一，因此String实例是堆使用的主要组成部分。据观察，大多数字符串对象只包含Latin-1字符，只需要一个字节的存储空间。所以内部存储总是UTF-16意味着一半的存储空间将被闲置。

为了使字符串更节省空间，Java9以后，String类的内部表示形式已从UTF-16字符数组修改为字节数组加上编码标志字段，编码标志字段标记了采用何种编码格式

根据Java压缩字符串特性，字符串的内容要么被存储在SO-8859-1/Latin-1编码的字节数组中(一个字符占用一个字节)要么存储在UTF-16 编码的字符数组中(一个字符占用两个字节)

```java
/** The value is used for character storage. */
// private final char value[];  has been changed to byte[] array

private final byte[] value;
Encoding-flag field is named as coder and is of type byte-
private final byte coder;
```



## 总结

如果你还对String 的其他知识和技巧感兴趣请告诉我。这篇就是我们的String 系列的完结篇了，更多精彩请看

[String基础](https://blog.csdn.net/king14bhhb/article/details/111314482)

[String进阶之字符串常量池](https://blog.csdn.net/king14bhhb/article/details/111414710)

[String进阶之不可变性](https://blog.csdn.net/king14bhhb/article/details/111415444)

[String进阶之StringBuilder与StringBuffer](https://blog.csdn.net/king14bhhb/article/details/111566093)

[String扩展](https://blog.csdn.net/king14bhhb/article/details/111721430)
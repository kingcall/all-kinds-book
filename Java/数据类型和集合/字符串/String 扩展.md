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

## String 扩展工具类
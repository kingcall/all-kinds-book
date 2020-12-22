```java
    public static void main(String[] args) {
        String[] a1 = new String[5];
        a1[0] = "a";
        System.out.println(a1.length);
        a1 = Arrays.copyOf(a1, 10);
        System.out.println(a1.length);
    }
    // 运行结果
    5
	10
	a
```


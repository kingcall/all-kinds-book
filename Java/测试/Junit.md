[toc]
# 《Junit》
## 常见注解
### Rule
- Rule是JUnit4中的新特性，它让我们可以扩展JUnit的功能，灵活地改变测试方法的行为。
#### @Rule
- 是方法级别的，每个测试方法执行时都会调用被注解的Rule 
#### @ClassRule
- 是类级别的，在执行一个测试类的时候只会调用一次被注解的Rule

### before

 **@BeforeAll **和 **@AfterAll \**，它们定义了整个测试类在开始前以及结束时的操作，只能修饰静态方法，主要用于在测试过程中所需要的全局数据和外部资源的初始化和清理。与它们不同，\**@BeforeEach** 和 **@AfterEach** 所标注的方法会在每个测试用例方法开始前和结束时执行，主要是负责该测试用例所需要的运行环境的准备和销毁。



#### @BeforeAll
- 必须是static 
- 只执行一次，执行时机是在所有测试和 @BeforeEach 注解方法之前。
#### @BeforeEach 
### after
#### @AfterAll
- 必须是static 
- 只执行一次，执行时机是在所有测试和 @AfterEach 注解方法之后。
#### @AfterEach 

## 测试顺序
### 字母顺序
- junit 默认是字母顺序
```
对于方法名字是 testXXX()的方法，第一个XXX 就是按照XXX的字母顺序，testXXX中test是junit5中的规范
```
### order 注解
## 断言
- 准备好测试实例、执行了被测类的方法以后，断言能确保你得到了想要的结果
- 一般的断言，无非是检查一个实例的属性（比如，判空与判非空等），或者对两个实例进行比较（比如，检查两个实例对象是否相等）等。
- 无论哪种检查，断言方法都可以接受一个字符串作为最后一个可选参数，它会在断言失败时提供必要的描述信息。如果提供出错信息的过程比较复杂，它也可以被包装在一个 lambda 表达式中，这样，只有到真正失败的时候，消息才会真正被构造出来。





```java
@DisplayName("我的第一个测试用例")
public class MyFirstTestCaseTest {

    @BeforeAll
    public static void init() {
        System.out.println("初始化数据");
    }

    @AfterAll
    public static void cleanup() {
        System.out.println("清理数据");
    }

    @BeforeEach
    public void tearup() {
        System.out.println("当前测试方法开始");
    }

    @AfterEach
    public void tearDown() {
        System.out.println("当前测试方法结束");
    }

    @DisplayName("我的第一个测试")
    @Test
    void testFirstTest() {
        System.out.println("我的第一个测试开始测试");
    }

    @DisplayName("我的第二个测试")
    @Test
    void testSecondTest() {
        System.out.println("我的第二个测试开始测试");
    }
}
```


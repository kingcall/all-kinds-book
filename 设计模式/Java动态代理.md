本文主要介绍`Java`中两种常见的动态代理方式：`JDK原生动态代理`和`CGLIB动态代理`。

## 什么是代理模式
就是为其他对象提供一种代理以控制对这个对象的访问。代理可以在不改动目标对象的基础上，增加其他额外的功能（扩展功能）。

代理模式角色分为 3 种：

- `Subject`（抽象主题角色）：定义代理类和真实主题的公共对外方法，也是代理类代理真实主题的方法；
- `RealSubject`（真实主题角色）：真正实现业务逻辑的类；
- `Proxy`（代理主题角色）：用来代理和封装真实主题；

如果根据字节码的创建时机来分类，可以分为静态代理和动态代理：

- 所谓静态也就是在程序运行前就已经存在代理类的字节码文件，代理类和真实主题角色的关系在运行前就确定了。
- 而动态代理的源码是在程序运行期间由JVM根据反射等机制动态的生成，所以在运行前并不存在代理类的字节码文件

## 静态代理
学习动态代理前，有必要来学习一下静态代理。

静态代理在使用时,需要定义接口或者父类,被代理对象（目标对象）与代理对象（Proxy）一起实现相同的接口或者是继承相同父类。

来看一个例子，模拟小猫走路的时间。
```java
// 接口
public interface Walkable {
    void walk();
}

// 实现类
public class Cat implements Walkable {

    @Override
    public void walk() {
        System.out.println("cat is walking...");
        try {
            Thread.sleep(new Random().nextInt(1000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

如果我想知道走路的时间怎么办？可以将实现类`Cat`修改为：

```java
public class Cat implements Walkable {

    @Override
    public void walk() {
        long start = System.currentTimeMillis();
        System.out.println("cat is walking...");
        try {
            Thread.sleep(new Random().nextInt(1000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        long end = System.currentTimeMillis();
        System.out.println("walk time = " + (end - start));
    }
}
```
这里已经侵入了源代码，如果源代码是不能改动的，这样写显然是不行的，这里可以引入时间代理类`CatTimeProxy`。

```java
public class CatTimeProxy implements Walkable {
    private Walkable walkable;

    public CatTimeProxy(Walkable walkable) {
        this.walkable = walkable;
    }

    @Override
    public void walk() {
        long start = System.currentTimeMillis();

        walkable.walk();

        long end = System.currentTimeMillis();
        System.out.println("Walk time = " + (end - start));
    }
}
```
如果这时候还要加上常见的日志功能，我们还需要创建一个日志代理类`CatLogProxy`。
```java
public class CatLogProxy implements Walkable {
    private Walkable walkable;

    public CatLogProxy(Walkable walkable) {
        this.walkable = walkable;
    }

    @Override
    public void walk() {
        System.out.println("Cat walk start...");

        walkable.walk();

        System.out.println("Cat walk end...");
        
    }
}
```

如果我们需要先记录日志，再获取行走时间，可以在调用的地方这么做：
```java
public static void main(String[] args) {
    Cat cat = new Cat();
    CatLogProxy p1 = new CatLogProxy(cat);
    CatTimeProxy p2 = new CatTimeProxy(p1);

    p2.walk();
}
```
这样的话，计时是包括打日志的时间的。

### 静态代理的问题
如果我们需要计算`SDK`中100个方法的运行时间，同样的代码至少需要重复100次，并且创建至少100个代理类。往小了说，如果`Cat`类有多个方法，我们需要知道其他方法的运行时间，同样的代码也至少需要重复多次。因此，静态代理至少有以下两个局限性问题：
- 如果同时代理多个类，依然会导致类无限制扩展
- 如果类中有多个方法，同样的逻辑需要反复实现

所以，我们需要一个通用的代理类来代理所有的类的所有方法，这就需要用到动态代理技术。

## 动态代理
学习任何一门技术，一定要问一问自己，这到底有什么用。其实，在这篇文章的讲解过程中，我们已经说出了它的主要用途。你发现没，使用动态代理我们居然可以在不改变源码的情况下，直接在方法中插入自定义逻辑。这有点不太符合我们的一条线走到底的编程逻辑，这种编程模型有一个专业名称叫`AOP`。所谓的`AOP`，就像刀一样，抓住时机，趁机插入。

![](https://user-gold-cdn.xitu.io/2018/3/2/161e5ba2dfeb24bb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


## Jdk动态代理
JDK实现代理只需要使用newProxyInstance方法,但是该方法需要接收三个参数:
```java
@CallerSensitive
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        Objects.requireNonNull(h);

        final Class<?>[] intfs = interfaces.clone();
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }

        /*
         * Look up or generate the designated proxy class.
         */
        Class<?> cl = getProxyClass0(loader, intfs);

        /*
         * Invoke its constructor with the designated invocation handler.
         */
        try {
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }

            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }
```
方法是在`Proxy`类中是静态方法,且接收的三个参数依次为:
- `ClassLoader loader`  //指定当前目标对象使用类加载器
- `Class<?>[] interfaces`  //目标对象实现的接口的类型,使用泛型方式确认类型
- `InvocationHandler h`  //事件处理器

主要是完成`InvocationHandler h`的编写工作。

接口类`UserService`：
```java
public interface UserService {

    public void select();

    public void update();
}
```
接口实现类，即要代理的类`UserServiceImpl`：
```java
public class UserServiceImpl implements UserService {
    @Override
    public void select() {
        System.out.println("查询 selectById");
    }

    @Override
    public void update() {
        System.out.println("更新 update");
    }
}
```
代理类`UserServiceProxy`：
```java
public class UserServiceProxy implements UserService {
    private UserService target;

    public UserServiceProxy(UserService target){
        this.target = target;
    }

    @Override
    public void select() {
        before();
        target.select();
        after();
    }

    @Override
    public void update() {
        before();
        target.update();
        after();
    }

    private void before() {     // 在执行方法之前执行
        System.out.println(String.format("log start time [%s] ", new Date()));
    }
    private void after() {      // 在执行方法之后执行
        System.out.println(String.format("log end time [%s] ", new Date()));
    }
}
```
主程序类：
```java
public class UserServiceProxyJDKMain {
    public static void main(String[] args) {

        // 1. 创建被代理的对象，即UserService的实现类
        UserServiceImpl userServiceImpl = new UserServiceImpl();

        // 2. 获取对应的classLoader
        ClassLoader classLoader = userServiceImpl.getClass().getClassLoader();

        // 3. 获取所有接口的Class, 这里的userServiceImpl只实现了一个接口UserService，
        Class[] interfaces = userServiceImpl.getClass().getInterfaces();

        // 4. 创建一个将传给代理类的调用请求处理器，处理所有的代理对象上的方法调用
        //     这里创建的是一个自定义的日志处理器，须传入实际的执行对象 userServiceImpl
        InvocationHandler logHandler = new LogHandler(userServiceImpl);

        /*
		   5.根据上面提供的信息，创建代理对象 在这个过程中，
               a.JDK会通过根据传入的参数信息动态地在内存中创建和.class 文件等同的字节码
               b.然后根据相应的字节码转换成对应的class，
               c.然后调用newInstance()创建代理实例
		 */

        // 会动态生成UserServiceProxy代理类，并且用代理对象实例化LogHandler，调用代理对象的.invoke()方法即可
        UserService proxy = (UserService) Proxy.newProxyInstance(classLoader, interfaces, logHandler);

        // 调用代理的方法
        proxy.select();
        proxy.update();
        
        // 生成class文件的名称
        ProxyUtils.generateClassFile(userServiceImpl.getClass(), "UserServiceJDKProxy");
    }
}
```
这里可以保存下来代理生成的实现了接口的代理对象：
```java
public class ProxyUtils {
    /*
     * 将根据类信息 动态生成的二进制字节码保存到硬盘中，
     * 默认的是clazz目录下
     * params :clazz 需要生成动态代理类的类
     * proxyName : 为动态生成的代理类的名称
     */
    public static void generateClassFile(Class clazz, String proxyName) {

        //根据类信息和提供的代理类名称，生成字节码
        byte[] classFile = ProxyGenerator.generateProxyClass(proxyName, clazz.getInterfaces());
        String paths = clazz.getResource(".").getPath();
        System.out.println(paths);
        FileOutputStream out = null;

        try {
            //保留到硬盘中
            out = new FileOutputStream(paths + proxyName + ".class");
            out.write(classFile);
            out.flush();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                out.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

}
```
### 动态代理实现过程

1. 通过`getProxyClass0()`生成代理类。`JDK`生成的最终真正的代理类，它继承自`Proxy`并实现了我们定义的接口.
2. 通过`Proxy.newProxyInstance()`生成代理类的实例对象，创建对象时传入`InvocationHandler`类型的实例。
3. 调用新实例的方法，即原`InvocationHandler`类中的`invoke()`方法。

代理对象不需要实现接口，但是目标对象一定要实现接口，否则不能用动态代理

## Cglib动态代理
`JDK`的动态代理机制只能代理实现了接口的类，而不能实现接口的类就不能实现`JDK`的动态代理，`cglib`是针对类来实现代理的，他的原理是对指定的目标类生成一个子类，并覆盖其中方法实现增强，但因为采用的是继承，所以不能对`final`修饰的类进行代理。 

`Cglib`代理，也叫作子类代理，它是在内存中构建一个子类对象从而实现对目标对象功能的扩展。

`Cglib`子类代理实现方法:
1. 需要引入`cglib`的`jar`文件,但是`Spring`的核心包中已经包括了`Cglib`功能,所以直接引入`Spring-core.jar`即可.
2. 引入功能包后,就可以在内存中动态构建子类
3. 代理的类不能为`final`,否则报错
4. 目标对象的方法如果为`final/static`,那么就不会被拦截,即不会执行目标对象额外的业务方法.

基本使用
```xml
<!-- https://mvnrepository.com/artifact/cglib/cglib -->
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>2.2</version>
</dependency>
```
方法拦截器
```java
public class LogInterceptor implements MethodInterceptor{

    /*
     * @param o 要进行增强的对象
     * @param method 要拦截的方法
     * @param objects 参数列表，基本数据类型需要传入其包装类
     * @param methodProxy 对方法的代理，
     * @return 执行结果
     * @throws Throwable
     */
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        before();
        Object result = methodProxy.invokeSuper(o, objects);
        after();
        return result;

    }

    private void before() {
        System.out.println(String.format("log start time [%s] ", new Date()));
    }
    private void after() {
        System.out.println(String.format("log end time [%s] ", new Date()));
    }
}
```
测试用例

这里保存了代理类的`.class`文件
```java
public class CglibMain {

    public static void main(String[] args) {
        // 创建Enhancer对象，类似于JDK动态代理的Proxy类
        Enhancer enhancer = new Enhancer();
        // 设置目标类的字节码文件
        enhancer.setSuperclass(UserDao.class);
        // 设置回调函数
        enhancer.setCallback(new LogInterceptor());
        // create会创建代理类
        UserDao userDao = (UserDao)enhancer.create();
        userDao.update();
        userDao.select();
    }
}
```
结果
```
log start time [Mon Nov 30 17:26:39 CST 2020] 
UserDao 更新 update
log end time [Mon Nov 30 17:26:39 CST 2020] 
log start time [Mon Nov 30 17:26:39 CST 2020] 
UserDao 查询 selectById
log end time [Mon Nov 30 17:26:39 CST 2020] 
```

## JDK动态代理与CGLIB动态代理对比
#### JDK 动态代理
- 为了解决静态代理中，生成大量的代理类造成的冗余；
- `JDK` 动态代理只需要实现 `InvocationHandler` 接口，重写 `invoke` 方法便可以完成代理的实现，
- jdk的代理是利用反射生成代理类 `Proxyxx.class` 代理类字节码，并生成对象
- jdk动态代理之所以只能代理接口是因为代理类本身已经`extends`了`Proxy`，而java是不允许多重继承的，但是允许实现多个接口

**优点**：解决了静态代理中冗余的代理实现类问题。

**缺点**：`JDK` 动态代理是基于接口设计实现的，如果没有接口，会抛异常。


#### CGLIB 代理

- 由于`JDK` 动态代理限制了只能基于接口设计，而对于没有接口的情况，`JDK`方式解决不了；
- `CGLib` 采用了非常底层的字节码技术，其原理是通过字节码技术为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑，来完成动态代理的实现。
- 实现方式实现 `MethodInterceptor` 接口，重写 `intercept` 方法，通过 `Enhancer` 类的回调方法来实现。
- 但是`CGLib`在创建代理对象时所花费的时间却比JDK多得多，所以对于单例的对象，因为无需频繁创建对象，用`CGLib`合适，反之，使用`JDK`方式要更为合适一些。
- 同时，由于`CGLib`由于是采用动态创建子类的方法，对于`final`方法，无法进行代理。

**优点**：没有接口也能实现动态代理，而且采用字节码增强技术，性能也不错。

**缺点**：技术实现相对难理解些。
[toc]

## 概念

### 方法和函数
- Scala 有方法与函数，二者在语义上的区别很小。Scala 方法是类的一部分，而函数是一个对象可以赋值给一个变量。换句话来说在类中定义的函数即是方法。

#### 多态方法
- Scala 中的方法可以按类型和值进行参数化。 语法和泛型类类似。 类型参数括在方括号中，而值参数括在圆括号中
- 在java 中也存在泛型，但是用的更多的实方法的重载(其实这里我们可以给一个结论性的东西，对于仅仅是类型不一样的方法就使用泛型，对于参数个数不一样的时候，就使用重载)

```
def listOfDuplicates[A](x: A, length: Int): List[A] = {
  if (length < 1)
    Nil
  else
    x :: listOfDuplicates(x, length - 1)
}
println(listOfDuplicates[Int](3, 4))  // List(3, 3, 3, 3)
println(listOfDuplicates("La", 8)) 
// // List(La, La, La, La, La, La, La, La)
```
- 上例中第一次调用方法时，我们显式地提供了类型参数 [Int]。 因此第一个参数必须是 Int 类型，并且返回类型为 List[Int]。

- 上例中第二次调用方法，表明并不总是需要显式提供类型参数。 编译器通常可以根据上下文或值参数的类型来推断。 在这个例子中，"La" 是一个 String，因此编译器知道 A 必须是 String

#### 函数的定义
- =>的左边是参数列表，右边是一个包含参数的表达式
#### 方法的定义
- 方法的表现和行为和函数非常类似，但是它们之间有一些关键的差别。方法由def关键字定义。def后面跟着一个名字、参数列表、返回类型和方法体。

#### 可变参数函数
#### 默认值参数函数
#### 内嵌函数
-  Scala 函数内定义函数，定义在函数内的函数称之为局部函数。

####  偏应用函数
- Scala 偏应用函数是一种表达式，你不需要提供函数需要的所有参数，只需要提供部分，或不提供所需参数。

#### 强制转换方法为函数
```
object T{
    def convertCtoF(temp: Int) = temp * 1.8 + 32
}
val salaries = Seq(20000, 70000, 40000)
val newSalaries = salaries.map(T.convertCtoF)
```
#### 高阶函数
- 高阶函数可以使用其他函数作为参数，或者使用函数作为输出结果。
##### map

#### 柯里化函数
- 方法可以定义多个参数列表，当使用较少的参数列表调用多参数列表的方法时，会产生一个新的函数，该函数接收剩余的参数列表作为其参数。这被称为柯里化。
- 隐式（IMPLICIT）参数如果要指定参数列表中的某些参数为隐式（implicit），应该使用多参数列表

##### foldLeft
- 常规操作
```
val numbers = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
val res = numbers.foldLeft(0)((m, n) => m + n)
```
- 柯里化操作
```
val numbers = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
val numberFunc = numbers.foldLeft(List[Int]())_
```
##### 单一的函数参数
- 注意使用**多参数列表**时，我们还可以利用Scala的类型推断来让代码更加简洁（如下所示），而如果没有多参数列表，这是不可能的
```
numbers.foldLeft(0)(_ + _)
```

#### 命名参数函数(和函数无关)
- 因为使用命名参数可忽略参数的顺序，在调用时候非常方便，尤其是参数多的情况，调用时用参数名称和参数值同时出现的方法，同时提高代码的可读性.
- 一般情况下函数调用参数，就按照函数定义时的参数顺序一个个传递。但是我们也可以通过指定函数参数名，并且不需要按照顺序向函数传递参数
- 当调用方法时，实际参数可以通过其对应的形式参数的名称来标记
- 注意使用命名参数时，顺序是可以重新排列的。 但是，如果某些参数被命名了，而其他参数没有，则未命名的参数要按照其方法签名中的参数顺序放在前面
```
object Test {
  def main(args: Array[String]) {
    printInt(b=5, a=7);
  }
  def printInt( a:Int, b:Int ) = {
    println("Value of a : " + a );
    println("Value of b : " + b );
  }
}
```

#### 传名参数函数
##### 传值参数函数
- 传值参数在函数调用之前表达式会被求值，例如Int，Long等数值参数类型
- 传名参数在函数调用前表达式不会被求值，而是会被包裹成一个匿名函数作为函数参数传递下去

##### 传名参数函数
- 传名参数 仅在被使用时触发实际参数的求值运算。 它们与 传值参数 正好相反。 要将一个参数变为传名参数，只需在它的类型前加上 =>
- 传名参数的优点是，如果它们在函数体中未被使用，则不会对它们进行求值。
```
如果参数是计算密集型或长时间运行的代码块，如获取 URL，这种延迟计算参数直到它被使用时才计算的能力可以帮助提高性能。
```
```
def whileLoop(condition: => Boolean)(body: => Unit): Unit =
  if (condition) {
    body
    whileLoop(condition)(body)
  }

var i = 2

whileLoop (i > 0) {
  println(i)
  i -= 1
}  // print
```

```
object ByNameParameterGunction {
  /**
   * 编译器检测到strToInt接受一个传值参数，所以先对传入的参数表达式{println("eval parameter expression"); "123"}，然后再讲求值结果传递给strToInt
   * @param s
   * @return
   */
  def strToInt(s: String) = {
    println("call strToInt")
    s.toInt
  }

  /**
   * 传名参数在函数调用前表达式不会被求值，而是会被包裹成一个匿名函数作为函数参数传递下去
   * 在strToInt函数声明中添加一个=>，参数s的类型就变成了无参函数，类型为:() => String，按照Scala针对无参函数的简化规则，此处省略了()。因为参数s的类型是无参函数，所以此处是按名传递。
   * @param s
   * @return
   */
  def strToInt2(s: => String) = {
    println("call strToInt")
    s.toInt
  }

  def main(args: Array[String]) {
    strToInt({println("eval parameter expression"); "123"})
    strToInt2({println("eval parameter expression"); "123"})
  }
}
```

###  其他概念
#### 代码块
#### 类型推断
- Scala 编译器通常可以推断出表达式的类型，因此你不必显式地声明它。
    - 编译器可以推断出赋值表达式的类型，所以无需申明变量的类型
    ```
    val businessName = "Montreux Jazz Café"
    ```
    - 编译器可以推断出方法的返回类型，所以无需申明方法的返回类型
    ```
    def squareOf(x: Int) = x * x
    ```
    - 对于递归方法，编译器无法推断出结果类型。（下面的方法是编译不通过的）
    ```
    def fac(n: Int) = if (n == 0) 1 else n * fac(n - 1)
    ```
    - 当调用 多态方法 或实例化 泛型类 时，也不必明确指定类型参数。 Scala 编译器将从上下文和实际方法的类型/构造函数参数的类型推断出缺失的类型参数。
    ```
    case class MyPair[A, B](x: A, y: B)
    val p = MyPair(1, "scala") // type: MyPair[Int, String]

    def id[T](x: T) = x
    val q = id(1)              // type: Int
    ```
```
编译器从不推断方法形式参数的类型——形参的类型一定是可知的
但是在某些情况下，当函数作为参数传递时，编译器可以推断出匿名函数形式参数的类型
```
##### 类型推断总结——什么时候会发生类型推断
###### 变量定义时的推断
###### 方法返回值的推断(递归方法除外)
###### 多态方的调用
###### 泛型类的调用
###### 实参的类型推断(函数作为参数传递的时候)
```
Seq(1, 3, 4).map(x => x * 2)  // List(2, 6, 8)
方法 map 的形式参数是 f: A => B。 因为我们把整数放在 Seq 中，编译器知道 A 是 Int 类型 (即 x 是一个整数)。 因此，编译器可以从 x * 2 推断出 B 是 Int 类型。
```
##### 类型推断总结——何时不要依赖类型推断
- 公开可访问的 API 成员应该具有显示类型声明以增加可读性。 因此，我们建议你将代码中向用户公开的任何 API 明确指定类型


#### 运算符
- 在Scala中，运算符即是方法（scala 才是真正的面向对象）。 任何具有单个参数的方法都可以用作 中缀运算符
```
10.+(1) 而中缀运算符则更易读:10 + 1
```
#### 注解
##### @deprecated
##### @tailrec
##### @inline
##### @interface

#### 包和导入
##### 包
- Scala 使用包来创建命名空间，从而允许你创建模块化程序。
- 通过在 Scala 文件的头部声明一个或多个包名称来创建包
- 一个惯例是将包命名为与包含 Scala 文件的目录名相同。 但是，Scala 并未对文件布局作任何限制
- 明包的另一种方式是使用大括号
```
package users {
  package administrators {
    class NormalUser
  }
  package normalusers {
    class NormalUser
  }
}
```
##### 导入
- import 语句用于导入其他包中的成员（类，特质，函数等）。 使用相同包的成员不需要 import 语句。
```
import users._  // 导入包 users 中的所有成员
import users.User  // 导入类 User
import users.{User, UserPreferences}  // 仅导入选择的成员
import users.{UserPreferences => UPrefs}  // 导入类并且设置别名
```
- Scala 不同于 Java 的一点是 Scala 可以在任何地方使用导入

##### 包对象
- Scala 提供包对象作为在整个包中方便的共享使用的容器。
- 包对象中可以定义任何内容，而不仅仅是变量和方法。 -              - 包对象经常用于保存包级作用域的类型别名和隐式转换。
    - 包对象甚至可以继承 Scala 的类和特质。
- 包对象的代码通常放在名为 package.scala 的源文件中。
- 每个包都允许有一个包对象。在包对象中的任何定义都被认为是包自身的成员
- 包对象与其他对象类似，这意味着你可以使用继承来构建它们。 例如，一个包对象可能会混入多个特质
- 方法重载在包对象中不起作用。

### 样例类
- Scala有一种特殊的类叫做样例类（case class）。默认情况下，样例类一般用于不可变对象，并且可作值比较。你可以使用case class关键字来定义样例类

### 对象
- 对象是它们自己定义的单实例，你可以把它看作它自己的类的单例。你可以使用object关键字定义对象。

### For 表达式
- Scala 提供一个轻量级的标记方式用来表示 序列推导。推导使用形式为 for (enumerators) yield e 的 for 表达式，此处 enumerators 指一组以分号分隔的枚举器。一个 enumerator 要么是一个产生新变量的生成器，要么是一个过滤器。for 表达式在枚举器产生的每一次绑定中都会计算 e 值，并在循环结束后返回这些值组成的序列
- 其实就是通过yield 将for 循环中的变量返回，生成一个列表；For 表达式可以有守卫，堆循环中的变量进行返回

### 型变
- 型变是复杂类型的子类型关系与其组件类型的子类型关系的相关性。 
- Scala支持 泛型类 的类型参数的型变注释，允许它们是协变的，逆变的，或在没有使用注释的情况下是不变的。 在类型系统中使用型变允许我们在复杂类型之间建立直观的连接，而缺乏型变则会限制类抽象的重用性。
- 进一步提升方法的适用性

#### 协变
- 协变使用注释 +A，可以使一个泛型类的类型参数 A 成为协变。 对于某些类 class List[+A]，使 A 成为协变意味着对于两种类型 A 和 B，如果 A 是 B 的子类型，那么 List[A] 就是 List[B] 的子类型。 这允许我们使用泛型来创建非常有用和直观的子类型关系。
- 从上述描述我们知道java 它是支持协变的(只不过是没有这个概念而已)。

#### 逆变
- 通过使用注释 -A，可以使一个泛型类的类型参数 A 成为逆变。 与协变类似，这会在类及其类型参数之间创建一个子类型关系，但其作用与协变完全相反。 也就是说，对于某个类 class Writer[-A] ，使 A 逆变意味着对于两种类型 A 和 B，如果 A 是 B 的子类型，那么 Writer[B] 是 Writer[A] 的子类型。

#### 不变
- 默认情况下，Scala中的泛型类是不变的。 这意味着它们既不是协变的也不是逆变的。 在下例中，类 Container 是不变的。 Container[Cat] 不是 Container[Animal]，反之亦然。

### 类型上下界
#### 类型上界
- 在Scala中，类型参数和抽象类型都可以有一个类型边界约束。这种类型边界在限制类型变量实际取值的同时还能展露类型成员的更多信息。比如像T <: A这样声明的类型上界表示类型变量T应该是类型A的子类

#### 类型下界
- 类型上界 将类型限制为另一种类型的子类型，而 类型下界 将类型声明为另一种类型的超类型。 术语 B >: A 表示类型参数 B 或抽象类型 B 是类型 A 的超类型。 在大多数情况下，A 将是类的类型参数，而 B 将是方法的类型参数

## 关键字
### Protected
在 scala 中，对保护（Protected）成员的访问比 java 更严格一些。因为它**只允许保护成员在定义了该成员的的类的子类中被访问**。而在java中，用protected关键字修饰的成员，除了定义了该成员的类的子类可以访问，同一个包里的其他类也可以进行访问。
### trait
- Scala Trait(特征)相当于Java的接口，实际上它比接口还功能强与接口不同的是，它还可以定义属性和方法的实现。

### case
#### case class
#### 模式匹配
- 它是有返回值的

##### 普通变量的模式匹配

##### 集合的模式匹配
##### 案例类的模式匹配
##### 类型匹配
##### 模式守卫（Pattern gaurds）
#### 偏函数的定义

## scala 类型层次
![image-20201127203954079](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/20:39:55-image-20201127203954079.png)
- scala 中一切皆对象，不像java 中还有基本类型和引用类型的区别，其实我们可以将scala 中值类型java 中基本类型的包装类,需要注意的是java 中的String 是对象类型
```
List<int> x = new ArrayList<>();
List<String> xx = new ArrayList<>();
```
- 对于scala 中提供的一些特殊类型(Option,Either,Try)，都存在着一些比较高级的操作，可以帮助我们减少很多代码量和不优美的操作
### Any
- 是所有类型的超类型，也称为顶级类 型。它定义了一些通用的方法如equals、hashCode和toString。Any有两个直接子类：AnyVal和AnyRef。

### AnyVal
- 代表值类型。有9个预定义的非空的值类型分别是：Double、Float、Long、Int、Short、Byte、Char、Unit和Boolean。
- Unit是不带任何意义的值类型，它仅有一个实例可以像这样声明：()。所有的函数必须有返回，所以说有时候Unit也是有用的返回类型

#### java 的 void
- java 的void 不是一中类型，仅仅用来标识方法无返回值
- java 中的void 也不允许当做变量使用
```
 // 这段代码是编译不通过的
 void x = null;
 public static void main(String[] args) throws IOException {
     System.out.println(t());
 }
 public static void t(){
 }
```
#### scala 的Unit
- 首先它是一种基本的类型(值类型)
- 因为是基本类型，当然可以声明该类型的变量
```
val x:Unit=()
println(x)
println(x.isInstanceOf[Unit])
```

### AnyRef
- 代表引用类型。所有非值类型都被定义为引用类型。在Scala中，每个用户自定义的类型都是AnyRef的子类型。如果Scala被应用在Java的运行环境中，AnyRef相当于java.lang.Object。

### Nothing和Null
#### Nothing
- 是所有类型的子类型，也称为底部类型。没有一个值是Nothing类型的。它的用途之一是给出非正常终止的信号，如抛出异常、程序退出或者一个无限循环（可以理解为它是一个不对值进行定义的表达式的类型，或者是一个不能正常返回的方法）。

#### Null
- 是所有引用类型的子类型（即AnyRef的任意子类型）。它有一个单例值由关键字null所定义。
- Null主要是使得Scala满足和其他JVM语言的互操作性，但是几乎不应该在Scala代码中使用。我们将在后面的章节中介绍null的替代方案。
- 该类型的唯一值是 null，因此无法分配其他的值

### Option
#### Some
#### None
### Either
#### Left
#### Right
### Try

#### Success
#### Failure
### scala 的类型转换

![image-20201127204015424](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/27/20:40:16-image-20201127204015424.png)

## 面向对象
### class
#### 成员
- 类包含的方法、常量、变量、类型、对象、特质、类，这些统称为成员。(和java 不一样，java 区分字段和方法，字段和方法又区分类变量和类方法、实例变量和实例方法)
- 其实scala 这样统称是有原因的，因为scala 的类是不能直接运行的，但是java 可以的。

#### 构造器
- 默认无参构造器
- 有参构造器
- 默认值构造器
- 主构造器
- 辅助构造器

#### 混入(MIXIN)
- 当某个特质被用于组合类时，被称为混入
### trait
- 特质 (Traits) 用于在类 (Class)之间共享程序接口 (Interface)和字段 (Fields)。 它们类似于Java 8的接口。 类和对象 (Objects)可以扩展特质，但是特质不能被实例化，因此特质没有参数。

#### 子类型
- 凡是需要特质的地方，都可以由该特质的子类型来替换。

### 密封类
- 特质（trait）和类（class）可以用sealed标记为密封的，这意味着其所有子类都必须与之定义在相同文件中，从而保证所有子类型都是已知的
```
sealed abstract class Furniture
case class Couch() extends Furniture
case class Chair() extends Furniture

def findPlaceToSit(piece: Furniture): String = piece match {
  case a: Couch => "Lie on the couch"
  case b: Chair => "Sit on the chair"
}
```
- 这对于模式匹配很有用，因为我们不再需要一个匹配其他任意情况的case

### 泛型类
- 泛型类使用方括号 [] 来接受类型参数。一个惯例是使用字母 A 作为参数标识符，当然你可以使用任何参数名称。
- 泛型类在集合中用的比较多

### 单例对象
- 单例对象是一种特殊的类，有且只有一个实例。和惰性变量一样，单例对象是延迟创建的，当它第一次被使用时创建。
- 当对象定义于顶层时(即没有包含在其他类中)，单例对象只有一个实例。
- 当对象定义在一个类或方法中时，单例对象表现得和惰性变量一样。（需要注意的实单例对象的方法是直接可以导入的，这一点和java 不一样）

### 伴生对象
- 当一个单例对象和某个类共享一个名称时，这个单例对象称为 伴生对象。 同理，这个类被称为是这个单例对象的伴生类。
- 类和它的伴生对象可以互相访问其私有成员。
- 使用伴生对象来定义那些在伴生类中**不依赖于实例化对象而存在的成员变量或者方法**。
- 伴生对象中的方法，对于伴生类的实例对象是不可见的
- 在 Java 代码中调用伴生对象时，伴生对象的成员会被定义成伴生类中的static成员。这称为静态转发。这种行为发生在当你自己没有定一个伴生类时。

### 构造器对象

### 提取器对象
- 提取器对象是一个包含有 unapply 方法的单例对象。apply 方法就像一个构造器，接受参数然后创建一个实例对象，反之 unapply 方法接受一个实例对象然后**返回最初创建它所用的参数**。提取器常用在模式匹配和偏函数中。

### 内部类
- 在Scala中，一个类可以作为另一个类的成员。 在一些类似 Java 的语言中，内部类是外部类的成员，而 Scala 正好相反，内部类是绑定到外部对象的。 
- 假设我们希望编译器在编译时阻止我们混淆节点 nodes 与图形 graph 的关系，**路径依赖类型**提供了一种解决方案。

### 抽象类
- 特质和抽象类可以包含一个抽象类型成员，意味着实际类型可由具体实现来确定
```
trait Buffer {
  type T
  val element: T
}
```
- 一般情况下，抽象类型的实现都可以使用抽象参数类型来实现，此外要说明的是，有些情况下用类型参数替换抽象类型是行不通的。
### 复合类型
- 有时需要表明一个对象的类型是其他几种类型的子类型。 在 Scala 中，这可以表示成 复
- 合类型，即多个类型的交集。
```
    trait Resetable {
      def reset: Unit
    }
    trait Cloneable extends java.lang.Cloneable {
      override def clone(): Cloneable = {
        super.clone().asInstanceOf[Cloneable]
      }
    }
  class T extends Cloneable with Resetable {
    /**
      * 注意一下这种写法，可以帮助你通过编译，有点类似python 里面的 pass
      */
    override def reset: Unit = ???
  }

  def cloneAndReset(obj: Cloneable with Resetable): Cloneable = {
    val cloned = obj.clone()
    obj.reset
    cloned

  }
```
### 自类型
- 自类型用于声明一个特质必须混入其他特质，尽管该特质没有直接扩展其他特质。 这使得所依赖的成员可以在没有导入的情况下使用。
- 自类型是一种细化 this 或 this 别名之类型的方法。 语法看起来像普通函数语法，但是意义完全不一样

## 隐式
### 隐式参数
- 方法可以具有 隐式 参数列表，由参数列表开头的 implicit 关键字标记。 如果参数列表中的参数没有像往常一样传递， Scala 将查看它是否可以获得正确类型的隐式值，如果可以，则自动传递。

#### 查找隐式参数
- Scala 将查找这些参数的位置分为两类：
- Scala 在调用包含有隐式参数块的方法时，将首先查找可以直接访问的隐式定义和隐式参数 (无前缀)。
- 然后，它在所有伴生对象中查找与隐式候选类型相关的有隐式标记的成员。

### 隐式转换
- 一个从类型 S 到类型 T 的隐式转换由一个函数类型 S => T 的**隐式值**来定义，或者由一个可转换成所需值的**隐式方法**来定义。
```
从定义我们了解到它可能是一个方法，也可能是一个值
```
#### 隐式转换——方法
- 隐式转换方法是指在同一个作用域下面，一个给定输入类型并自动转换为指定返回类型的函数，
- 这个函数和函数名字无关，和入参名字无关，只和入参类型以及返回类型有关。注意是同一个作用域。
- 而且会自动被调用，而且只要输入参数和输出参数的类型相同就被认为是一样的隐式函数，可能会出现二意性
```
object ImplicitDemo {
    /*
    * 在这个例子中，其实想着我为什么不定义一系列的重载函数，但是如果重载函数很长的话，就比较啰嗦了，
    * 可以看出隐式转换函数有缩减代码量的好处
    * */
    def display(input: String): Unit = println(input)

    implicit def typeConvertor(input: Int): String ={
        println("Int->String")
        input.toString
    }

    implicit def typeConvertor(input: Boolean): String ={
        println("Boolean->String")
        if (input) "true" else "false"
    }

    implicit val implicitValue=(x:Double)=>{println("Double->String");x.toString}

    def main(args: Array[String]): Unit = {
        display("1212")
        display(12)
        display(true)
        display(123.4D)
    }

}
```

#### 隐式转换——值(函数)
```
 implicit val implicitValue=(x:Double)=>{println("Double->String");x.toString}
```
#### 隐式转换堆方法重载的意义
##### 重载的定义
- 重载（Overload）是让类以统一的方式处理不同类型数据的一种手段，实质表现就是多个具有不同的参数个数或者类型的同名函数（返回值类型可随意，不能以返回类型作为重载函数的区分标准）同时存在于同一个类中，**是一个类中多态性的一种表现**（调用方法时通过传递不同参数个数和参数类型来决定具体使用哪个方法的多态性）。
- 隐式转换的重载的意义在于，不用去写那么多的重载方法，而是通过定义隐式转换（值或者方法），完成参数类型的转换，更加符合设计模式的解耦
- 需要注意的实，隐式转换不能完全代替重载，因为隐式转换不能改变参数的个数(其实这可以通过隐式转换的第二种特性完成，调用不存在的方法)
- 重载也不能完全替代隐式转换，因为隐式转换的调用更加简洁——不需要手动调用


#### 隐式转换的使用情况
- 如果一个表达式 e 的类型为 S， 并且类型 S 不符合表达式的期望类型 T
```
1. 存在一个方法或者函数的参数类型为T,当传入的参数的类型为S，这个时候存在一个隐式转换 S=>T,那么这个隐式转换将被调用
2. 赋值的时候不对的类型的时候也需要隐式转换 val x:Int=3.5D
```

```
implicit def double2Int(x: Double) = x.toInt
var x: Int = 3.5
print(x)
```

- 在一个类型为 S 的实例对象 e 中调用 e.m， 如果被调用的 m 并没有在类型 S 中声明。
```
在一个对象上调用一个不存在的方法，但是却存在这样的隐式方法(发现单纯的隐式方法不行，必须借助隐式类来定义)
```
```
object One {
  implicit class OneMethod(val second: Int) {
    implicit def read(num: Int) = num + second
  }
  def main(args: Array[String]): Unit = {
    println(1.read(2)) // 隐式对象转换  Int类找不到read方法
  }
}
```
#### 例子
- 当调用一个接受 java.lang.Integer 作为参数的 Java 方法时，你完全可以传入一个 scala.Int。那是因为 Predef 包含了以下的隐式转换：
```
import scala.language.implicitConversions

implicit def int2Integer(x: Int) =
  java.lang.Integer.valueOf(x)
```

## 数据类型
- 元组是一个可以容纳不同类型元素的类。 元组是不可变的。当我们需要从函数返回多个值时，元组会派上用场。
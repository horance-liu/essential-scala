# 编程元素

本章讲述`Scala`最基本的概念和习惯用法，包括变量定义，函数定义，类定义等，帮助读者快速融入到`Scala`社区之中。

## 程序设计元素

任何一门程序设计语言存在三种本质的机制，从而将若干简单的认识组合为一个复合的认识，由此产生出各种复杂的认识。

- 原子：表示语言中最简单的个体；
- 组合：组合简单的原子或复合对象为更复杂的对象，常称为「组合式」；
- 抽象：通过对组合对象「命名」，并将其当作黑盒去操作，而无需关心内部细节；

### 一切皆表达式

最简单的原子表达式莫过于一个数字。

```scala
1
```

可以将数字的表达式组合起来，形成复合的「组合式」。

```scala
1 + 2
```

也可以使用「标示符」命名一个表达式，它可以引用一个简单的原子表达式；

```scala
val pi = 3.14159
val raidus = 10
```

也可一个引用一个复合对象。

```scala
val circumference = 2 * pi * radius
```

更有甚者，可以将复合运算命名为一个函数。

```scala
def circumference(radius: Double): Double = 2 * pi * radius
```

`val`用于定义变量，`def`用于定义函数，虽然形式不一样，但它们的本质是相同的；它们都是使用一个简单的名字去引用一个组合运算的结果。

但是，`val`与`def`还是有区别的，这主要体现在求值的方式上。

### 组合式的求值

一个组合表达式的求值是一个递归的过程。

- 首先对组合式的子表达式进行求值；
- 再将其作为复合表达式的实际参数进行替换，从而实现复合表达式的值。



## 变量定义

可以使用`var`和`val`定义变量。`var(variable)`表示「可变」的变量，在它的生命周期内可以重新赋值。

```scala
var name: String = "Alice"
name = "Bob"   // OK: reassignment with the same type to var.
```

但是，`val(value)`表示不可变的变量，它一旦被初始化后，就不能再被赋值。

```scala
val name: String = "Alice"
name = "Bob"   // ERROR: reassignment to val
```

`val`类似于`Java`中的`final`变量，等价的`Java`实现为：

```java
final String name = "Alice";
```

### 后置类型

这是一段`C++`代码，其包含两层含义。其一，`ptr`是一个`const`指针，表示`ptr`不能再指向其它对象了；其二，`ptr`指向`std::string`类型的`const`对象，表示所指向的对象不能被修改；其中，`std::string`是一个可变类。

```c++
const std::string* ptr const = new std::string("Alice");
```

如果使用`Scala`，应该如此定义：

```scala
val s: String = "Alice"
```

> `Scala`将变量的「修饰符与类型」相分离，不仅能够改善代码的可读性，而且避免了歧义的发生。

同样地，它也蕴含了两层含义。其一，`val`表示`s`是一个不变的引用变量，`s`初始化后不能再引用其它对象了；其二，`String`本身是一个不可变类，其对象的状态不能被修改。

### 类型推演

定义变量时，编译器可以根据初始值的类型自动推演变量的类型。

```scala
import scala.collection.mutable.ArrayBuffer
val buff: ArrayBuffer[String] = new ArrayBuffer[String]
```

等价于

```scala
val buff = new ArrayBuffer[String]
```

相对于`Java`，`Scala`消除了冗余的语法噪声，程序更加简单明了。其中，`ArrayBuffer`类似于`Java`中的`ArrayList`，是一个可变长度的数组实现。

### 理解val

`val`意味着变量本身是一个不可变的引用变量，但`val`变量可以引用可变的对象。

```scala
val buff = new ArrayBuffer[String]
```

`ArrayBuffer[String]`类似于`Java`中的`ArrayList<String>`，都是一个可变长度的数组实现。只不过`ArrayBuffer`提供了更为丰富，更为人性化的操作方法。

```scala
buff += "Alice"  // 尾部追加元素
buff(0) = "Bob"  // 修改第0个元素的值
```

> 尽量使用`val`，而不是`var`。
> 
> 事实上，`var`的存在代表了指令式的设计思想。`Scala`倡导函数式的代码风格，但在某些场景下，`var`具有至关重要的作用。
> - 改善性能；
> - 相对于函数式，指令式的设计有时会更加简单、易懂；

## 方法定义

```scala
def min(a: Int, b: Int): Int = { 
  if (a < b) a else b
}
```

一个方法定义以`def`开头，包括参数列表，返回值类型，函数体。其中，函数原型与函数体之间使用`=`相隔。

### 面向表达式

`Scala`是一个面向表达式的程序设计语言，程序中每一个元素都是有值的，包括函数，`if-else, throw`语句等。

`if (a > b) a else b`的值要么为`a`，要么为`b`，它等价于`C`语言中的三元表达式`?:`；但是，由于功能重复，`Scala`并未提供`?:`的语法。

### 函数值

在`Scala`中，函数也是有值的，函数调用类似于表达式求值的过程。特殊地，函数体只有一条语句时，常常省略函数体的大括号。

```scala
def max(a: Int, b: Int): Int = if (a > b) a else b
```

类似于变量可以推演类型，函数的返回值也能推演类型。函数原型与函数体之间的`=`更形象地体现了这个事实。

```scala
def max(a: Int, b: Int) = if (a > b) a else b
```

其中，`max`函数原型后面的`=`不能略去，否则会限制函数的类型推演的能力，编译器将一律推演函数返回值类型为`Unit`(等价于`Java`中的`void`)。

> 不要省略函数返回值的类型声明，一则可以提高代码的可读性，二则可以避免错误的发生。

另外，函数值为函数体中最后一个表达式的值，而且常常略去`return`关键字。相反，`return`需要显式地声明函数返回值的类型。

```scala
// 错误, 需要显式地声明函数返回值的类型
def max(a: Int, b: Int) = return if (a > b) a else b 
```

## 分号推断

`Scala`程序的语句末尾的分号是可选的。一般地，编译器能够自动地推断出语句的终止条件，因此语句末尾的分号常常被省略。

例如，不用为`toScalaSyntax`的每个`case`都显式地加上分号。

```scala
def toScalaSyntax(ch: Char): String =
  ch match {
    case '"'  => "\\\""
    case '\n' => "\\n"
    case '\r' => "\\r"
    case '\t' => "\\t"
    case '\\' => "\\\\"
    case _    => ch
  }
```

### 合并行

如果一行中包含了多条语句，它们之间的分号则是必须的。

```scala
import java.lang.Math.{abs, sqrt}

def distance(p1: Point, p2: Point): Double = { 
  val dx = abs(p1.x - p2.x); val dy = abs(p1.y - p2.y)  // 合并语句，不能略去分号
  sqrt(dx*dx + dy*dy)
}
```

### 最佳实践

为了保证编译器能够合理的断行，正确地识别语句的终止条件，需要遵循一致的代码风格和习惯用法。例如，`Scala`社区习惯于`K&R`的代码风格，代码块的大括号起始于语句末尾；同样地，操作符也常常放置在语句末尾，表示该语句还未结束。

```scala
def msg(header: String, body: String): String =
  "header: " + 
  header     + 
  "\nbody: " + 
  body
```

事实上，当下一行语句开始于一个不能作为语句开始的符号时，编译器默认地认为上一语句还未结束。例如，「链式函数调用」常常习惯于以`.`开头。

```scala
sc.textFile("hdfs://...")
  .flatMap(_ split " ")
  .map(_ -> 1)
  .reduceByKey(_ + _)
```

> 在`REPL`环境下，`Scala`解释器更易将每行视为单独的表达式；如果断行存在歧义，请使用`:paste`模式，当输入完毕时，使用`Ctrl+D`退出`:paste`模式。

## 类定义

先使用`Java`构造一个`Point`的类，并计算两点之间的距离。

```java
import static java.lang.Math.*;

public class Point {
  private double x;
  private double y;
  
  public Point(int x, int y) {
    this.x = x;
    this.y = y;
  }
  
  public double x() {
    return x;
  }
  
  public double y() {
    return y;
  }
  
  public double distance(Point other) {
    double dx = abs(x - other.x)
    double dy = abs(y - other.y)
    sqrt(dx*dx + dy*dy)
  }
}
```

对于值对象而言，构造函数，字段，存取方法，甚至包括`equals, hashCode`方法，存在特定的规律；但是，程序员不得不承受重复的样板代码。

> 如果精通诸如`Idea`等集成开发环境，这些样板代码可以使用快捷键快速实现。`Scala`从本质上消除了重复设计，减少用户的关注点，降低用户的负担。

### 主构造器

使用`Scala`，可以消除大量的「样板代码」。`Scala`使用「构造参数」构造类的字段，整个类体也成为了「主构造器」的函数体；其中使用`val`将自动提供`Getter`接口，，

```scala
import java.lang.Math._

class Point(val x: Double, val y: Double) {
  def distance(other: Point): Double = { 
    val dx = abs(x - other.x)
    val dy = abs(y - other.y)
    sqrt(dx*dx + dy*dy)
  }
}
```

### 伴生对象

如果要定义原点这个常量，可以放在`Point`的伴生对象里。

```scala
object Point {
  val ORIGIN = new Point(0, 0)
}
```

`Scala`没有`static`的概念，使用「单键对象」取而代之。如果单键对象在编译单元内存在同名的类，该对象称为该类的「伴生对象」。例如，`object Point`是`class Point`的伴生对象。

另外，当客户需要一个`Point`实例时，需要显式地使用`new`进行构造。事实上，可以提供`apply`的工厂方法，略去`new`关键字。

```scala
object Point {
  def apply(x: Double, y: Double): Point = new Point(x, y)
  ...
}
```

当客户构造`Point`对象时，可以省略`new`关键字。

```scala
object Point {
  val ORIGIN = Point(0, 0)
  ...
}
```

### case类

对于`Point`的值对象，可以使用`case class`简化实现；`case class`将默认地提供`Getter`方法，及其伴生对象，其中，伴生对象默认地提供`apply`的工厂方法。

```scala
case class Point(x: Double, y: Double) {
  def distance(other: Point): Double = { 
    val dx = abs(x - other.x)
    val dy = abs(y - other.y)
    sqrt(dx*dx + dy*dy)
  }
}

object Point {
  val ORIGIN = Point(0, 0)
}
```


## Scala数据类型

本章讲述`Scala`最常见的数据类型，包括`Boolean`，整型，浮点型；字符串，符号，数组，元组；及其常见的集合类型。

## 实作Boolean

为了揭秘`scala.Boolean`的工作机制，使用`Bool`的实现进行模拟。`Bool`是一个递归的数据结构，对于任意的一个`Bool`实例`pred`，`pred.eval(t, e)`等价于`if(pred.toBoolean) t else e`。

```scala
sealed abstract class Bool {
  def eval[T](t: => T, e: => T): T

  def &&(x: => Bool): Bool = eval(x, False)
  def ||(x: => Bool): Bool = eval(True, x)
  def unary_! : Bool = eval(False, True)
  
  def ==(x: Bool): Bool = eval(x, !x)
  def !=(x: Bool): Bool = eval(!x, x)
}

object True extends Bool {
  def eval[T](t: => T, e: => T): T = t
  override def toString: String = "True"
}

object False extends Bool {
  def eval[T](t: => T, e: => T): T = e
  override def toString: String = "False"
}
```

```scala
object Bool {
  implicit  def toBool(p: Boolean): Bool = if(p) True else False
}
```

## 探秘整型

### 区分int, Int, Integer

### 探秘Int

`Scala`是一门纯的面向对象的程序设计语言，它没有像`Java`一样，单独地定义原生的数据类型，例如`int, short, long, char`等。

```scala
1 + 2
```

等价于

```scala
1.+(2)
```

事实上，`+`定义在`Int`的类里。

```scala
final abstract class Int private extends AnyVal {
  def +(x: Int): Int = ???
  ...
}
```

### 探秘RichInt

求取`1`至`10`的和，可以如此实现。

```scala
(1 to 10).sum
```

它等价于：

```scala
1.to(10).sum
```

但是，`Int`中并没有定义`to`方法，`1 to 10`为什么能够工作呢？事实上，在`Predef`中定义了`Int`到`RichInt`的隐式转换，而且`Predef`被编译器默认地导入，对所有程序可见。

```scala
package scala

object Predef {
  implicit def intWrapper(x: Int) = new scala.runtime.RichInt(x)
  ...
}
```

而`RichInt`中刚好定义了一个`to`方法，它创建了一个`Range.Inclusive`类型的实例。

```scala
package scala.runtime

class RichInt {
  def to(end: Int): Range.Inclusive = Range.inclusive(self, end)
  ...
}
```

## 浮点数



## 字符串

### 字符串内插

### 多行字符串

## 元组

## 符号

## 数组

`Scala`并没有将数组内置于语言内核，它只不过是一个具有类型参数的类而已；也就是说，`Array[String](10)`类似于原生的`Java`字符串数组`String[10]`。

其中，`Scala`使用中括号定义「类型参数」，而不是使用尖括号。

```scala
final class Array[T](length: Int) {
  def apply(i: Int): T = ???
  def update(i: Int, x: T): Unit = ???
}
```

# Scala控制结构

本章讲述`Scala`基本的控制结构，包括条件分支，循环；并简单介绍模式匹配，`for`推导式的基本用法；最后讲述自定义控制结构的实现技术。

## 条件分支

在以前的章节讲过，`if-else`是一个表达式，它类似于「三元表达式」的功能。它拥有一个值，要么是`if`分支的值，要么是`else`分支的值。

例如，`abs`的返回值类型可以根据`if-else`表达式的类型自动推演为`Int`。

```scala
def abs(n: Int) = if (n > 0) n else -n
```

### 最近公共父类

两个分支的类型可以不同，将自动推演为两者之间最近的父类型。例如，`abs`的返回值类型被推演为`None`与`Some`的最近公共父类`Option[Int]`。

```scala
def abs(n: Int) = if (n > 0) Some(n) else None
```

### 单分支的值

对于单分支`if`，编译器默认地引入`else ()`分支。

```scala
def abs(n: Int) = if (n > 0) n
```

等价于

```scala
def abs(n: Int) = if (n > 0) n else ()
```

其中，`()`的类型为`Unit`类型，而`Unit`和`Int`最近公共类型为`AnyVal`，所以此时`abs`的返回值类型推演为`AnyVal`。

### else抛出异常

当`else`分支抛出异常时，函数返回值类型推演为`if`分支的值类型。这是因为`throw`表达式的值类型为`Nothing`，它是所有类的子类。

对于本例，`abs`的返回值类型推演为`Int`。

```scala
def abs(n: Int) = 
  if (n > 0) n else throw new IllegalArgumentException("negative")
```

### 函数式风格

当需要从某种结构中获取值之前，先进行状态查询。例如，调用`Iterator`的`next`之前，先调用`hasNext`查阅当前迭代器是否已经到了尾部；调用`Option`的`get`方法之前，先调用`isEmpty`查阅当前状态是否为空。

下面这个例子，`head`函数要从`Seq`中取得头部第`0`个元素；其中，当`Seq`为空时，返回空字符串。

```scala
def head(seq: Seq[String]): String = {
  val headOption = seq.headOption
  if (!headOption.isEmpty) headOption.get else ""
}
```

其中，`head`的实现可以使用更加函数式的方式进行表达，不仅可以提高表达力，而且可以消除客户端的重复代码。

```scala
def head(names: Seq[String]): String = 
  names.headOption.getOrElse("")
```

其中，`getOrElse`被封装在`Option`的内部了。

```scala
sealed abstract class Option[+A] {
  def isEmpty: Boolean
  def get: A

  def getOrElse[B >: A](default: => B): B =
    if (isEmpty) default else get
  ...
}
```

## 模式匹配

当存在多路分支，可以使用多路`case`进行匹配操作。

```scala
def toScalaSyntax(ch: Char): String =
  ch match {
    case '"'  => "\\\""
    case '\n' => "\\n"
    case '\r' => "\\r"
    case '\t' => "\\t"
    case '\\' => "\\\\"
    case _    => ch
  }
```

它并不是像`switch-case`那么简单。首先，它也是一个表达式，它的值等于第一个匹配成功的`case`表达式。其次，它具有丰富的「模式匹配」的功能。

例如，在不关心效率的问题，可以使用「模式匹配」和线性的「递归」优雅地实现快速排序算法。

```scala
def quicksort[T](list: List[T]): List[T] = 
  list match {
    case Nil => Nil
    case x :: xs =>
      val (before, after) = xs.partition(_ < x)
      quicksort(before) ++ (x :: quicksort(after))
  }
```

模式匹配是`Scala`的重要特性之一，我们将在后续章节重点介绍。

## 循环

`Scala`的`while/do-while`与`C`的行为一致，唯一的区别在于，它们也是表达式。例如，`factorial`用于求解`n(n <= 16)`的阶乘，使用`while`迭代的实现方式如下。

```scala
def factorial(n: Int): Long = {
  var r = 1; var m = n
  while (m > 0) {
    r = r * m
    m -= 1
  }
  r
}
```

## for推导式

可以使用`for`推导式改善程序的表达力。

```scala
def factorial(n: Int): Long = {
  var r = 1
  for(i < 1 to n) r = r * i
  r
}
```

`i < 1 to n`称为`for`推导式的「生成器」。`for`推导式是`Scala`的重要特性之一，我们将在后续章节重点介绍。

### 线性递归

也可以使用模式匹配实现`factorial`的递归算法。

```scala
def factorial(n: Int): Long = n match {
  case 0 => 1
  case _ => n * factorial(n - 1)
}
```

但是，该实现不具有尾递归的优化效果。

### 尾递归

为了得到优化的尾递归实现，可以使用「局部方法」设计迭代的结构来实现，并显式地加上`@tailrec`的注解，用于编译时的检查。

```scala
import scala.annotation.tailrec

def factorial(n: Int): Long = {
  @tailrec
  def fact(r: Int, n: Int): Long = n match {
    case 0 => r
    case _ => fact(r * n, n - 1)
  }
  fact(1, n)
}
```

### 函数式风格

> 提倡函数式的编程风格。

可以使用`foldLeft`实现`factorial`，这种函数式的风格简单优雅，并具有高度的代码复用的价值。

```scala
def factorial(n: Int): Long = 
  (1 to n).foldLeft(1) { _ * _ }
```

## 表达式与语句

在`Scala`里，没有语句这样的概念，一切皆为表达式。`if-else`称为分支表达式，而不是分支语句；`while/do-while`也被称为循环表达式，而不是循环语句；依次类推，模式匹配，函数，`for`推导式都是表达式。

## 自定义控制结构

`Scala`是一门具有高度可扩展性的程序设计语言，它可以设计出自定义的控制结构，以便提升代码复用价值，并改善代码的表达力。

本文通过`2`个例子，阐述`Scala`在设计抽象的控制结构的相关技术。

### 探秘forall

`Scala`是一门函数式的程序设计语言，所以它的语言内核拒绝指令式的`break, continue`。但借助于`Scala`强大的可扩展能力，可以模拟实现`break, continue`指令的实现。

`Traversable`是一个可以遍历的，元素类型为`A`的集合。它定义了一个`foreach`的内部迭代器，其它方法的实现都是基于`foreach`的抽象，使用`for`推导式实现算法逻辑的。

例如，`forall`用于判断集合元素是否全部满足于某个谓词，否则返回`false`。算法无需遍历整个集合，实现使用了`break`提前终止迭代过程。

```scala
import scala.util.control.Breaks._

trait Traversable[+A] {
  def foreach[U](f: A => U): Unit
  
  def forall(p: A => Boolean): Boolean = {
    var result = true
    breakable {
      for (x <- this if !p(x)) { 
        result = false; break 
      }
    }
    result
  }
}
```

### 探秘`breakable/break`

事实上，`break`的工作机制是通过异常来实现的。`breakable`定义了一个执行`op: => Unit`的上下文环境，`break`通过抛出异常终止当前执行的`op`操作。

```scala
class Breaks {
  private val breakException = new BreakControl 

  def break(): Nothing = throw breakException

  def breakable(op: => Unit): Unit = 
    try { 
      op 
    } catch {
      case ex: BreakControl =>
        if (ex ne breakException) throw ex
    }
```

### 模拟break/continue

`break`只是终止了`breakable`的上下文。如果要模拟指令式的`break`，可以通过将循环放在`breakable`之内实现。

```scala
breakable {  for (x <- xs) {    if (cond) break  } 
}
```

如果要模拟指令式的`continue`，可以将循环放在`breakable`之外实现，`break`时只是终止了当前的迭代，而不是整个迭代过程。

```scala
for (x <- xs) { 
  breakable { 
    if (cond) break  }
}
```

### 一个更具体的例子

再以一个简单的例子，通过若干迭代，遵循正交设计的基本原则，灵活地应用重构，逐渐改进设计，实现控制结构的抽象。

> 需求1：搜索当前目录下扩展名为`.scala`的所有文件

#### 快速实现

```scala
object FileMatchers {
  def ends(file: File, ext: String) =
    for (file <- file.listFiles if file.getName.endsWith(ext))
      yield filelist
}
```

放在`for`推导式中的`if`语句称为「守卫」，用于在迭代过程中设置过滤条件。例如上例判断文件名以`ext`结尾。

`yield`是`for`推导式的「产生式」，用于生成新的集合，并成为`for`推导式的值。

> 需求2：搜索当前目录下名字包含`Test`的所有文件，及其搜索目录下名字正则匹配特定模式的所有文件。

#### 重复设计

`ends，contains, matches`三者之间的逻辑存在明显的重复，它们仅过滤条件不一样。

```scala
object FileMatcher {
  def ends(file: File, ext: String) =
    for (file <- file.listFiles if file.getName.endsWith(ext))
      yield file
  
  def contains(file: File, query: String) =
    for (file <- file.listFiles if file.getName.contains(query))
      yield file
  
  def matches(file: File, regex: String) =
    for (file <- file.listFiles if file.getName.matches(regex))
      yield file
}
```

#### 提取抽象

为了消除上述实现的重复逻辑，可以提取一个抽象: `Matcher: (String, String) => Boolean`。

```scala
object FileMatcher {
  private def list(file: File, query: String, 
  matcher: (String, String) => Boolean) =
    for (file <- file.listFiles if matcher(file.getName, query))
      yield file

  def ends(file: File, ext: String) =
    list(file, ext, (fileName, ext) => fileName.endsWith(ext))
    
  def contains(file: File, query: String) =
    list(file, query, (fileName, query) => fileName.contains(query))
          
  def matches(file: File, regex: String) =
    list(file, regex, (fileName, regex) => fileName.matches(regex))
}
```

#### 类型推演

借助于`Scala`强大的类型推演能力，可以得到更为简洁的函数字面值。

```scala
object FileMatcher {
  private def list(file: File, query: String, 
    matcher: (String, String) => Boolean) = {
    for (file <- file.listFiles if matcher(file.getName, query))
      yield file
  }

  def ends(file: File, ext: String) =
    list(file, ext, _.endsWith(_))
    
  def contains(file: File, query: String) =
    list(file, query, _.contains(_))
          
  def matches(file: File, regex: String) =
    list(file, regex, _.matches(_))
}
```

#### 类型别名

`list`的参数由于类型修饰，显得有点过长而影响阅读；可以通过「类型别名」的机制缩短函数的类型修饰符，以便改善表达力。

```scala
object FileMatcher {
  private type Matcher = (String, String) => Boolean

  private def list(file: File, query: String, matcher: Matcher) =
    for (file <- file.listFiles if matcher(file.getName, query))
      yield file
      
  def ends(file: File, ext: String) =
    list(file, ext, _.endsWith(_))
    
  def contains(file: File, query: String) =
    list(file, query, _.contains(_))
          
  def matches(file: File, regex: String) =
    list(file, regex, _.matches(_))
}
```

#### 简化参数

简化参数传递，消除不必要的冗余。

```scala
object FileMatcher {
  private type Matcher = String => Boolean

  private def list(file: File, matcher: Matcher) =
    for (file <- file.listFiles if matcher(file.getName))
      yield file

  def ends(file: File, ext: String) =
    list(file, _.endsWith(ext))
    
  def contains(file: File, query: String) =
    list(file, _.contains(query))
          
  def matches(file: File, regex: String) =
    list(file, _.matches(regex))
}
```

#### 替换`for comprehension`

可以通过定制「高阶函数」替代语法较为复杂的`for`推导式，改善代码的表达力。

```scala
object FileMatcher {
  private type Matcher = String => Boolean

  private def list(file: File, matcher: Matcher) =
    file.listFiles.filter(f => matcher(f.getName))

  def ends(file: File, ext: String) =
    list(file, _.endsWith(ext))
    
  def contains(file: File, query: String) =
    list(file, _.contains(query))
          
  def matches(file: File, regex: String) =
    list(file, _.matches(regex))
}
```

#### 柯里化

应用「柯里化」，漂亮的「大括号」终于登上了舞台。

```scala
object FileMatcher {
  private type Matcher = String => Boolean

  def list(file: File)(matcher: Matcher) =
    file.listFiles.filter(f => matcher(f.getName))

  def ends(file: File, ext: String) =
    list(file) { _.endsWith(ext) }

  def contains(file: File, query: String) =
    list(file) { _.contains(query) }

  def matches(file: File, regex: String) =
    list(file) { _.matches(regex) }
}
```

#### 更自然

对于中缀表达式，并且只有一个参数时，可以省略函数调用的点号和小括号，使得程序更加自然。

```scala
object FileMatcher {
  private type Matcher = String => Boolean

  def list(file: File)(matcher: Matcher) =
    file.listFiles.filter(f => matcher(f.getName))

  def ends(file: File, ext: String) =
    list(file) { _ endsWith ext }

  def contains(file: File, query: String) =
    list(file) { _ contains query }

  def matches(file: File, regex: String) =
    list(file) { _ matches regex }
}
```


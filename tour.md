# 破冰之旅

> 即使水墨丹青，何以绘出半妆佳人。

`Scala`是一门优雅而又复杂的程序设计语言，初学者很容易陷入细节而迷失方向。这也给我的写作带来了挑战，如果从基本的控制结构，再深入地介绍高级的语法结构，难免让人生厌。

为此，本文另辟蹊径，尝试通过一个简单有趣的例子，概括性地介绍`Scala`常见的语言特性。它犹如一个迷你版的`Scala`教程，带领大家一起领略`Scala`的风采。

### 问题的提出

有一名体育老师，在某次离下课还有五分钟时，决定玩一个游戏。此时有`100`名学生在上课，游戏的规则如下：

1. 老师先说出三个不同的特殊数(都是个位数)，比如`3, 5, 7`；让所有学生拍成一队，然后按顺序报数；

2. 学生报数时，如果所报数字是「第一个特殊数(`3`)」的倍数，那么不能说该数字，而要说`Fizz`；如果所报数字是「第二个特殊数(`5`)」的倍数，要说`Buzz`；如果所报数字是「第三个特殊数(`7`)」的倍数，要说`Whizz`。

3. 学生报数时，如果所报数字同时是「两个特殊数」的倍数，也要特殊处理。例如，如果是「第一个(`3`)」和「第二个(`5`)」特殊数的倍数，那么也不能说该数字，而是要说`FizzBuzz`。以此类推，如果同时是三个特殊数的倍数，那么要说`FizzBuzzWhizz`。

4. 学生报数时，如果所报数字包含了「第一个特殊数」，那么也不能说该数字，而是要说`Fizz`。例如，要报`13`的同学应该说`Fizz`。

5. 如果数字中包含了「第一个特殊数」，需要忽略规则`2`和`3`，而使用规则`4`。例如要报`35`，它既包含`3`，同时也是`5`和`7`的倍数，要说`Fizz`，而不能说`BuzzWhizz`；

6. 否则，直接说出要报的数字。

##### 形式化

以`3, 5, 7`为例，该问题可形式化地描述为：

```scala
r1: times(3) => Fizz || 
    times(5) => Buzz ||
    times(7) => Whizz

r2: times(3) && times(5) && times(7) => FizzBuzzWhizz ||
    times(3) && times(5) => FizzBuzz  ||
    times(3) && times(7) => FizzWhizz ||
    times(5) && times(7) => BuzzWhizz

r3: contains(3) => Fizz

rd: others => string of others

spec: r3 || r2 || r1 || rd
```

其中，`times(3) => Fizz`表示：当要报的数字是`3`的倍数时，则说`Fizz`，其他以此类推。

### 建立测试环境

首先搭建测试环境，建立反馈系统。这里使用`scalatest`的测试框架，它也是作者偏爱的测试框架之一。

```scala
package scalaspec.fizzbuzz

import org.scalatest.Matchers._
import org.scalatest._

class RuleSpec extends FunSpec {
  describe("World") {
    it ("should not be work" ) {
      true should be(false)
    }
  }
}
```

运行测试用例，与预期相符，测试失败；证明测试环境可工作，删除该用例，然后开启新的旅程。

### 第一个测试用例

先建立了一个规则：`new Times(3, "Fizz")`；它表示如果是`3`的倍数，则输出`Fizz`。此时，如果输入数字`3*2`，它恰好是`3`的倍数，断言预期的结果为`Fizz`。

```scala
it ("times(3) -> fizz" ) {
  new Times(3, "Fizz").apply(3 * 2) should be("Fizz")
}
```

##### 如果使用Java

为了快速通过测试，可以做简单实现。如果使用`Java`，`Times`可以如下实现。

```java
public class Times {
  private final int n;
  private final String word;

  public Times(int n, String word) {
    this.n = n;
    this.word = word;
  }

  public apply(int m) {
    return word;
  }
}
```

##### 构造参数

从上述`Java`实现可以看出，当定义一个私有字段时，需要在构造函数对它进行初始化。类似重复的「样板代码」在`Scala`中，可以在「主构造函数」中使用「构造参数」定义字段，彻底消除重复设计。

```scala
class Times(n: Int, word: String) {
  def apply(m: Int): String = word
}
```

##### 类型的后缀修饰

`Scala`将类型的修饰放在后面，以便实现风格的「一致性」，包括：

- 变量的类型修饰
- 函数返回值的类型修饰

```scala
def apply(m: Int): String = word
```

##### 类型推演

定义变量时，可以通过初始化值的类型推演出变量类型。

```scala
val i = 0  // 等价于 val i: Int = 0
```

事实上，当函数体比较短小时，可以一眼看出函数返回值类型，也可以略去函数返回值的类型。例如`Times.apply`的返回值类型推演为`String`类型。

```scala
def apply(m: Int) = word  // 等价于 def apply(m: Int): String = word
```

##### apply方法

`apply`方法是一个特殊的方法，它可以简化方法调用的表现形式，使其行为更贴近函数的语义。在特殊的场景下，能够改善代码的表达力。

```scala
it ("times(3) -> fizz" ) {
  new Times(3, "Fizz").apply(3 * 2) should be("Fizz")
}
```

等价于：

```scala
it ("times(3) -> fizz" ) {
  new Times(3, "Fizz")(3 * 2) should be("Fizz")
}
```

### 实现Times

至此，测试通过了，但`apply`实现被写死了。因为`Times`的逻辑较为简单，可以快速实现它。

```scala
class Times(n: Int, word: String) {
  def apply(m: Int): String = 
    if (m % n == 0) word else ""
}
```

##### 万物皆是对象

`Scala`并没有像`Java`一样，针对「基本类型」(例如`int`)，「数组类型」(例如`int[]`)定义特殊的语法，它将世间万物都看成对象。

其中，`m % n`等价于`m.%(n)`，而`%`只不过是`Int`的一个普通方法而已。

```scala
package scala

final abstract class Int private extends AnyVal {
  def %(x: Int): Int
  ...
}
```

注意，`Int`的实现由编译器完成，后续章节将讲述`Int`的修饰语法。

##### 面向表达式

`Scala`是一门面向表达式的语言，它所有的程序结构都具有值，包括`if-else`，函数调用等。其中，`if (m % n == 0) word else ""`类似于`Java`中的三元表达式：`m % n == 0 ? word : ""`。

因为两者功能重复，因此`Scala`并没有提供三元表达式的特性。

##### 使用case类

可以将`Times`设计为`case`类。

```scala
case class Times(n: Int, word: String) {
  def apply(m: Int): String =
    if (m % n == 0) word else ""
}
```

当构造一个`Times`实例时，可以使用其「伴生对象」提供的工厂方法，从而略去`new`关键字，简化代码实现。

```scala
it ("times(3) -> fizz" ) {
  Times(3, "Fizz")(3 * 2) should be("Fizz")
}
```

### 实现Contains

有了`Times`实现的基础，可以很轻松地实现`Contains`的测试用例。

```scala
it ("contains(3) -> fizz" ) {
  Contains(3, "Fizz")(13) should be("Fizz")
}
```

依次类推，`Contains`可以快速实现为：

```scala
case class Contains(n: Int, word: String) {
  def apply(m: Int): String =
    if (m.toString.contains(n.toString)) word else ""
}
```

恭喜，测试通过了。

##### 省略括号

`m.toString`等价于`m.toString()`。按照惯例，如果函数没有副作用，则可以略去小括号；相反，如果产生副作用，则显式地加上小括号用于警示。

如果函数定义时就没有使用小括号，用于表达函数无副作用；此时用户不能画蛇添足，添加多余的小括号，否则与函数定义的语义相驳了。

### 实现默认规则

对于默认规则，它只是简单地将输入的数字转变为字符串表示形式。

```scala
it ("default rule" ) {
  Default()(2) should be("2")
}
```

其中，`Default`可以快速实现为：

```scala
case class Default() {
  def apply(m: Int): String = m.toString
}
```

##### 定制伴生对象

上述实现中，`case class Default()`，及其调用点`Default()(2)`，不能略去`()`。这非常讨厌，可以自行定制伴生对象的`apply`工厂方法，改善表达力。

```scala
class Default {
  def apply(m: Int) = m.toString
}

object Default {
  def default = new Default
}
```

此时，便可以删除测试用例中冗余的`()`。

```scala
import Default._

it ("default rule" ) {
  default(2) should be("2")
}
```

值得庆幸的是，`default`并非`Scala`的保留字。

### 提取抽象

至此，发现`Times, Contains, Default`都具有相同的结构，可抽象出`Rule`的概念。

```scala
trait Rule {
  def apply(n: Int): String
}
```

事实上，`trait`不仅仅等价于`Java`中的`interface`，它是`Scala`实现对象组合的重要机制。

##### 实现特质

例如`Times`实现`Rule`特质，它使用`extends Rule`语法混入该「特质」。

```scala
case class Times(n: Int, word: String) extends Rule {
  def apply(m: Int): String =
    if (m % n == 0) word else ""
}
```

以此类推，`Contains, Default`实现方式相同，不再重述。

### 实现AnyOf

接下来，实现具有两个之间具有「逻辑与」关系的复合规则。先建立一个简单的测试用例：

```scala
it ("times(3) && times(5) -> FizzBuzz" ) {
  AllOf(Times(3, "Fizz"), Times(5, "Buzz"))(3*5) should be("FizzBuzz")
}
```

为了快速通过测试，可以先打桩实现。

```scala
case class AllOf(rules: Rule*) extends Rule {
  def apply(n: Int): String = "FizzBuzz"
}
```

##### 变长参数

`rules: Rule*`表示变长的`Rule`列表，表示可以向`AllOf`的构造函数传递任意多的`Rule`实例。

事实上，`rules: Rule*`的真正类型为`scala.collection.mutable.WrappedArray[Rule]`，所以`rules: Rule*`拥有普通集合类的一般特征，例如调用`map, foreach, foldLeft`等方法。

##### 快速实现`AllOf`

```scala
case class AllOf(rules: Rule*) extends Rule {
  def apply(n: Int): String = {
    val result = new StringBuilder
    rules.foreach((r: Rule) => result.append(r.apply(n)))
    result.toString
  }
}
```

其中，`r`的静态类型为`Rule`，而其运行时类型为`Times`，它将在运行时发生「子类化多态」。

##### 高阶函数

一般地，可以传递或返回「函数值」的函数常称为「高阶函数」。例如`foreach`就是一个高阶函数，它通过传递`(r: Rule) => result.append(r.apply(n))`的函数值实现容器的遍历；其中，函数字面值的类型为`Function1[Rule, StringBuilder]`，表示参数为`Rule`，返回值为`StringBuilder`的函数。

对于此例子，如果你偏爱大括号，则可以如此实现。

```scala
rules.foreach { (r: Rule) => 
  result.append(r.apply(n))
}
```

借助类型推演，可以去除`r`的类型修饰。

```scala
rules.foreach { r => result.append(r.apply(n)) }
```

其中，`apply`有特殊的调用语义，因此代码可以更简洁。

```scala
rules.foreach { r => result.append(r(n)) }
```

甚至，可以略去一些冗余的语法符号。

```scala
rules foreach { r => result append r(n) }
```

##### 使用foldLeft

事实上，上述`AllOf.apply`实现可以简化为函数式中常见的「规约」操作。

```scala
case class AllOf(rules: Rule*) extends Rule {
  def apply(n: Int): String = 
    rules.foldLeft("") { r => _ + r.apply(n) }
}
```

因为`r`在函数字面值中仅过一次，可以使用占位符代替。

```scala
case class AllOf(rules: Rule*) extends Rule {
  def apply(n: Int): String = 
    rules.foldLeft("") { _ + _.apply(n) }
}
```

同样地，因为`apply`方法具有特殊的函数调用语义，可以进一步简化实现。

```scala
case class AllOf(rules: Rule*) extends Rule {
  def apply(n: Int): String = 
    rules.foldLeft("") { _ + _(n) }
}
```

##### 剖析`foldLeft`

`foldLeft`实现在`TraversableOnce`中。

```scala
trait TraversableOnce[+A] {
  ...
  def foreach[U](f: A => U): Unit

  def foldLeft[B](z: B)(op: (B, A) => B): B = {
    var result = z
    foreach(x => result = op(result, x))
    result
  }
}
```

`foldLeft`使用函数式中一个重要的技术：「柯里化」。其中，`z`为迭代的初始值，`op: (B, A) => B`中第一个参数为「收集参数」，通过遍历将容器的元素收集起来。

### 实现AnyOf

接下来，实现具有两个之间具有「逻辑或」关系的复合规则。先建立一个简单的测试用例：

```scala
it ("times(3) -> Fizz || times(5) -> Buzz" ) {
  AnyOf(Times(3, "Fizz"), Times(5, "Buzz"))(3*5) should be("Fizz")
}
```

为了快速通过测试，可以先打桩实现。

```scala
case class AnyOf(rules: Rule*) extends Rule {
  def apply(n: Int): String = "Fizz"
}
```

##### 链式调用

鉴于`AnyOf`的基础，可以快速地实现`AnyOf`。

```scala
case class AnyOf(rules: Rule*) extends Rule {
  def apply(n: Int): String = 
    rules.map(_(n))
         .filterNot(_.isEmpty)
         .headOption
         .getOrElse("")
}
```

此时，测试用例通过了。

##### 隐式树

`AllOf, AnyOf`是一个「复合规则」，而`Times, Contains, Default`表示「原子规则」。它们之间构成了一棵「隐式树」，它们的关键在于抽象的`Rule`特质的提取。

### 提供工厂方法

因为`Times, Contains, Default, AnyOf, AllOf`都具有相同的句法结构，是一种典型的结构性重复设计，可以通过「工厂方法」消除它们之间的重复设计。

另外，为了简单函数调用的方式，可以使用`Int => String`的一元函数代替`Rule`特质。

##### 重构测试用例

此时，可以定义一组新的测试用例集合，并使用`describe`分离用例组，并通过显示地导入所依赖的类型，与既有的用例集共存，互不干扰。

> 切忌删除既有的`Rule`特质，以及`Times, Contains, Default, AllOf, AnyOf`的实现，包括既有的测试用例；否则既有的测试用例失败，重构的安全网被撕破，将会让重构陷入一个极度危险的境界。
> 
> 总之，重构应该保持小步快跑的基本原则。

按照`TDD`的规则，可以小步地，安全地逐一驱动实现各个工厂方法。

```scala
class RuleSpec extends FunSpec {
  ...
  describe("functional style") {
    import Rule._
    it ("times(3) -> fizz" ) {
      times(3, "Fizz")(3 * 2) should be("Fizz")
    }
  }
}
```

##### 实现工厂

`times`的工厂方法也较容易实现，可以通过搬迁`Times`的逻辑至此即可。

```scala
object Rule {
  def times(n: Int, word: String): Int => String =
    m => if (m % n == 0) word else ""
}
```

至此，`times`实现通过测试。

##### 小步快跑

以此类推，通过小步地`TDD`的微循环，将其他工厂方法驱动实现出来。

```scala
class RuleSpec extends FunSpec {
  ...

  describe("Rule using factory method") {
    import Rule._

    it ("times(3) -> fizz" ) {
      times(3, "Fizz")(3 * 2) should be("Fizz")
    }

    it ("contains(3) -> fizz" ) {
      contains(3, "Fizz")(13) should be("Fizz")
    }

    it ("default rule" ) {
      default(2) should be("2")
    }

    it ("times(3) && times(5) -> FizzBuzz" ) {
      anyof(times(3, "Fizz"), times(5, "Buzz"))(3*5) should be("FizzBuzz")
    }

    it ("times(3) -> Fizz || times(5) -> Buzz" ) {
      anyof(times(3, "Fizz"), times(5, "Buzz"))(3*5) should be("Fizz")
    }
  }
}
```

最终，在`Rule`伴生对象中实现了所有方法。

```scala
object Rule {
  def times(n: Int, word: String): Int => String =
    m => if (m % n == 0) word else ""
    
  def contains(n: Int, word: String): Int => String = 
    m => if (m.toString.contains(n.toString)) word else ""
    
  def default: Int => String =
    m => m.toString
  
  def anyof(rules: (Int => String)*): Int => String = 
    m => rules.foldLeft("") { _ + _(m) }
    
  def allof(rules: (Int => String)*): Int => String = 
    m => rules.map(_(m))
      .filterNot(_.isEmpty)
      .headOption
      .getOrElse("")
}
```

恭喜，通过所有测试。此时可以安全地删除`Times, Contains, Default, AnyOf, AllOf`, `trait Rule`，以及相关遗留的测试用例了。

##### 类型别名

可以对`Int => String`定义「类型别名」，消除类型的重复定义。

```scala
object Rule {
  type Rule = Int => String

  def times(n: Int, word: String): Rule =
    m => if (m % n == 0) word else ""

  def contains(n: Int, word: String): Rule =
    m => if (m.toString.contains(n.toString)) word else ""

  def default: Rule =
    m => m.toString

  def anyof(rules: Rule*): Rule =
    m => rules.foldLeft("") { _ + _(m) }

  def allof(rules: Rule*): Rule =
    m => rules.map(_(m))
      .filterNot(_.isEmpty)
      .headOption
      .getOrElse("")
}
```

至此，设计已经较为干净了。如果将`default`稍微进行改造，很容易发现`times, contains, default`之间存在微妙的重复结构。

```scala
def times(n: Int, word: String): Rule =
  m => if (m % n == 0) word else ""

def contains(n: Int, word: String): Rule =
  m => if (m.toString.contains(n.toString)) word else ""

def default: Rule =
  m => if (true) m.toString else ""
```

它们各自拥有隐晦的「匹配规则」，当匹配成功时，执行相应的「转换规则」；其中，`default`的「匹配规则」比较特殊，因为它总是匹配成功。

因此，三者实现可归结为一种统一的抽象行为:

```scala
n => if (matcher) action(n) else ""
```

### 匹配器：`Matcher`

先提取抽象的「匹配器」概念：`Matcher`。事实上，`Matcher`是一个「一元函数」，入参为`Int`，返回值为`Boolean`，是一种典型的「谓词」。

> 从`OO`的角度看，`always`是一个典型的`Null Object`实现模式。

```scala
object Matcher {
  type Matcher = Int => Boolean

  def times(n: Int): Matcher = _ % n == 0
  def contains(n: Int): Matcher = _.toString.contains(n.toString)
  def always(bool: Boolean): Matcher = _ => bool
}
```

### 执行器：`Action`

然后再提取抽象的「执行器」概念：`Action`。事实上，`Action`也是一个「一元函数」，入参为`Int`，返回值为`String`。其本质类似于`map`操作，将定义域映射到值域。

```scala
object Action {
  type Action = Int => String

  def to(str: String): Action = _ => str
  def nop: Action = _.toString
}
```

### 原子操作

接下来开始抽取`times, contains, default`三者之间的重复逻辑。

##### 尝试构建

此时，新建一组用例集合，使用`describe`隔离新老用例集，显式地`import`所依赖的类型，保证既有测试用例可用。然后按照`TDD`微循环驱动实现三者之间共同的本质操作：`atom`。

> 在`Rule.atom, Matcher, Action`可运行之前，切忌删除`Rule`中既有的`times, contains, default`，及其相应的测试用例。  

```scala
class RuleSpec extends FunSpec {
  ...
  describe("using atom rule") {
    import Rule.atom
    import Matcher._
    import Action._

    it ("times(3) -> fizz" ) {
      atom(times(3), to("Fizz"))(3 * 2) should be("Fizz")
    }
  }
}
```

以此类推，可以使用`atom`构造其他测试场景，在此不再重述。

##### 实现atom

至此，可以提取`times, contains, default`三者之间的共公抽象：`atom`。它表示的语义为：给定一个整数`m`，如果与`Matcher`匹配成功，则执行`Action`转换；否则返回空字符串。

```scala
def atom(matcher: => Matcher, action: => Action): Rule =
  m => if (matcher(m)) action(m) else ""
```

测试通过。此时可以安全地删除`Rule`中`times, contains, default`的实现，只保留`atom`一个即可；然后删除遗留的测试用例集。

##### 按名传递

`matcher: => Matcher, action: => Action`是按照`by-name`传递参数的，在实参传递形参过程中，并未对实参进行立即求值，而将求值推延至形参调用点。

也就是说，求值推延至`if (matcher(m)) action(m)`语句才展开调用的。

### 规则库

> Composition Everywhere

至此，`Rule`最终实现如下：

```scala
object Rule {
  type Rule = Int => String

  import Matcher.Matcher
  import Action.Action

  def atom(matcher: => Matcher, action: => Action): Rule =
    n => if (matcher(n)) action(n) else ""

  def anyof(rules: Rule*): Rule =
    n => rules.map(_(n))
      .filterNot(_.isEmpty)
      .headOption
      .getOrElse("")

  def allof(rules: Rule*): Rule =
    n => rules.foldLeft("") { _ + _(n) }
}
```

`Rule`是`FizzBuzzWhizz`最核心的抽象，也是设计的灵魂所在。从语义上`Rule`分为两种基本类型，并且它们之间形成了隐式的「树型」结构，体现了「组合式设计」的强大威力。

- 原子规则：`atom`
- 复合规则: `anyof, anyof`

### 实现门面

例如，给定`100`学生，如果输入的三个特殊数为`3, 5, 7`，如下调用可得到最终的结果序列。

```scala
start(100)(3, 5, 7)
```

从某种意义讲，`spec`使用其内部基本组件构建了该领域的一个`DSL`实现，它具有很强的表现力。

```scala
import Rule._
import Matcher._
import Action._

object Game {  
  def spec(n1: Int, n2: Int, n3: Int): Rule = {
    val r1_1 = atom(times(n1), to("Fizz"))
    val r1_2 = atom(times(n2), to("Buzz"))
    val r1_3 = atom(times(n3), to("Whizz"))

    val r1 = anyof(r1_1, r1_2, r1_3)

    val r2 = anyof(
      allof(r1_1, r1_2, r1_3),
      allof(r1_1, r1_2),
      allof(r1_1, r1_3),
      allof(r1_2, r1_3))

    val r3 = atom(contains(n1), to("Fizz"))
    val rd = atom(always(true), nop);

    anyof(r3, r2, r1, rd)
  }
  
  def start(n: Int)(n1: Int, n2: Int, n3: Int): Seq[String] = {
    val saying = spec(n1, n2, n3)
    (1, n) map saying
  }
}
```

##### 消除重复

事实上，可以将`Game.spec`中的`r1, r2`进行合并。首先，将`r1`做一次等价变换：

```scala
val r1 = anyof(allof(r_n1), allof(r_n2), allof(r_n3))
```

因为`r2`的优先级高于`r1`，因此可以将`r1`合并至`r2`的尾部。

```scala
val r2 = anyof(
  allof(r_n1, r_n2, r_n3),
  allof(r_n1, r_n2),
  allof(r_n1, r_n3),
  allof(r_n2, r_n3),
  allof(r_n1), 
  allof(r_n2), 
  allof(r_n3))
```

实际上，`r2`的本质就是枚举集合`{r_n1, r_n2, r_n3}`的所有「真子集」，然后将每个子集`{R1, R2, ..., Rn}`变换为`allof(R1, R2, ..., Rn)`。

因此，可以提取一个抽象操作：`comb(rules: Rule*)`，用于自动地枚举出`rules`的所有「真子集」。

```scala
val r2 = comb(r_n1, r_n2, r_n3)
```

##### 本地方法

当一个私有函数比较短小时，可以实现为「本地方法」，这样即可保证函数的职责单一，并取得良好的封装性，同时还可以避免函数间的参数传递。

```scala
def comb(rules: Rule*) = {
  def subsets: Seq[Rule] =
    rules.toSet.subsets.toList.reverse.map { subset =>
       allof(subset.toList: _*)
    }
  anyof(subsets: _*)
}
```

其中，`Set.subsets`求取集合的所有子集，然后调用`toList`转换为`List`，并通过`reverse`取得其「逆序」。

然后针对于每个子集合`subset: {R1, R2, ..., Rn}`，通过`map`变换为`allof(R1, R2, ..., Rn)`。

### 完备用例集

而对于测试用例，以`3, 5, 7`为例，可以对测试用例进行整理，形成完备的用例集。此处使用「数据驱动」的方式组织用例，消除用例的重复代码，并改善表达力。

```scala
import org.scalatest._
import prop._

class RuleSpec extends PropSpec with TableDrivenPropertyChecks with Matchers {
  val specs = Table(
    ("n",         "expect"),
    (3,           "Fizz"),
    (5,           "Buzz"),
    (7,           "Whizz"),
    (3 * 5,       "FizzBuzz"),
    (3 * 7,       "FizzWhizz"),
    ((5 * 7) * 2, "BuzzWhizz"),
    (3 * 5 * 7,   "FizzBuzzWhizz"),
    (13,          "Fizz"),
    (35/*5*7*/,   "Fizz"),
    (2,           "2")
  )

  property("fizz buzz whizz") {
    val spec = Game.spec(n1, n2, n3)
    forAll(specs) { spec(_) should be (_) }
  }
}
```

### 语义模型

归纳上述设计，可以得到问题的语义模型。

```scala
Rule:    Int => String
Matcher: Int => Boolean
Action:  Int => String
```

其中，`Rule`存在三种基本的类型：

```scala
Rule: atom | allof | anyof
```

三者之间构成了隐式的「树型结构」。

```scala
atom: (Matcher, Action) => String
allof: rule1 && rule2 ... 
anyof: rule1 || rule2 ... 
```

### 总结

本文通过对`FizzBuzzWhizz`小游戏的设计和实现，首先尝试使用`Scala`的面向对象技术，然后采用函数式的设计；过程采用`TDD`小步快跑，演进式地完成了所有功能。

中间遇到了「特质」，「子类化多态」，「case类」，「类型别名」，「伴生对象」，「变长参数」，「惰性求值」，「高阶函数」，「柯里化」，「本地方法」等常用的技术。经过这个例子的实践，相信大家对`Scala`有了一个大体的印象和感觉，接下来让我们开启`Scala`的星际之旅吧。


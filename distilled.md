# Scala精粹

> 戏路如流水，从始至终，点滴不漏。一路百折千回，本性未变，终归大海。一步一戏，一转身一变脸，扑朔迷离。真心自然流露，举手投足都是风流戏。一旦天幕拉开，地上再无演员。

[Scala](http://www.scala-lang.org/)是由`Martin Ordersky`设计的一门混合「面向对象」和「函数式」，并具备完备的「泛型编程」能力，支持多种范式的程序设计语言。

`Scala`取名为「可扩展的语言」，因为它拥有良好的弹性。它使用不变的语言内核，构建万千变化的世界。同时，`Scala`也是设计「`DSL`(领域描述语言)」的利器，让编程充满轻松，愉快的气氛，并富有成就感。

`Scala`拥有强大的类型系统，具有丰富的表达能力，语法简洁，优雅，灵活。它应用广泛，从编写简单的脚本，到构建大型的企业级系统。

`Scala`运行于`JVM`之上，并与现有的`JVM`生态系统无缝链接，而无需定义额外的胶水代码。它兼容既有`Java`的类库，让成千上万的程序可以继续工作，并能够得以复用。

## Scala的哲学

`Scala`的设计吸收了众多程序设计语言优秀的思想，取精除粕，形成了自身特有的哲学思维体系。

- 自由：释放自由，方能创造奇迹；
- 复用：讨厌重复，重用既有代码；
- 抽象：正交设计，拥抱未来变化；
- 开放：对外扩展开放，对内拒绝修改；
- 友好：专家级的瑞士军刀；

## Scala的基因

`Scala`首先偏向`Java`社区的使用习惯，包括表达式，代码块，还有包和引用的等语法习性。而对于`Java`用户唯一提出挑战的就是「类型声明」被放在后面了；但是，当习惯了`Scala`代码风格后，你会发现后置类型修饰具有很多优势。

`Scala`也借鉴了`Smalltalk`的对象模型，修正了`Java`对象模型存在的诸多不足。例如，在没有损失性能的前提下，将`AnyVal, AnyRef`两者完美统一起来；不仅考虑层次的顶端，还设计了层次的末端，例如，`Nothing`的抽象，对「类型推演」具有重大意义。

`Scala`也借鉴了`Haskell`类型系统的设计，及其函数式的思维，并结合自身特性，优雅地将`OO`和`FP`整合在一起，取长补短，极大地增强了`Scala`的威力。

`Scala`也借鉴了`C++`多重继承，并吸收了`Ruby`的`Mixin`的特性，设计了强大的`trait`机制实现灵活的对象组合机制。

`Scala`也借鉴了`Erlang`的思想，在没有改变内核的情况下，通过扩展类库的方式支持`actor`的并发编程模式。

`Scala`也借鉴了`C++`语言的一些特性，例如操作符重载，隐式转换等特性；尤其增强了的「隐式转换」成为`Scala`可扩展性的重要机制。

## Scala的特质

接下来，通过几个简单的例子，阐述`Scala`所具有的一些特点，并阐述选择`Scala`的动机。

### Scala是自由的

> No One Size Fits All. 

`Scala`既增强了`OO`的语义，也引入了`FP`的思维，同时也拥有完备的「泛型编程」能力，它们互相截长补短，并融为一体。

`Scala`更像一把瑞士军刀，支持多种编程范式。`Scala`程序员拥有丰富的工具箱，当面对具体问题时拥有很大的自由空间，力求使用最简单的方法解决问题。

> 需求：要定义一个「读取器」，可以从字符中直接获取，可以从文件中读取，甚至从网络`IO`读取。

为了得到一致的抽象，可以定义`Reader`的抽象体，并对「数据源」这一变化方向进行抽象。`Scala`是一门多范式的程序设计语言，这里尝试使用两种不同的思维尝试解决这个问题。

##### 类型参数

先定义泛型的`Reader[+T]`，并赋予协变的能力。

```scala
trait Reader[+T] {
  val source: T
  def read: String
}
```

然后，再子类化一个`StringReader`，数据源从字符串中直接获取。

```scala
class StringReader(val source: String) extends Reader[String] {
  def read: String = source
}
```

再定义一个`FileReader`，数据源从文本文件中获取。

```scala
import scala.io.Source
import scala.util.Properties.lineSeparator
import scalaspec.control.using

class FileReader(val source: File) extends Reader[File] {
  def read: String = 
    using(Source.fromFile(source)) { file => 
      file.getLines.mkString(lineSeparator) 
    }
}
```

> `using`是一个自定义的抽象控制结构，用于保证资源的安全释放，它是「借贷模式」的经典应用，后文将对它进行阐述。

##### 抽象类型

首先定义一个抽象类型：`type In`；基于抽象类型`In`，再定义了一个抽象字段：`val source: In`；此外，`Reader`还定义了一个抽象方法`read`。

```scala
trait Reader {
  type In
  val source: In
  def read: String
}
```

然后，再子类化一个`StringReader`，数据源从字符串中直接获取。

```scala
class StringReader(val source: String) extends Reader {
  type In = String
  def read: String = source
}
```

再定义一个`FileReader`，数据源从文本文件中获取。

```scala
import scala.io.Source
import scala.util.Properties.lineSeparator

import scalaspec.control.using

class FileReader(val source: File) extends Reader {
  type In = File
  def read: String = 
    using(Source.fromFile(source)) { file =>
      file.getLines.mkString(lineSeparator) 
    }
}
```

### Scala是抽象的

`Scala`对于控制系统的复杂度拥有强大的抽象能力。甚至，`Scala`具备「控制结构」的抽象能力，使得设计更加正交，合理，程序更加简单，优雅。

> 需求1：判断某个单词是否包含数字

##### 使用Java

可以使用迭代快速实现这个需求。

```java
public static boolean hasDigit(String word) {
  for (int i = 0; i < word.length(); i++)
    if (Character.isDigit(word.charAt(i)))
      return true;
    return false;
}
```

> 需求2：判断某个单词是否包含大写字母

当然，可以通过复制粘贴，重复实现相同的逻辑；但是，如此将导致重复设计。

```java
public static boolean hasDigit(String word) {
  for (int i = 0; i < word.length(); i++)
    if (Character.isUpperCase(word.charAt(i)))
      return true;
    return false;
}
```

为了得到更为抽象的设计，使得代码具有高度的可复用性，可以提取一个抽象的接口。

```java
public interface CharacterSpec {
  boolean satisfy(char c);
}
```

为了保持抽象在同一个层次，可以定义一个新的抽象函数。

```java
public static boolean exists(String word, CharacterSpec spec) {
  for (int i = 0; i < word.length(); i++)
    if (spec.satisfy(word.charAt(i)))
      return true;
    return false;
}
```

可以如下使用这个抽象函数，判断单词是否包含数字。

```java
exists(word, new CharacterSpec() {
  @Override
  public boolean satisfy(char c) {
    return Character.isDigit(c);
  }
});
```

对于判断是否包含大写字母，则可以实现为：

```java
exists(word, new CharacterSpec() {
  @Override
  public boolean satisfy(char c) {
    return Character.isUpperCase(c);
  }
});
```

但是，使用匿名内部类，将导致复杂的程序结构和冗余的语法噪声。

##### 使用`Java8`

可以使用`Java8`的`lambda`表达式简化设计。

```java
exists(word, c -> Character.isDigit(c));
```

使用「方法引用」，可以进一步改善程序的表达力。

```java
exists(word, Character::isDigit);
```

但是，即使使用`Java8`，设计依然还是美中不足。其一，`exists`拥有两个参数，如果能够做到如下的代码块那就太神奇了。

```java
// 假设可以定义代码块
exists(word) { Character::isDigit };
```

其二，如果将`exists`成为`String`的一个方法，用户代码将更加具有`OO`的风格了。

```java
// 假设String拥有exists方法
word.exists(Character::isDigit);
```

可惜的是，`Java`并没有上述的能力。

##### 使用Scala

使用`Scala`，可以复用`Java`中既有的结构。按照`Scala`惯例，对`Java`实现的`StringUtil.exists`可以做一个简单的包装。

```scala
def exists(s: String)(p: Char => Boolean): Boolean =
  return StringUtil.exists(s, new CharacterSpec {
    override def satisfy(c: Char): Boolean = p(c)
  })
```

相比`Java`的实现，`Scala`可以借助「柯里化」的机制，进一步改善代码的表达力。

```scala
exists(word) { _.isUpper }
```

事实上，`Scala`运用「隐式转换」的神奇魔法，可以将`String`的功能增强为`StringOps`，使其直接能够调用`exists`方法。

```scala
word.exists(_.isUpper)
```

如果你偏爱大括号，也可以写成这样：

```scala
word.exists { _.isUpper }
```

### Scala是简洁的

`Scala`极度讨厌「重复」，严格坚持`DRY(Don't Repeat Youself)`原则。不仅仅体现在语法上，还包括类库的设计上。

> 需求：设计一个货币的值对象。

##### 使用Java

`Java`实现「值对象」时，语法较为啰嗦，并具有相同的模式，很容易形成重复的「样板代码」。

例如使用`private`定义字段，并在构造函数进行初始化；定义字段的`Getter`接口；即使`equals，hashCode`等具有逻辑的方法时，也表现为固定的模式。

```java
public class Currency {
  private final int amount;
  private final String designation;

  public Currency(int amount, String designation) {
    this.amount = amount;
    this.designation = designation;
  }

  public String getDesignation() {
    return designation;
  }

  public int getAmount() {
    return amount;
  }

  @Override
  public int hashCode() {
    ...
  }

  @Override
  public boolean equals(Object obj) {
    ...
  }
}
```

在`Java`社区，也存在一个实用方法，或者类库，支持`equals, hashCode`的自动生成，但因此也要让程序员付出额外的成本。

##### 使用Scala

`Scala`对于重复的事情从来不说两次。对于固定的模式，具备最直接、最简洁的表达方式，从而大幅地削减了代码量。

```scala
case class Currency(amount: Int, designation: String)
```

在`Scala`社区，`case class`是定义「值对象」的最佳实践。它天然地拥有`apply`的静态工厂方法，及其`Getter, equals, hashCode`等方法实现。

### Scala是性感的

`Scala`语法轻量，并具备丰富的表达力。借助于`Scala`强大的「类型推演」能力，`Scala`的简洁程度可以和动态语言相媲美。

> 需求：建立一个电话簿的数据表格。

##### 使用Java

使用`Java`建立一个简单的电话簿数据表格，类型`HashMap<String, String>`重复地进行声明，构造静态表也使用了特殊的初始化语义。

```java
Map<String, String> phonebook = new HashMap<String, String>() {{
  put("Horance", "+9519728872");
  put("Dave", "+9599820012");
}};
```

##### 使用Scala

使用`Scala`，代码实现不仅轻量，表现力也相当不错。

```scala
val phonebook = Map(
  "Horance" -> "+9519728872"
  "Dave"    -> "+9599820012"
)
```

没有冗余的类型噪声，而且具有更形象的语法描述。更重要的是，`->`操作符并不是语言内核所支持的，而是通过简单地扩展而实现的。

也就是说，`"Horance" -> "+9519728872"`构造了一个类型为`Tuple2[String, String]`的二元组，它等价于`("Horance", "+9519728872")`。

事实上，`->`定义在`Predef`中。

```scala
object Predef {
  implicit final class ArrowAssoc[A](self: A) extends AnyVal {
    def ->[B](y: B) = (self, y)
  }
}
```

### Scala是多变的

`Scala`犹如变形金刚，拥有无穷变化的空间。`Scala`倡导一个问题，拥有多种解法的思维习惯。当面对具体问题时，使得程序员拥有更多的自由选择权。

> 需求：打印程序选项列表

##### 使用`var`

`Scala`虽然倡导函数式的思维，但在某些性能苛刻的场景，指令式可能成为最后的救命稻草。

```scala
object Main extends App {
  var i = 0
  while (i < args.length) {
    println(args(i));
    i += 1
  }
}
```

##### `for`推导式

```scala
object Main extends App {
  for (arg <- args)
    println(arg)
}
```

##### 函数字面值

`Scala`的函数具有一等公民的地位，跟其他值一样可以被传递和存储。`foreach`就是一个高阶函数，它接受函数字面值作为参数进行传递。

```scala
object Main extends App {
  args.foreach((arg: String) => println(arg))
}
```

##### 类型自动推演

```scala
object Main extends App {
  args.foreach(arg => println(arg))
}
```

##### 占位符

因为`arg`在`foreach`中仅出现过一次，可以使用`_`的占位符简化实现。

```scala
object Main extends App {
  args.foreach(println(_))
}
```

##### 部分应用函数

也可以将`println`看成一个整体进行传递。需要注意的是，`println`与`_`之间有且仅有一个空格。

```scala
object Main extends App {
  args.foreach(println _)
}
```

对于此例，可以将传递`println`函数名，简化实现，提高表现力。

```scala
object Main extends App {
  args.foreach(println)
}
```

##### 可选的逗号和括号

在特殊场景下，逗号和括号是可选的，代码的表现力犹如自然语言一样直白。

```scala
object Main extends App {
  args foreach println
}
```

### Scala是开放的

> Open for Extension.

`Scala`具有高度可扩展性的架构，借助于`trait`，抽象类型和泛型，`Self Type`，隐式转换等机制，使得它具备强大的灵活性。

以`using`的设计为例，讲解`Scala`对于扩展性的支持，及其设计内部`DSL`的技术，加深对`Scala`的理解。

##### 形式化

对于资源管理，可以简化为如下的数学模型：给定一个资源`R`，并将资源传递给用户空间，并回调算法`f: R => T`，当过程结束时资源自动释放，其算法可形式化地描述为：

```
Input: Given resource: R
Output：T
Algorithm：Call back to user namespace: f: R => T, and make sure resource be closed on done.
```

因此，`using`常常被称为「借贷模式」，是保证资源自动回收的重要机制。

##### using实现

```scala
import scala.language.reflectiveCalls

object using {
  def apply[R <: { def close(): Unit }, T](resource: => R)(f: R => T): T = {
    var source: Option[R] = None
    try {
      source = Some(resource)
      f(source.get)
    } finally {
      for (s <- source)
        s.close
    }
  }
}
```

##### 控制抽象

使得`using`形如内置于语言的控制结构，其行为类似于`if, while`一样。

```scala
def read: String = using(Source.fromFile(source)) { file =>
  file.getLines.mkString(lineSeparator) 
}
```

##### 鸭子编程

`R <: { def close(): Unit }`表明`R`类型必须是具有`def close(): Unit`方法的子类型，这是`Scala`支持「鸭子编程」的一种重要技术。

例如，`File`满足`R`类型的特征，因为它拥有`close`方法。

##### 惰性求值

`resource: => R`是按照`by-name`传递，在实参传递形参过程中，并未对实参进行立即求值，而将求值推延至`resource: => R`的调用点。

例如，`using(Source.fromFile(source))`并没有马上调用`fromFile`方法，并传递给形参，而将求值推延至`source = Some(resource)`语句，即调用`Some.apply`方法时才展开计算。

##### Option

`Option`存在两种形式：`Some, None`，分别表示存在和不存在。在`Scala`社区，`Option`常常用于表示「存在与不存在」两者之间的语义。

##### for推导式

在`finally`关闭资源时，使用`for`推导式过滤掉`None`。也就是说，如下三种形式是等价的。

- 过滤掉`None`，并自动提取`Option`中的元素。

```scala
for (s <- source)
  s.close
```

- 使用`if`，但需要从`Some`中手动`get`。

```scala
if (source != None)
  source.get.close
```

## Scala的明天

软件设计的目的就是为了控制复杂性，让软件应对未来变化具有更好的弹性。`Scala`强大而自由，当程序员设计一个应用和类库时，具有很大的自由空间。

但是，`Scala`过度的灵活性，往往会诱惑他人掉进复杂性的深渊而不能自拔。它犹如具有「魔戒」的力量，虽然强大，但也很致命。

> Complexity is like a bug light for smart people. We can't resist it, even though we know it's bad for us. -- Alex Payne.

因此，应该理智地抵制复杂性的诱惑，才能真正地发挥`Scala`的威力。使用`Scala`不是为了炫技，而应该尽最大的可能让设计保持简单。

`Martin Ordersky`也在`2016`年元旦之初发文，号召社区有志之士在未来的时间里尽最大可能地降低`Scala`的复杂度。

我坚信，`Scala`的明天会更简单，更加漂亮。




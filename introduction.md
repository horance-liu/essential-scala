# 破冰之旅

> 戏路如流水，从始至终，点滴不漏。一路百折千回，本性未变，终归大海。一步一戏，一转身一变脸，扑朔迷离。真心自然流露，举手投足都是风流戏。一旦天幕拉开，地上再无演员。

[Scala](http://www.scala-lang.org/)是由`Martin Ordersky`设计的一门混合「面向对象」和「函数式」，并具备完备的「泛型编程」能力，支持多种范式的程序设计语言。

`Scala`的设计吸收了众多程序设计语言优秀的思想，取其精华，去其糟粕。`Scala`博大精深，并拥有与众不同的气质，及其鲜明的哲学思维体系。

## 选择Scala

`Scala`取名为「可扩展的语言」，因为它拥有良好的弹性。它使用不变的语言内核，构建万千变化的世界。同时，`Scala`也是设计「`DSL`(领域描述语言)」的利器，让编程充满轻松，愉快的气氛，并富有成就感。

`Scala`拥有强大的类型系统，具有丰富的表达能力，语法简洁，优雅，灵活。它应用广泛，从编写简单的脚本，到构建大型的企业级系统。

`Scala`运行于`JVM`之上，并与现有的`JVM`生态系统无缝链接，而无需定义额外的胶水代码。它兼容既有`Java`的类库，让成千上万的程序可以继续工作，并能够得以复用。

接下来，通过几个简单的例子，阐述`Scala`所具有的一些特点，并以此阐述选择`Scala`的动机。

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

但是，即使使用`Java8`，设计依然还是美中不足。因为`exists`被设计为一个实用方法，如果将`exists`成为`String`的一个方法，用户代码将更加具有面向对象的风格了。

```java
// 假设String拥有exists方法
word.exists(Character::isDigit);
```

可惜的是，`Java`并没有这样的能力。

##### 使用Scala

而使用`Scala`，则可以使用更为抽象的语义来表达。

```scala
word.exists(_.isUpperCase)
```

事实上，`java.lang.String`类中并不存在`exists`方法，但通过`Scala`「隐式转换」的神奇魔法，从而得到了一个功能增强的字符串类。

### Scala是简洁的

`Scala`极度讨厌「重复」，严格坚持`DRY(Don't Repeat Youself)`原则。不仅仅体现在语法上，还包括类库的设计上。

> 需求：设计一个货币的值对象。

##### 使用Java

`Java`实现「值对象」时，语法较为啰嗦，并具有相同的模式，很容易形成重复的「样板代码」。例如使用`private`定义字段，并在构造函数时赋予初始值；接着定义字段的`Getter`接口；即使设计和实现`equals，hashCode`等具有逻辑的方法时，也表现为固定的模式。

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
    return 17 * designation.hashCode() + amount;
  }

  @Override
  public boolean equals(Object obj) {
    if (obj instanceof Currency) {
      Currency other = (Currency)obj;
   return designation.equals(other.designation) && 
     amount == other.amount;
 }
 return false;
  }
}
```

在`Java`社区，也存在一个实用方法，或者类库，支持`equals, hashCode`的自动生成，但因此也要让程序员付出额外的成本。

##### 使用Scala

`Scala`对于重复的事情从来不说两次。对于固定的模式，具备最直接、最简洁的表达方式，从而大幅地削减了代码量。

```scala
case class Currency(amount: Int, designation: String)
```

在`Scala`社区，`case class`是定义「值对象」的最佳实践。它天然地拥有`apply`的静态工厂方法；自动地拥有`Getter, equals, hashCode`等方法实现。

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

当然，可以利用`Java7`部分的类型推演能力简化实现，但它没有彻底解决问题。

```java
Map<String, String> phonebook = new HashMap<>() {{
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

`Scala`摒弃了`static`的语义，使用`object`定义单键对象，体现了`Scala`纯正的`OO`血统。诸如静态工厂方法，工具类等实现模式，是单键对象最佳的使用场景。

`App`是一个特殊的「特质」，单键对象`Main`通过混入特质`App`，从根本上消除了`main`函数这一固定模式的重复实现，但自动地拥有`main`函数的所有特性，包括自动拥有`args: Array[String]`的程序选项。

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

例如，`Scala`在没有改变语言内核和基本特性的前提下，通过扩展类库便支持了`actor`的并发编程模式。

```scala
val service = actor {
  loop {
    receive {
   case Add(x, y) => reply(x + y)
   case Sub(x, y) => reply(x - y)
 }
  }
}

service ! Add(2, 3)
```

其中，`actor, loop, receive, !`等都不是语言的关键字。归功于其良好的可扩展性，`Scala`已然成为设计`DSL`的利器。

## Scala的明天

软件设计的目的就是为了控制复杂性，让软件应对未来变化具有更好的弹性。`Scala`强大而自由，当程序员设计一个应用和类库时，具有很大的自由空间。但过度的灵活性，往往诱惑他人掉进复杂性的深渊而不能自拔。

> Complexity is like a bug light for smart people. We can't resist it, even though we know it's bad for us. -- Alex Payne.

我们应该理智地控制`Scala`的威力，抵制复杂性的诱惑，尽最大的可能让设计保持简单。`Scala`的明天会更简单，更加漂亮，对此我充满信心。




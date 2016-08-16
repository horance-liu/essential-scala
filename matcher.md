# Scala.new

> 即使水墨丹青，何以绘出半妆佳人。

`Scala`是一门优雅而又复杂的程序设计语言，初学者很容易陷入细节而迷失方向。这也给写作带来了挑战，如果从基本的控制结构，再深入地介绍高级的语法，难免让读者生厌。

为此，本文另辟蹊径，尝试通过一个简单有趣的例子，概括性地介绍`Scala`常见的语言特性。它犹如一个迷你版的`Scala`教程，带领大家一起领略`Scala`的风采。

> **需求1：** 存在一个学生的列表，查找一个年龄等于`18`岁的学生

### 快速实现

```scala
def findByAge(students: Array[Student]): Student = {
  var i = 0
  while (i < students.length) {
    if (students(i).age == 18) {
      return students(i)
    } else {
      i += 1
    }
  }
  null
}
```

首先，这样的实现方式不是`Scala`所倡导的，实现存在很多设计的「坏味道」。

- 缺乏弹性参数类型：只支持数组类型，其他容器类型都被拒之门外；
- 容易出错：操作数组下标，往往引入不经意的错误；
- 幻数：硬编码，将算法与配置高度耦合；
- 返回`null`：再次给用户打开了犯错的大门；

先不用关心这些问题，我们看看这个实现的一些细节。

##### 类型的后缀修饰

- 变量的类型修饰：`students: Array[Student]`
- 函数返回值的类型修饰：`def findByAge(students: Array[Student]): Student`

因此，类型修饰便可达成一致的风格。

##### 类型推演

函数声明体最后的`=`将函数也看成普通的表达式，并具有类型推演的功能。关于类型推演，需要注意的是：

- 不能略去`=`，否则会限制函数的类型推演；
- 常常略去`return`，否则会限制函数的类型推演；
- 变量，函数返回值的类型，都可以略去；但会损失代码的表达力，除非逻辑非常简单；

```scala
var i: Int = 0  // 类型修饰显得冗余
```

相反地，如果略去函数返回值得类型修饰，表达力将大大折扣。事实上，此处不能略去函数返回值的类型修饰，因为函数体内使用了提前终止的`return`语句。

> 为了倡导函数式的设计风格，`Scala`并未提供`break, continue`的中断语句；但可以使用类库提供的`break`抽象控制结构进行模拟，但此处未使用该机制。

```scala
// 编译失败，因为使用了显式的return语句，必须显式地提供函数返回值类型
def findByAge(students: Array[Student]) = {
  ... 
  return students(i)
}
```

##### 数组与泛型

`Scala`使用中括号表示「类型参数」，而不是尖括号。另外，获取数组的元素，使用的是小括号，而不是中括号。

因为`Scala`并没有像`Java`一样，针对「基本类型」(例如`int`)，「数组类型」(例如`int[]`)定义特殊的语法，它将一切都看成对象。

`Array[String]`是一个对「原生数组」包装的普通类。获取元素的方法，与普通方法没有什么区别，只不过它默认调用的是`Array.apply`方法；如果要修改数组元素，则默认调用`Array.update`方法。

```scala
final class Array[T](val length: Int) {
  def apply(i: Int): T = ???            // 当调用arr(i)获取元素时
  def update(i: Int, x: T): Unit = ???  // 当调用arr(i) = t修改元素时
}
```

值得庆幸的是，`Scala`编译器会尽最大可能将其还原为原生的数组类型，以便获得最佳的执行效率。

##### var与val

- `var`用于修饰可变的变量
- `val`用于修饰不可变的变量

其中，`val`的效果等价于`Java`中的`final`。但需要注意，`val`仅表示引用本身不可变，它可以引用可变的对象。

##### 没有i++/++i

`Scala`认为`i++`违背了函数式的设计哲学，为此并未提供`i++/++i`的语法，可以使用`i += 1`所取代。

### 使用for推导式

使用`for`推导式，可以进一步简化代码的实现。

```scala
def findByAge(students: Array[Student]): Student = {
  for (s <- students)
    if (s.age == 18)
      return students(i)
  null
}
``` 

##### 生成器

`s <- students`称为`for`推导式的「生成器」，其中`s`默认地被`val`所修饰。

##### 守卫

可以使用`for`推导式的「守卫」，改进代码的可读性。

```scala
def findByAge(students: Array[Student]): Student = {
  for (s <- students if s.age == 18)
    return s
  null
}
```

> **需求2：** 查找一个名字为`horance`的学生

### 重复设计
    
新手常常通过「复制-粘贴」最快地实现此需求，但会产生明显的「重复设计」。

```scala
def findByName(students: Array[Student]): Student = {
  for (s <- students if s.name == "horance")
    return s
  null
}
```

##### 对象比较

在`Java`社区，比较对象的逻辑相等性使用的是`equals`，比较对象的物理相等性使用的是`==`。而在`Scala`社区，比较比较对象的逻辑相等性使用的是`==`，比较对象的物理相等性使用的是`eq, ne`。

这是因为`Scala`支持操作符命名的方法定义，自然需要将`Java`社区的习惯颠倒过来。

##### 消除重复

为了消除重复，可以将「查找算法」与「比较准则」这两个「变化方向」进行分离。

### 抽象准则
    
首先将比较的准则进行抽象化，让其独立变化。

```scala
trait StudentPredicate {
 def apply(s:Student): Boolean
}
```

##### 特质

此处使用`trait`定义了一个接口。事实上，`trait`不仅仅等价于`Java`中的`interface`，它是`Scala`实现对象组合的重要机制。

##### Boolean

在`Scala`里，没有原生的基本数据类型，它们都被看成了类。例如`Boolean, Char, Byte, Short, Int, Long, Float, Double`。

```scala
1 to 10   // 等价于1.to(10)
```

它们都是`AnyVal`的子类，它们的值只能通过「文字字面值」而存在，不能被`new`出来，也不能被赋予`null`值。

### 定义算子

将各个「变化原因」对象化，为此建立了两个简单的算子。
    
```scala
case class AgePredicate(age: Int) extends StudentPredicate {
  def apply(s: Student) = s.age == age
}
```
    
```scala
case class NamePredicate(name: String) extends StudentPredicate {
  def apply(s: Student) = s.name == name
}
```

此刻，查找算法的方法名也应该被「重命名」，使其保持在同一个「抽象层次」上。
    
```scala
def find(students: Array[Student], pred: StudentPredicate): Student = {
  for (s <- students if pred(s))
    return s
  null
}
```
    
客户端的调用根据场景，提供算子的配置。
    
```scala
find(students, AgePredicate(18)) == ???
find(students, NamePredicate(18)) == ???
```

##### apply方法

此处的`pred(s)`，等价于`pred.apply(s)`。

```scala
def find(students: Array[Student], pred: StudentPredicate): Student = {
  for (s <- students if pred(s))
    return s
  null
}
```

另外，此处的`AgePredicate(18)`，等价于`new AgePredicate(18)`。它调用了`AgePredicate`伴生对象的`apply`方法。

```scala
find(students, AgePredicate(18)) == ???
```

##### 样本类

使用`case class`定义的类称为「样本类」，使得它默认具有字段的`Getter`方法，并自定生成`equals, hashCode`等方法。

另外，「样本类」在其伴生对象中自动生成成类`apply`的工厂方法，从而生成对象时，可以略去`new`的关键字，语义更加明确。

也就是说，`case class AgePredicate...`等价于：

```scala
class AgePredicate(val age: Int) extends StudentPredicate {
  def apply(s: Student) = s.age == age
  
  override def equals(obj: Any): Boolean = ???
  override def hashCode(): Int = ???
}

object AgePredicate {
  def apply(age: Int) = new AgePredicate(age)
}
```

##### 伴生对象

`object AgePredicate`常常称为`class AgePredicate`的伴生对象。事实上，伴生对象中的方法，类似于`Java`中的`static`方法。但`Scala`摒弃了`static`的关键字，将面向对象的语义进行统一。

如果用`Java`设计`case class AgePredicate`，其实现类似于：

```java
public class AgePredicate implements StudentPredicate {
  private int age;
  
  public AgePredicate(int age) {
    this.age = age;
  }
  
  public int age() {
    return age;
  }
  
  @Override
  public Boolean apply(Student s) {
    return s.age == age;
  }
  
  @Override
  public boolean equals(Object obj) {
    ...
  }
  
  @Override
  public int hashCode() {
    ...
  }
  
  public static AgePredicate apply(int age) {
    return new AgePredicate(age);
  }
}
```

##### 构造参数

```java
public class AgePredicate implements StudentPredicate {
  private int age;
  
  public AgePredicate(int age) {
    this.age = age;
  }
  
  public int age() {
    return age;
  }
  
  ...
}
```

而`Scala`则直接使用`case class AgePredicate(age: Int)`声明一次`age`即可，从语法结构上彻底地消除了重复。在类名后面的参数列表，被称为「构造参数」，它极度地缩减了`Java`中大量重复的样板代码。


### 引入`GenTraversable`

按照「向稳定的方向依赖」的原则，为了适应诸如`List, Set, Seq, Array`等多种数据结构，可以将`find`的入参类型重构为更加抽象的`GenTraversable`类型。

```scala
import scala.collection.GenTraversable

def find(students: GenTraversable[Student], pred: StudentPredicate): Student = {
  for (s <- students if pred(s))
    return s
  null
}
```

> **需求3：** 存在一个老师列表，查找第一个女老师

### 类型重复
    
按照既有的代码结构，可以通过「复制-粘贴」快速地实现这个功能，但产生了明显的重复设计。
    
```scala
trait TeacherPredicate {
 def apply(s: Teacher): Boolean
}
```
 
```scala
import scala.collection.GenTraversable

def find(teachers: GenTraversable[Teacher], pred: TeacherPredicate): Teacher = {
  for (t <- teachers if pred(t))
    return t
  null
}
```

### 类型参数化

分析`StudentPredicate/TeacherPredicate`, `find(students: GenTraversable[Student])/find(teachers: GenTraversable[Teacher])`之间的重复，可以引入「类型参数化」的设计，以便消除重复设计。
    
首先消除`StudentPredicate`和`TeacherPredicate`之间的重复设计。

```scala
trait Predicate[E] {
  def apply(e: E): Boolean
}
```

再对`find`进行类型参数化设计。
    
```java
def find[E](c: GenTraversable[E], p: Predicate[E]): E = {
  for (e <- c if p(e))
    return e
  null.asInstanceOf[E]
}
```

其中，`null.asInstanceOf[E]`将`null`强制转换为`E`类型，否则会导致编译失败。

### 型变

但`Predicate`的「类型参数」缺乏「型变」的能力，为此引入「型变」能力的支持，接口更加具有可复用性。因为`Predicate`消费`E`类型的实例，为此为其引入「逆变」的能力。

```scala
trait Predicate[-E] {
  def apply(e: E): Boolean
}
```

### 绝不返回null

> Billion-Dollar Mistake
    
在最开始就遗留了一个问题：`find`返回了`null`。用户调用返回`null`的接口时，常常忘记`null`的检查，导致在运行时发生`NullPointerException`异常。
    
按照「向稳定的方向依赖」的原则，`find`的返回值应该设计为`Option<E>`，以便获取到「类型系统」的优势：
    
- 显式地表达了不存在的语义；
- 编译时保证错误的发生；
    
```scala
def find[E](c: GenTraversable[E], p: Predicate[E]): Option[E] = {
  for (e <- c if p(e))
    return Some(e)
  None
}
```

### 柯里化

将多个参数进行分解，使得函数调用更加具有弹性。

```scala
def find[E](c: GenTraversable[E])(p: Predicate[E]): Option[E] = {
  for (e <- c if p(e))
    return Some(e)
  None
}
```

这样，用户接口可以实现为：

```scala
find(students) { AgePredicate(30) } == ???
```

### 消灭`Predicate`

事实上，可以消灭`Predicate`特质，用更为一般的「函数类型」代替。

```scala
def find[E](c: GenTraversable[E])(p: E => Boolean): Option[E] = {
  for (e <- c if p(e))
    return Some(e)
  None
}
```

用户接口可以直接使用「函数字面值」，从而可以删除`AgePredicate`等算子。

```scala
find(students) { (s: Student) => s.age == 18 } == ???
```

借助「类型推演」的能力，其等价于：

```scala
find(students) { s => s.age == 18 } == ???
```

因为`s`在函数体内仅出现一次，可以使用「占位符」简化实现：

```scala
find(students) { _.age == 18 } == ???
```

### 复用`lambda`

观察如下两个测试用例，如果做到极致，可认为两个`lambda`表达式也是重复的。从「分离变化的方向」的角度分析，此`lambda`表达式承载的「比较算法」与「参数配置」两个职责，应该对其进行分离。

```scala
find(students) { _.name == "Horance" } == ???
find(students) { _.name == "Tomas" } == ???
```

可以通过「工厂方法」生产`lambda`表达式，将比较算法封装起来；而配置参数通过引入「参数化」设计，将「逻辑」与「配置」分离，从而达到最大化的代码复用。

```scala
object Predicate {
  def age(expectAge: Int): Student => Boolean =
    _.age == expectAge

  def name(expectName: String): Student => Boolean =
    _.name == expectName
}
```

客户端代码可以简化为：
   
```java
import scalaspec.matcher.Predicates._
    
find(students) { name("horance") }
find(students) { age(18) }
```

### 组合查询

但是，将上述`lambda`表达式封装在「工厂方法」的设计是及其脆弱的。例如，增加如下的需求：

> **需求4：** 查找年龄不等于18岁的女生

最简单的方法就是往`Predicate`不停地增加工厂方法，但这样的设计严重违反了`「OCP」(开放封闭)`原则。

```scala
object Predicate {
  def ageEq(expectAge: Int): Student => Boolean =
    _.age == expectAge

  def ageNe(expectAge: Int): Student => Boolean =
    _.age != expectAge
}
```

从需求看，比较准则增加了众多的语义，再次运用「分离变化方向」的原则，可发现存在两类运算的规则:
    
- 比较运算：`==, !=`
- 逻辑运算：`&&, ||`

##### 比较语义
    
先处理比较运算的变化方向，为此建立一个`Matcher`的抽象：
    
```scala
object Matcher {
  def equalTo[T](expected: T): T => Boolean = 
    expected == _  

  def notEqualTo[T](expected: T): T => Boolean = 
    expected != _
}
```

> Composition everywhere.
    
此刻，`age`的设计运用了「函数式」的思维，其行为表现为「高阶函数」的特性，通过函数的「组合式设计」完成功能的自由拼装组合，简单、漂亮。

```scala
object Predicate {
  def age(matcher: Int => Boolean): Student => Boolean = 
    s => matcher(s.age)
  ...
}
```

*查找年龄不等于18岁的学生*，可以如此描述。
    
```scala
find(students) { age(notEqualTo(18)) } == ???
```

##### 逻辑语义
    
为了使得逻辑「谓词」变得更加人性化，可以引入「流式接口」的`「DSL」`设计，增强表达力。
    
```scala
class Predicate[A](pred: A => Boolean) extends (A => Boolean) {
  override def apply(a: A) = pred(a)

  def &&(that: A => Boolean) = new Predicate[A](x => pred(x) && that(x))
  def ||(that: A => Boolean) = new Predicate[A](x => pred(x) || that(x))
  def unary_! = new Predicate[A](x => !pred(x))
}
```

其中`!`是一个一元操作符。

```scala
object Predicate {
  def age(matcher: Int => Boolean): Predicate[Student] = 
    new Predicate(s => matcher(s.age))
  ...
}
```
    
*查找年龄不等于18岁的女生*，可以表述为：
    
```java
find(students) { age(notEqualTo(18)) && name(equalTo("horance")) } == ???
```




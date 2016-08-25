# Scala类型推演

> 一纸，一笔，一人记录；一字，一句，千种情怀。

`Scala`支持多种编程范式，包括面向对象，函数式，泛型编程，并发编程等。要深入地理解和运用`Scala`将是一个非常漫长的过程。

而对于转瞬即逝的互联网时代，能够做到快速入门，降低学习的门槛，让既有的编程思维平滑迁移，是每个`Java`程序员所期待的。

本章重点讲述`Java`程序员迁移至`Scala`社区的一些习惯用法，以便帮助你快速融入到`Scala`编程的世界之中。

### 类型推演

因为`result`由`true`初始化，编译器能够准确地推演`result`为`Boolean`类型。

```scala
var result = true
```

其次，当函数实现较为简单，也可以略去函数返回值类型的声明，编译器也能够进行自动推演。例如`StringAdder.+`方法的返回值将自动推演为`String`。

> 当函数实现相对复杂时，如果略去函数返回值类型的声明，函数可读性将大大折扣。

```scala
class StringAdder[A](private val self: A) extends AnyVal {
  def +(other: String) = String.valueOf(self) + other
}
```

更为常见的是，当传递或返回「函数」类型时，根据宿主函数的原型信息可以自动推演函数文字的类型。

```scala
def exists(s: String)(p: Char => Boolean): Boolean =
  return StringUtil.exists(s, new CharacterSpec {
    override def satisfy(c: Char): Boolean = p(c)
  })
```

当调用`exists`时，可以传递`(c: Char) => c.isUpper`的函数值。

```scala
exists(word) { (c: Char) => c.isUpper }
```

事实上，`Scala`根据`exists`的类型声明，能够自动推演`(c: Char) => c.isUpper`中`c`的类型为`Char`，从而`exists`可以简化实现为：

```scala
exists(word) { c => c.isUpper }
```

但是，并非所有场景下都能够去除类型声明；相反，在某些场景下必须显式地进行类型的声明。

##### 抽象成员

抽象字段`source`，及其抽象方法`read`，都必须显式地进行类型声明。

```scala
trait Reader[+T] {
  val source: T
  def read: String
}
```

##### 参数类型

函数的参数类型必须显式地进行类型声明，例如`other: String`。

```scala
def +(other: String): String = ???
```

##### 显式地return

一般地，不要显示地`return`，编译器默认地讲最后一条语句的值作为函数的返回值。但时，在某些场景下，需要提前终止函数，`return`成为最后的杀手锏。

此时，应该显式地声明函数的返回值类型，即使函数实现逻辑相当简单。

```scala
def exists(s: String, spec: CharacterSpec): Boolean = {
  for (c <- s)
    if (spec.satisfy(c)) return true
  false
}
```

##### 重载函数

这是两个重载函数，必须显式地声明函数的返回值类型。

```scala
def join(strings: String*): String = strings.mkString("-")def join(strings: List[String]): String = join(strings: _*)
```

不能因为第二个`join`实现中调用了第一个`join`，就认为编译器能够根据第一个重载函数的返回值类型，自动地推演第二个重载函数的返回值类型。

```scala
def join(strings: String*): String = strings.mkString("-")def join(strings: List[String]) = join(strings: _*)  // Compile Fail.
```

##### 递归函数

`factorial`使用递归实现`n`的阶乘运算，它必须显式地声明函数返回值的类型。

```scala
@tailrecdef factorial(n: BigInt): BigInt =  if(n == 1) n else n * factorial(n - 1)
```

##### 推演太泛

```scala
val result = if(!opt.isEmpty) opt.get else 
    throw new RuntimeException("empty")
```

### 分号可选

使用`Scala`编程，当语句结束时，分号是可选。相对于强制「使用分号」或者「拒绝分号」，「分号可选」的方案具有很大的灵活性。

`forall`是一个很简单的泛型算法，它接受一个类型为`GenTraversableOnce[A]`的集合`c`，及其一个谓词`p`，用于判定集合中所有元素是否都满足于该谓词。

```scala
import scala.collection.GenTraversableOnce
import scala.util.control.Breaks._

def forall[A](c: GenTraversableOnce[A], p: A => Boolean): Boolean = {
  var result = true
  breakable {
    for (x <- c)
      if (!p(x)) { result = false; break }
  }
  result
}
```

一方面，当编译器能够准确地判定语句的结尾，略去分号可以使得代码更加简洁；否则，程序将冗余了过多的语法噪声。

```scala
def forall[A](c: GenTraversableOnce[A], p: A => Boolean): Boolean = {
  var result = true;  // 冗余
  ...
  result;
}
```

另一方面，如果两个语句关系紧密，将它们放在同一行，并用分号隔离，将使得代码更加紧凑；否则，程序相对就较为复杂、冗长。

```scala
if (!p(x)) { 
  result = false
  break
}
```

### 面向表达式

### 控制结构

### 构造参数

### 特质

### 



# Scala隐身术

> 带着最初的感受，在如烟的故事里，搜章劫律，凑一曲忧伤的旋律，默默的弹奏着，凄美而又伤情的韵律，到也能稍慰心怀。

「隐式转换」和「隐式参数」是`Scala`具备可扩展性的基石之一。利用这个机制，复用和扩展既有类库变得得心应手，而不用关心细枝末节。

但是，因为其工作机制太过隐晦，隐式转换也成为了`Scala`最具争议的特性之一。本文通过透析`Scala`隐式转换的原理，习惯用法，及其剖析标准库中常见的隐式转换，以便能够熟练地掌握隐式转换。

### 类增强

如果要为`String`增加一个`exists`方法，用于判断字符串中的字符是否具有某种特征。传统地，可以通过如下方法扩展`String`。

```scala
object String {
  def exists(s: String)(p: Char => Boolean): Boolean = ???
}
```

用户如此使用`exists`函数：

```scala
import scalaspec.ext.String._

exists("Alibaba") { _.isUpper }
```

按照惯例，我们更希望`exist`的调用方式更加具有`OO`的风格：

```scala
"Alibaba".exists { _.isUpper }
```

通过隐式转换，可以将`String`增强为一个具有`exists`方法的类。

```scala
object Predef {
  implicit def augmentString(x: String) = new StringOps(x)
}
```

其中，`StringOps`混入了特质`StringLike[String]`。

```scala
class StringOps extends StringLike[String] with ... {
  ...
}
```

而`StringLike`扩展自`IndexedSeqOptimized`

```scala
trait StringLike[+Repr] extends IndexedSeqOptimized[Char, Repr] with ... {
  ...
}
```

`exists`就定义在`IndexedSeqOptimized`之中，从而使得`String`高度地与集合类库集成在一起，实现最大化的代码复用。

```scala
trait IndexedSeqOptimized[+A, +Repr] extends Any with IndexedSeqLike[A, Repr] {
  private def prefixLength(p: A => Boolean, shortcut: Boolean): Int = {
    var i = 0
    while (i < length && p(apply(i)) == shortcut) i += 1
    i
  }

  override def forall(p: A => Boolean): Boolean = 
    prefixLength(p, true) == length

  override def exists(p: A => Boolean): Boolean = 
    prefixLength(p, false) != length
   
  ...
}
```

## Implicit Resolution Rules

### Marking Rule

> Only definitions marked implicit are available.

```scala
object Predef {
  implicit def intWrapper(x: Int) = new scala.runtime.RichInt(x)
}
```

```scala
object Predef {
  implicit final class any2stringadd[A](self: A) extends AnyVal {
    def +(other: String): String = String.valueOf(self) + other
  }
}
```

```scala
object Ordering {
  trait IntOrdering extends Ordering[Int] {
    def compare(x: Int, y: Int) =
      if (x < y) -1
      else if (x == y) 0
      else 1
  }
  implicit object Int extends IntOrdering
}
```

### Scope Rule

> An inserted implicit conversion must be in scope as a **single identifier**, or be associated with the source or target type of the conversion.

```scala
case class Yard(val amount: Int)
case class Mile(val amount: Int)
```

`mile2yard`可以定义在`object Mile`

```scala
object Mile {
  implicit def mile2yard(mile: Mile) = new Yard(10*mile.amount)
}
```

也可以定义在`object Yard`中

```scala
object Yard {
  implicit def mile2yard(mile: Mile) = new Yard(10*mile.amount)
}
```

转换为目标类型时，常常发生如下两个场景：

- 传递参数时，但类型匹配失败；

```scala
def accept(yard: Yard) = println(yard.amount + " yards")
accept(Mile(10))
```

- 赋值表达式，但类型匹配失败

```scala
val yard: Yard = Mile(10)
```

### Other Rules

- **One-at-a-time Rule**: Only one implicit is tried.
- **Explicits-First Rule**: Whenever code type checks as it is written, no implicits are attempted.
- **No-Ambiguty Rule**: An implicit conversion is only inserted if there is no other possible conversion is inserted.

## Where implicits are tried?

- Conversions to an expected type
  - 传递参数时，但类型匹配失败；
  - 赋值表达式，但类型匹配失败

- Conversions of the receiver of a selection
  - 调用方法，方法不存在
  - 调用方法，方法存在，但参数类型匹配失败

- Implicit parameters


# 函数论

> 一纸，一笔，一人记录；一字，一句，千种情怀。

软件设计的挑战在于管理系统的复杂度，其本质是一个「分离-组合」的过程。当系统变得庞大时，需要将其分解为更小，更易管理的部件，每一个部件的职责应该是单一的，它只完成它自己分内的任务；而要完成系统的功能，又要将各个部件协助起来才能实现。

`Scala`的函数被赋予了神奇的魔法，它的美感在于其「可组合性」。本文通过实战简化版的`Hamcrest`，以此体会`Scala`函数式设计的魅力。

## 函数类型

```scala
students.filter

val lines: Array[String] = ???
lines.filter(starts("ERROR:"))
```

这里定义了一个函数`starts`。

```scala
def contains(s: String): String => Boolean =
  _.contains(s)
```

返回值类型为`String => Boolean`



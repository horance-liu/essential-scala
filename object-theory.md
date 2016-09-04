# 对象论


## 揭秘Case类

使用`case class`定义的类称为「样本类」，它默认具有字段的`Getter`方法，并天然地拥有`equals, hashCode`等方法。

另外，「样本类」在其「伴生对象」中自动生成`apply`的工厂方法。当生产对象时，可以略去`new`关键字，使得语义更加简洁。

也就是说，`case class Times...`等价于：

```scala
class Times(val n: Int, val word: String) {
  def apply(m: Int): String =
    if (m % n == 0) word else ""

  override def equals(obj: Any): Boolean = ???
  override def hashCode(): Int = ???
  ...
}

object Times {
  def apply(n: Int, word: String) = new Times(n, word)
}
```

> 在使用`TDD`时，可以使用`???`快速通过编译。在本书中，为了缩减篇幅，也常用`???`表示函数实现的占位表示。
> 
> 其中，`???`定义在`Predef`中；因为`Predef`被编译器默认导入，其包含的成员对所有程序公开。
> 
> ```scala
> def ??? = throw new NotImplementedError
> ```

##### 伴生对象

`object Times`称为`class Times`的「伴生对象」。事实上，「伴生对象」类似于`Java`中的`static`语义。`Scala`摒弃了`static`的关键字，将面向对象的语义进行统一。

如果用`Java`设计`case class Times`，其实现类似于：

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
  
  // Getter方法
  public int n() {
    return n;
  }
  
  public String word() {
    return word;
  }
  
  // 自动生成equals, hashCode方法  
  @Override
  public boolean equals(Object obj) {
    ...
  }
  
  @Override
  public int hashCode() {
    ...
  }
  
  // 静态工厂方法
  public static Times apply(int n, String word) {
    return new Times(n, word);
  }
  
  ...
}
```


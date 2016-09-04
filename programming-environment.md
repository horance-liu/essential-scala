# 编程环境

## 安装

### 安装JDK

使用`Scala`之前，需要先安装`JDK(Java Development Kit)`，并配置相关环境变量。以`Linux`为例，安装`JDK`过程如下。

首先，从[`Oracle`官网](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载最新版的`JDK`，并解压至`/usr/local/lib/jvm`目录。

```bash
$ sudo mkdir -p /usr/local/lib/jvm
$ sudo tar zxvf jdk-8u65-linux-x64.gz -C /usr/local/lib/jvm
$ sudo ln -s /usr/local/lib/jvm/jdk1.8.0_65 /usr/local/lib/jvm/default
```

配置`JAVA_HOME`环境变量，并将`$JAVA_HOME/bin`添加到`PATH`环境变量之中。

```bash
$ echo "export JAVA_HOME=/usr/local/lib/jvm/default" >> ~/.bashrc
$ echo "export PATH=$PATH:$JAVA_HOME/bin" >> ~/.bashrc
```

运行`source`命令生效环境变量的修改。

```bash
$ source ~/.bashrc
```

最后，验证`Java`环境。

```bash
$ java -version
java version "1.8.0_65"
Java(TM) SE Runtime Environment (build 1.8.0_65-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.65-b01, mixed mode)
```

### 安装Scala

然后再安装`Scala`，以`Linux`为例，安装`Scala`过程如下。

首先，从[`Scala`官网](http://www.scala-lang.org)下载最新版本的`Scala`版本，并解压至`/usr/local/lib/scala`目录。

```bash
$ sudo mkdir -p /usr/local/lib/scala
$ sudo tar zxvf scala-2.11.7.tgz -C /usr/local/lib/scala
$ sudo ln -s /usr/local/lib/scala/scala-2.11.7 /usr/local/lib/scala/default
```

然后配置`SCALA_HOME`环境变量，并将`$SCALA_HOME/bin`添加到`PATH`环境变量之中。

```bash
$ echo "export SCALA_HOME=/usr/local/lib/scala/default" >> ~/.bashrc
$ echo "export PATH=$PATH:$SCALA_HOME/bin" >> ~/.bashrc
```

最后，验证`Scala`环境。

```bash
$ scala -version
Scala code runner version 2.11.7 -- Copyright 2002-2013, LAMP/EPFL
```

## Scala解释器：REPL

`REPL(Read-Evaluate-Print-Loop)`是`Scala`的解释器，对于学习和实践`Scala`语言的特性特别有帮助。

在命令行直接输入`scala`回车即可启动`REPL`。

```bash
$ scala
Welcome to Scala version 2.11.7 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_65).
Type in expressions to have them evaluated.
Type :help for more information.
```

然后就可以在`REPL`环境中编写简单的`Scala`程序了。

```bash
scala> println("hello, world!")
hello, world!
```

> 可以在`.bashrc`中为`scala`定义一个别名：`alias scala='scala -Dscala.color=true'`，启动`scala`时，终端就能有颜色了。

## 构建工具：SBT

## 集成开发环境

## 测试环境：ScalaTest

## 文档：ScalaDoc

## 资源



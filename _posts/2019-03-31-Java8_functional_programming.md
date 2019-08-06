---
layout:     post
title:      "Java 函数式编程"
subtitle:   "Java Functional Programming"
date:       2019-04-01 22:40:00
author:     "jk"
header-img: "img/post-bg-os-metro.jpg"
catalog: true
tags:
    - Java
    - Lambda
    - Functional Programming
---

![](https://img.charflow.com/2019-04-01-Java8_functional_programming/java8.jpeg)

##  default & static methods  in interfaces

### default methods

##### 特性：

- 隐式 **public** 关键字 (其他方法public atbstract)
- 显示 **default** 关键字

##### why？

向后兼容(backward compatibility)——直接在原有接口中加入新的方法，而不需重写原有实现该接口的类

联想：抽象类(模板设计模式)——不同：不可实例化、无构造器、多(单)继承、访问控制等

### static methods

特性：

- 显示 **static** 关键字 (和类的静态方法没啥区别)
- 接口中的其他静态方法和默认方法可调用

##### why?

避免单独的工具类，增加内聚


### Thinking...好像有什么问题？

##### 多重继承问题！！

- > （compile-time error）interface "xxx" inherits unrelated defaults for defaultCommon() from types "xxx" and "yyy"

- 接口允许多重继承，如果实现或继承多个接口，重名方法需要重写提供一个实现。

- 继承的父类和实现的接口中有相同签名，**且返回类型相同**， 可不重写，会调用父类方法。

```java
@Override
public String turnAlarmOn() {
    return Vehicle.super.turnAlarmOn() + " " + Train.super.turnAlarmOn();
}
     
@Override
public String turnAlarmOff() {
    return Vehicle.super.turnAlarmOff() + " " + Train.super.turnAlarmOff();
}
```

联想：Java 的重写、隐藏、*覆盖*

- 重写的概念——类型升级，多态，隐藏的概念——子类引用调用子类，父类引用调用父类
- 重写——动态绑定，隐藏——静态绑定
- 重写，隐藏都是对于父子继承而出现的，区别重载的作用域范围为同类内部
- 隐藏主要有两个关注点
  - 变量只能被隐藏，无论静态 or final
  - 静态方法：参数一样隐藏，参数不一样，子类继承(重载)；
- 覆盖在Java中的概念较弱，主要是C++中，C++中默认不是重写，加上virtual 且参数完全一致的同名方法，才形成覆盖，其他情况各调各的(隐藏)。
- [Java和C++里面的重写/隐藏/覆盖](https://www.cnblogs.com/charlesblc/p/6133605.html)

扩展：其他语言的多重继承问题 [Multiple inheritance](https://en.wikipedia.org/wiki/Multiple_inheritance)

- C++ 

  - 作用域限定符 （非同一基类）
  - 虚拟继承（菱形问题）

  【Ref】

  [c++ 多重继承歧义及其解决办法](https://blog.csdn.net/u014489596/article/details/38488927)

  [C++对象模型：单继承，多继承，虚继承](https://www.cnblogs.com/raichen/p/5744300.html)

  [C++ 多继承和虚继承的内存布局](https://blog.csdn.net/freeking101/article/details/60139209)

  [C++学习之多重继承与虚继承](https://songlee24.github.io/2014/07/17/cpp-inheritance/)

- Python MRO

  - 经典类 DFS

    ![](https://img.charflow.com/2019-04-01-Java8_functional_programming/python-MRO.jpg)

    问题：菱形继承，C只能继承无法重写

  - BFS——存在单调性问题

  - 新式类：C3线性算法(大杀器)，和拓扑排序相似，但有区别。手动计算C3 Linearization！

    ![](https://img.charflow.com/2019-04-01-Java8_functional_programming/multiple_inheritance.png)

  【Ref】

  [你真的理解Python中MRO算法吗？](http://python.jobbole.com/85685/)

  [C3 线性化算法与 MRO](http://kaiyuan.me/2016/04/27/C3_linearization/)

## Lambda

### Thinking Ahead

- 为什么要引入Lambda ？
- Java 8 Lambda为什么是这样 ?
- Lambda 8 背后有什么 其他的 设计思考？

### Lambda or Anonymous Classes

#### 无参函数简写

```java
// JDK7 匿名内部类写法
new Thread(new Runnable(){// 接口名
	@Override
	public void run(){// 方法名
		System.out.println("Thread run()");
	}
}).start();
```

```java
// JDK8 Lambda表达式写法
new Thread(
		() -> System.out.println("Thread run()")// 省略接口名和方法名
).start();
```

接口名和函数名全都省略，聚焦实现逻辑

#### 带参函数简写

```java
// JDK7 匿名内部类写法
List<String> list = Arrays.asList("I", "love", "you", "too");
Collections.sort(list, new Comparator<String>(){// 接口名
    @Override
    public int compare(String s1, String s2){// 方法名
        if(s1 == null)
            return -1;
        if(s2 == null)
            return 1;
        return s1.length()-s2.length();
    }
});
```

```java
// JDK8 Lambda表达式写法
List<String> list = Arrays.asList("I", "love", "you", "too");
Collections.sort(list, (s1, s2) ->{// 省略参数表的类型
    if(s1 == null)
        return -1;
    if(s2 == null)
        return 1;
    return s1.length()-s2.length();
});
```

### So, Lambda instead of Anonymous Classes ，Why? and what's the differences？

- 语法冗余
- 匿名类中的`this`和变量名容易使人产生误解(
  - 匿名内部类：引用掩盖问题
  - lambda：相同的词法作用域

```java
public class Hello {
	Runnable r1 = () -> { System.out.println(this); };
	Runnable r2 = () -> { System.out.println(toString()); };
	public static void main(String[] args) {
		new Hello().r1.run();
		new Hello().r2.run();
	}
	public String toString() { return "Hello Hoolee"; }
}
```

> - [ ] `Hello$1@5b89a773`和`Hello$2@537a7706`
>
> - [x] `"Hello Hoolee"`打印两遍    

> 内部类中通过继承得到的成员（包括来自`Object`的方法）可能会把外部类的成员掩盖（shadow），此外未限定（unqualified）的`this`引用会指向内部类自己而非外部类。
>
> Lambda不会从超类（supertype）中继承任何变量名，也不会引入一个新的作用域。Lambda表达式函数体里面的变量和它外部环境的变量具有相同的语义（也包括lambda表达式的形式参数）。此外，'this'关键字及其引用在Lambda表达式内部和外部也拥有相同的语义。

- 无法捕获非`final`的局部变量

  匿名内部类中对引用的外部变量——捕获的变量 (*源于翻译 Java编译器实现的只是capture-by-value，并没有实现capture-by-reference*)强制被声明为final，否则产生编译错误。Lambda将这个要求降低为Effectively Final。

  - 匿名内部类为什么强制final?  —— 和闭包有关系 —— Java中存在闭包么？[java为什么匿名内部类的参数引用时final？](https://www.zhihu.com/question/21395848)

    > Java编译器支持了闭包，但支持地不完整。编译器编译的时候把外部环境局部变量，拷贝了一份到匿名内部类里。
    >
    > Java编译器实现的只是capture-by-value，并没有实现capture-by-reference（lambda expressions close over *values*, not *variables*）

  - Effectively Final又是啥？如果一个局部变量在初始化后从未被修改过，那么它就符合有效只读的要求，换句话说，加上final后也不会导致编译错误的局部变量就是有效只读变量。

    ```java
    Callable<String> helloCallable(String name) {
      String hello = "Hello";
      return () -> (hello + ", " + name);
    }
    ```

    尽管放宽了对捕获变量的语法限制，但试图修改捕获变量的行为仍然会被禁止。

    ```java
    int sum = 0;
    list.forEach(e -> { sum += e.size(); });
    ```

    

  *额外补充：*

  *lambda表达式对值封闭，对变量开放 (lambda expressions close over values, not variables)*

  ```java
  int sum = 0;
  list.forEach(e -> { sum += e.size(); }); // Illegal, close over values
  
  List<Integer> aList = new List<>();
  list.forEach(e -> { aList.add(e); }); // Legal, open over variables
  ```

  *和函数式编程的**immutable data** 思想很像*。[函数式编程](https://coolshell.cn/articles/10822.html)

#### 瞅一眼JVM层面

- 匿名内部类仍然是一个类，只是不需要程序员显示指定类名，编译器会自动为该类取名，编译之后将会产生两个class文件。
- Lambda表达式通过invokedynamic指令实现，书写Lambda表达式不会产生新的类。

[Lambda and Anonymous Classes(II)](https://github.com/CarpenterLee/JavaLambdaInternals/blob/master/2-Lambda%20and%20Anonymous%20Classes(II).md)

### 为什么Lambda能够这样简写

Java是强类型语言，也意味着不能在代码的任何地方任性的写Lambda表达式。所以Lambda表达式简写有两个重要的支撑：

- 函数式接口
- 类型推断机制

> 尽管匿名内部类有着种种限制和问题，但是它有一个良好的特性，它和Java类型系统结合的十分紧密：每一个函数对象都对应一个接口类型。
>
> - 接口是Java类型系统的一部分
> - 接口天然就拥有其运行时表示（Runtime representation）

函数式接口为Lambda融入现有Java体系提供了通路。

## Functional Interfaces

### Definition

函数式接口：(SAM)Single Abstract Method。顾名思义，只有一个抽象方法的接口。

but，有歧义。函数式接口允许有default和static方法。

函数式接口并不是Java8 中才有的，所有符合上述条件的都是函数式接口，如Runnable。since 1.8的是`@FunctionalInterface`注解和为了Lambda而生的 *java.util.function* 包。`@FunctionalInterface`注解的主要作用是声明函数式接口和供编译器验证函数式接口要求。

### java.util.function

#### Function

```java
/**
 * Represents a function that accepts one argument and produces a result.
 */
@FunctionalInterface
public interface Function<T, R> {
       R apply(T t);
    
       /**
        * Returns a composed function that first applies the {@code before}
        * function to its input, and then applies this function to the result.
        */
       default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        	Objects.requireNonNull(before);
        	return (V v) -> apply(before.apply(v));
}
```

compose 事例

```java
Function<Integer, String> intToString = Object::toString;
Function<String, String> quote = s -> "'" + s + "'";
 
Function<Integer, String> quoteIntToString = quote.compose(intToString);
 
assertEquals("'5'", quoteIntToString.apply(5));
```

#### 基本类型函数式接口

- IntFunction, LongFunction, DoubleFunction: 参数是基本类型，返回类型是引用类型
- ToIntFunction, ToLongFunction, ToDoubleFunction： 参数是引用类型，返回类型是基本类型
- DoubleToIntFunction, DoubleToLongFunction, IntToDoubleFunction, IntToLongFunction, LongToIntFunction, LongToDoubleFunction 全都是基本类型

#### 二元函数式接口

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
```

名字上加了Bi

#### 函数式接口类型总结

| 名称                                                         | 一元接口      | 说明                           | 二元接口       |              说明              |
| :----------------------------------------------------------- | :------------ | :----------------------------- | :------------- | :----------------------------: |
| 一般函数                                                     | Function      | 一元函数，抽象apply方法        | BiFunction     |    二元函数，抽象apply方法     |
| [Operator](https://en.wikipedia.org/wiki/Operation_(mathematics)) 算子函数（输入输出同类型） | UnaryOperator | 一元算子，抽象apply方法        | BinaryOperator |    二元算子，抽象apply方法     |
| **Predicates**谓词函数（输出boolean）                        | Predicate     | 一元谓词，抽象test方法         | BiPredicate    |     二元谓词，抽象test方法     |
| **Consumers**消费者（无返回值）                              | Consumer      | 一元消费者函数，抽象accept方法 | BiConsumer     | 二元消费者函数，抽象accept方法 |
| **Suppliers**供应者（无参数，只有返回值）                    | Supplier      | 供应者函数，抽象get方法        | -              |               -                |

补充：二元算子 可以用于归约，二元算子的可结合性(Associative)使得归约操作很好实现并行化。

【Ref】

[一起爪哇Java 8（一）——函数式接口](https://www.zybuluo.com/changedi/note/616132)

## 再看Lambda——类型推导机制

### 目标类型——Lambda类型是什么

> Lambda类型推导机制帮助我们把匿名内部类的高度问题转化成宽度问题，lambda表达式并不是第一个拥有上下文相关类型的Java表达式：泛型方法调用和“菱形”构造器调用也通过目标类型来进行类型推导

### Lambda类型及常见写法

Lambda表达式可以看做一个函数式接口的实例！同一个Lambda表达式在不同的上下文中可能是不同的类型，Lambda类型由上下文推导而来。

```java
@FunctionalInterface
public interface ConsumerInterface<T>{
	void accept();
}

@FunctionalInterface
public interface ConsumerInterface1<T>{
	void accept();
}


（）-> (System.out.println("hi");) 啥类型？
```

> 译器负责推导lambda表达式的类型。它利用lambda表达式所在上下文**所期待的类型**进行推导，这个**被期待的类型**被称为*目标类型*。**`lambda表达式只能出现在目标类型为函数式接口的上下文中`。**
>
> 
>
> 当然，lambda表达式对目标类型也是有要求的。编译器会检查lambda表达式的类型和目标类型的方法签名（method signature）是否一致。当且仅当下面所有条件均满足时，lambda表达式才可以被赋给目标类型`T`：
>
> - `T`是一个函数式接口
> - lambda表达式的参数和`T`的方法参数在数量和类型上一一对应
> - lambda表达式的返回值和`T`的方法返回值相兼容（Compatible）
> - lambda表达式内所抛出的异常和`T`的方法`throws`类型相兼容

```java
// Lambda表达式的书写形式
Runnable run = () -> System.out.println("Hello World");// 无参函数简写
ActionListener listener = event -> System.out.println("button clicked");// 单个参数函数简写
Runnable multiLine = () -> {// 3 代码块 写法
    System.out.print("Hello");
    System.out.println(" Hoolee");
};
```

### 目标类型上下文

- 变量声明

  ```java
  Comparator<String> c;
  ```

- 赋值

  ```java
  c = (String s1, String s2) -> s1.compareToIgnoreCase(s2);
  ```

- 返回语句

  ```java
  public Runnable toDoLater() {
    return () -> {
      System.out.println("later");
    }
  }
  ```

- 数组初始化器

  ```java
  filterFiles(new FileFilter[] {
                f -> f.exists(), f -> f.canRead(), f -> f.getName().startsWith("q")
              });
  ```

  > 数组初始化器和赋值类似，类型是从数组类型中推导得知

- 方法和构造方法的参数

  > 方法参数的类型推导要相对复杂些：目标类型的确认会涉及到其它两个语言特性：`重载解析（Overload resolution）和参数类型推导（Type argument inference）`。
  >
  > 重载解析会为一个给定的方法调用（method invocation）寻找最合适的方法声明（method declaration）。由于不同的声明具有不同的签名，当lambda表达式作为方法参数时，重载解析就会影响到lambda表达式的目标类型。编译器会通过它所得之的信息来做出决定。如果lambda表达式具有`显式类型`（参数类型被显式指定），编译器就可以直接 使用lambda表达式的返回类型；如果lambda表达式具有`隐式类型`（参数类型被推导而知），重载解析则会忽略lambda表达式函数体而只依赖lambda`表达式参数的数量`。

  ```java
  BinaryOperator<Long> add = (Long x, Long y) -> x + y;// 显式类型
  BinaryOperator<Long> addImplicit = (x, y) -> x + y;//  隐式类型
  ```

  举例：

  ```java
  List<Person> ps = ...
  Stream<String> names = ps.stream().map(p -> p.getName());
  ```

  > 在上面的代码中，`ps`的类型是`List<Person>`，所以`ps.stream()`的返回类型是`Stream<Person>`。`map()`方法接收一个类型为`Function<T, R>`的函数式接口，这里`T`的类型即是`Stream`元素的类型，也就是`Person`，而`R`的类型未知。由于在重载解析之后lambda表达式的目标类型仍然未知，我们就需要推导`R`的类型：通过对lambda表达式函数体进行类型检查，我们发现函数体返回`String`，因此`R`的类型是`String`，因而`map()`返回`Stream<String>`

- lambda表达式函数体

  ```java
  upplier<Runnable> c = () -> () -> { System.out.println("hi"); }; //高阶函数 -- 联想闭包
  ```

- 条件表达式（`? :`）

  ```java
  Callable<Integer> c = flag ? (() -> 23) : (() -> 42);
  ```

- 转型（Cast）表达式

  ```java
  // Object o = () -> { System.out.println("hi"); }; 这段代码是非法的
  // Target type of a lambda conversion must be an interface
  Object o = (Runnable) () -> { System.out.println("hi"); };
  ```

出现二义性怎么办？

- 使用显式lambda表达式（为参数`p`提供显式类型）以提供额外的类型信息
- 把lambda表达式转型为`Function<T>`
- 为泛型参数`R`提供一个实际类型。（`<String>(p -> p.getName())`）

## Method Reference

函数式接口不仅给我们带来了Lambda，还给我们带来了另一个大杀器——方法引用。

方法引用，能让我们以及其简洁的方式来服用我们已有java8 之前的代码。方法引用也可以看作是一种特殊类型的Lambda表达式。

```java
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.forEach(System.out::println);
```

### 类别

|                             kind                             |             **Example**              |
| :----------------------------------------------------------: | :----------------------------------: |
|         Reference to a static method（静态方法引用）         |  ContainingClass::staticMethodName   |
| Reference to an instance method of a particular object(实例上的实例方法引用) | containingObject::instanceMethodName |
| Reference to an instance method of an arbitrary object of a particular type(类型上的实例方法引用) |      ContainingType::methodName      |
|           Reference to a constructor(构造方法引用)           |            ClassName::new            |

| 类型                 | 方法引用         | 等价Lambda表达式       |
| -------------------- | ---------------- | ---------------------- |
| 静态                 | String::valueOf  | x ->String.valueOf(x)  |
| 实例上实例方法引用   | x::toString      | () -> x.toString()     |
| 类型上的实例方法引用 | Object::toString | x -> x.toString()      |
| 构造方法引用         | ArrayList::new   | () ->new ArrayList<>() |
|                      | String::new      | s->new String(s)       |
|                      | int[]::new       | x -> new int[x]        |
|                      | this::equals     | s -> this.equals(x)    |
|                      | super::equals    | s -> super.equals(s)   |

需要特别关注的是类型上的实例方法引用，因为这和java之前不允许使用类名去调用成员方法好像有歧义。

[java lambda方法引用总结——烧脑吃透](https://my.oschina.net/polly/blog/917214)，这篇文章可以参考，作者在末尾有测试，但是其实可以简单的去理解，第一个参数成为调用该实例方法的对象，这里的类型名用来指示用这个类型的对象去调用该实例方法。

### 方法引用的参数类型匹配

> 因为函数式接口的方法参数对应于隐式方法调用时的参数，所以被引用方法签名可以通过放宽类型，装箱以及组织到参数数组中的方式对其参数进行操作，就像在调用实际方法一样

```java
Consumer<Integer> b1 = System::exit;    // void exit(int status)
Consumer<String[]> b2 = Arrays:sort;    // void sort(Object[] a)
Consumer<String> b3 = MyProgram::main;  // void main(String... args)
Runnable r = Myprogram::mapToInt        // void main(String... args)
```

## forEach —> JCF

### forEach

#### 方法签名

forEach, 从命名可以看出来是个方法，方法签名如下：

```java
void forEach(Consumer<? super T> action)
void forEach(BiConsumer<? super K,? super V> action)
```

#### 代码进化论

```java
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
for(String str : list){
        System.out.println(str);
}
```

```java
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.forEach(new Consumer<String>(){
    @Override
    public void accept(String str){
            System.out.println(str);
    }
});
```

```java
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.forEach( str -> {
            System.out.println(str);
 });
```

```
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.forEach(System.out::println);
-------------------------------------------------------------
namesMap.forEach((key, value) -> System.out.println(key + " " + value));// Map中的新迭代方式
```

### JCF

Java 8 中 JCF 添加了Lambda的支持，背后的原理就是接口中加入了对default和static的支持。

![JCF_Collection_Interfaces](https://img.charflow.com/2019-04-01-Java8_functional_programming/JCF_Collection_Interfaces.png)

| 接口名     | Java8新加入的方法                                            |
| ---------- | ------------------------------------------------------------ |
| Collection | removeIf() spliterator() stream() parallelStream() forEach() |
| List       | replaceAll() sort()                                          |
| Map        | getOrDefault() forEach() replaceAll() putIfAbsent() remove() replace() computeIfAbsent() computeIfPresent() compute() merge() |

【Ref】

[Lambda and Collections](https://github.com/CarpenterLee/JavaLambdaInternals/blob/master/3-Lambda%20and%20Collections.md)

## Best Practices

### Item 42: Prefer lambdas to anonymous classes —— Effective Java 3rd Edition

> In summary, as of Java 8, lambdas are by far the best way to represent small function objects. Don’t use anonymous classes for function objects unless you have to create instances of types that aren’t functional interfaces.

[Is using Lambda expressions whenever possible in java good practice?](https://softwareengineering.stackexchange.com/questions/341066/is-using-lambda-expressions-whenever-possible-in-java-good-practice)

### Item 43: Prefer method references to lambdas——Effective Java 3rd Edition

> In summary, method references often provide a more succinct alternative to lambdas. Where method references are shorter and clearer, use them; where they aren’t, stick with lambdas.

### Item 44: Favor the use of standard functional interfaces——Effective Java 3rd Edition

> To use the standard interfaces provided in java.util.function.Function, but keep our eyes open for the relatively rare cases where we would be better off writing our own functional interfaces

## Stream API

Lambda 表达式遇到Stream 时，才是威力终极所在！

Stream特性：

- 无存储
- 为函数式编程而生
- 惰式执行
- 可消费性

### Stream 基本操作

- 中间操作惰式执行
- 结束操作出发实际计算以pipleline方式执行

| 操作类型 | 接口方法                                                     |
| -------- | ------------------------------------------------------------ |
| 中间操作 | concat() distinct() filter() flatMap() limit() map() peek()  skip() sorted() parallel() sequential() unordered() |
| 结束操作 | allMatch() anyMatch() collect() count() findAny() findFirst()  forEach() forEachOrdered() max() min() noneMatch() reduce() toArray() |

#### forEach()     

`void forEach(Consumer<? super E> action)`

```java
// 使用Stream.forEach()迭代
Stream<String> stream = Stream.of("I", "love", "you", "too");
stream.forEach(str -> System.out.println(str));
```

#### filter()

`Stream<T> filter(Predicate<? super T> predicate)`

```java
// 保留长度等于3的字符串
Stream<String> stream= Stream.of("I", "love", "you", "too");
stream.filter(str -> str.length()==3)
    .forEach(str -> System.out.println(str));
```

![1553517036254](https://img.charflow.com/2019-04-01-Java8_functional_programming/stream-fileter.png)

#### distinct()

`Stream<T> distinct()`

```java
Stream<String> stream= Stream.of("I", "love", "you", "too", "too");
stream.distinct()
    .forEach(str -> System.out.println(str));
```

![](https://img.charflow.com/2019-04-01-Java8_functional_programming/stream-distinct.png)

#### sorted()

`Stream<T>　sorted(Comparator<? super T> comparator)`

```java
Stream<String> stream= Stream.of("I", "love", "you", "too");
stream.sorted((str1, str2) -> str1.length()-str2.length())
    .forEach(str -> System.out.println(str));
```

#### map()

`<R> Stream<R> map(Function<? super T,? extends R> mapper)`

```java
Stream<String> stream　= Stream.of("I", "love", "you", "too");
stream.map(str -> str.toUpperCase())
    .forEach(str -> System.out.println(str));
```

![](https://img.charflow.com/2019-04-01-Java8_functional_programming/stream-map.png)

#### flatMap()

`<R> Stream<R> flatMap(Function<? super T,? extends Stream<? extends R>> mapper)`

```java
Stream<List<Integer>> stream = Stream.of(Arrays.asList(1,2), Arrays.asList(3, 4, 5));
stream.flatMap(list -> list.stream())
    .forEach(i -> System.out.println(i));
```

### reduce

> *reduce*操作可以实现从一组元素中生成一个值，`sum()`、`max()`、`min()`、`count()`等都是*reduce*操作，将他们单独设为函数只是因为常用。

```java
Optional<T> reduce(BinaryOperator<T> accumulator)
T reduce(T identity, BinaryOperator<T> accumulator)
<U> U reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator<U> combiner)
```

![Stream.reduce_parameter](https://img.charflow.com/2019-04-01-Java8_functional_programming/Stream.reduce_parameter.png)

```java
// 求单词长度之和
Stream<String> stream = Stream.of("I", "love", "you", "too");
Integer lengthSum = stream.reduce(0,　// 初始值　// (1)
        (sum, str) -> sum+str.length(), // 累加器 // (2)
        (a, b) -> a+b);　// 部分和拼接器，并行执行时才会用到 // (3)
// int lengthSum = stream.mapToInt(str -> str.length()).sum();
System.out.println(lengthSum);
```

### collect

![Stream.collect_parameter](https://img.charflow.com/2019-04-01-Java8_functional_programming/Stream.collect_parameter.png)

```java
<R> R collect(Supplier<R> supplier, BiConsumer<R,? super T> accumulator, BiConsumer<R,R> combiner) // collect 签名
<R,A> R collect(Collector<? super T,A,R> collector) //Collectors 工具类
    
    
 Examples:
//　将Stream规约成List
Stream<String> stream = Stream.of("I", "love", "you", "too");
List<String> list = stream.collect(ArrayList::new, ArrayList::add, ArrayList::addAll);// 方式１
//List<String> list = stream.collect(Collectors.toList());// 方式2
System.out.println(list);


// 将Stream转换成List或Set
Stream<String> stream = Stream.of("I", "love", "you", "too");
List<String> list = stream.collect(Collectors.toList()); // (1)
Set<String> set = stream.collect(Collectors.toSet()); // (2)


// 使用toCollection()指定规约容器的类型
ArrayList<String> arrayList = stream.collect(Collectors.toCollection(ArrayList::new));// (3)
HashSet<String> hashSet = stream.collect(Collectors.toCollection(HashSet::new));// (4)
```

#### collect 与 map

```java
// 使用toMap()统计学生GPA
Map<Student, Double> studentToGPA =
     students.stream().collect(Collectors.toMap(Function.identity(),// 如何生成key
                                     student -> computeGPA(student)));// 如何生成value

// Partition students into passing and failing
Map<Boolean, List<Student>> passingFailing = students.stream()
         .collect(Collectors.partitioningBy(s -> s.getGrade() >= PASS_THRESHOLD));

// Group employees by department
Map<Department, List<Employee>> byDept = employees.stream()
            .collect(Collectors.groupingBy(Employee::getDepartment));

// 使用下游收集器统计每个部门的人数
Map<Department, Integer> totalByDept = employees.stream()
                    .collect(Collectors.groupingBy(Employee::getDepartment,
                                                   Collectors.counting()));// 下游收集器

// 按照部门对员工分布组，并只保留员工的名字
Map<Department, List<String>> byDept = employees.stream()
                .collect(Collectors.groupingBy(Employee::getDepartment,
                        Collectors.mapping(Employee::getName,// 下游收集器
                                Collectors.toList())));// 更下游的收集器
```



【Ref】

[Java Functional Programming Internals](https://github.com/CarpenterLee/JavaLambdaInternals)

[深入理解Java 8 Lambda（语言篇——lambda，方法引用，目标类型和默认方法）](https://www.cnblogs.com/figure9/p/java-8-lambdas-insideout-language-features.html)

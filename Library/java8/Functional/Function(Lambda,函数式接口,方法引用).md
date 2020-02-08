## 接口的默认方法和静态方法
Java 8的接口中允许有默认方法和静态方法。如果一个类继承了一个类还实现了一个接口，而且接口中的默认方法和父类中的方法同名，这时采用类优先原则。也就是说，子类使用的是父类的方法，而不是接口中的同名方法。

**默认方法使用default关键字修饰**

Java8的接口和Scala的trait有点类似了。

## Lambada表达式
java的Lambada表达式 和 scala的函数字面量写法基本一样。不同的是

* 1、java用`->`scala用`=>`

* 2、scala中return可以省略，java中多行语句有返回值必须加return(java这点和js一样)

```java
Collections.sort(names, (String a, String b) -> b.compareTo(a));
Collections.sort(names, (a, b) -> b.compareTo(a));
Comparator<String> comparator = (a, b) -> b.compareTo(a);
```

Lambada表达式访问外部变量被隐式声明为final，可以省略，之前匿名内部类的方式必须声明为final。

## 函数式接口(FunctionalInterface)
Lambda表达式是如何在java的类型系统中表示的呢？每一个lambda表达式都对应一个类型，就是函数式接口类型。如果没有函数式接口，就不能写Lambda表达式。

**只包含一个抽象方法的接口，称为函数式接口**。默认方法 不算抽象方法，所以你也可以给你的函数式接口添加默认方法。

可以通过 Lambda 表达式来创建该接口的对象。（若 Lambda 表达式 抛出一个受检异常(即：非运行时异常)，那么该异常需要在目标接口的抽 象方法上进行声明）。 

我们可以在一个接口上使用 **@FunctionalInterface** 注解，这样做可以检查它是否是一个函数式接口，编译器如果发现你标注了这个注解的接口有多于一个抽象方法的时候会报错的。同时 javadoc 也会包含一条声明，说明这个 接口是一个函数式接口。 @FunctionalInterface不是必须要加的，只是启到标识和检查的作用，不加也可以是函数式接口。

**在java.util.function包下定义了Java 8 的丰富的函数式接口**

### Java8内置四大核心函数式接口
Java8提供的四大核心函数接口如下：

| 接口             | 方法               | 描述       |
| ---------------- | ------------------ | ---------- |
| `Comsumer<T>`    | void accept (T t)  | 消费型接口 |
| `Supplier<T>`    | T get()            | 供给型接口 |
| `Function<T, R>` | R apply (T t)      | 函数型接口 |
| `Predicate<T>`   | Boolean test (T t) | 断言型接口 |

* `Consumer<T>`：消费型接口(void accept(T t))，接收一个参数，无返回值。    
* `Supplier<T>`：供给型接口(T get())，无参数，有返回值。    
* `Function<T,R>`：函数型接口(R apply(T t))，接收一个参数，有返回值。    
* `Predicate<T>`：断言型接口(boolean test(T t))，接收一个参数，返回Boolean值。

```scala
Consumer[T]：T => Unit
Supplier[T]：() => T
Function[T, R]：T => R
Predicate[T]：T => boolean
```

### Java8其他内置接口 
| 接口                   | 方法                       | 描述                                      |
| ---------------------- | -------------------------- | ----------------------------------------- |
| `BiFunction <T, U, R>` | R apply (T t, U u)         | 对传入参数进行操作，返回R类型处理结果     |
| `UnaryOperator <T>`    | T apply (T t)              | 对参数进行一元运算操作，返回类型为T的结果 |
| `BinaryOperator <T>`   | T apply (T t1, T t2)       | 对参数进行二元运算操作，返回T类型的结果   |
| `BiConsumer <T, U>`    | void accept (T t, U u)     | 对参数t，u进行操作，不返回结果            |
| `ToIntFunction <T>`    | int applyAsInt (T t)       | 将传入参数强转成int类型返回               |
| `ToLongFunction <T>`   | long applyAsLong (T t)     | 将传入参数强转成long类型返回              |
| `ToDoubleFunction <T>` | double applyAsDouble (T t) | 将传入参数强成double类型返回              |
| `IntFunction <R>`      | R apply(int param)         | 将一个int类型经过处理后强转成R类型返回    |
| `LongFunction <R>`     | R apply(long param)        | 将一个long类型经过处理后强转成R类型返回   |
| `DoubleFunction <R>`   | R apply(double param)      | 将一个double参数经过处理后强转成R类型返回 |

```scala
BiFunction[T, U, R]：(T, U) => R
UnaryOperator[T]：T => T
BinaryOperator[T]：(T, T) => T
BiConsumer[T, U]：(T, U) => Unit
ToIntFunction[T]：T => int
ToLongFunction[T]：T => long
ToDoubleFunction[T]：T => double
IntFunction[R]：int => R
LongFunction[R]：long => R
DoubleFunction[R]：double => R
```

## 方法引用(Method References)
当要传递给Lambda体的操作，已经有实现的方法了，可以使用方法引用！ 

要求：实现接口的抽象方法的参数列表和返回值类型，必须与方法引用的 方法的参数列表和返回值类型保持一致！

格式：使用操作符 “::” 将类(或对象) 与 方法名分隔开来。 

如下三种主要使用情况： 
```java
//相当于(a,b,c ...) -> method(a,b,c ...)   Integer::compare
类::静态方法名
//相当于(a,b,c ...) -> a.method(b,c ...)   String::compareTo
类::实例方法名 
//相当于(a,b,c ...) -> e.method(a,b,c ...)  System.out::println
对象::实例方法名  
```
例子：
```java
//相当于：(a, b) -> Integer.compare(a, b)
Integer::compare
//相当于：(a, b) -> a.compareTo(b)
String::compareTo
//相当于：(a) -> System.out.println(a)
System.out::println
```

## 构造器引用
格式： ClassName::new 

与函数式接口相结合，自动与函数式接口中方法兼容。 

可以把构造器引用赋值给定义的方法，要求构造器参数列表要与接口中抽象 方法的参数列表一致！且方法的返回值即为构造器对应类的对象。

```java
Supplier<Employee> supplier = () -> new Employee();
//可以改写成这样
Supplier<Employee> supplier = Employee::new;
```




```java

```
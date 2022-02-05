## jdk8
### Lambda表达式
用于简化代码结构，Lambda表达式通常使用(param)->(body)语法书写，示例如下：


```java
//未使用
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("启动一个线程");
    }
}).start();

```

```java
//使用
new Thread(()->{
    System.out.println("启动一个线程");
}).start();

```
Lambda 表达式简化了匿名内部类的形式，可以达到同样的效果，但是 Lambda 要优雅的多。虽然最终达到的目的是一样的，但其实**内部的实现原理却不相同**。
匿名内部类在编译之后会创建一个新的匿名内部类出来，而 Lambda 是调用` JVM invokedynamic`指令实现的，并不会产生新类。

* Lambda表达式可以有0~n个参数。
* 参数类型可以显式声明，也可以让编译器从上下文自动推断类型。如(int x)和(x)是等价的。
* 多个参数用小括号括起来，逗号分隔。一个参数可以不用括号。
* 没有参数用空括号表示。
* Lambda表达式的正文可以包含零条，一条或多条语句，如果有返回值则必须包含返回值语句。如果只有一条可省略大括号。如果有一条以上则必须包含在大括号（代码块）中。

### 函数式接口
函数式接口(Functional Interface)是Java8对一类特殊类型的接口的称呼。这类接口**只定义了唯一的抽象方法的接口**（除了隐含的Object对象的公共方法），因此最开始也就做SAM类型的接口（Single Abstract Method）。

java.lang.Runnable就是一种函数式接口，在其内部只定义了一个void run()的抽象方法，同时在该接口上注解了`@FunctionalInterface`。

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

### 方法引用
方法引用可以**将一个方法赋给一个变量或者作为参数传递给另外一个方法**。`::`双冒号作为方法引用的符号，如：

```java
Function<String, Integer> s = Integer::parseInt;
Integer i = s.apply("10");
```

对于方法引用的返回值，**引用方法的参数个数、类型，返回值类型要和函数式接口中的方法声明一一对应才行**，如下：

```java
//Function接口的定义
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

```java
//parseInt方法的定义
public static int parseInt(String s) throws NumberFormatException {
    return parseInt(s,10);
}
```
### Stream

![](https://img-blog.csdnimg.cn/20191009131714290.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9oZWxsby5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70)

[图源](https://www.choupangxia.com/2019/10/09/java8-stream%e6%96%b0%e7%89%b9%e6%80%a7%e8%af%a6%e8%a7%a3%e5%8f%8a%e5%ae%9e%e6%88%98/)






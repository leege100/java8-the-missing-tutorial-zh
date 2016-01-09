Lambdas
-----

lambda表达式是java8中最重要的特性之一，它让代码变得简洁并且可以当作参数进行传递。之前一段时间，Java总因为缺少函数式编程的能力而饱受批评。随着函数式编程变得越来越流行，Java也被迫开始拥抱函数式编程。否则，Java会被大家逐步抛弃。

Java8使得这个世界上最流行的编程语言在函数式编程上实现大的跨越。一门语言要支持函数式编程，就必须把函数作为一等公民。在Java8之前，只能通过匿名内部类来写出函数式编程的代码。而随着lambda表达式的引入，函数变成了一等公民，并且可以被当作参数传递。

lambda表达式运行开发者定义一个匿名函数，开发者可以像在程序中使用变量的定义一样来使用lambda表达式。在支持高阶函数的编程语言中，lambda表达式是必须的。高阶函数是指能够把函数作为参数或者返回值是函数的那些函数。

> 这个章节的代码如下[ch02 package](https://github.com/shekhargulati/java8-the-missing-tutorial/tree/master/code/src/main/java/com/shekhargulati/java8_tutorial/ch02).

随着Java8中lambda表达式的引入，Java也支持高阶函数。接下来让我们来分析这个经典的lambda表达式示例--Java中Collections类的一个排序（sort）函数。sort函数有两种调用方法，一种需要一个List作为参数，另一种需要一个List参数和一个Comparator。第二种sort函数是一个接收lambda表达式的高阶函数的实例，如下：

```java
List<String> names = Arrays.asList("shekhar", "rahul", "sameer");
Collections.sort(names, (first, second) -> first.length() - second.length());
```

上面的代码是根据names的长度来进行排序，运行的结果如下：

```
[rahul, sameer, shekhar]
```

上面代码片段中的`（first,second) -> first.length() - second.length()`表达式是一个`Comparator<String>`的lambda表达式。

* `（first,second）`是`Comparator` 中`compare`方法的参数。

* `first.length() - second.length()`比较name字符串长度的函数体。

* `->` 是用来把参数从函数体中分离出来的操作符。

在我们深入研究Java8中的lambda表达式之前，我们先来追溯一下他们的历史，了解它们为什么会存在。

## lambda表达式的历史

Lambda expressions have their roots in the Lambda Calculus. [Lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus) originated from the work of [Alonzo Church](https://en.wikipedia.org/wiki/Alonzo_Church) on formalizing the concept of expressing computation with functions. Lambda calculus is turing complete formal mathematical way to express computations. Turing complete means you can express any mathematical computation via lambdas.

lambda表达式源自于`λ演算`.[λ演算](https://en.wikipedia.org/wiki/Lambda_calculus)是[Alonzo Church](https://en.wikipedia.org/wiki/Alonzo_Church)在用函数式来制定表达式计算概念的得出的。`λ演算`是等价于图灵机的。等价于图灵机意味着你可以用lambda表达式来表达任何数学算式。



Lambda calculus became the basis of strong theoretical foundation of functional programming languages. Many popular functional programming languages like Haskell, Lisp are based on Lambda calculus. The idea of higher order functions i.e. a function accepting other functions came from Lambda calculus.

`λ演算`后来成为了函数式编程语言强有力的理论基础。诸如 Hashkell、Lisp等著名的函数式编程语言都是基于`λ演算`.高阶函数的概念就来自于`λ演算`。

The main concept in Lambda calculus is expressions. An expression can be expressed as:

`λ演算`最主要的概念就是表达式，一个表达式可以用如下形式来表示：

```
<expression> := <variable> | <function>| <application>
```



* **variable** -- A variable is a placeholder like x, y, z for values like 1, 2, etc or lambda functions.

* **variable** -- 一个variable就是一个类似x、y、z占位符或者1、2这类值,或者lambda函数式


* **function** -- It is an anonymous function definition that takes one variable and produces another lambda expression. For example, `λx.x*x` is a function to compute square of a number.

* **function** -- 函数的定义，需要一个变量，生产出另一个lambda表达式。比如`λx.x*x`是一个求平方的函数。


* **application** -- This is the act of applying a function to an argument. Suppose you want a square of 10, so in lambda calculus you will write a square function `λx.x*x` and apply it to 10. This function application  would result in `(λx.x*x) 10 = 10*10 = 100`.You can not only apply simple values like 10 but, you can apply a function to another function to produce another function. For example, `(λx.x*x) (λz.z+10)` will produce a function `λz.(z+10)*(z+10)`. Now, you can use this function to produce number plus 10 squares. This is an example of higher order function.

* **application** -- 把一个函数当成一个参数的行为。简书你想要10的平台，那你需要写一个λ演算式 `λx.x*x`并把10当真自变量传进去，这个函数程序就会返回`(λx.x*x) 10 = 10*10 = 100`。但是你不仅仅可以只传10，你可以把一个函数传给另一个函数然后生成另一个函数。比如，`(λx.x*x) (λz.z+10)` 会生成这样一个新的函数 `λz.(z+10)*(z+10)`。现在，你可以用这个函数来生成一个数加上10的平方。这就是一个高阶函数的实例。


Now, you understand Lambda calculus and its impact on functional programming languages. Let's learn how they are implemented in Java 8.

现在，你已经理解了`λ演算`和它对函数式编程语言的影响。下面我们继续学习它们在java8中如何实现。

## 在java8之前传递行为

Before Java 8, the only way to pass behavior was to use anonymous classes. Suppose you want to send an email in another thread after user registration. Before Java 8, you would write code like one shown below.

Java8之前，传递行为的唯一方法就是通过匿名内部类。假设你在用户完成注册功能之后，需要通过另外一个线程发送一封邮件。在Java8之前，可以通过如下方式：

```java
sendEmail(new Runnable() {
            @Override
            public void run() {
                System.out.println("Sending email...");
            }
        });
```

The sendEmail method has following method signature.

sendEmail方法定义如下：

```java
public static void sendEmail(Runnable runnable)
```

The problem with the above mentioned code is not only that we have to encapsulate our action i.e. `run` method in an object but, the bigger problem is that it misses the programmer intent i.e. to pass behavior to `sendEmail` function. If you have used libraries like Guava then, you would have certainly felt the pain of writing anonymous classes. A simple example of filtering all the tasks with **lambda** in their title is shown below.

上面的代码的问题不仅仅在于我们需要把行为封装进去，比如`run`方法在一个对象里面；更糟糕的是，它丢失了开发者原本的意图，比如把行为传递给`sendEmail`函数。如果你用过一些其他库，如Guava,那么你就会切身的感受到写匿名内部类的痛苦。下面是一个简单的例子，过滤所有标题中包含**lambda**字符串的task。


```java
Iterable<Task> lambdaTasks = Iterables.filter(tasks, new Predicate<Task>() {
            @Override
            public boolean apply(Task task) {
                return input.getTitle().contains("lambda");
            }
});
```
With Java 8 Stream API, you can write the above mentioned code without the use of a third party library like Guava. We will cover streams in [chapter 3](./03-streams.md). So, stay tuned!!

使用Java8的Stream API,开发者不用太第三方库额可以写出上面的代码，我们将在下一章[chapter 3](./03-streams.md)讲述streams相关的知识。这里，暂不多提。

## Java 8 Lambda expressions

## Java 8 Lambda表达式

In Java 8, we would write the code using a lambda expression as show below. We have mentioned the same example in the code snippet above.

在Java8中，我们可以用lambda表达式写出如下的代码。

```java
sendEmail(() -> System.out.println("Sending email..."));
```

The code shown above is concise and does not pollute the programmer's intent to pass behavior. `()` is used to represent no function parameters i.e. `Runnable` interface `run` method does not have any parameters. `->` is the lambda operator that separates the parameters from the function body which prints `Sending email...` to the standard output.

上面的代码非常简洁，并且能够清晰的传递编码者的意图。`()`用来传递无参函数，上例中的`Runnable`接口的中`run`方法不含任何参数，直接就可以用`()`代替。`->`是将参数和函数体分开的lambda操作符，上例中，`->`后面打印`Sending email`。

Let's look at the Collections.sort example again so that we can understand how lambda expressions work with the parameters. To sort a list of names by their length, we passed a `Comparator` to the sort function. The `Comparator` is shown below.

下面再次通过Collections.sort这个例子来了解带参数的lambda表达式如何使用。通过传递一个`Comparator`到sort函数，来将names列表按照字符串的长度排序。`Comparator`如下

```java
Comparator<String> comparator = (first, second) -> first.length() - second.length();
```

The lambda expression that we wrote was corresponding to compare method in the Comparator interface. The signature of `compare` function is shown below.

上述的lambda表达式等同于Comparator接口中的compare方法。`compare`方法的定义如下:

```java
int compare(T o1, T o2);
```
`T` is the type parameter passed to `Comparator` interface. In this case it will be a `String` as we are working over a List of `String` i.e. names.

`T`是传递给`Comparator`接口的参数类型，在本例中names列表是由`String`组成，所以`T`代表的是`String`。

In the lambda expression, we didn't have to explicitly provide the type -- String. The `javac` compiler inferred the type information from its context. The Java compiler inferred that both parameters should be String as we are sorting a List of String and `compare` method use only one T type. The act of inferring type from the context is called **Type Inference**. Java 8 improves the already existing type inference system in Java and makes it more robust and powerful to support lambda expressions. `javac` under the hoods look for the information close to your lambda expression and uses that information to find the correct type for the parameters.

在lambda表达式中，我们每必要明确指出参数类型，`javac`编译器会通过上下文自动推断参数的类型信息。由于我们是在对一个由`String`类型组成的List进行排序并且`compare`方法仅仅用一个T类型，所以Java编译器自动推断出两个参数都是`String`类型。根据上下文推断类型的行为称为**类型推断**。Java8提升了Java中已经存在的类型推断系统，使得对lambda表达式的支持变得更加强大。`javac`通过紧邻lambda表达式的一些信息来找出参数的正确类型。

> In most cases, `javac` will infer the type from the context. In case it can't resolve type because of missing or incomplete context information then the code will not compile. For example, if we remove `String` type information from `Comparator` then code will fail to compile as shown below.

> 在大多数情况下，`javac`会根据上下文自动推断类型。如果因为丢失了上下文信息或者上下文信息不完整而导致无法推断出类型，代码就不会编译通过。例如，下面我们将`String`类型从`Comparator`中移除，代码就会编译失败。


```java
Comparator comparator = (first, second) -> first.length() - second.length(); // compilation error - Cannot resolve method 'length()'
```

## How does Lambda expressions work in Java 8?
## Lambda表达式在Java8中的运行机制

You may have noticed that the type of a lambda expression is some interface like Comparator in the above example. You can't use any interface with lambda expression. ***Only those interfaces which have only one non-object abstract method can be used with lambda expressions***. These kinds of interfaces are called **Functional interfaces** and they can be annotated with `@FunctionalInterface` annotation. Runnable interface is an example of functional interface as shown below.

你可能已经发现lambda表达式的类型是一些类似上例中Comparator
的接口。但是不是每个接口都可以使用lambda表达式，***只有那些仅仅包含一个非实例化抽象方法的接口才能使用lambda表达式***。这样的接口被称着**函数式接口**并且它们能够被`@FunctionalInterface`注解注释。下面的Runnable接口就是函数式接口的一个例子。

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

`@FunctionalInterface` annotation is not mandatory but, it can help tools know that an interface is a functional interface and perform meaningful actions. For example, if you try to compile an interface that annotates itself with `@FunctionalInterface` annotation and has multiple abstract methods then compilation will fail with an error ***Multiple non-overriding abstract methods found***. Similarly, if you add `@FunctionInterface` annotation to an interface without any method i.e. a marker interface then you will get error message ***No target method found***.

`@FunctionalInterface`注解不是必须的，但是它能够让工具知道一个接口是一个函数式接口并且表现出有意义的行为。比如，如果你是这编译一个用`@FunctionalInterface`注解自己并且含有多个抽象方法的接口，编译就会报出这样一个错***Multiple non-overriding abstract methods found***。同样的，如果你给一个不含有任何方法的接口添加`@FunctionalInterface`注解，会得到如下错误信息,***No target method found***.

Let's answer one of the most important questions that might be coming to your mind. ***Are Java 8 lambda expressions just the syntactic sugar over anonymous inner classes or how does functional interface gets translated to bytecode?***

下面来回答一个你心中的疑问，***Java8的lambda表达式只是一个匿名内部类的语法糖吗或者函数式接口是如何被转换成字节码的？***

The short answer is **NO**. Java 8 does not use anonymous inner classes mainly for two reasons:

答案是**NO**,Java8不采用匿名内部类的原因主要有两点：

1. **Performance impact**: If lambda expressions were implemented using anonymous classes then each lambda expression would result in a class file on disk. When these classes are loaded by JVM at startup, then startup time of JVM will increase as all the classes needs to be first loaded and verified before they can be used.

1. **性能影响**: 如果lambda表达式是采用匿名内部类实现的，那么每一个lambda表达式都会生成一个class文件，所有的class文嘉都需要在进入时加载并且在使用前确认，从而导致JVM的启动变慢。

2. **Possibility to change in future**: If Java 8 designers would have used anonymous classes from the start then it would have limited the scope of future lambda implementation changes.

2. **向后的扩展性**: 如果Java8的设计者从一开始就采用匿名内部类的方式，那么这将会lambda表达式未来的使用范围。

### Using invokedynamic

### 使用动态启用

Java 8 designers decided to use `invokedynamic` instruction added in Java 7 to defer the translation strategy at runtime. When`javac` compiles the code, it captures the lambda expression and generates an `invokedynamic` call site (called lambda factory). The `invokedynamic` call site when invoked returns an instance of the functional interface to which the lambda is being converted. For example, if we look at the byte code of our Collections.sort example, it will look like as shown below.


Java8的设计者采用在Java7中新增的`动态启用`来延迟在运行时的加载策略。当`javac`编译代码时，会找出代码中的lambda表达式并且生成一个`动态启用`的调用地址（又称lambda工厂）。当`动态启用`被调用时，就会向lambda表达式转换的地方返回一个函数式接口的实例。比如，在Collections.sort这个例子中，它的字节码如下：

```
public static void main(java.lang.String[]);
    Code:
       0: iconst_3
       1: anewarray     #2                  // class java/lang/String
       4: dup
       5: iconst_0
       6: ldc           #3                  // String shekhar
       8: aastore
       9: dup
      10: iconst_1
      11: ldc           #4                  // String rahul
      13: aastore
      14: dup
      15: iconst_2
      16: ldc           #5                  // String sameer
      18: aastore
      19: invokestatic  #6                  // Method java/util/Arrays.asList:([Ljava/lang/Object;)Ljava/util/List;
      22: astore_1
      23: invokedynamic #7,  0              // InvokeDynamic #0:compare:()Ljava/util/Comparator;
      28: astore_2
      29: aload_1
      30: aload_2
      31: invokestatic  #8                  // Method java/util/Collections.sort:(Ljava/util/List;Ljava/util/Comparator;)V
      34: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
      37: aload_1
      38: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/Object;)V
      41: return
}
```

The interesting part of the byte code shown above is the line 23 `23: invokedynamic #7,  0              // InvokeDynamic #0:compare:()Ljava/util/Comparator;` where a call to `invokedynamic` is made.

上面代码的关键部分位于第23行`23: invokedynamic #7,  0              // InvokeDynamic #0:compare:()Ljava/util/Comparator;`这里创建了一个`动态启用`的调用。

The second step is to convert the body of the lambda expression into a method that will be invoked through the `invokedynamic` instruction. This is the step where JVM implementers have the liberty to choose their own strategy.

接下来是将lambda表达式转换成一个将会通过`动态启用`指引调用的方法。在这一步中，JVM实现者有自由选择策略的权利。

I have only glossed over this topic. You can read about internals at http://cr.openjdk.java.net/~briangoetz/lambda/lambda-translation.html.

这里我仅粗略的概括一下，具体的内部标准见这里 http://cr.openjdk.java.net/~briangoetz/lambda/lambda-translation.html.

## Anonymous classes vs lambdas

## 匿名类 vs lambda表达式

Let's compare anonymous classes with lambdas to understand the differences between them.

下面我们一起来对比一下匿名类和lambda表达式，以此来理解它们的不同。

1. In anonymous classes, `this` refers to the anonymous class itself whereas in lambda expression `this` refers to the class enclosing the lambda expression.

1. 在匿名类中，`this` 指代的是匿名类本身；而在lambda表达式中，`this`指代的是lambda表达式所在的这个类。

2. You can shadow variables in the enclosing class inside the anonymous class. This gives compile time error when done inside lambda expression.

2. 在匿名类中可以shadow包含此匿名类的变量，而在lambda表达式中就会报编译错误。

3. Type of the lambda expression is determined from the context where as type of the anonymous class is specified explicitly as you create the instance of anonymous class.

3. lambda表达式的类型是由上下文决定的，而匿名类中必须在创建实例的时候明确指定。

## Do I need to write my own functional interfaces?

## 我需要自己去写函数式接口吗?

By default, Java 8 comes with many functional interfaces which you can use in your code. They exist inside `java.util.function` package. Let's have a look at few of them.

默认的，Java8带有许多可以直接在代码中使用的函数式接口。它们位于`java.util.function`包中，下面简单介绍几个:

### java.util.function.Predicate<T>

This functional interface is used to define check for some condition i.e. a predicate. Predicate interface has one method called `test` which takes a value of type `T` and return boolean. For example, from a list of `names` if we want to filter out all the names which starts with **s** then we will use a predicate as shown below.

此函数式接口是用来定义对一些条件的检查，比如一个判断。Predicate接口有一个叫`test`的方法，它需要一个`T`类型的值作为参数，返回布尔值。例如，在一个`names`列表中找出所有以**s**开头的名字就可以像如下代码这样使用判断。

```java
Predicate<String> namesStartingWithS = name -> name.startsWith("s");
```

### java.util.function.Consumer<T>

This functional interface is used for performing actions which does not produce any output. Predicate interface has one method called `accept` which takes a value of type `T` and return nothing i.e. it is void. For example, sending an email with given message.

这个函数式接口用于那些不需要产生任何输出的行为。Predicate接口有一个叫做`accept`的方法，它需要一个`T`类型的参数并且返回值。例如，发一个指定信息的邮件：

```java
Consumer<String> messageConsumer = message -> System.out.println(message);
```

### java.util.function.Function<T,R>

This functional interface takes one value and produces a result. For example, if we want to uppercase all the names in our `names` list, we can write a Function as shown below.

这个函数式接口需要一个值并返回一个结果。例如，将所有`names`列表中的name转换为大写，可以像下面这样写一个Function:

```java
Function<String, String> toUpperCase = name -> name.toUpperCase();
```

### java.util.function.Supplier<T>

This functional interface does not take any input but produces a value. This could be used to generate unique identifiers as shown below.

这个函数式接口不需要传值，但是会返回一个值。可以像下面这样，用来生成唯一的标识符

```java
Supplier<String> uuidGenerator= () -> UUID.randomUUID().toString();
```

We will cover more functional interfaces throughout this tutorial.

在接下来的章节中，我们会学习更多的函数式接口。

## Method references

There would be times when you will be creating lambda expressions that only calls a specific method like `Function<String, Integer> strToLength = str -> str.length();`. The lambda only calls `length()` method on the `String` object. This could be simplified using method references like `Function<String, Integer> strToLength = String::length;`. They can be seen as shorthand notation for lambda expression that only calls a single method. In the expression `String::length`, `String` is the target reference, `::` is the delimiter, and `length` is the function that will be called on the target reference. You can use method references on both the static and instance methods.

有时候，你需要为一个特定方法创建lambda表达式时，比如`Function<String, Integer> strToLength = str -> str.length();`，这个表达式仅仅在`String`对象上调用`length()`方法。可以这样来简化它，`Function<String, Integer> strToLength = String::length;`。仅调用一个方法的lambda表达式，可以用缩写符号来表示。在`String::length`中，`String`是目标参考，`::`是定界符，`length`是目标参考要调用的方法。方法参考可以用在静态方法和实例方法上。

### Static method references

Suppose we have to find a maximum number from a list of numbers then we can write a method reference `Function<List<Integer>, Integer> maxFn = Collections::max`. `max` is a static method in the `Collections` class that takes one argument of type `List`. You can then call this like `maxFn.apply(Arrays.asList(1, 10, 3, 5))`. The above lambda expression is equivalent to `Function<List<Integer>, Integer> maxFn = (numbers) -> Collections.max(numbers);` lambda expression.

假设我们需要从一个数字列表中找出最大的一个数字，那我们可以像这样写一个方法参考`Function<List<Integer>, Integer> maxFn = Collections::max`。`max`是一`Collections`里的一个静态方法，它需要传入一个`List`类型的参数。接下来你就可以这样调用它，`maxFn.apply(Arrays.asList(1, 10, 3, 5))`。上面的lambda表达式等价于`Function<List<Integer>, Integer> maxFn = (numbers) -> Collections.max(numbers);`。

### Instance method references

This is used for method reference to an instance method for example `String::toUpperCase` calls `toUpperCase` method on a `String` reference. You can also use method reference with parameters for example `BiFunction<String, String, String> concatFn = String::concat`. The `concatFn` can be called as `concatFn.apply("shekhar", "gulati")`. The `String` `concat` method is called on a String object and  passed a parameter like `"shekhar".concat("gulati")`.

这样的情况用于一个实例方法，比如`String::toUpperCase`是在一个`String`参考上调用 `toUpperCase`方法。还可以使用带参的方法参考，比如：`BiFunction<String, String, String> concatFn = String::concat`。`concatFn`可以向这样调用:`concatFn.apply("shekhar", "gulati")`。`String``concat`方法在一个String对象上调用并且传递一个类似`"shekhar".concat("gulati")`的参数。

## Exercise >> Lambdify me

Let's look at the code shown below and apply what we have learnt so far.

下面通过一段代码，来应用所学到的。

```java
public class Exercise_Lambdas {

    public static void main(String[] args) {
        List<Task> tasks = getTasks();
        List<String> titles = taskTitles(tasks);
        for (String title : titles) {
            System.out.println(title);
        }
    }

    public static List<String> taskTitles(List<Task> tasks) {
        List<String> readingTitles = new ArrayList<>();
        for (Task task : tasks) {
            if (task.getType() == TaskType.READING) {
                readingTitles.add(task.getTitle());
            }
        }
        return readingTitles;
    }

}
```
The code shown above first fetches all the Tasks from a utility method `getTasks`. We are not interested in `getTasks` implementation. The `getTasks` could fetch tasks by accessing a web-service or database or in-memory. Once you have tasks, we filter all the reading tasks and extract the title field from the task. We add extracted title to a list and then finally return all the reading titles.

上面这段代码首先通过工具方法`getTasks`取得所有的Task,这里我们不去关心`getTasks`方法的具体实现，`getTasks`能够通过webservice或者数据库或者内存获取task。一旦得到了tasks,我们就过滤所有处于reading状态的task，并且从task中提取他们的标题，最后放回所有处于reading状态task的标题。

Let' start with the simplest refactor -- using foreach on a list with method reference.

下面我们简单的重构下--在一个list上使用foreach和方法参考。

```java
public class Exercise_Lambdas {

    public static void main(String[] args) {
        List<Task> tasks = getTasks();
        List<String> titles = taskTitles(tasks);
        titles.forEach(System.out::println);
    }

    public static List<String> taskTitles(List<Task> tasks) {
        List<String> readingTitles = new ArrayList<>();
        for (Task task : tasks) {
            if (task.getType() == TaskType.READING) {
                readingTitles.add(task.getTitle());
            }
        }
        return readingTitles;
    }

}
```

Using `Predicate<T>` to filter out tasks.

使用`Predicate<T>`来过滤task

```java
public class Exercise_Lambdas {

    public static void main(String[] args) {
        List<Task> tasks = getTasks();
        List<String> titles = taskTitles(tasks, task -> task.getType() == TaskType.READING);
        titles.forEach(System.out::println);
    }

    public static List<String> taskTitles(List<Task> tasks, Predicate<Task> filterTasks) {
        List<String> readingTitles = new ArrayList<>();
        for (Task task : tasks) {
            if (filterTasks.test(task)) {
                readingTitles.add(task.getTitle());
            }
        }
        return readingTitles;
    }

}
```

Using `Function<T,R>` for extracting out title from the Task.

使用`Function<T,R>`来将task中的title提取出来。

```java
public class Exercise_Lambdas {

    public static void main(String[] args) {
        List<Task> tasks = getTasks();
        List<String> titles = taskTitles(tasks, task -> task.getType() == TaskType.READING, task -> task.getTitle());
        titles.forEach(System.out::println);
    }

    public static <R> List<R> taskTitles(List<Task> tasks, Predicate<Task> filterTasks, Function<Task, R> extractor) {
        List<R> readingTitles = new ArrayList<>();
        for (Task task : tasks) {
            if (filterTasks.test(task)) {
                readingTitles.add(extractor.apply(task));
            }
        }
        return readingTitles;
    }
}
```

Using method reference for extractor

把方法参考当着提取器。



```java
public static void main(String[] args) {
    List<Task> tasks = getTasks();
    List<String> titles = filterAndExtract(tasks, task -> task.getType() == TaskType.READING, Task::getTitle);
    titles.forEach(System.out::println);
    List<LocalDate> createdOnDates = filterAndExtract(tasks, task -> task.getType() == TaskType.READING, Task::getCreatedOn);
    createdOnDates.forEach(System.out::println);
    List<Task> filteredTasks = filterAndExtract(tasks, task -> task.getType() == TaskType.READING, Function.identity());
    filteredTasks.forEach(System.out::println);
}
```

We can also write our own **Functional Interface** that clearly tells the reader intent of the developer. We can create an interface `TaskExtractor` that extends `Function` interface. The input type of interface is fixed to `Task` and output type depend on the implementing lambda. This way developer will only have to worry about the result type as input type will always remain Task.

我们也可以自己编写`函数式接口`，这样可以清晰的把开发者的意图传递给读者。我们可以写一个继承自`Function`接口的`TaskExtractor`接口。这个接口的输入类型是固定的`Task`类型，输出类型由实现的lambda表达式来决定。这样开发者就只需要关注输出结果的类型，因为输入的类型只会是Task。

```java
public class Exercise_Lambdas {

    public static void main(String[] args) {
        List<Task> tasks = getTasks();
        List<Task> filteredTasks = filterAndExtract(tasks, task -> task.getType() == TaskType.READING, TaskExtractor.identityOp());
        filteredTasks.forEach(System.out::println);
    }

    public static <R> List<R> filterAndExtract(List<Task> tasks, Predicate<Task> filterTasks, TaskExtractor<R> extractor) {
        List<R> readingTitles = new ArrayList<>();
        for (Task task : tasks) {
            if (filterTasks.test(task)) {
                readingTitles.add(extractor.apply(task));
            }
        }
        return readingTitles;
    }

}


interface TaskExtractor<R> extends Function<Task, R> {

    static TaskExtractor<Task> identityOp() {
        return t -> t;
    }
}
```

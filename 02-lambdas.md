Lambda表达式
-----

> - 原文链接： [Lambdas](https://github.com/shekhargulati/java8-the-missing-tutorial/blob/master/02-lambdas.md)  
- 原文作者： [shekhargulati](https://github.com/shekhargulati)  
- 译者： leege100  
- 状态： 完成

lambda表达式是java8中最重要的特性之一，它让代码变得简洁并且允许你传递行为。曾几何时，Java总是因为代码冗长和缺少函数式编程的能力而饱受批评。随着函数式编程变得越来越受欢迎，Java也被迫开始拥抱函数式编程。否则，Java会被大家逐步抛弃。

Java8是使得这个世界上最流行的编程语言采用函数式编程的一次大的跨越。一门编程语言要支持函数式编程，就必须把函数作为其一等公民。在Java8之前，只能通过匿名内部类来写出函数式编程的代码。而随着lambda表达式的引入，函数变成了一等公民，并且可以像其他变量一样传递。

lambda表达式允许开发者定义一个不局限于定界符的匿名函数，你可以像使用编程语言的其他程序结构一样来使用它，比如变量申明。如果一门编程语言需要支持高阶函数，lambda表达式就派上用场了。高阶函数是指把函数作为参数或者返回结果是一个函数那些函数。

> 这个章节的代码如下[ch02 package](https://github.com/shekhargulati/java8-the-missing-tutorial/tree/master/code/src/main/java/com/shekhargulati/java8_tutorial/ch02).

随着Java8中lambda表达式的引入，Java也支持高阶函数。接下来让我们来分析这个经典的lambda表达式示例--Java中Collections类的一个sort函数。sort函数有两种调用方式，一种需要一个List作为参数，另一种需要一个List参数和一个Comparator。第二种sort函数是一个接收lambda表达式的高阶函数的实例，如下：

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

lambda表达式源自于`λ演算`.[λ演算](https://en.wikipedia.org/wiki/Lambda_calculus)起源于用函数式来制定表达式计算概念的研究[Alonzo Church](https://en.wikipedia.org/wiki/Alonzo_Church)。`λ演算`是图灵完整的。图灵完整意味着你可以用lambda表达式来表达任何数学算式。

`λ演算`后来成为了函数式编程语言强有力的理论基础。诸如 Hashkell、Lisp等著名的函数式编程语言都是基于`λ演算`.高阶函数的概念就来自于`λ演算`。

`λ演算`中最主要的概念就是表达式，一个表达式可以用如下形式来表示：

```
<expression> := <variable> | <function>| <application>
```

* **variable** -- 一个variable就是一个类似用x、y、z来代表1、2、n等数值或者lambda函数式的占位符。

* **function** -- 它是一个匿名函数定义，需要一个变量，并且生成另一个lambda表达式。例如，`λx.x*x`是一个求平方的函数。

* **application** -- 把一个函数当成一个参数的行为。假设你想求10的平方，那么用λ演算的方式的话你需要写一个求平方的函数`λx.x*x`并把10应用到这个函数中去，这个函数程序就会返回`(λx.x*x) 10 = 10*10 = 100`。但是你不仅可以求10的平方，你可以把一个函数传给另一个函数然后生成另一个函数。比如，`(λx.x*x) (λz.z+10)` 会生成这样一个新的函数 `λz.(z+10)*(z+10)`。现在，你可以用这个函数来生成一个数加上10的平方。这就是一个高阶函数的实例。

现在，你已经理解了`λ演算`和它对函数式编程语言的影响。下面我们继续学习它们在java8中的实现。

## 在java8之前传递行为

Java8之前，传递行为的唯一方法就是通过匿名内部类。假设你在用户完成注册后，需要在另外一个线程中发送一封邮件。在Java8之前，可以通过如下方式：

```java
sendEmail(new Runnable() {
            @Override
            public void run() {
                System.out.println("Sending email...");
            }
        });
```
sendEmail方法定义如下：

```java
public static void sendEmail(Runnable runnable)
```

上面的代码的问题不仅仅在于我们需要把行为封装进去，比如`run`方法在一个对象里面；更糟糕的是，它容易混淆开发者真正的意图，比如把行为传递给`sendEmail`函数。如果你用过一些类似Guava的库,那么你就会切身感受到写匿名内部类的痛苦。下面是一个简单的例子，过滤所有标题中包含**lambda**字符串的task。


```java
Iterable<Task> lambdaTasks = Iterables.filter(tasks, new Predicate<Task>() {
            @Override
            public boolean apply(Task task) {
                return input.getTitle().contains("lambda");
            }
});
```

使用Java8的Stream API,开发者不用太第三方库就可以写出上面的代码，我们将在下一章[chapter 3](./03-streams.md)讲述streams相关的知识。所以，继续往下阅读！

## Java 8 Lambda表达式

在Java8中，我们可以用lambda表达式写出如下代码，这段代码和上面提到的是同一个例子。

```java
sendEmail(() -> System.out.println("Sending email..."));
```

上面的代码非常简洁，并且能够清晰的传递编码者的意图。`()`用来表示无参函数，比如`Runnable`接口的中`run`方法不含任何参数，直接就可以用`()`来代替。`->`是将参数和函数体分开的lambda操作符，上例中，`->`后面是打印`Sending email`的相关代码。

下面再次通过Collections.sort这个例子来了解带参数的lambda表达式如何使用。要将names列表中的name按照字符串的长度排序,需要传递一个`Comparator`给sort函数。`Comparator`的定义如下

```java
Comparator<String> comparator = (first, second) -> first.length() - second.length();
```

上面写的lambda表达式相当于Comparator接口中的compare方法。`compare`方法的定义如下:

```java
int compare(T o1, T o2);
```

`T`是传递给`Comparator`接口的参数类型，在本例中names列表是由`String`组成，所以`T`代表的是`String`。

在lambda表达式中，我们不需要明确指出参数类型，`javac`编译器会通过上下文自动推断参数的类型信息。由于我们是在对一个由`String`类型组成的List进行排序并且`compare`方法仅仅用一个T类型，所以Java编译器自动推断出两个参数都是`String`类型。根据上下文推断类型的行为称为**类型推断**。Java8提升了Java中已经存在的类型推断系统，使得对lambda表达式的支持变得更加强大。`javac`会寻找紧邻lambda表达式的一些信息通过这些信息来推断出参数的正确类型。

> 在大多数情况下，`javac`会根据上下文自动推断类型。假设因为丢失了上下文信息或者上下文信息不完整而导致无法推断出类型，代码就不会编译通过。例如，下面的代码中我们将`String`类型从`Comparator`中移除，代码就会编译失败。


```java
Comparator comparator = (first, second) -> first.length() - second.length(); // compilation error - Cannot resolve method 'length()'
```

## Lambda表达式在Java8中的运行机制

你可能已经发现lambda表达式的类型是一些类似上例中Comparator的接口。但并不是每个接口都可以使用lambda表达式，***只有那些仅仅包含一个非实例化抽象方法的接口才能使用lambda表达式***。这样的接口被称着**函数式接口**并且它们能够被`@FunctionalInterface`注解注释。Runnable接口就是函数式接口的一个例子。

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

`@FunctionalInterface`注解不是必须的，但是它能够让工具知道这一个接口是一个函数式接口并表现有意义的行为。例如，如果你试着这编译一个用`@FunctionalInterface`注释自己并且含有多个抽象方法的接口，编译就会报出这样一个错***Multiple non-overriding abstract methods found***。同样的，如果你给一个不含有任何方法的接口添加`@FunctionalInterface`注解，会得到如下错误信息,***No target method found***.

下面来回答一个你大脑里一个非常重大的疑问，***Java8的lambda表达式是否只是一个匿名内部类的语法糖或者函数式接口是如何被转换成字节码的？***

答案是**NO**,Java8不采用匿名内部类的原因主要有两点：

1. **性能影响**: 如果lambda表达式是采用匿名内部类实现的，那么每一个lambda表达式都会在磁盘上生成一个class文件。当JVM启动时，这些class文件会被加载进来，因为所有的class文件都需要在启动时加载并且在使用前确认，从而会导致JVM的启动变慢。

2. **向后的扩展性**: 如果Java8的设计者从一开始就采用匿名内部类的方式，那么这将限制lambda表达式未来的使发展范围。

### 使用动态启用

Java8的设计者决定采用在Java7中新增的`动态启用`来延迟在运行时的加载策略。当`javac`编译代码时，它会捕获代码中的lambda表达式并且生成一个`动态启用`的调用地址(称为lambda工厂）。当`动态启用`被调用时，就会向lambda表达式发生转换的地方返回一个函数式接口的实例。比如，在Collections.sort这个例子中，它的字节码如下：

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

上面代码的关键部分位于第23行`23: invokedynamic #7,  0              // InvokeDynamic #0:compare:()Ljava/util/Comparator;`这里创建了一个`动态启用`的调用。

接下来是将lambda表达式的内容转换到一个将会通过`动态启用`来调用的方法中。在这一步中，JVM实现者有自由选择策略的权利。

这里我仅粗略的概括一下，具体的内部标准见这里 http://cr.openjdk.java.net/~briangoetz/lambda/lambda-translation.html.

## 匿名类 vs lambda表达式

下面我们对匿名类和lambda表达式做一个对比，以此来区分它们的不同。

1. 在匿名类中，`this` 指代的是匿名类本身；而在lambda表达式中，`this`指代的是lambda表达式所在的这个类。

2. 在匿名类中可以shadow包含此匿名类的变量，而在lambda表达式中就会报编译错误。

3. lambda表达式的类型是由上下文决定的，而匿名类中必须在创建实例的时候明确指定。

## 我需要自己去写函数式接口吗?

Java8默认带有许多可以直接在代码中使用的函数式接口。它们位于`java.util.function`包中，下面简单介绍几个:

### java.util.function.Predicate<T>

此函数式接口是用来定义对一些条件的检查，比如一个predicate。Predicate接口有一个叫`test`的方法，它需要一个`T`类型的值，返回值为布尔类型。例如，在一个`names`列表中找出所有以**s**开头的name就可以像如下代码这样使用predicate。

```java
Predicate<String> namesStartingWithS = name -> name.startsWith("s");
```

### java.util.function.Consumer<T>

这个函数式接口用于表现那些不需要产生任何输出的行为。Consumer接口中有一个叫做`accept`的方法，它需要一个`T`类型的参数并且没有返回值。例如，用指定信息发送一封邮件：

```java
Consumer<String> messageConsumer = message -> System.out.println(message);
```

### java.util.function.Function<T,R>

这个函数式接口需要一个值并返回一个结果。例如，如果需要将所有`names`列表中的name转换为大写，可以像下面这样写一个Function:

```java
Function<String, String> toUpperCase = name -> name.toUpperCase();
```

### java.util.function.Supplier<T>

这个函数式接口不需要传值，但是会返回一个值。它可以像下面这样，用来生成唯一的标识符

```java
Supplier<String> uuidGenerator= () -> UUID.randomUUID().toString();
```

在接下来的章节中，我们会学习更多的函数式接口。

## Method references

有时候，你需要为一个特定方法创建lambda表达式，比如`Function<String, Integer> strToLength = str -> str.length();`，这个表达式仅仅在`String`对象上调用`length()`方法。可以这样来简化它，`Function<String, Integer> strToLength = String::length;`。仅调用一个方法的lambda表达式，可以用缩写符号来表示。在`String::length`中，`String`是目标引用，`::`是定界符，`length`是目标引用要调用的方法。静态方法和实例方法都可以使用方法引用。

### Static method references

假设我们需要从一个数字列表中找出最大的一个数字，那我们可以像这样写一个方法引用`Function<List<Integer>, Integer> maxFn = Collections::max`。`max`是一`Collections`里的一个静态方法，它需要传入一个`List`类型的参数。接下来你就可以这样调用它，`maxFn.apply(Arrays.asList(1, 10, 3, 5))`。上面的lambda表达式等价于`Function<List<Integer>, Integer> maxFn = (numbers) -> Collections.max(numbers);`。

### Instance method references

在这样的情况下，方法引用用于一个实例方法，比如`String::toUpperCase`是在一个`String`引用上调用 `toUpperCase`方法。还可以使用带参数的方法引用，比如：`BiFunction<String, String, String> concatFn = String::concat`。`concatFn`可以这样调用:`concatFn.apply("shekhar", "gulati")`。`String``concat`方法在一个String对象上调用并且传递一个类似`"shekhar".concat("gulati")`的参数。

## Exercise >> Lambdify me

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

上面这段代码首先通过工具方法`getTasks`取得所有的Task,这里我们不去关心`getTasks`方法的具体实现，`getTasks`能够通过webservice或者数据库或者内存获取task。一旦得到了tasks,我们就过滤所有处于reading状态的task，并且从task中提取他们的标题，最后返回所有处于reading状态task的标题。

下面我们简单的重构下--在一个list上使用foreach和方法引用。

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

使用`Predicate<T>`来过滤tasks

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

把方法引用当着提取器来使用。



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

我们也可以自己编写`函数式接口`，这样可以清晰的把开发者的意图传递给读者。我们可以写一个继承自`Function`接口的`TaskExtractor`接口。这个接口的输入类型是固定的`Task`类型，输出类型由实现的lambda表达式来决定。这样开发者就只需要关注输出结果的类型，因为输入的类型永远都是Task。

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

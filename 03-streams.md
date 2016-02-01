Streams
------

在[第二章中](./02-lambdas.md), we learnt how lambdas can help us write clean concise code by allowing us to pass behavior without the need to create a class. Lambdas is a very simple language construct that helps developer express their intent on the fly by using functional interfaces. The real power of lambdas can be experienced when an API is designed keeping lambdas in mind i.e. a fluent API that makes use of Functional interfaces (we discussed them in [lambdas chapter](./02-lambdas.md#do-i-need-to-write-my-own-functional-interfaces)).

在[第二章中](./02-lambdas.md)，我们学习到了lambda表达式允许我们在不创建新类的情况下传递行为，从而帮助我们写出干净简洁的代码。lambda表达式是一种简单的语法接口，它通过使用函数式接口来帮助开发者轻松传递意图。当对API采用lambda表达式的思想设计时，lambda表达式的强大就会得到体现，比如我们在第二节讨论的使用函数式接口编程的API[lambdas chapter](./02-lambdas.md#do-i-need-to-write-my-own-functional-interfaces)).。

One such API that makes heavy use of lambdas is Stream API introduced in JDK 8. Streams provide a higher level abstraction to express computations on Java collections in a declarative way similar to how SQL helps you declaratively query data in the database. Declarative means developers write what they want to do rather than how it should be done. In this chapter, we will discuss why need a new data processing API, difference between Collection and Stream, and how to use Stream API in your applications.

Stream是java8中引入的一个重度使用lambda表达式的API。Stream使用一种类似通过SQL语句从数据库查询数据的的简明方式来提供一种对Java集合的运行表达的高阶抽象。简明意味着开发者在写代码时只需关注他们想要的结果而无需关注具体的实现方式。这一章节中，我们将介绍为什么我们需要一种新的数据处理API、Collection和Stream的不同之处、以及在我们的程序中怎样去应用StreamAPI。

> 本节的代码见 [ch03 package](https://github.com/shekhargulati/java8-the-missing-tutorial/tree/master/code/src/main/java/com/shekhargulati/java8_tutorial/ch03).


## 为什么我们需要一种新的数据处理抽象概念?

在我看来，主要有两点：

1. Collection API 不能提供更高阶的结构来查询数据，因而开发者不得不为大多数的琐碎任务写一大堆样板代码。

2、语法对集合数据的并行处理有一定的限制。对Java语言并发结构的使用、高效处理诗句以及高效的并发处理数据都需要由程序员自己来支配。


2. It has limited language support to process Collection data in parallel. It is left to the developer to use Java language concurrency constructs and process data effectively and efficiently in parallel.

##Java 8之前的数据处理

阅读下面这一段代码，猜猜看它是拿来做什么的。

```java
public class Example1_Java7 {

    public static void main(String[] args) {
        List<Task> tasks = getTasks();

        List<Task> readingTasks = new ArrayList<>();
        for (Task task : tasks) {
            if (task.getType() == TaskType.READING) {
                readingTasks.add(task);
            }
        }
        Collections.sort(readingTasks, new Comparator<Task>() {
            @Override
            public int compare(Task t1, Task t2) {
                return t1.getTitle().length() - t2.getTitle().length();
            }
        });
        for (Task readingTask : readingTasks) {
            System.out.println(readingTask.getTitle());
        }
    }
}
```

上面这段代码是用来按照字符串长度的顺序打印所有正在阅读task的title。所有Java开发者每天都会写这样的代码。为了写出这样一个简单的程序，我们不得不写下15行Java代码。然而上面这段代码最大的问题不在于其代码长度，而在于不能清晰传达开发者的意图：过滤出阅读的task、通过长度排序然后转化为String类型的List。

The code shown above prints all the reading task titles sorted by their title length. All Java developers write this kind of code everyday. To write such a simple program we had to write 15 lines of Java code. The bigger problem with the above mentioned code is not the number of lines a developer has to write but, it misses the developer's intent i.e. filtering reading tasks, sorting by title length, and transforming to List of String.

##Java8中的数据处理

在Java8中，可以像下面这段代码这样使用java8中的Stream API来实现与上面代码同等的效果。

```java
public class Example1_Stream {

    public static void main(String[] args) {
        List<Task> tasks = getTasks();

        List<String> readingTasks = tasks.stream()
                .filter(task -> task.getType() == TaskType.READING)
                .sorted((t1, t2) -> t1.getTitle().length() - t2.getTitle().length())
                .map(Task::getTitle)
                .collect(Collectors.toList());

        readingTasks.forEach(System.out::println);
    }
}
```


上面这段代码中，形成了一个由多个stream操作组成的管道。

* **stream()** - 通过在类似上面说的`tasks` `List<Task>`的集合源上调用 `stream()`方法来创建一个stream的管道。

* **filter(Predicate<T>)** - This operation extract elements in the stream matching the condition defined by the predicate. Once you have a stream you can call zero or more intermediate operations on it. The lambda expression `task -> task.getType() == TaskType.READING` defines a predicate to filter all reading tasks. The type of lambda expression is `java.util.function.Predicate<Task>`.这个操作用来提取stream中匹配predicate定义规则的元素。如果你有一个stream，你可以在它上面调用零次或者多次间断的操作。lambda表达式`task -> task.getType() == TaskType.READING`定义了一个用来过滤出所有正在阅读task的断言。

* **sorted(Comparator<T>)**: This operation returns a stream consisting of all the stream elements sorted by the Comparator defined by lambda expression i.e.  in the example shown above.此操作返回一个通过lambda表达式定义的comparator排序后的所有stream元素组成的stream,在上面代码中排序的表达式是(t1, t2) -> t1.getTitle().length() - t2.getTitle().length().

* **map(Function<T,R>)**: This operation returns a stream after applying the Function<T,R> on each element of this stream.此操作返回一个对每个stream元素应用Function<T,R>后的stream。

* **collect(toList())** - This operation collects result of the operations performed on the Stream to a List.此操作把上面对stream进行各种操作后的结果生成一个list。

### Why Java 8 code is better?

In my opinion Java 8 code is better because of following reasons:
在我看来，Java8的代码更好主要有以下几点原因：

1. Java 8 code clearly reflect developer intent of filtering, sorting, etc.Java8代码能够清晰的表现开发者对数据过滤、排序等操作的意图。 

2. Developers express what they want to do rather than how they want do it by using a higher level abstraction in the form of Stream API.通过使用Stream API格式的更高抽象，开发者表达他们所想要的结果的而不是怎么实现这些效果。

3. Stream API provides a unified language for data processing. Now developers will have the common vocabulary when they are talking about data processing. When two developers talk about `filter` function you can be sure that they both are applying a data filtering operation.Stream API为数据处理提供一种统一的语言，现在开发者在谈论数据处理时就有共同的词汇，当两个开发者在讨论`filter`函数时，他们都会明白他们都是在进行一个数据过滤操作。

4. No boilerplate code required to express data processing. Developers now don't have to write explicit for loops or create temporary collections to store data. All is taken care by the Stream API itself.不需要表达数据处理的样板代码，现在开发者不需要写详细的loop代码或者临时集合来储存数据，Stream API会处理好所有的这一切。

5. Streams does not modify your underlying collection. They are non mutating.Stream不会修改原本的集合，它是非 交换的。

## Stream是什么

Stream is an abstract view over some data. For example, Stream can be a view over a list or lines in a file or any other sequence of elements. Stream API provides aggregate operations that can be performed sequentially or in parallel. ***One thing that developers should keep in mind is that Stream is an higher level abstraction not a data structure. Stream does not store your data.*** Streams are **lazy** by nature and they are only computed when accessed. This allows us to produce infinite streams of data. In Java 8, you can very easily write a Stream that will produce infinite unique identifiers as shown below.

Stream是一个在数据上的抽象视图。比如，Stream可以是一个列表或者文件中的几行或者其他任意的一个元素序列的视图。Stream API提供可以顺序表现或者并行表现的操作总和。***开发者需要明白的是，Stream是一种高阶的抽象概念，而不是一种数据结构***Stream天生就是**懒的**，只有在使用时才会执行计算。它允许我们产生无限的数据流(stream of data)。在Java8中，你可以非常轻松的写出如下一个生成无限特定标识符的代码：

```
public static void main(String[] args) {
    Stream<String> uuidStream = Stream.generate(() -> UUID.randomUUID().toString());
}
```

There are various static factory methods like `of`, `generate`, and `iterate` in the Stream interface that one can use to create Stream instances. The `generate` method shown above takes a `Supplier`. `Supplier` is a functional interface to describe a function that does not take any input and produce a value. We passed the `generate` method a supplier that when invoked generates a unique identifier.在Stream接口中有诸如`of`、`generate`、`iterate`等多种静态工厂方法可以用来创建stream实例。上面提到的`generate`方法带有一个`Supplier`，`Supplier`是一个用来描述一个不需要任何输入且会产生一个值的函数的函数式接口。

```java
Supplier<String> uuids = () -> UUID.randomUUID().toString()
```

运行上面这段代码，什么都不会发生，因为Stream是懒加载的，而且直到他们被使用时才会执行。如果我们改成如下这段代码，我们就会在控制台看到打印出来的UUID。这段程序永远都不会终止。
If you run this program nothing will happen as Streams are lazy and until they are accessed nothing will be computed. If we update the program to the one shown below we will see UUID printing to the console. The program will never terminate.

```java
public static void main(String[] args) {
    Stream<String> uuidStream = Stream.generate(() -> UUID.randomUUID().toString());
    uuidStream.forEach(System.out::println);
}
```
Java8运行开发者通过在一个Collection上调用`stream`方法来创建Stream。Stream支持数据处理操作，从而开发者可以同高阶的数据处理结构来表达计算。

Java 8 allows you to create Stream from a Collection by calling the `stream` method on it. Stream supports data processing operations so that developers can express computations using higher level data processing constructs.

## Collection vs Stream

下面这张表阐述了Collection和Stream的不同之处

The table shown below explains the difference between a Collection and a Stream.

![Collection vs Stream](https://whyjava.files.wordpress.com/2015/10/collection_vs_stream.png)

下面我们来探讨内部迭代和外部迭代的区别，以及懒赋值
Let's discuss External iteration vs internal iteration and Lazy evaluation in detail.

### External iteration vs internal iteration外部迭代vs 内部迭代

上面谈到的java8Stream API代码和Collection API代码的区别在于谁控制迭代，是迭代器自己还是开发者。Stream API仅仅提供他们想要应用的操作，然后迭代器把这些操作应用到潜在集合的每个元素中去。当对潜在的Collection进行的迭代操作是由迭代器本身控制时，就叫着`内部迭代`；反之，当迭代的对象是由开发者控制时，就叫着`外部迭代`。Collection API中`for-each`结构的使用就是一个`外部迭代`的例子。
The difference between Java 8 Stream API code and Collection API code shown above is who controls the iteration, the iterator or the client that uses the iterator. Users of the Stream API just provide the operations they want to apply, and iterator applies those operations to every element in the underlying Collection. When iterating over the underlying collection is handled by the iterator itself, it is called **internal iteration**. On the other hand, when iteration is handled by the client it is called **external iteration**. The use of `for-each` construct in the Collection API code is an example of **external iteration**.

有人会辩解到，在Collection API中我们也不需要对潜在的迭代器进行操作，因为`for-each`结构已经替我们处理得很好了，但是`for-each`结构其实不过是一种iterator API的语法糖罢了。`for-each`尽管很简单，但是它有一些缺点 --1)只有固有顺序 2)容易写出急切的代码 3)难以并行。
Some might argue that in the Collection API code we didn't have to work with the underlying iterator as the `for-each` construct took care of that but, `for-each` is nothing more than syntactic sugar over manual iteration using the iterator API. The `for-each` construct although very simple has few disadvantages -- 1) It is inherently sequential 2) It leads to imperative code 3) It is difficult to parallelize.

### Lazy evaluation懒加载

stream表达式在被终极操作方法调用之前，不会被赋值计算。Stream API中的大多数操作会返回一个Stream。这些操作不会表现任何执行，它们只会构建这个管道。看着下面这段代码，预测一下它的输出会是什么。

Streams are not evaluated until a terminal operation is called on them. Most of the operations in the Stream  API return a Stream. These operations does not perform any execution they just builds the pipeline. Let's look at the code shown below and try to predict its output.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
Stream<Integer> stream = numbers.stream().map(n -> n / 0).filter(n -> n % 2 == 0);
```

上面这段代码中，我们将stream元素中的数字除以0，我们也许会认为这段代码在运行时会抛出`ArithmeticExceptin`异常，而事实上不会。因为stream表达式只有在有终极操作被调用时才会被赋值计算。如果我们为上面的stream加上终极操作，stream就会被执行并抛出异常。

In the code shown above, we are dividing elements in numbers stream by 0. We might expect that this code will throw `ArithmeticException` when the code is executed. But, when you run this code no exception will be thrown. This is because streams are not evaluated until a terminal operation is called on the stream. If you add terminal operation to the stream pipeline, then stream is executed, and exception is thrown.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
Stream<Integer> stream = numbers.stream().map(n -> n / 0).filter(n -> n % 2 == 0);
stream.collect(toList());
```

我们会得到如下的stack trace:
You will get stack trace as shown below.

```
Exception in thread "main" java.lang.ArithmeticException: / by zero
	at org._7dayswithx.java8.day2.EagerEvaluationExample.lambda$main$0(EagerEvaluationExample.java:13)
	at org._7dayswithx.java8.day2.EagerEvaluationExample$$Lambda$1/1915318863.apply(Unknown Source)
	at java.util.stream.ReferencePipeline$3$1.accept(ReferencePipeline.java:193)
	at java.util.Spliterators$ArraySpliterator.forEachRemaining(Spliterators.java:948)
	at java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:512)
	at java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:502)
	at java.util.stream.ReduceOps$ReduceOp.evaluateSequential(ReduceOps.java:708)
	at java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234)
	at java.util.stream.ReferencePipeline.collect(ReferencePipeline.java:499)
```

## 使用Stream API

Stream API提供了一大堆开发者可以用来从集合中查询数据的操作，这些操作分为两种--过渡操作和终极操作。

Stream API provides a lot of operations that developers can use to query data from collections. Stream operations fall into either of the two categories -- intermediate operation or terminal operation.

**过渡操作**是那些从已经存在的stream上产生另一个新stream的函数，比如`filter`,`map`, `sorted`, 等。

**Intermediate operations** are functions that produce another stream from the existing stream like `filter`, `map`, `sorted`, etc.

**终极操作**是那些从stream上产生一个非stream结果的函数，如`collect(toList())` , `forEach`, `count`等。

**Terminal operations** are functions that produce a non-stream result from the Stream like `collect(toList())` , `forEach`, `count` etc.

过渡操作允许开发者构建只有调用终极操作时才会执行的管道。下面是Stream API的部分函数列表：

Intermediate operations allows you to build the pipeline which gets executed when you call the terminal operation. Below is the list of functions that are part of the Stream API.

<a href="https://whyjava.files.wordpress.com/2015/07/stream-api.png"><img class="aligncenter size-full wp-image-2983" src="https://whyjava.files.wordpress.com/2015/07/stream-api.png" alt="stream-api" height="450" /></a>

### Example domain

在本教程中，我们将会用Task来解释这个概念。例子中，有一个叫Task的类，它是一个由用户表现的类，其定义如下：

Throughout this tutorial we will use Task management domain to explain the concepts. Our example domain has one class called Task -- a task to be performed by user. The class is shown below.

```java
import java.time.LocalDate;
import java.util.*;

public class Task {
    private final String id;
    private final String title;
    private final TaskType type;
    private final LocalDate createdOn;
    private boolean done = false;
    private Set<String> tags = new HashSet<>();
    private LocalDate dueOn;

    // removed constructor, getter, and setter for brevity
}
```
例子中的数据集如下，在整个Stream API例子中我们都会使用到它。
The sample dataset is given below. We will use this list throughout our Stream API examples.

```java
Task task1 = new Task("Read Version Control with Git book", TaskType.READING, LocalDate.of(2015, Month.JULY, 1)).addTag("git").addTag("reading").addTag("books");

Task task2 = new Task("Read Java 8 Lambdas book", TaskType.READING, LocalDate.of(2015, Month.JULY, 2)).addTag("java8").addTag("reading").addTag("books");

Task task3 = new Task("Write a mobile application to store my tasks", TaskType.CODING, LocalDate.of(2015, Month.JULY, 3)).addTag("coding").addTag("mobile");

Task task4 = new Task("Write a blog on Java 8 Streams", TaskType.WRITING, LocalDate.of(2015, Month.JULY, 4)).addTag("blogging").addTag("writing").addTag("streams");

Task task5 = new Task("Read Domain Driven Design book", TaskType.READING, LocalDate.of(2015, Month.JULY, 5)).addTag("ddd").addTag("books").addTag("reading");

List<Task> tasks = Arrays.asList(task1, task2, task3, task4, task5);
```
这个章节我们暂不讨论Java8的Data Time API，这里我们就把它当着一个普通的关于日期的API。
> We will not discuss about Java 8 Date Time API in this chapter. For now, just think of as the fluent API to work with dates.

### Example 1: Find all reading task titles sorted by their creation date找出所有正在阅读任务的标题，并根据它们的创建时间排序。

第一个例子我们将要实现的是，从Task列表中找出所有正在阅读的任务的标题，并根据它们的创建时间排序。我们要做的操作如下：

The first example that we will discuss is to find all the reading task titles sorted by creation date. The operations that we need to perform are:

1. 过滤出所有TaskType为READING的Task。
2. Sort the filtered values tasks by `createdOn` field.将第一步中过滤出来的task数据按照它们的创建时间排序。
3. Get the value of title for each task.获取每个task的title
4. Collect the resulting titles in a List.将得到的这些title放进一个List中。

上面的四个操作步骤可以非常简单的翻译成下面这段代码：

The following four operations can be easily translated to the code as shown below.

```java
private static List<String> allReadingTasks(List<Task> tasks) {
        List<String> readingTaskTitles = tasks.stream().
                filter(task -> task.getType() == TaskType.READING).
                sorted((t1, t2) -> t1.getCreatedOn().compareTo(t2.getCreatedOn())).
                map(task -> task.getTitle()).
                collect(Collectors.toList());
        return readingTaskTitles;
}
```

在上面的代码中，我们使用了StreamAPI中的如下一些方法：

In the code shown above, we used following methods of the Stream API:


* **filter**:  Allows you to specify a predicate to exclude some elements from the underlying stream. The predicate **task -> task.getType() == TaskType.READING** selects all the tasks whose TaskType is READING.允许开发者定义一个断言来从潜在的stream中提取部门符合规则的元素。规则**task -> task.getType() == TaskType.READING**从stream中选取所有TaskType 为READING的元素。

* **sorted**: Allows you to specify a Comparator that will sort the stream. In this case, you sorted based on the creation date. The lambda expression **(t1, t2) -> t1.getCreatedOn().compareTo(t2.getCreatedOn())** provides implementation of the `compare` method of Comparator functional interface.允许开发者定义一个比较器来排序stream。上面的例子中，我们根据创建时间来排序，其中的lambda表达式**(t1, t2) -> t1.getCreatedOn().compareTo(t2.getCreatedOn())**就实现了函数式接口Comparator的`compare`函数。


* **map**: It takes a lambda that implements `Function<? super T, ? extends R>` which transforms one stream to another stream. The lambda expression **task -> task.getTitle()** transforms a task into a title.需要一个实现自`Function<? super T, ? extends R>`的lambda表达式，Function<? super T, ? extends R>接口能够将一个stream转换为另一个stream。lambda表达式**task -> task.getTitle()**将一个task转化为标题。

* **collect(toList())** It is a terminal operation that collects the resulting reading titles into a List.这是一个终极操作，它将所有正在阅读的镖旗装进一个列表中。

我们可以通过使用`Comparator`接口的`comparing`方法和方法参考来将上面的代码简化成入戏代码：

We can improve the above Java 8 code by using `comparing` method of `Comparator` interface and method references as shown below.

```java
public List<String> allReadingTasks(List<Task> tasks) {
    return tasks.stream().
            filter(task -> task.getType() == TaskType.READING).
            sorted(Comparator.comparing(Task::getCreatedOn)).
            map(Task::getTitle).
            collect(Collectors.toList());

}
```

> 从Java8开始，接口接口可以通过静态和默认方法的方式来实现方法，在[ch01](./01-default-static-interface-methods.md)已经介绍过了。
> From Java 8, interfaces can have method implementations in the form of static and default methods. This is covered in [ch01](./01-default-static-interface-methods.md).参考方法`Task::getCreatedOn`是用`Function<Task,LocalDate>`决定的。

上面代码中，我们使用了`Comparator`接口中的静态帮助方法`comparing`，此方法`函数`来提取`Comparator`值并返回一个用来比较的`Comparator`。
In the code shown above, we used a static helper method `comparing` available in the `Comparator` interface which accepts a `Function` that extracts a `Comparable` key, and returns a `Comparator` that compares by that key. The method reference `Task::getCreatedOn` resolves to `Function<Task, LocalDate>`.

使用函数组成，可以通过在Comparator上调用`reversed()`方法轻松的写出反转排序的如下代码。
Using function composition, we can very easily write code that reverses the sorting order by calling `reversed()` method on Comparator as shown below.

```java
public List<String> allReadingTasksSortedByCreatedOnDesc(List<Task> tasks) {
    return tasks.stream().
            filter(task -> task.getType() == TaskType.READING).
            sorted(Comparator.comparing(Task::getCreatedOn).reversed()).
            map(Task::getTitle).
            collect(Collectors.toList());
}
```

### Example 2: Find distinct tasks 去除重复的tasks

假设我们有一个有很多重复task的数据集，通过调用`distinct`方法可以非常轻松的去除stream中重复的元素，如下
Suppose, we have a dataset which contains duplicate tasks. We can very easily remove the duplicates and get only distinct elements by using the `distinct` method on the stream as shown below.

```java
public List<Task> allDistinctTasks(List<Task> tasks) {
    return tasks.stream().distinct().collect(Collectors.toList());
}
```

`distinct()`方法把一个stream转换成一个不含重复元素的stream,它通过对象的`equals`方法来判断对是否相等。根据对象相等方法的判定，如果两个对象相等，它的副本就会从结果stream中移除。
The `distinct()` method converts one stream into another without duplicates. It uses the Object's `equals` method for determining the object equality. According to Object's equal method contract, when two objects are equal, they are considered duplicates and will be removed from the resulting stream.

### Example 3: Find top 5 reading tasks sorted by creation date 根据创建时间找出前5个处于reading状态的task

`limit`方法可以用来把结果集限定在一个给定的数字。`limit`是一个短路操作，意味着它不会为了找出结果而去估值所有元素。
The `limit` function can be used to limit the result set to a given size. `limit` is a short circuiting operation which means it does not evaluate all the elements to find the result.

```java
public List<String> topN(List<Task> tasks, int n){
    return tasks.stream().
            filter(task -> task.getType() == TaskType.READING).
            sorted(comparing(Task::getCreatedOn)).
            map(Task::getTitle).
            limit(n).
            collect(toList());
}
```

可以同时使用`skip`方法和`limit`方法来创建某一页
You can use `limit` along with `skip` method to create pagination as shown below.

```java
// page starts from 0. So to view a second page `page` will be 1 and n will be 5.
//page从0开始，所以要查看第二页的话,`page`应该为1，n应该为5
List<String> readingTaskTitles = tasks.stream().
                filter(task -> task.getType() == TaskType.READING).
                sorted(comparing(Task::getCreatedOn).reversed()).
                map(Task::getTitle).
                skip(page * n).
                limit(n).
                collect(toList());
```

### Example 4: Count all reading tasks统计状态为reading的task的数量

要得到所有正处于reading的任务的数量，我们可以在stream中使用`count`方法来获得，这个方法是一个终极方法。
To get the count of all the reading tasks, we can use `count` method on the stream. This method is a terminal operation.

```java
public long countAllReadingTasks(List<Task> tasks) {
    return tasks.stream().
            filter(task -> task.getType() == TaskType.READING).
            count();
}
```

### Example 5: Find all unique tags from all tasks从所有task中找出所有不重复的标签

要找出不重复的标签，我们需要下面几个步骤
To find all the distinct tags we have to perform following operations:

1. Extract tags for each task.从每个task中提取标签
2. Collect all the tags into one stream.把所有的标签放到一个stream中
3. Remove the duplicates.删除重复的标签
4. Finally collect the result into a list.把结果放进一个列表中

第一步和第二步可以通过在`stream`上调用`flatMap`来得到。`flatMap`操作可以把通过调用`task.getTags().stream`得到的各个stream合成一个stream。一旦我们把所有的tag放到一个stream中后，我们就可以通过调用`distinct`方法来得到非重复的tag。
The first and second operations can be performed by using the `flatMap` operation on the `tasks` stream. The `flatMap` operation flattens the streams returned by each invocation of `tasks.getTags().stream()` into one stream. Once we have all the tags in one stream, we just used `distinct` method on it to get all unique tags.

```java
private static List<String> allDistinctTags(List<Task> tasks) {
        return tasks.stream().flatMap(task -> task.getTags().stream()).distinct().collect(toList());
}
```

### Example 6: Check if all reading tasks have tag `books` 检查是否所有处于readeing状态的task都有`book`标签

Stream API有一些可以用来判断元素中是否含有给定属性的方法，`allMatch`,`anyMatch`,`noneMatch`,`findFirst`,`findAny`。要判断是否所有状态为reading的task的title中都包含`books`标签，可以用如下代码来实现:
Stream API has methods that allows the users to check if elements in the dataset match a given property. These methods are `allMatch`, `anyMatch`, `noneMatch`, `findFirst`, and `findAny`. To check if all reading titles have a tag with name `books` we can write code as shown below.

```java
public boolean isAllReadingTasksWithTagBooks(List<Task> tasks) {
    return tasks.stream().
            filter(task -> task.getType() == TaskType.READING).
            allMatch(task -> task.getTags().contains("books"));
}
```

要判断所有reading的task中是否有一个task包含`java8`标签，可以通过`anyMatch`来试下，代码如下：
To check whether any reading task has a `java8` tag, then we can use `anyMatch` operation as shown below.

```java
public boolean isAnyReadingTasksWithTagJava8(List<Task> tasks) {
    return tasks.stream().
            filter(task -> task.getType() == TaskType.READING).
            anyMatch(task -> task.getTags().contains("java8"));
}
```

### Example 7: Creating a summary of all titles创建一个所有title的总览

当你想要创建一个所有title的总览时就可以使用`reduce`操作，`reduce`能够把stream减小成一个值。`reduce`函数接受一个可以用来连接stream中所有元素的lambda表达式。
Suppose, you want to create a summary of all the titles then you can use `reduce` operation, which reduces the stream to a value. The `reduce` function takes a lambda which joins elements of the stream.

```java
public String joinAllTaskTitles(List<Task> tasks) {
    return tasks.stream().
            map(Task::getTitle).
            reduce((first, second) -> first + " *** " + second).
            get();
}
```

### Example 8: Working with primitive Streams 对基本类型的stream的操作

除了基于对象的一般stream，Java8对诸如int,long,double等基本类型也提供了特定的stream。下面一起来看一些基本类型的stream的例子。
Apart from the generic stream that works over objects, Java 8 also provides specific streams that work over primitive types like int, long, and double. Let's look at few examples of primitive streams.

要创建一个值的域，可以调用`range`方法。`range`方法创建一个值为0到9的stream,不包含10。
To create a range of values, we can use `range` method that creates a stream with value starting from 0 and ending at 9. It excludes 10.

```java
IntStream.range(0, 10).forEach(System.out::println);
```

`rangeClosed`方法允许我们创建一个包含上限值的stream。因此，下面的代码会产生一个从1到10的stream。
The `rangeClosed` method allows you to create streams that includes the upper bound as well. So, the below mentioned stream will start at 1 and end at 10.
```java
IntStream.rangeClosed(1, 10).forEach(System.out::println);
```
还可以像下面这样，通过在基本类型的stream上使用`iterate`方法来创建无限的stream：
You can also create infinite streams using iterate method on the primitive streams as shown below.

```java
LongStream infiniteStream = LongStream.iterate(1, el -> el + 1);
```
要从一个无限的stream中过滤出所有偶数，可以用如下代码来实现:
To filter out all even numbers in an infinite stream, we can write code as shown below.

```java
infiniteStream.filter(el -> el % 2 == 0).forEach(System.out::println);
```
可以通过使用`limit`操作来现在结果stream的个数，代码如下：
We can limit the resulting stream by using the `limit` operation as shown below.

```java
infiniteStream.filter(el -> el % 2 == 0).limit(100).forEach(System.out::println);
```

### Example 9: Creating Streams from Arrays为数组创建stream

可以像如下代码这样，通过调用`Arrays`类的静态方法`stream`来把为数组建立stream：
You can create streams from arrays by using the static `stream` method on the `Arrays` class as shown below.

```java
String[] tags = {"java", "git", "lambdas", "machine-learning"};
Arrays.stream(tags).map(String::toUpperCase).forEach(System.out::println);
```

还可以像如下这样，根据数组中特定起始下标和结束下标来创建stream。这里的起始下标包括在内，而结束下标不包含在内。
You can also create a stream from the array by specifying the start and end indexes as shown below. Here, starting index is inclusive and ending index is exclusive.

```java
Arrays.stream(tags, 1, 3).map(String::toUpperCase).forEach(System.out::println);
```

## Parallel Streams并发的stream


使用Stream还有一个优势在于，由于stream采用内部迭代，所有java库可以优先的处理并发。可以通过在一个stream上调用`parallel`方法来使一个stream并行。‘parallel’方法的底层实现基于JDK7中引入的'fork-join'API,它可以产生与你机器CPU数量相等的线程。下面的代码中，我们根据线程的数量来对将数字分组。你将在第4节中学习`collect`和`groupingBy`函数，现在暂时理解为它可以根据一个key来对元素进行分组。
One advantage that you get by using Stream abstraction is that now library can effectively manage parallelism as iteration is internal. You can make a stream parallel by calling `parallel` method on it. The `parallel` method underneath uses the fork-join API introduced in JDK 7. By default, it will spawn up threads equal to number of CPU in your machine. In the code show below, we are grouping numbers by thread that processed them. You will learn about `collect` and `groupingBy` functions in chapter 4. For now just understand that they allow you to group elements based on a key.

```java
public class ParallelStreamExample {

    public static void main(String[] args) {
        Map<String, List<Integer>> numbersPerThread = IntStream.rangeClosed(1, 160)
                .parallel()
                .boxed()
                .collect(groupingBy(i -> Thread.currentThread().getName()));

        numbersPerThread.forEach((k, v) -> System.out.println(String.format("%s >> %s", k, v)));
    }
}
```

在我的机器上，打印的结果如下：
The output of the above program on my machine looks like as shown below.

```
ForkJoinPool.commonPool-worker-7 >> [46, 47, 48, 49, 50]
ForkJoinPool.commonPool-worker-1 >> [41, 42, 43, 44, 45, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 111, 112, 113, 114, 115, 116, 117, 118, 119, 120, 121, 122, 123, 124, 125, 126, 127, 128, 129, 130]
ForkJoinPool.commonPool-worker-2 >> [146, 147, 148, 149, 150]
main >> [106, 107, 108, 109, 110]
ForkJoinPool.commonPool-worker-5 >> [71, 72, 73, 74, 75]
ForkJoinPool.commonPool-worker-6 >> [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 151, 152, 153, 154, 155, 156, 157, 158, 159, 160]
ForkJoinPool.commonPool-worker-3 >> [21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 76, 77, 78, 79, 80]
ForkJoinPool.commonPool-worker-4 >> [91, 92, 93, 94, 95, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 131, 132, 133, 134, 135, 136, 137, 138, 139, 140, 141, 142, 143, 144, 145]
```

并不是每个工作的线程都处理相等数量的数字，可以通过更改系统属性来控制fork-join线程池的数量。
Not every thread process same number of elements. You can control the size of fork join thread pool by setting a system property `System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "2")`.

另外一个会用到`parallel'操作的例子是，当你像下面这样要处理一个URL的列表时：

Another example where you can use `parallel` operation is when you are processing a list of URLs as shown below.

```java
String[] urls = {"https://www.google.co.in/", "https://twitter.com/", "http://www.facebook.com/"};
Arrays.stream(urls).parallel().map(url -> getUrlContent(url)).forEach(System.out::println);
```

如果你想更好的掌握什么时候应该使用并发的stream,推荐你阅读由Doug Lea和其他几个Java大牛写的文章[http://gee.cs.oswego.edu/dl/html/StreamParallelGuidance.html](http://gee.cs.oswego.edu/dl/html/StreamParallelGuidance.html)。
If you need to understand when to use Parallel Stream I would recommend you read this article by Doug Lea and few other Java folks [http://gee.cs.oswego.edu/dl/html/StreamParallelGuidance.html](http://gee.cs.oswego.edu/dl/html/StreamParallelGuidance.html) to gain better understanding.

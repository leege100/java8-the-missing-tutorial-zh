收集器(Collectors)
------

> - 原文链接： [Collectors](https://github.com/shekhargulati/java8-the-missing-tutorial/blob/master/04-collectors.md)  
- 原文作者： [shekhargulati](https://github.com/shekhargulati)  
- 译者： leege100  
- 状态： 完成

在前一节中，我们已经了解到StreamAPI能够帮助我们用更直观简洁的方式来处理集合。现在我们来看一下`collect`方法，它是一个能够把stream管道中的结果集装进一个`List`集合的终极操作。	`collect`是一个把stream规约成一个value的规约操作，这里的value可以是一个Collection、Map或者一个value对象。在下面这几种情况下，可以使用`collect`操作。

1. **把stream规约到一个单独的值** stream的执行结果可以规约成一个单独的值，这个单独的值可以是`Collection`或者数值型的值如int、double等，还可以是一个自定义的值对象。 

2. **在stream中对元素进行分组** 对stream中的所有task按照TaskType分组。这会生成一个一个`Map<TaskType,List<Task>`,其中的每个entry都包含一个TaskType和与它相关联的Task。也可以使用其它任何的Collection来替代List。如果不需要把所有的task对应到一个TaskType,也可以生成一个`Map<TaskType,Task>`。
3. **分离stream中的元素** 可以把一个stream分离到两个组中--正在进行中的和已经完成的task。 

## Collector in Action

下面我们通过这个根据type来对task进行分组的例子，来体验`Collector`的作用。在java8中，我们可以像下面这样来实现根据TaskType分组。**请参考博客[day 2](http://shekhargulati.com/2015/07/26/day-2-lets-learn-about-streams/)**

```java
private static Map<TaskType, List<Task>> groupTasksByType(List<Task> tasks) {
    return tasks.stream().collect(Collectors.groupingBy(task -> task.getType()));
}
```

上面的代码使用了`Collectors`工具类中定义的`groupingBy` `Collector`方法。它创建一个map，其中key为`TaskType`、value为所有具有相同`TaskType`的task组成的一个list列表。在java7中要实现相同的功能，需要写如下的代码。

```java
public static void main(String[] args) {
    List<Task> tasks = getTasks();
    Map<TaskType, List<Task>> allTasksByType = new HashMap<>();
    for (Task task : tasks) {
        List<Task> existingTasksByType = allTasksByType.get(task.getType());
        if (existingTasksByType == null) {
            List<Task> tasksByType = new ArrayList<>();
            tasksByType.add(task);
            allTasksByType.put(task.getType(), tasksByType);
        } else {
            existingTasksByType.add(task);
        }
    }
    for (Map.Entry<TaskType, List<Task>> entry : allTasksByType.entrySet()) {
        System.out.println(String.format("%s =>> %s", entry.getKey(), entry.getValue()));
    }
}
```

## 收集器（Collectors）:常用规约操作

`Collectors` 工具类提供了许多静态工具方法来为大多数常用的用户用例创建收集器，比如将元素装进一个集合中、将元素分组、根据不同标准对元素进行汇总等。本文中将覆盖大多数常见的`收集器(Collector)`。


## 规约到一个单独的值

如上面所说，收集器(collector)可以用来把stream收集到一个collection中或者产生一个单独的值。

### 把数据装进一个list列表中

下面我们给出第一个测试用例--将给定的任务列表的所有标题收集到一个List列表中。


```java
import static java.util.stream.Collectors.toList;

public class Example2_ReduceValue {
    public List<String> allTitles(List<Task> tasks) {
        return tasks.stream().map(Task::getTitle).collect(toList());
    }
}
```

`toList`收集器通过使用List的`add`方法将元素添加到一个结果List列表中，`toList`收集器使用`ArrayList`作为List的实现。


### 将数据收集到一个Set中

如果要保证所收集的title不重复并且我们对数据的排序没有要求的话，可以采用`toSet`收集器。

```java
import static java.util.stream.Collectors.toSet;

public Set<String> uniqueTitles(List<Task> tasks) {
    return tasks.stream().map(Task::getTitle).collect(toSet());
}
```

`toSet` 方法采用`HashSet`作为Set的实现来储存结果集。

### 把数据收集到一个Map中

可以使用`toMap`收集器将一个stream转换成一个Map。`toMap`收集器需要两个集合函数来提取map中的key和value。下面的代码中，`Task::getTitle`需要一个task并产生一个仅有一个标题的key。**task -> task**是一个用来返回自己的lambda表达式，上例中返回一个task。

```java
private static Map<String, Task> taskMap(List<Task> tasks) {
  return tasks.stream().collect(toMap(Task::getTitle, task -> task));
}
```

可以使用`Function`接口中的默认方法`identity`来让上面的代码代码变得更简洁明了、传递开发者意图时更加直接，下面是采用identity函数的代码。

```java
import static java.util.function.Function.identity;

private static Map<String, Task> taskMap(List<Task> tasks) {
  return tasks.stream().collect(toMap(Task::getTitle, identity()));
}
```

代码创建了一个Map，当出现相同的key时就会抛出如下的异常。

```
Exception in thread "main" java.lang.IllegalStateException: Duplicate key Task{title='Read Version Control with Git book', type=READING}
at java.util.stream.Collectors.lambda$throwingMerger$105(Collectors.java:133)
```

`toMap`还有一个可以指定合并函数的变体，我们可以采用它来处理重复的副本。合并函数允许开发者指定一个解决同一个key冲突的规则。在下面的代码中，我们简单地使用最后一个value，当然你也可以写更加智能的算法来处理冲突。

```java
private static Map<String, Task> taskMap_duplicates(List<Task> tasks) {
  return tasks.stream().collect(toMap(Task::getTitle, identity(), (t1, t2) -> t2));
}
```

我们还可以使用`toMap`的第三种变体方法来使用任何其它的Map实现，这需要指定`Map` 和`Supplier`来存放结果。

```
public Map<String, Task> collectToMap(List<Task> tasks) {
    return tasks.stream().collect(toMap(Task::getTitle, identity(), (t1, t2) -> t2, LinkedHashMap::new));
}
```

与`toMap`收集器类似，`toConcurrentMap`收集器可以产生`ConcurrntMap`来替代`HashMap`。

### Using other collections 使用其它的集合

`toList`和`toSet`等特定的收集器不支持指定潜在的list或set的实现，当你想要像下面这样这样把结果聚合到其它类型的集合时可以采用`toCollection`收集器。

```
private static LinkedHashSet<Task> collectToLinkedHaskSet(List<Task> tasks) {
  return tasks.stream().collect(toCollection(LinkedHashSet::new));
}
```

### 找出标题最长的task

```java
public Task taskWithLongestTitle(List<Task> tasks) {
    return tasks.stream().collect(collectingAndThen(maxBy((t1, t2) -> t1.getTitle().length() - t2.getTitle().length()), Optional::get));
}
```

### 统计tags的总数

```java
public int totalTagCount(List<Task> tasks) {
    return tasks.stream().collect(summingInt(task -> task.getTags().size()));
}
```

### 生成task标题的汇总

```java
public String titleSummary(List<Task> tasks) {
    return tasks.stream().map(Task::getTitle).collect(joining(";"));
}
```

## 将元素分组

Collector收集器一个最常见的用户用例就是对元素进行分组，下面我们通过几个示例来理解我们可以如何来分组。

### Example 1: 根据type对tasks分组

下面这个例子，我们根据`TaskType`对task进行分组。通过使用`Collectors`工具类的`groupingBy`收集器，我们可以非常简单的完成这个功能。可以使用方法引用和静态引入来让代码变得更加简洁。

```java
import static java.util.stream.Collectors.groupingBy;
private static Map<TaskType, List<Task>> groupTasksByType(List<Task> tasks) {
       return tasks.stream().collect(groupingBy(Task::getType));
}
```
会产生如下的输出：
```
{CODING=[Task{title='Write a mobile application to store my tasks', type=CODING, createdOn=2015-07-03}], WRITING=[Task{title='Write a blog on Java 8 Streams', type=WRITING, createdOn=2015-07-04}], READING=[Task{title='Read Version Control with Git book', type=READING, createdOn=2015-07-01}, Task{title='Read Java 8 Lambdas book', type=READING, createdOn=2015-07-02}, Task{title='Read Domain Driven Design book', type=READING, createdOn=2015-07-05}]}
```

### Example 2: 根据tags分组

```java
private static Map<String, List<Task>> groupingByTag(List<Task> tasks) {
        return tasks.stream().
                flatMap(task -> task.getTags().stream().map(tag -> new TaskTag(tag, task))).
                collect(groupingBy(TaskTag::getTag, mapping(TaskTag::getTask,toList())));
}

    private static class TaskTag {
        final String tag;
        final Task task;

        public TaskTag(String tag, Task task) {
            this.tag = tag;
            this.task = task;
        }

        public String getTag() {
            return tag;
        }

        public Task getTask() {
            return task;
        }
    }
```

### Example 3: 根据tag和tag的个数分组

```java
private static Map<String, Long> tagsAndCount(List<Task> tasks) {
        return tasks.stream().
        flatMap(task -> task.getTags().stream().map(tag -> new TaskTag(tag, task))).
        collect(groupingBy(TaskTag::getTag, counting()));
    }
```

### Example 4: 根据TaskType和createdOn分组

```java
private static Map<TaskType, Map<LocalDate, List<Task>>> groupTasksByTypeAndCreationDate(List<Task> tasks) {
        return tasks.stream().collect(groupingBy(Task::getType, groupingBy(Task::getCreatedOn)));
    }
```

## 分割

有时候，你需要根据一定的规则将一个数据集分成两个数据集。比如，我们可以定义一个分割函数，根据规则`进行时间早于今天和进行时间晚于今天`将task分成两组。

```java
private static Map<Boolean, List<Task>> partitionOldAndFutureTasks(List<Task> tasks) {
  return tasks.stream().collect(partitioningBy(task -> task.getDueOn().isAfter(LocalDate.now())));
}
```

## 生成统计

另外，一些产生统计结果的收集器也非常有用。它们主要用于int、double、long等基本类型上，它们可以用来产生类似如下的统计结果。
```java
IntSummaryStatistics summaryStatistics = tasks.stream().map(Task::getTitle).collect(summarizingInt(String::length));
System.out.println(summaryStatistics.getAverage()); //32.4
System.out.println(summaryStatistics.getCount()); //5
System.out.println(summaryStatistics.getMax()); //44
System.out.println(summaryStatistics.getMin()); //24
System.out.println(summaryStatistics.getSum()); //162
```

还有一些其它的基本类型的变体，比如`LongSummaryStatistics` 和 `DoubleSummaryStatistics`

还可以用`combine`操作来把两个`IntSummaryStatistics`结合到一起。

```java
firstSummaryStatistics.combine(secondSummaryStatistics);
System.out.println(firstSummaryStatistics)
```

## 把所有的titles连在一起

```java
private static String allTitles(List<Task> tasks) {
  return tasks.stream().map(Task::getTitle).collect(joining(", "));
}
```

## 编写一个自定义的收集器

```java
import com.google.common.collect.HashMultiset;
import com.google.common.collect.Multiset;

import java.util.Collections;
import java.util.EnumSet;
import java.util.Set;
import java.util.function.BiConsumer;
import java.util.function.BinaryOperator;
import java.util.function.Function;
import java.util.function.Supplier;
import java.util.stream.Collector;

public class MultisetCollector<T> implements Collector<T, Multiset<T>, Multiset<T>> {

    @Override
    public Supplier<Multiset<T>> supplier() {
        return HashMultiset::create;
    }

    @Override
    public BiConsumer<Multiset<T>, T> accumulator() {
        return (set, e) -> set.add(e, 1);
    }

    @Override
    public BinaryOperator<Multiset<T>> combiner() {
        return (set1, set2) -> {
            set1.addAll(set2);
            return set1;
        };
    }

    @Override
    public Function<Multiset<T>, Multiset<T>> finisher() {
        return Function.identity();
    }

    @Override
    public Set<Characteristics> characteristics() {
        return Collections.unmodifiableSet(EnumSet.of(Characteristics.IDENTITY_FINISH));
    }
}
```

```java
import com.google.common.collect.Multiset;

import java.util.Arrays;
import java.util.List;

public class MultisetCollectorExample {

    public static void main(String[] args) {
        List<String> names = Arrays.asList("shekhar", "rahul", "shekhar");
        Multiset<String> set = names.stream().collect(new MultisetCollector<>());

        set.forEach(str -> System.out.println(str + ":" + set.count(str)));

    }
}
```

## Word Count in Java 8 Java8中的单词统计

下面，我们通过java8中的Streams和Collectors编写一个非常著名的单词统计示例来结束本文。

```java
public static void wordCount(Path path) throws IOException {
    Map<String, Long> wordCount = Files.lines(path)
            .parallel()
            .flatMap(line -> Arrays.stream(line.trim().split("\\s")))
            .map(word -> word.replaceAll("[^a-zA-Z]", "").toLowerCase().trim())
            .filter(word -> word.length() > 0)
            .map(word -> new SimpleEntry<>(word, 1))
            .collect(groupingBy(SimpleEntry::getKey, counting()));
    wordCount.forEach((k, v) -> System.out.println(String.format("%s ==>> %d", k, v)));
}
```

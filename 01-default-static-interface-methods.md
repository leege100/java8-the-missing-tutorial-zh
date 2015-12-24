接口的默认和静态方法
--------

> - 原文链接： [Default and Static Methods for Interfaces](https://github.com/shekhargulati/java8-the-missing-tutorial/blob/master/01-default-static-interface-methods.md)  
- 原文作者： [shekhargulati](https://github.com/shekhargulati)  
- 译文出自： [leege100](http://www.jianshu.com/users/bf691c3709d6/latest_articles)  
- 译者： leege100  
- 状态： 完成

众所周知，我们应该使用接口编程，接口使得在交互时不需要关注具体的实现细节，从而保持程序的松散耦合。在API的设计中，设计简约而清晰的接口非常重要。被称作固定定律的接口分离定律，其中有一条就讲到了应该设计更小的特定客户端接口而不是一个通用目的的接口。良好的接口设计是让应用程序和库的API保持简洁高效的关键。

如果你曾有过接口API设计的经验，那么有时候你会感觉到为API增加方法的必要。但是，如果API一旦发布了，几乎无法在保证已接口类的代码不变的情况下来增加新的方法。举个例子，假设你设计了一个简单的API Calculator，里面有add、subtract、devide和multiply函数。我们可以如下申明。简便起见，我们使用int类型。

    public interface Calculator {
    
      int add(int first, int second);
    
      int subtract(int first, int second);
    
      int divide(int number, int divisor);
    
      int multiply(int first, int second);
    }

为了实现Calculator这个接口，需要写如下一个BasicCalculator类

    public class BasicCalculator implements Calculator {
    
      @Override
      public int add(int first, int second) {
        return first + second;
      }
    
      @Override
      public int subtract(int first, int second) {
        return first - second;
      }
    
      @Override
      public int divide(int number, int divisor) {
        if (divisor == 0) {
          throw new IllegalArgumentException("divisor can't be           zero.");
        }
        return number / divisor;
      }
    
      @Override
      public int multiply(int first, int second) {
        return first * second;
      }
    }

静态工厂方法

乍一看，Calculator这个API还是非常简单实用，其他开发者只需要创建一个BasicCalculator就可以使用这个API。比如下面这两段代码：

    Calculator calculator = new BasicCalculator();
    int sum = calculator.add(1, 2);
    
    BasicCalculator cal = new BasicCalculator();
    int difference = cal.subtract(3, 2);

然而，事实上给人的感觉却是此API的用户并不是面向这个接口进行编程，而是面向这个接口的实现类在编程。因为你的API并没有强制要求用户把BasicCalculator当着一个public类来对接口进行编程，如果你把BasicCalculator类变成protected类型你就需要提供一个静态的工厂类来谨慎的提供Calculator的实现，代码如下:

    class BasicCalculator implements Calculator {
      // rest remains same
    }
接下来，写一个工厂类来提供Calculator的实例，代码如下：

    public abstract class CalculatorFactory {
    
      public static Calculator getInstance() {
      return new BasicCalculator();
      }
    }

这样，用户就被强制要求对Calculator接口进行编程，并且不需要关注接口的详细实现。

尽管我们通过上面的方式达到了我们想要的效果，但是我们却因增加了一个新的CalculaorFactory类而增加了API的表现层次。如果其他开发者要有效的利用这个Calculator API，就必须要先学会一个新的类的使用。但在java8之前，这是唯一的实现方式。

java8允许用户在接口中定义静态的方法，允许API的设计者在接口的内部定义类似getInstance的静态工具方法，从而保持API简洁。静态的接口方法可以用来替代我们平时专为某一类型写的helper类。比如，Collections类是用来为Collection和相关接口提供各种helper方法的一个类。Collections类中定义的方法应该直接添加到Collection及其它相关子接口上。

下面的代码就是Java8中直接在Calculator接口中添加静态getInstance方法的例证：

    public interface Calculator {
    
      static Calculator getInstance() {
      return new BasicCalculator();
      }
    
      int add(int first, int second);
    
      int subtract(int first, int second);
    
      int divide(int number, int divisor);
    
      int multiply(int first, int second);
    
    }

随着时间来演进API

有些用户打算通过新建一个Calculator子接口并在子接口中添加新方法或是通过重写一个继承自Calculator接口的实现类来增加类似remainder的方法。与他们沟通之后你会发现，他们可能仅仅是想向Calculator接口中添加一个类似remainder的方法。感觉这仅仅是一个非常简单的API变化，所以你加入了一个方法，代码如下:
    public interface Calculator {
    
      static Calculator getInstance() {
        return new BasicCalculator();
      }
    
      int add(int first, int second);
    
      int subtract(int first, int second);
    
      int divide(int number, int divisor);
    
      int multiply(int first, int second);
    
      int remainder(int number, int divisor); // new method added to API
    }
在接口中增加方法会破坏代码的兼容性，这意味着那些实现Calculator接口的用户需要为remainder方法添加实现，否则，代码无法编译通过。这对API设计者来说非常麻烦，因为它让代码演进变得非常困难。Java8之前，开发者不能直接在API中实现方法，这也使得在一个API中添加一个或者多个接口定义变得十分困难。

为了让API的演进变得更加容易，Java8允许用户对定义在接口中的方法提供默认的实现。这就是通常的default或者defender方法。接口的实现类不需要再提供接口的实现方法。如果接口的实现类提供了方法，那么实现类的方法会被调用，否则接口中的方法会被调用。List接口有几个定义在接口中的默认方法，比如replaceAll,sort和splitIterator。

    default void replaceAll(UnaryOperator<E> operator) {
      Objects.requireNonNull(operator);
      final ListIterator<E> li = this.listIterator();
      while (li.hasNext()) {
        li.set(operator.apply(li.next()));
      }
    }

我们可以向下面的代码一样定义一个默认的方法来解决API的问题。默认方法通常用在使用那些已经存在的方法，比如remainder是用来定义使用subtract,multiply和divice的一个方法。

    default int remainder(int number, int divisor) {
      return subtract(number, multiply(divisor, divide(number, divisor)));
    }

多重继承

一个类只能继承自一个父类，但可以实现多个接口。既然在接口中实现方法是可行的，接口方法的多实现也是可行的。之前，Java已经在类型上支持多重继承，如今也支持在表现阶段的多重继承。下面这三条规则决定了哪个方法会被调用。

规则一：定义在类中的方法优先于定义在接口的方法：

    interface A {
      default void doSth(){
      System.out.println("inside A");
      }
    }
    class App implements A{
    
      @Override
      public void doSth() {
        System.out.println("inside App");
      }
    
      public static void main(String[] args) {
        new App().doSth();
      }
    }
  
运行结果：inside App。调用的是在类中定义的方法。

规则二：否则，调用定制最深的接口中的方法。

    interface A {
      default void doSth() {
        System.out.println("inside A");
      }
    }
    interface B {}
    interface C extends A {
      default void doSth() {
        System.out.println("inside C");
      } 
    }
    class App implements C, B, A {
    
      public static void main(String[] args) {
        new App().doSth();
      }
    }
   
运行结果：inside C

规则三：否则，直接调用指定接口的实现方法

    interface A {
      default void doSth() {
        System.out.println("inside A");
      }
    }
    interface B {
      default void doSth() {
        System.out.println("inside B");
      }
    }
    class App implements B, A {
    
      @Override
      public void doSth() {
        B.super.doSth();
      }
    
      public static void main(String[] args) {
          new App().doSth();
      }
    }

运行结果： inside B

水平有限，欢迎大家指点和建议，^-^
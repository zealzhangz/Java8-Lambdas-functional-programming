![alt](https://i.imgur.com/N5q8tne.png)
# 第一章
## 1.1 修改Java的目的
`java.util.concurrent`的不足，让代码在多核 `CPU` 上高效运行。为了编写这类处理批量数据的并行类库，需要在语言层面上修改现有的`Java`:增加 `Lambda` 表达式。

面向对象编程是对数据进 行抽象，而函数式编程是对行为进行抽象。

Java 8还让集合类可以拥有一些额外的方法:default方法。程序员在维护自己的类库时， 可以使用这些方法。

## 1.2 什么是函数式编程
其核心是:在思考问题时，使用不可变值和函 数，函数对一个值进行处理，映射成另一个值。

# 第二章
## 2.1 Lambda表达式
- 一个`Swing Button`点击事件处理的例子：

```java
button.addActionListener(new ActionListener() { 
    public void actionPerformed(ActionEvent event) {
             System.out.println("button clicked");
         }
});
```

`ActionListener`是动作监听接口，动作处理需实现接口中的`actionPerformed`方法，一般的做法如上直接`new`一个匿名类，然后直接实现`actionPerformed`方法。设计匿名内部类的目的，就是为了方便 `Java` 程序员将代码作为数据传递。

上述代码有两个比较明显的问题：

1. `4`行冗繁的样板代码。
2. 这些代码还相当难读，因为它没有清楚地表达程 序员的意图。我们不想传入对象，只想传入`行为`。

- `Lambda`表达式解决以上两个问题

```java
button.addActionListener(event -> System.out.println("button clicked"));
```

## 2.2 辨别`Lambda`表达式
### 2.2.1 Lambda 表达式不包含参数
`Lambda` 表达式不包含参数，使用空括号 `()` 表示没有参数。该 `Lambda` 表达式 实现了 `Runnable` 接口，该接口也只有一个 `run` 方法，没有参数，且返回类型为 `void`。注意这里有无参数取决于对应接口中方法是否有返回值。

```java
Runnable noArguments = () -> System.out.println("Hello World");
```

### 2.2.2 包含一个参数
中所示的 `Lambda` 表达式包含且只包含一个参数，可省略参数的括号，这和例 `2-2` 中的 形式一样。

```java
button.addActionListener(event -> System.out.println("button clicked"));
```

### 2.2.2 Lambda代码块
`Lambda` 表达式的主体不仅可以是一个表达式，而且也可以是一段代码块，使用大括号 `({})`将代码块括起来

```java
Runnable multiStatement = () -> {
    System.out.print("Hello");
    System.out.println(" World");
};

ExecutorService executorService = new ThreadPoolExecutor(2, 10, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
executorService.execute(multiStatement);
```

### 2.2.3 Lambda 表达式包含多个参数
这行代码并不是将两个数字相加，而是创建了一个函数，用来计算 两个数字相加的结果。变量 add 的类型是 BinaryOperator<Long>，它不是两个数字的和， 而是将两个数字相加的那行代码。

```java
BinaryOperator<Long> add = (x, y) -> x + y;
```

所有 `Lambda` 表达式中的参数类型都是由编译器推断得出的。这当然不错， 但有时最好也可以显式声明参数类型，此时就需要使用小括号将参数括起来，多个参数的 情况也是如此。

```java
BinaryOperator<Long> addExplicit = (Long x, Long y) -> x + y;
```

## 2.3 引用值，而不是变量
在匿名类内引用外部的变量，在`Java7`中必须将变量显示声明为`final`，`Java8`中可以省略`final`(`Java 8`虽然放松了这一限制，可以引用非`final`变量，但是该变量在既成事实上必须是 `final`，虽然无需将变量声明为final，但在Lambda表达式中，也无法用作非终态变量。如 果坚持用作非终态变量，编译器就会报错。
)，但是变量的还是`final`类型，不能被赋值。

```java
final String name = getUserName(); 
button.addActionListener(new ActionListener({
    public void actionPerformed(ActionEvent event) { 
        System.out.println("hi " + name);
    } 
});
```

`Lambda`表达式也是一样，既成事实上的 final 是指只能给该变量赋值一次。换句话说，Lambda 表达式引用的是值， 而不是变量。在例 2-6 中，name 就是一个既成事实上的 final 变量。

```java
String name = getUserName();
button.addActionListener(event -> System.out.println("hi " + name));
```

如果你试图给该变量多次赋值，然后在 Lambda 表达式中引用它，编译器就会报错。无法通过编译，并显示出错信息:local variables referenced from a Lambda expression must be final or effectively final1。

```java
    //无法编译通过
     String name = getUserName();
     name = formatUserName(name);
     button.addActionListener(event -> System.out.println("hi " + name));
```
注意：在匿名类或`Lambda`表达式中`Java8`中虽然可以省略`final`关键词，但是该变量已经成为既成事实上的 `final` 变量，尝试给他赋值的话编译也会报错。

## 2.4 函数接口
__函数接口是只有一个抽象方法的接口，用作 Lambda 表达式的类型。__

如果一个接口中有多个抽象方法就不是函数接口，就不能用作 `Lambda` 表达式。多个抽象方法情况编译会直接报如下错误信息：

```
Error:(373, 59) java: The target type of this expression must be a functional interface
```

- `ActionListener`就是函数接口

```java
public interface ActionListener extends EventListener {
    public void actionPerformed(ActionEvent e);
}
```

函数接口，接口中单一方法的命名并不重要，只要方法签名和 `Lambda` 表达式的类 型匹配即可。可在函数接口中为参数起一个有意义的名字，增加代码易读性，便于更透彻 地理解参数的用途。

这里的函数接口接受一个 `ActionEvent` 类型的参数，返回空(`void`)，但函数接口还可有其他形式。例如，函数接口可以接受两个参数，并返回一个值，还可以使用泛型，这完全取 决于你要干什么。

以后我将使用图形来表示不同类型的函数接口。指向函数接口的箭头表示参数，如果箭头 从函数接口射出，则表示方法的返回类型。`ActionListener` 的函数接口如下图所示：

![alt](https://www.zhangaoo.com/upload/2018/09/o1vd65dpe4h9mqo1o2u11s9bcv.jpeg)

- Java中重要的函数接口

可简单理解为可以把lambda表达式，一组行为（函数接口的实现）传递给方法。以前想传递函数(行为)，必须先将函数封装成对象的方法。然后传递改对象。lambda表达式则可以直接传递函数（行为）。

接口|参数|返回类型|说明|示例
:---|:---|:---|:---|:---
Predicate<T>|T|boolean|通过Lambda实现该接口中的test方法，返回一个布尔值，用作判断用|Predicate<Integer> boolValue = x -> x > 5;</br>System.out.println(boolValue.test(1));
Consumer<T>|T|void|通过Lambda实现该接口中的accept方法，不返回值，用于执行一些操作|Consumer<Integer> consumer = x -> System.out.println(x);</br>consumer.accept(123);
Function<T,R>|T|R|接受一个输入值T，处理后返回R类型数据|Function<Integer,Integer> function = t -> t+2;</br>System.out.println(function.apply(4));
Supplier<T>|None|T|类似于工厂方法，返回一个T类型的变量|Supplier<Integer> supplier = () -> 2;</br>Integer i = supplier.get();
UnaryOperator<T>|T|T|继承自接口`Function`，感觉和`Function`类似没看出什么差别|
BinaryOperator<T>|(T, T)|T|作用于于两个同类型操作符的操作，并且返回了操作符同类型的结果|BinaryOperator<Integer> binaryOperator = (x,y) -> x*y;</br>binaryOperator.apply(2,3);

- Consumer补充列子

```java
public class ConsumerTest {
    public static void main(String[] args) {
        Consumer<Integer> consumer = (x) -> {
            int num = x * 2;
            System.out.println(num);
        };
        Consumer<Integer> consumer1 = (x) -> {
            int num = x * 3;
            System.out.println(num);
        };
        consumer.andThen(consumer1).accept(10);
    }
}
/**
结果
20
30
**/
```

- UnaryOperator

```java
    UnaryOperator<Integer> unaryOperator = x -> x + 1;
    Integer i = unaryOperator.apply(10);
    System.out.println(i);
    System.out.println(UnaryOperator.identity().apply(5));
    System.out.println(UnaryOperator.identity().apply("1234567890"));

    /***
    结果：
    11
    5
    1234567890
    **/    
```

## 2.5 类型推断

Lambda表达式中的类型推断，实际上是Java 7就引入的目标类型推断的扩展。Java 7 中的菱形操作符，它可使 javac 推断出泛型参数的类型：

```java
Map<String, Integer> diamondWordCounts = new HashMap<>();
```

如果将构造函数直接传递给一个方法，也可根据方法签名来推断类型：

```java
//在Java7中不能通过编译
private void useHashmap(Map<String, String> values);
useHashmap(new HashMap<>());
```

Java 8 更进一步，可省略 `Lambda` 表达 式中的所有参数类型。

- Predicate——用来判断 真假的函数接口

```java
Predicate<Integer> atLeast5 = x -> x > 5;
```
接口

```java
public interface Predicate<T> { 
    boolean test(T t);
}
```
接口图示

![alt](https://www.zhangaoo.com/upload/2018/09/7pps3e4guggskru5fju9sv03na.png)

`Predicate` 接口图示，接受一个对象，返回一个布尔值

## 2.6 要点回顾
- `Lambda` 表达式是一个匿名方法，将行为像数据一样进行传递。
- `Lambda` 表达式的常见结构:`BinaryOperator<Integer> add = (x, y) → x + y`。
- 函数接口指仅具有单个抽象方法的接口，用来表示Lambda表达式的类型。

## 2.7 练习
1. 请看 Function 函数接口并回答下列问题。

```java
public interface Function<T, R> { 
    R apply(T t);
}
```

a. 请画出该函数接口的图示。
![alt](https://www.zhangaoo.com/upload/2018/09/u92ieabbssh28p3h2vcmfro8lg.jpg)

b. 若要编写一个计算器程序，你会使用该接口表示什么样的 Lambda 表达式?

```java
Function<Double,Double> calc = t -> t * 2.0; 
```
c. 下列哪些 `Lambda` 表达式有效实现了 Function<Long,Long> ?

```java
x->x+1; //OK
(x,y)->x+1; //NG
x->x==1;//NG
```

2. `ThreadLocal Lambda`表达式。`Java`有一个`ThreadLocal`类，作为容器保存了当前线程里局部变量的值。`Java 8`为该类新加了一个工厂方法，接受一个`Lambda`表达式，并产生 一个新的 `ThreadLocal` 对象，而不用使用继承，语法上更加简洁。

a. 在 `Javadoc` 或集成开发环境(IDE)里找出该方法。

```java
/**
 * Represents a supplier of results.
 *
 * <p>There is no requirement that a new or distinct result be returned each
 * time the supplier is invoked.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #get()}.
 *
 * @param <T> the type of results supplied by this supplier
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

b. DateFormatter 类是非线程安全的。使用构造函数创建一个线程安全的 `SimpleDateFormat`对象，并输出日期，如“01-Jan-1970”。

```java
    Supplier<ThreadLocal> threadLocal = () -> ThreadLocal.withInitial(() -> new SimpleDateFormat("dd-MMM-yyyy"));
    SimpleDateFormat df = (SimpleDateFormat)threadLocal.get().get();
    System.out.println(df.format(new Date()));
```

3. 类型推断规则。下面是将 Lambda 表达式作为参数传递给函数的一些例子。javac 能正确推断出 Lambda 表达式中参数的类型吗?换句话说，程序能编译吗?
a. Runnable helloWorld = () -> System.out.println("hello world");//OK
b. 使用 Lambda 表达式实现 ActionListener 接口://OK
```java
JButton button = new JButton();
     button.addActionListener(event ->
System.out.println(event.getActionCommand()));
```

c. 以如下方式重载 check 方法后，还能正确推断出 check(x -> x > 5) 的类型吗?
__不能正确推导，有歧义，因为输入的值都是Integer，返回都是boolean__

> No - the lambda expression could be inferred as IntPred or Predicate<Integer> so the overload is ambiguous.

```java
interface IntPred {
boolean test(Integer value);
}
boolean check(Predicate<Integer> predicate);
boolean check(IntPred predicate);
```

写成如下形式就可以推导类型：

```java
public interface IntPred {
    boolean test(Double value);
    public boolean check(IntPred intPred, Double i){
        return intPred.test(i);
    }
    public boolean check(Predicate<Integer> predicate,Integer i){
        return predicate.test(i);
    }

    //执行
    System.out.println(check(x -> x > 5, 6.0));
    System.out.println(check(x -> x > 10, 6));
}
```
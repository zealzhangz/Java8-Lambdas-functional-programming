# 第4章 类库
接下来将详细阐述另一个重要方面:如何使用 Lambda 表达式。即使不需要编写像 `Stream` 这样重度使用函数式编程风格的类库，学会如 何使用 Lambda 表达式也是非常重要的。

`Java 8` 中的另一个变化是引入了默认方法和接口的静态方法，它改变了人们认识类库的方 式，接口中的方法也可以包含代码体了。

本章还对前 3 章疏漏的知识点进行补充，比如，Lambda 表达式方法重载的工作原理、基 本类型的使用方法等

## 4.1 在代码中使用Lambda表达式
我们来看一个日志系统中的具体案例。在 `slf4j` 和 `log4j` 等几种常用的日志系统中，有 一些记录日志的方法，当日志级别不低于某个固定级别时就会开始记录日志。如此一来， 在日志框架中设置类似`void debug(String message)`这样的方法，当级别为`debug`时，它 们就开始记录日志消息。

问题在于，频繁计算消息是否应该记录日志会对系统性能产生影响。程序员通过显式调用 `isDebugEnabled` 方法来优化系统性能。即使直接调用 `debug` 方法能省去记录文本信息，也仍然需要调用 `expensiveOperation` 方法，并且需要将执行结果和已有字符串连接起来，因此，使用 `if` 语句显式判断，可以让程序跑得更快。

- 使用 isDebugEnabled 方法降低日志性能开销

```java
Logger logger = new Logger(); 
if (logger.isDebugEnabled()) {
         logger.debug("Look at this: " + expensiveOperation());
     }
```

这里我们想做的是传入一个 `Lambda` 表达式，生成一条用作日志信息的字符串。只有日志 级别在调试或以上级别时，才会执行该 `Lambda` 表达式。使用这个方式重写上面的代码。

- 使用 `Lambda` 表达式简化日志代码

```java
    Logger logger = new Logger();
    logger.debug(() -> "Look at this: " + expensiveOperation());
```

那么在 `Logger` 类中该方法是如何实现的呢?从类库的角度看，我们可以使用内置的 `Supplier` 函数接口，它只有一个 `get` 方法。然后通过调用 isDebugEnabled 判断是否需要记录日志，是否需要调用 `get` 方法，如果需要，就调用 `get` 方法并将结果传给 `debug` 方法。

- 启用 `Lambda` 表达式实现的日志记录器

```java
public void debug(Supplier<String> message) { 
    if (isDebugEnabled()) {
             debug(message.get());
    }
}
//调用如下
debug(()-> "Log information")
```

调用 `get()` 方法，相当于调用传入的 `Lambda` 表达式。这种方式也能和匿名内部类一起工作，如果用户暂时无法升级到 `Java 8`，这种方式可以实现向后兼容。值得注意的是，不同的函数接口有不同的方法。如果使用 `Predicate`，就应该调用 `test` 方法，如果使用 `Function`，就应该调用 `apply` 方法。

__思考：没感觉换成Lambda表达式带来了其他优势，似乎还没GET到作者的意图，难道就是增加了一个方法，简化了几行代码？__

## 4.2 基本类型
在 `Java` 中，有一些相伴的类型，比如 `int` 和 `Integer`—— 前者是基本类型，后者是装箱类型。基本类型内建在语言和运行环境中，是基本的程序构 建模块;而装箱类型属于普通的 `Java` 类，只不过是对基本类型的一种封装。

`Java` 的泛型是基于对泛型参数类型的擦除——换句话说，假设它是 `Object` 对象的实例—— 因此只有装箱类型才能作为泛型参数。这就解释了为什么在 `Java` 中想要一个包含整型值的 列表 `List<int>`，实际上得到的却是一个包含整型对象的列表 `List<Integer>`。

- 关于`Java`泛型类型擦除的补充
`Java` 的泛型在编译器有效，在运行期被删除，也就是说所有泛型参数类型在编译后都会被清除掉，看下面一个列子，代码如下：

```java
public class Foo {  
    public void listMethod(List<String> stringList){  
    }  
    public void listMethod(List<Integer> intList) {  
    }  
}
```

编译上面的代码报错如下：

```java
//此错误的意思是说listMethod(List<String>) 方法在编译时擦除类型后的方法是listMethod(List<E>)，它与另外一个方法重复，也就是方法签名重复。
Method listMethod(List<String>) has the same erasure listMethod(List<E>) as another method in type Foo
```
- 反编译之后的方法代码如下：

```java
public void listMethod(List list)  {  }
```
从上面代码可以看出 `Java` 编译后的字节码中已经没有泛型的任何信息，在编译后所有的泛型类型都会做相应的转化，转化如下：
1. List<String>、List<T> 擦除后的类型为 List。
2. List<String>[]、List<T>[] 擦除后的类型为 List[]。
3. List<? extends E>、List<? super E> 擦除后的类型为 List<E>。
4. List<T extends Serialzable & Cloneable> 擦除后类型为 List<Serializable>。

__回归正题__

由于装箱类型是对象，因此在内存中存在额外开销。比如，整型在内存中占用 `4` 字节，整型对象却要占用 `16` 字节。这一情况在数组上更加严重，整型数组中的每个元素 只占用基本类型的内存，而整型对象数组中，每个元素都是内存中的一个指针，指向 `Java` 堆中的某个对象。在最坏的情况下，同样大小的数组，`Integer[]` 要比 `int[]` 多占用 `6` 倍内存。

将基本类型转换为装箱类型，称为装箱，反之则称为拆箱，两者都需要额外的计算开销。 对于需要大量数值运算的算法来说，装箱和拆箱的计算开销，以及装箱类型占用的额外内 存，会明显减缓程序的运行速度。

为了减小这些性能开销，`Stream` 类的某些方法对基本类型和装箱类型做了区分。下图所示的高阶函数`mapToLong`和其他类似函数即为该方面的一个尝试。在`Java 8`中，仅对整型、 长整型和双浮点型做了特殊处理，因为它们在数值计算中用得最多，特殊处理后的系统性 能提升效果最明显。`mapToLong`参数接受如下`Lambda`表达式：

![alt](https://www.zhangaoo.com/upload/2018/10/4jcmmdt3iuh4qoiut45hrdnpm2.png)

对基本类型做特殊处理的方法在命名上有明确的规范。如果方法返回类型为基本类型，则 在基本类型前加 `To`，如上图中的`ToLongFunction`。如果参数是基本类型，则不加前缀只需类型名即可，如下图中的 `LongFunction`。如果高阶函数使用基本类型，则在操作后加 后缀 `To` 再加基本类型，如 `mapToLong`。

![alt](https://www.zhangaoo.com/upload/2018/10/6m55aaimmsjp0os1rsq16hj1fa.png)

这些基本类型都有与之对应的 `Stream`，以基本类型名为前缀，如 `LongStream`。事实上， `mapToLong` 方法返回的不是一个一般的 `Stream`，而是一个特殊处理的 `Stream`。在这个特 殊的 `Stream` 中，`map` 方法的实现方式也不同，它接受一个 `LongUnaryOperator` 函数，将 一个长整型值映射成另一个长整型值，如下图所示。通过一些高阶函数装箱方法，如 `mapToObj`，也可以从一个基本类型的 `Stream` 得到一个装箱后的 `Stream`，如 `Stream<Long>`。

![alt](https://www.zhangaoo.com/upload/2018/10/322vdqqpcsirlq360jmgcqlkcc.png)

如有可能，应尽可能多地使用对基本类型做过特殊处理的方法，进而改善性能。这些特殊的`Stream`还提供额外的方法，避免重复实现一些通用的方法，让代码更能体现出数值计算 的意图。
- 使用 summaryStatistics 方法统计曲目长度

```java
    Album album = new Album(Sets.newHashSet(
            new Track("wwwww", 120, "wrwerwer"),
            new Track("sdsds", 30, "sfsdfsdfsdf"),
            new Track("wqeqweqwe", 120, "sasdasd"),
            new Track("asdasd", 120, "123qeew")));
    IntSummaryStatistics trackStatistic = album.getTracksStream()
            .mapToInt(track -> track.getLength())
            .summaryStatistics();

    System.out.printf("Max:%d,Min:%d,Avg:%f,Sum:%d",trackStatistic.getMax(),trackStatistic.getMin(),trackStatistic.getAverage(),trackStatistic.getSum());
```
无需手动计算这些信息，这里使用对基 本类型进行特殊处理的方法`mapToInt`，将每首曲目映射为曲目长度。因为该方法返回一个 `IntStream`对象，它包含一个 `summaryStatistics` 方法，这个方法能计算出各种各样的统计 值，如 `IntStream` 对象内所有元素中的最小值、最大值、平均值以及数值总和。

这些统计值在所有特殊处理的 `Stream`，如 `DoubleStream、LongStream` 中都可以得出。如无 需全部的统计值，也可分别调用 `min、max、average` 或 `sum` 方法获得单个的统计值，同样， 三种基本类型对应的特殊 `Stream` 也都包含这些方法。

## 4.3 重载解析
在 `Java` 中可以重载方法，造成多个方法有相同的方法名，但签名确不一样。这在推断参数 类型时会带来问题，因为系统可能会推断出多种类型。这时，`javac` 会挑出最具体的类型。 如下面的例子输出 `String`，而不是 `Object`。

```java
overloadedMethod("abc");

private void overloadedMethod(Object o) { 
    System.out.print("Object");
}

private void overloadedMethod(String s) { 
    System.out.print("String");
}
```

`BinaryOperator` 是一种特殊的 `BiFunction` 类型，参数的类型和返回值的类型相同。比如，
两个整数相加就是一个 `BinaryOperator`。

Lambda 表达式的类型就是对应的函数接口类型，因此，将 Lambda 表达式作为参数 传递时，情况也依然如此。操作时可以重载一个方法，分别接受 BinaryOperator 和该接口的一个子类作为参数。调用这些方法时，Java 推导出的 Lambda 表达式的类型正 是最具体的函数接口的类型。

```java
public interface IntegerBiFunction extends BinaryOperator<Integer> {}

private void overloadedMethod(BinaryOperator<Integer> Lambda) {
    System.out.print("BinaryOperator");
}

private void overloadedMethod(IntegerBiFunction Lambda) {
    System.out.print("IntegerBinaryOperator");
}

@Test
public void overloadMethodTest() {
    overloadedMethod((x, y) -> x + y);
}
//结果输出 IntegerBinaryOperator，这里IntegerBiFunction接口继承了BinaryOperator<Integer>，因此IntegerBiFunction更具体
```

当然，同时存在多个重载方法时，哪个是“最具体的类型”可能并不明确。重载方法导致的编译错误

```java
private interface IntPredicate { 
    public boolean test(int value);
}
private void overloadedMethod(Predicate<Integer> predicate) { 
    System.out.print("Predicate");
}
private void overloadedMethod(IntPredicate predicate) { 
    System.out.print("IntPredicate");
}
//编译报错
overloadedMethod((x) -> true);
//强制指定类型
overloadedMethod((IntPredicate)(x) -> true);

```

传入 `overloadedMethod` 方法的 `Lambda` 表达式和两个函数接口 `Predicate`、`IntPredicate` 在 类型上都是匹配的。在这段代码块中，两种情况都定义了相应的重载方法，这时，`javac` 就无法编译，在错误报告中显示 `Lambda` 表达式被模糊调用。`IntPredicate` 没有继承 `Predicate`，因此编译器无法推断出哪个类型更具体。

将 `Lambda` 表达式强制转换为 `IntPredicate` 或 `Predicate<Integer>` 类型可以解决这个问 题，至于转换为哪种类型则取决于要调用哪个函数接口。当然，如果以前你曾自行设计过 类库，就可以将其视为“代码异味”，不该再重载，而应当开始重新命名重载方法。

`Lambda` 表达式作为参数时，其类型由它的目标类型推导得出，推导过程遵循 如下规则:
- 如果只有一个可能的目标类型，由相应函数接口里的参数类型推导得出;
- 如果有多个可能的目标类型，由最具体的类型推导得出;
- 如果有多个可能的目标类型且最具体的类型不明确，则需人为指定类型。

## 4.4 @FunctionalInterface
前面已讨论过函数接口定义的标准，但未提及 `@FunctionalInterface` 注释。事实上，每个用作函数接口的接口都应该添加这个注释。

这究竟是什么意思呢? `Java` 中有一些接口，虽然只含一个方法，但并不是为了使用 `Lambda` 表达式来实现的。比如，有些对象内部可能保存着某种状态，使用带有一个方法 的接口可能纯属巧合。`java.lang.Comparable` 和 `java.io.Closeable` 就属于这样的情况。

如果一个类是可比较的，就意味着在该类的实例之间存在某种顺序，比如字符串中的字母 顺序。人们通常不会认为函数是可比较的，如果一个东西既没有属性也没有状态，拿什么 比较呢?

一个可关闭的对象必须持有某种打开的资源，比如一个需要关闭的文件句柄。同样，该接口也不能是一个纯函数，因为关闭资源是更改状态的另一种形式。

和 `Closeable` 和 `Comparable` 接口不同，为了提高 `Stream` 对象可操作性而引入的各种新接口，都需要有 `Lambda` 表达式可以实现它。它们存在的意义在于将代码块作为数据打包起来。因此，它们都添加了 `@FunctionalInterface` 注释。

该注释会强制 `javac` 检查一个接口是否符合函数接口的标准。如果该注释添加给一个枚举 类型、类或另一个注释，或者接口包含不止一个抽象方法，`javac` 就会报错。重构代码时， 使用它能很容易发现问题。

## 4.5 二进制接口的兼容性
如第3章开篇所言，`Java 8`中对`API`最大的改变在于集合类。虽然Java在持续演进，但它 一直在保持着向后二进制兼容。具体来说，使用`Java 1`到`Java 7`编译的类库或应用，可以 直接在`Java 8`上运行。

当然，错误也难免会时有发生，但和其他编程平台相比，二进制兼容性一直被视为 `Java` 的关键优势所在。除非引入新的关键字，如 enum，达成源代码向后兼容也不是没有可能实 现。可以保证，只要是 `Java 1` 到 `Java 7` 写出的代码，在 `Java 8` 中依然可以编译通过。

事实上，修改了像集合类这样的核心类库之后，这一保证也很难实现。我们可以用具体的 例子作为思考练习。`Java 8`中为`Collection`接口增加了`stream`方法，这意味着所有实现了 `Collection` 接口的类都必须增加这个新方法。对核心类库里的类来说，实现这个新方法(比如为 `ArrayList` 增加新的 `stream` 方法)就能就能使问题迎刃而解。

缺憾在于，这个修改依然打破了二进制兼容性，在 `JDK` 之外实现 `Collection` 接口的类， 例如`MyCustomList`，也仍然需要实现新增的`stream`方法。这个`MyCustomList`在`Java 8`中 无法通过编译，即使已有一个编译好的版本，在 `JVM` 加载 `MyCustomList` 类时，类加载器仍然会引发异常。

这是所有使用第三方集合类库的梦魇，要避免这个糟糕情况，则需要在`Java 8`中添加新的 语言特性:默认方法

## 4.6 
`Collection` 接口中增加了新的 `stream` 方法，如何能让 `MyCustomList` 类在不知道该方法的情况下通过编译?`Java 8`通过如下方法解决该问题:`Collection`接口告诉它所有的子类: “如果你没有实现 `stream` 方法，就使用我的吧。”接口中这样的方法叫作默认方法，在任何接口中，无论函数接口还是非函数接口，都可以使用该方法。

`Iterable` 接口中也新增了一个默认方法:`forEach`，该方法功能和 `for` 循环类似，但是允许用户使用一个 `Lambda` 表达式作为循环体。

- 默认方法示例:`forEach` 实现方式

```java
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
```

如果已经习惯了通过调用接口方法来使用 `Lambda` 表达式的方式，那么这个例子理解起来就相当简单。它使用一个常规的 `for` 循环遍历 `Iterable` 对象，然后对每个值调用 `accept` 方法。

既然如此简单，为何还要单独提出来呢?重点就在于代码段前面的新关键字 `default`。这 个关键字告诉 `javac` 用户真正需要的是为接口添加一个新方法。除了添加了一个新的关键字，默认方法在继承规则上和普通方法也略有区别。

和类不同，接口没有成员变量，因此默认方法只能通过调用子类的方法来修改子类本身， 避免了对子类的实现做出各种假设。

默认方法和子类
----
默认方法的重写规则也有一些微妙之处。从最简单的情况开始来看:没有重写。在下面的例子中，`Parent` 接口定义了一个默认方法 `welcome`，调用该方法时，发送一条信息。`ParentImpl` 类没有实现 `welcome` 方法，因此它自然继承了该默认方法。

```java
    public interface Parent {
        void message(String body);

        default void welcome() {
            message("Parents:Hi!");
        }
        String getLastMessage();
    }
```

- 在下面例子中，我们调用默认方法，可以看到断言正确。

```java
    @Test
    public void parentDefaultUsed() {
        Parent parent = new ParentImpl();
        parent.welcome();
        assertEquals("Parent: Hi!", parent.getLastMessage());
    }
//解析：实现类ParentImpl并未实现welcome()接口，而是调用接口中的默认方法
```

这时可新建一个接口 `Child`，继承自 `Parent` 接口，代码如下所示。`Child` 接口实现了自己的默认 `welcome` 方法，凭直觉判断可知，该方法重写了 `Parent` 的方法。同样在这个例子中，`ChildImpl` 类不会实现 `welcome` 方法，因此它自然也继承了接口的默认方法。

```java
//Child 继承了 Parent 但是重写了welcome默认方法
    public interface Child extends Parent{
        @Override
        default void welcome(){
            message("Child: Hi!");
        }
    }
```

- 调用 `Child` 接口的客户代码

```java
    @Test
    public void childOverrideDefault() {
        Child child = new ChildImpl();
        child.welcome();
        assertEquals("Child: Hi!", child.getLastMessage());
    }
    //最后输出的字符串自然是 "Child: Hi!"。
```

- 类继承体系如下图

![alt](https://www.zhangaoo.com/upload/2018/10/6u0koqvjlgjr1qpjeahegou10t.png)


现在默认方法成了虚方法——和静态方法刚好相反。任何时候，一旦与类中定义的方法产生冲突，都要优先选择类中定义的方法。例如下展示了这种情况，最终调用的 是 `OverridingParent` 的，而不是 `Parent` 的 `welcome`方法。

```java
// 重写 welcome 默认实现的父类
    public class OverridingParent extends ParentImpl {
        @Override
        public void welcome() {
            message("Class Parent: Hi!");
        }
    }
// 调用的是类中的具体方法，而不是默认方法
    @Test
    public void concreteBeatsDefault() {
        Parent parent = new OverridingParent();
        parent.welcome();
        assertEquals("Class Parent: Hi!", parent.getLastMessage());
    }    
```

下面的代码展示了另一种情况，或许不认为类中重写的方法能够覆盖默认方法。`OverridingChild` 本身并没有任何操作，只是继承了 `Child` 和 `OverridingParent` 中的 `welcome` 方法。最后，调 用的是 `OverridingParent` 中的 `welcome` 方法，而不是 `Child` 接口中定义的默认方法，原因在于，与接口中定义的默认方法相比，类中重写的方法更具体

```java
    public class OverridingChild extends OverridingParent implements Child {
    }

    @Test
    public void concreteBeatsCloserDefault() {
        Child child = new OverridingChild();
        child.welcome();
        assertEquals("Class Parent: Hi!", child.getLastMessage());
    }
```

简言之，类中重写的方法胜出。这样的设计主要是由增加默认方法的目的决定的，增加默认方法主要是为了在接口上向后兼容。让类中重写方法的优先级高于默认方法能简化很多继承问题。

假设已实现了一个定制的列表 `MyCustomList`，该类中有一个 `addAll` 方法，如果新的 `List` 接口也增加了一个默认方法 `addAll`，该方法将对列表的操作代理到 `add` 方法。如果类中重写的方法没有默认方法的优先级高，那么就会破坏已有的实现。

## 4.7 多重继承
接口允许多重继承，因此有可能碰到两个接口包含签名相同的默认方法的情况。比如下面代码中，接口 `Carriage` 和 `Jukebox` 都有一个默认方法 `rock`，虽然各有各的用途。类 `MusicalCarriage` 同时实现了接口 `Jukebox` 和 `Carriage`，它到底继承 了哪个接口的 `rock` 方法呢?

```java
    public interface Jukebox {
        default String rock() {
            return "... all over the world!";
        }
    }

    public interface Carriage {
        public default String rock() {
            return "... from side to side";
        }
    }

    public class MusicalCarriage implements Carriage, Jukebox {
    }
```

此时，`javac` 并不明确应该继承哪个接口中的方法，因此编译器会报错:`class Musical Carriage inherits unrelated defaults for rock() from types Carriage and Jukebox`。当然，在类 中实现 `rock` 方法就能解决这个问题
- 实现 rock 方法

```java
    public class MusicalCarriage implements Carriage, Jukebox {
        @Override
        public String rock() {
            return Carriage.super.rock();
        }
    }
```

该例中使用了增强的 `super` 语法，用来指明使用接口 `Carriage` 中定义的默认方法。此前，使用 `super` 关键字是指向父类，现在使用类似 `InterfaceName.super` 这样的语法指的是继承自父接口的方法。

三定律
----
如果对默认方法的工作原理，特别是在多重继承下的行为还没有把握，如下三条简单的定 律可以帮助大家:

1. 类胜于接口。如果在继承链中有方法体或抽象的方法声明，那么就可以忽略接口中定义的方法。
2. 子类胜于父类。如果一个接口继承了另一个接口，且两个接口都定义了一个默认方法， 那么子类中定义的方法胜出。
3. 没有规则三。如果上面两条规则不适用，子类要么需要实现该方法，要么将该方法声明为抽象方法。

其中第一条规则是为了让代码向后兼容。

## 4.8 权衡
在接口中定义方法的诸多变化引发了一系列问题，既然可用代码主体定义方法，那`Java 8` 中的接口还是旧有版本中界定的代码吗?现在的接口提供了某种形式上的多重继承功能， 然而多重继承在以前饱受诟病，`Java` 因此舍弃了该语言特性，这也正是 `Java` 在易用性方面优于 `C++` 的原因之一。

语言特性的利弊也在不断演化。很多人认为多重继承的问题在于对象状态的继承，而不是代码块的继承，默认方法避免了状态的继承，也因此避免了 `C++` 中多重继承的最大缺点。

突破语言上的局限性吸引着无数优秀的程序员不断尝试。现在已有一些博客文章，阐述在 `Java 8` 中实现完全的多重继承做出的尝试，包括状态的继承和默认方法。尝试突破 `Java 8` 这些有意为之的语言限制时，却往往又掉进 `C++` 的旧有陷阱之中。

接口和抽象类之间还是存在明显的区别。接口允许多重继承，却没有成员变量;抽象类可以继承成员变量，却不能多重继承。在对问题域建模时，需要根据具体情况进行权衡，而在以前的 `Java` 中可能并不需要这样。

## 4.9 接口的静态方法
前面已多次出现过 `Stream.of` 方法的调用，接下来将对其进行详细介绍。`Stream` 是个接口， `Stream.of` 是接口的静态方法。这也是 `Java 8` 中添加的一个新的语言特性，旨在帮助编写类库的开发人员，但对于日常应用程序的开发人员也同样适用。

人们在编程过程中积累了这样一条经验，那就是一个包含很多静态方法的类。有时，类是 一个放置工具方法的好地方，比如 `Java 7` 中引入的 `Objects` 类，就包含了很多工具方法， 这些方法不是具体属于某个类的。

当然，如果一个方法有充分的语义原因和某个概念相关，那么就应该将该方法和相关的类或接口放在一起，而不是放到另一个工具类中。这有助于更好地组织代码，阅读代码的人也更容易找到相关方法。

比如，如果想创建一个由简单值组成的 `Stream`，自然希望 `Stream` 中能有一个这样的方法。 这在以前很难达成，引入重接口的 `Stream` 对象，最后促使 `Java` 为接口加入了静态方法。

__Stream 和其他几个子类还包含另外几个静态方法。特别是 range 和 iterate 方法提供了产生 Stream 的其他方式。__
__静态方法，只能通过接口名调用，不可以通过实现类的类名或者实现类的对象调用。default方法，只能通过接口实现类的对象来调用。__

## 4.10 Optional
`reduce` 方法的一个重点尚未提及: `reduce` 方法有两种形式，一种如前面出现的需要有一个初始值，另一种变式则不需要有初始值。没有初始值的情况下，`reduce` 的第一步使用 `Stream` 中的前两个元素。有时，`reduce` 操作不存在有意义的初始值，这样做就是有意义的，此时，`reduce` 方法返回一个 `Optional` 对象。

`Optional` 是为核心类库新设计的一个数据类型，用来替换 `null` 值。人们对原有的 `null` 值有很多抱怨，甚至连发明这一概念的`Tony Hoare`也是如此，他曾说这是自己的一个“价值连城的错误”。作为一名有影响力的计算机科学家就是这样:虽然连一毛钱也见不到，却也可以犯一个“价值连城的错误”。

人们常常使用 `null` 值表示值不存在，`Optional` 对象能更好地表达这个概念。使用 `null` 代表值不存在的最大问题在于 `NullPointerException`。一旦引用一个存储 `null` 值的变量，程序会立即崩溃。使用 `Optional` 对象有两个目的:首先，`Optional` 对象鼓励程序员适时检查变量是否为空，以避免代码缺陷;其次，它将一个类的 `API` 中可能为空的值文档化，这比阅读实现代码要简单很多。

下面我们举例说明 `Optional` 对象的 `API`，从而切身体会一下它的使用方法。使用工厂方法 `of`，可以从某个值创建出一个 `Optional` 对象。`Optional` 对象相当于值的容器，而该值可以 通过 `get` 方法提取。如下代码所示。

```java
    Optional<String> a = Optional.of("a");
    assertEquals("a",a.get());
```

`Optional` 对象也可能为空，因此还有一个对应的工厂方法 `empty`，另外一个工厂方法 `ofNullable` 则可将一个空值转换成 `Optional` 对象。如下代码展示了这两个方法，同时展示了第三个方法 `isPresent` 的用法(该方法表示一个 `Optional` 对象里是否有值)。

```java
    Optional<String> a = Optional.of("a");
    Optional emptyOptional = Optional.empty();
    Optional alsoEmpty = Optional.ofNullable(null);
    assertFalse(emptyOptional.isPresent());
    assertTrue(a.isPresent());
```

使用 `Optional` 对象的方式之一是在调用 `get()` 方法前，先使用 `isPresent` 检查 `Optional` 对象是否有值。使用 `orElse` 方法则更简洁，当 `Optional` 对象为空时，该方法提供了一个备选值。如果计算备选值在计算上太过繁琐，即可使用 `orElseGet` 方法。该方法接受一个 `Supplier` 对象，只有在 `Optional` 对象真正为空时才会调用。如下代码展示了这两个方法。

- 使用 `orElse` 和 `orElseGet` 方法

```java
    assertEquals("b",emptyOptional.orElse("b"));
    assertEquals("c",emptyOptional.orElseGet(() -> "c"));
```

`orElseGet`用在需复杂计算的情况，因为可以传入一个`Supplier Lambda`表达式

`Optional` 对象不仅可以用于新的 `Java 8 API`，也可用于具体领域类中，和普通的类别无二致。当试图避免空值相关的缺陷，如未捕获的异常时，可以考虑一下是否可使用 `Optional` 对象。

## 4.11 要点回顾
- 使用为基本类型定制的`Lambda`表达式和`Stream`，如`IntStream`可以显著提升系统性能。
- 默认方法是指接口中定义的包含方法体的方法，方法名有`default`关键字做前缀。
- 在一个值可能为空的建模情况下，使用`Optional`对象能替代使用`null`值。

## 4.12 练习
1. 在下面的 `Performance` 接口基础上，添加 `getAllMusicians` 方法，该方法返回包含所有艺术家名字的 `Stream`，如果对象是乐队，则返回每个乐队成员的名字。例如，如果 `getMusicians` 方法返回甲壳虫乐队，则 `getAllMusicians` 方法返回乐队名和乐队成员， 如约翰 · 列侬、保罗 · 麦卡特尼等。

```java
    public interface Performance {
        String getName();

        Stream<Artist> getMusicians();

        default Stream<Artist> getAllMusicians(){
            return  getMusicians().flatMap(artist -> concat(Stream.of(artist), artist.getMembers()));
        }

    }

```
- 一个 `Artist` 对象拆成两个流，一个是`Artist`自身，一个是成员变量 `List<Artist> members`
- `flatMap` 方法可用 `Stream` 替换值，然后将多个 `Stream` 连接成一个 `Stream`
- `java.util.stream.Stream.concat`：the concatenation of the two input streams（把两个输入流连接级联在一起）

2. 根据前面描述的重载解析规则，能否重写默认方法中的 `equals` 或 `hashCode` 方法?
- 下面是官方的回答，理解的意思就是不需要重载

 > No - they are defined on java.lang.Object, and 'class always wins.'

3. 如面的代码所示的 `Artists` 类表示了一组艺术家，重构该类，使得 `getArtist` 方法返回一 个 `Optional<Artist>` 对象。如果索引在有效范围内，返回对应的元素，否则返回一个空 `Optional` 对象。此外，还需重构 `getArtistName` 方法，保持相同的行为。

- 重构前

```java
    public class Artists {
        private List<Artist> artists;

        public Artists(List<Artist> artists) {
            this.artists = artists;
        }

        public Artist getArtist(int index) {
            if (index < 0 || index >= artists.size()) {
                indexException(index);
            }
            return artists.get(index);
        }

        private void indexException(int index) {
            throw new IllegalArgumentException(index +
                    "doesn't correspond to an Artist");
        }

        public String getArtistName(int index) {
            try {
                Artist artist = getArtist(index);
                return artist.getName();
            } catch (IllegalArgumentException e) {
                return "unknown";
            }
        }
    }
```

- 重构后

```java
    public class Artists {
        private List<Artist> artists;

        public Artists(List<Artist> artists) {
            this.artists = artists;
        }

        public Optional<Artist> getArtist(int index) {
            if (index < 0 || index >= artists.size()) {
                return Optional.empty();
            }
            return Optional.of(artists.get(index));
        }

        private void indexException(int index) {
            throw new IllegalArgumentException(index +
                    "doesn't correspond to an Artist");
        }

        public String getArtistName(int index) {
            if (getArtist(index).isPresent()) {
                Artist artist = getArtist(index).get();
                return artist.getName();
            }
            return "unknown";
        }
    }
```

- 参考答案（`getArtistName`可以写的更简洁）

```java
    public String getArtistName(int index) {
        Optional<Artist> artist = getArtist(index);
        return artist.map(Artist::getName)
                     .orElse("unknown");
    }
```
## 4.13 开放练习
审阅工作代码库或熟悉的开源项目代码，找出哪些只包含静态方法的类适合用包含静态方法的接口替代。如有可能，和同事一起讨论，看他们是否赞同你找出的结果。
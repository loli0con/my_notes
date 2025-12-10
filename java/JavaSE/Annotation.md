# 注解
注解是那些<u>插入到源代码中</u>、<u>使用其他工具可以对其进行处理</u>的标签。

注解不会改变程序的编译方式。Java编译器对于包含注解和不包含注解的代码会生成相同的虚拟机指令。

为了能够利用注解，你需要选择一个**处理工具**，然后向代码中插入可被处理的注解，之后运用该处理工具处理这些代码。这种工具被统称为APT（Annotation Processing Tool）。

## 相关API
java.lang.reflect.AnnotatedElement：
* boolean isAnnotationPresent(Class<? extends Annotation> annotationType)：如果该项具有给定类型的注解，则返回true。
* &lt;T extends Annotation> T getAnnotation(Class&lt;T> annotationType)：获得给定类型的注解，如果该项不具有这样的注解，则返回null。
* &lt;T extends Annotation> T[] getAnnotationsByType(Class&lt;T> annotationType)：获得某个可重复注解类型的所有注解，或者返回长度为0的数组。
* Annotation[] getAnnotations()：获得作用于该项的所有注解，包括继承而来的注解。如果没有出现任何注解，那么将返回一个长度为0的数组。
* Annotation[] getDeclaredAnnotations()：获得为该项声明的所有注解，不包含继承而来的注解。如果没有出现任何注解，那么将返回一个长度为0的数组。

java.lang.annotation.Annotation：
* Class<? extends Annotation> annotationType()：返回Class对象，它用于描述该注解对象的注解接口。注意：调用注解对象上的getClass方法返回的是真正的类，而不是接口。
* boolean equals(object other)：如果other是一个实现了与该注解对象相同的注解接口的对象，并且如果该对象和other的所有元素彼此相等。那么返回True。
* int hashCode()：返回一个与 equals 方法兼容、由注解接口名以及元素值衍生而来的散列码。
* String tostring()：返回一个包含注解接口名以及元素值的字符串表示，例如：`@BugReport(assignedTo=[none], severity=0)`。

## 注解语法

### 定义注解
注解是由**注解接口**来定义的，@interface关键字声明并创建了一个注解接口：  
<code>
<i>modifiers</i> @interface <i>AnnotationName</i><br>
{<br>
&nbsp;&nbsp;&nbsp;&nbsp;<i>elementDeclaration</i><sub>1</sub><br>
&nbsp;&nbsp;&nbsp;&nbsp;<i>elementDeclaration</i><sub>2</sub><br>
&nbsp;&nbsp;&nbsp;&nbsp;....<br>
}
</code>

所有的注解接口都隐式地扩展自java.lang.annotation.Annotation接口，这个接口是一个常规接口，不是一个注解接口。

每个**元素**声明都具有下面这种形式，这些元素可以被读取并加以处理：  
<code>
<i>type</i> <i>elementName</i>();
</code>  
或者  
<code>
<i>type</i> <i>elementName</i> default <i>value</i>;
</code>

注解接口中的方法与注解中的元素相对应。注解接口的方法没有参数、也没有throws子句，它们不能是default或static方法，也不能有类型参数。

注解元素的类型为下列之一：
* 基本类型（int、short、long、byte、char、double、float或者boolean）。
* String。
* Class（具有一个可选的类型参数，例如 Class<? extends MyClass>）。
* enum类型。
* 注解类型。
* 由前面所述类型组成的数组（由数组组成的数组不是合法的元素类型）。

下面是一些合法的元素声明的例子：
```Java
public @interface BugReport {
    // 定义一个enum类型
    enum Status {UNCONFIRMED, CONFIRMED, FIXED, NOT_A_BUG};

    // 基本类型int
    int severity();
    
    // 基本类型boolean
    boolean showStopper() default false;

    // String类型
    String assignedTo() default "[none]";

    // Class类型
    Class<?> testCase() default Void.class;

    // enum类型
    Status status() default Status.UNCONFIRMED;

    // 注解类型
    Reference ref() default @Reference;

    // 由String类型组成的数组
    String[] reportedBy();

    // 由数组组成的数组（矩阵）不是一个合法的元素类型
    // String[][] matrix();
}

@interface Reference {
    String id() default "";
}
```

### 使用注解
在Java中，注解是当作修饰符来使用的，它被置于被注解项之前，中间没有分号。

每个注解都具有下面这种格式：  
<code>
@<i>AnnotationName</i>(<i>elementName</i><sub>1</sub>=<i>value</i><sub>1</sub>,&nbsp;<i>elementName</i><sub>2</sub>=<i>value</i><sub>2</sub>, ...)
</code>

元素的顺序无关紧要，下面两个注解是等价的：
```Java
@BugReport(assignedTo="Harry", severity=10)
@BugReport(severity=10, assignedTo="Harry")
```

如果某个元素的值并未指定，那么就使用声明时提供的默认值。

有两个特殊的方式可以用来简化注解：
* **标记注解**：如果注解中没有任何元素，或者所有元素都使用默认值，那么此时就不需要使用圆括号了。
* **单值注解**：如果一个元素具有特殊的名字value，并且在注解中没有指定其他元素，那么你就可以忽略掉这个元素名以及等号。

一个项可以有多个注解。如果注解被声明为可重复的，那么你就可以对同一个项多次重复使用同一个注解。

注解是由编译器计算而来的，所有元素值必须是编译期常量。

一个注解元素永远不能设置为null，甚至不允许其默认值为null。这样在实际应用中会相当不方便。你必须使用其他的默认值，例如""或者Void.class。

如果元素值是一个数组，那么要将它的值用括号括起来。如果该数组元素只有一个值，那么可以忽略括号。

既然一个注解元素可以是另一个注解，那么就可以创建出任意复杂的注解。例如：
```Java
@BugReport(ref=@Reference(id="3352627"), ...)
```

在注解中引入循环依赖是一种错误：因为BugReport具有一个注解类型为Reference的元素，所以Reference就不能再拥有一个类型为BugReport的元素。

### 注解的声明处
注解可以出现在许多地方，这些地方可以分为两类：声明和类型用法。

声明注解可以出现在下列声明处：
* 包
* 类（包括enum）
* 接口（包括注解接口）
* 方法
* 构造器
* 实例域（包含enum常量）
* 局部变量
* 参数变量
* 类型参数

对于类和接口，需要将注解放置在class和interface关键词的前面：
```Java
@Entity public class User { ... }
```

对于变量，需要将它们放置在类型的前面：
```Java
// 局部变量
@SuppressWarnings("unchecked") List<User> users = ...;

// 参数变量
public User getUser(@Param("id") String userId)
```

泛型类或方法中的类型参数可以像下面这样被注解：
```Java
// 泛型类的类型参数V被@Immutable所注解
public class Cache<@Immutable V> { ... }
```

包是在文件package-info.java中注解的，该文件只包含以注解先导的包语句。
```Java
/**
   Package-level Javadoc
*/
@GPL(version="3")
package com.horstmann.corejava;
import org.gnu.GPL;
```

对局部变量的注解只能在源码级别上进行处理。类文件并不描述局部变量。因此，所有的局部变量注解在编译完一个类的时候就会被遗弃掉。同样地，对包的注解不会在源码级别之外存在。

### 注解的类型用法
在Java8以前，只能在定义各种程序元素（定义类、定义接口、定义方法、定义成员变量...）时使用注解。从Java8开始，ElementType枚举增加了枚举值TYPE_USE，这种注解被称为**类型用法注解**，它可以在任何用到类型的地方使用。

现在，假设我们有一个类型为List&lt;String>的参数，并且想要表示其中所有的字符串都不为null。这就是类型用法注解大显身手之处，可以将该注解放置到类型参数之前：List<@NonNull String>。

类型用法注解可以出现在下面的位置：
* 与泛型类型参数一起使用：
  * List<@NonNull String>
  * Comparator.<@NonNull String>reverseOrder()
* 数组中的任何位置：
  * @NonNull String\[]\[] words（words\[i]\[j]不为null）
  * String @NonNull \[]\[] words（words不为null）
  * String\[] @NonNull \[] words（words\[i]不为null）
* 与超类和实现接口一起使用：
  * class Warning extends @Localized Message
  * class Warning implements @NonNull Serializable
* 与构造器调用一起使用：new @Localized String(...)
* 与强制转型和instanceof检査一起使用（这些注解只供外部工具使用，它们对强制转型和instanceof检査不会产生任何影响）：
  * (@Localized String) text
  * if (text instanceof @Localized String)
* 与异常规约一起使用：public String read() throws @Localized IOException
* 与通配符和类型边界一起使用：
  * List<@Localized ? extends Message>
  * List<? extends @Localized Message>
* 与方法和构造器引用一起使用：@Localized Message::getText

有多种类型位置是不能被注解的：
* @NonNull String.class // ERROR: Cannot annotate class literal
* import java.lang.@NonNull String; // ERROR: Cannot annotate import

可以将注解放置到诸如private和static这样的其他修饰符的前面或后面。习惯（但不是必需）的做法，是将类型用法注解放置到其他修饰符的后面和将声明注解放置到其他修饰符的前面。例如：
* private @NonNull String text; // Annotates the type use
* @Id private String userId; // Annotates the variable

如果一个注解可以同时应用于变量和类型用法，并且它确实被应用到了某个变量声明上，那么该变量和类型用法就都被注解了。例如，请考虑`public User getUser(@NonNull String userId)`，如果@NonNull可以同时应用于参数和类型用法，那么userId参数就被注解了，而其参数类型是@NonNull String。

### 注解this
可以用一种很少使用的语法变体来声明this，然后就可以添加注解了：
```Java
public class Point
{
    public boolean equals(@Readonly Point this, @Readonly Object other) { ... }
}
```
第一个参数被称为接收器参数，它必须被命名为this，而它的类型就是要构建的类。

传递给内部类构造器的是另一个不同的隐藏参数，即对其外围类对象的引用。你也可以让这个参数显式化：
```Java
public class Sequence
{
    private int from;
    private int to;

    // 实例内部类
    class Iterator implements java.util.Iterator<Integer>
    {
        private int current;
        // Sequence.this是指向外部类实例的引用
        public Iterator(@ReadOnly Sequence Sequence.this)
        {
            this.current = Sequence.this.from;
        }
        ...
    }
    ...
}
```
这个参数的名字必须像引用它时那样，叫作EnclosingClass.this，其类型为外围类。

## 标准注解
java.lang、java.lang.annotation和javax.annotation包中定义了大量的注解接口。其中四个是元注解，用于描述注解接口的行为属性，其他的是规则接口，可以用它们来注解你的源代码中的项。

|注解接口|应用场合|目的|
|---|---|---|
|Deprecated|全部|将项标记为过时的|
|SuppressWarnings|除了包和注解之外的所有情况|阻止某个给定类型的警告信息|
|SafeVarargs|方法和构造器|断言varargs参数可安全使用|
|Override|方法|检查该方法是否覆盖了某一个超类方法|
|Serial|方法|检查该方法是不是正确序列化的方法|
|FunctionalInterface|接口|将接口标记为只有一个抽象方法的函数式接口|
|Generated|全部|将项标记为由某个工具生成的源代码|
|Target|注解|指明这个注解可以应用到哪些项上|
|Retention|注解|指明这个注解可以保留多久|
|Documented|注解|指明这个注解应该包含在被注解项的文档中|
|Inherited|注解|指明当这个注解应用于一个类的时候，能够自动被它的子类继承|
|Repeatable|注解|指明这个注解可以在同一个项上应用多次|

### 用于编译的注解
@Deprecated注解可以被添加到任何不再鼓励使用的项上。所以，当你使用一个已过时的项时，编译器将会发出警告。该注解会一直持久化到运行时。

@SuppressWarnings注解会告知编译器阻止特定类型的警告信息，例如：`@SuppressWarnings("unchecked")`。

@Override这种注解只能应用到方法上。编译器会检査具有这种注解的方法是否真正覆盖了一个来自超类的方法。如果没有覆盖一个来自超类的方法，那么编译器会报告一个错误。

@Generated注解的目的是供代码生成工具来使用。任何生成的源代码都可以被注解，从而与程序员提供的代码区分开。每个注解都必须包含一个表示代码生成器的唯一标识符，而日期字符串（ISO8601格式）和注释字符串是可选的。例如：`@Generated("com.horstmann.beanproperty", "2008-01-04T12:08:56.235-0700");`。

### 元注解
@Target元注解可以应用于一个注解，以限制该注解可以应用到哪些项上。例如：
```Java
@Target(｛ElementType.TYPE, ElementType.METHOD｝)
public @interface BugReport{ ... }
```

下表显示了所有可能的取值情况，它们属于枚举类型ElementType。可以指定任意数量的元素类型，用括号括起来。

|元素类型|注解适用场台|
|---|---|
|ANNOTATION_TYPE|注解类型声明|
|PACKAGE|包|
|TYPE|类（包括enum）及接口（包括注解类型，即TYPE包括ANNOTATION_TYPE）|
|METHOD|方法|
|CONSTRUCTOR|构造器|
|FIELD|成员域（包括enum常量）|
|PARAMETER|方法或构造器参数|
|LOCAL_VARIABLE|局部变最|
|TYPE_PARAMETER|类型参数|
|TYPE_USE|类型用法|

一条没有@Target限制的注解可以应用于任何项上。编译器将检査你是否将一条注解只应用到了某个允许的项上。

@Retention元注解用于指定一条注解应该保留多长时间。只能将其指定为下表中的任意值，其默认值是RetentionPolicy.CLASS。

|保留规则|描述|
|---|---|
|SOURCE|源代码级别。不包括在类文件中的注解。|
|CLASS|字节码级别。包括在类文件中的注解，但是虚拟机不需要将它们载入。|
|RUNTIME|运行时级别。包括在类文件中的注解，并由虚拟机载入。通过反射API可获得它们。|

@Documented元注解为像Javadoc这样的归档工具提供了一些提示。应该像处理其他修饰符 （例如protected和static） 一样来处理归档注解，以实现其归档目的。其他注解的使用并不会 
纳入归档的范畴。

假定我们将@ActionListenerFor作为一个归档注解来声明：
```Java
@Documented // 将@ActionListenerFor声明为一个归档注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME) 
public @interface ActionListenerFor{ ... }
```

现在每一个被@ActionListenerFor注解标注过的方法的归档就会含有这条注解，如下图所示。

![20251209155647](https://raw.githubusercontent.com/loli0con/picgo/master/20251209155647.png)

@Inherited元注解只能应用于对类的注解。如果一个类具有继承注解，那么它的所有子类都自动具有同样的注解。

```Java
@Inherited @interface Persistent { } // 使用@Inherited来声明@Persistent
@Persistent class Employee { ... } // 在父类上使用@Persistent
class Manager extends Employee { ... } // 子类上同样具有@Persistent的效果
```

对于Java 8来说，将同种类型的注解多次应用于某一项是合法的。为了向后兼容，可重复注解的实现者需要提供一个容器注解，它可以将这些重复注解存储到一个数组中。

下面是如何定义@TestCase注解以及它的容器的代码：
```Java
// 定义可重复注解@TestCase，并提供容器注解@TestCases
@Repeatable(TestCases.class)
@interface TestCase
{
    String params(); 
    String expected();
}

@interface TestCases
{
    TestCase[] value();
}
```

无论何时，只要用户提供了两个或更多个@TestCase注解，那么它们就会自动地被包装到一个@TestCases注解中。

在处理可重复注解时必须非常仔细。如果调用getAnnotation来查找某个可重复注解，而该注解又确实重复了，那么就会得到null。这是因为重复注解被包装到了容器注解中。在这种情况下，应该调用getAnnotationsByType。这个调用会“遍历”容器，并给出一个重复注解的数组。如果只有一条注解，那么该数组的长度就为1。通过使用这个方法，你就不用操心如何处理容器注解了。
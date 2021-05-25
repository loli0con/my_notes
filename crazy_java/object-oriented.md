# 面向对象

## 🧩零散知识点
### 三大特征
面向对象的三大特征：封装、继承、多态
***

### 类型
Java里面有两种类型，基本类型和引用类型，变量可以指向这两种类型。
***

### 类中的成员
成员变量、初始化块、构造器、方法、内部类

属性：指的是一组setter方法和getter方法
***

### JavaBean
如果一个Java类的每个实例变量都被使用private修饰，并为每个实例变量都提供了public修饰的setter方法和getter方法，那么这个类就是一个符合JavaBean规范的类。
***

### this
this关键字总是指向调用该方法的对象。
***

### super
super用于限定该对象调用它从父类继承得到的实例变量或方法。为了在子类方法中访问父类中定义的、被隐藏的实例变量，或为了在子类方法中调用父类中定义的、被覆盖的（Override）的方法，可以通过`super.`作为限定来调用这些实例变量和实例方法。

[super关键字代表的就是“当前对象”的那部分父类型特征。](https://blog.csdn.net/MyxZxd/article/details/105891967#t8)

[关于超类：为什么super.super.method(); 在Java中不允许？](https://www.codenong.com/586363/)
***


### final
final关键字可用于修饰类、变量和方法，用于表示它修饰的类、变量和方法不可改变。
#### 成员变量
final成员变量必须由程序员显式初始化，系统不会对final成员进行隐式初始化。
***

### 运行流程
**纯个人理解，不一定正确！**
#### 类
1. 加载类
   1. 加载父类 *（注意此处会递归）*
   2. 给**类变量**和**方法**分配内存空间
2. 按照**类初始化块**和**显式初始化值**的顺序，先后执行初始化流程

#### 对象
1. 实例化对象
   1. 实例化父类的对象 *（注意此处会递归）*
   2. 给**实例变量**分配内存空间
2. 初始化
   1. 初始化父类的对象 *（注意此处会递归）*
   2. 按照**初始化块**和**显式初始化值**的顺序，先后执行初始化流程
   3. 执行构造器


***
## 修饰符
### 访问控制修饰符
|作用范围|类中|包中|子类|全局|
|-----|-----|-----|-----|-----|
|从小到大|private|~~default~~|protected|public|

### 静态 static
static用于区分成员变量、初始化块、方法、内部类这四种成员到底属于类本身还是属于实例。static修饰的成员表明它属于这个类本身，而不属于该类的单个实例，因此通常把static修饰的成员变量和方法称为类变量、类方法，把不使用static修饰的成员变量和方法称为实例变量、实例方法。

**静态成员不能访问非静态成员！！！**

### 抽象 abstract
归纳起来抽象类可用“有得有失”四个字来描述：
* “得”指的是抽象类得到了一个能力：抽象类可以包含抽象方法
* “失”指的是抽象类失去了一个能力：抽象类不可用于创建实例

定义抽象方法只需在普通方增加上abstract修饰符，并把普通方法的方法体全部去掉，并在方法后增加分号即可。

#### 注意事项
* abstract和final不能同时使用
* abstract和static不能同时修饰某个方法
* abstract和private不能同时修饰某个方法



***
## 方法
### 参数传递机制
Java里方法的参数传递方式只有一种：值传递。
所谓值传递，就是将实际参数值的副本（复制品）传入方法内，而参数本身不会受到影响。
### 方法重载 overload
如果同一个类中包含了两个或者两个以上方法的方法名相同，但形参列表不同，则被称为方法重载。
### 方法重写 override
方法的重写要遵循“两同两小一大”规则，“两同”即方法名相同、形参列表相同；“两小”指的是子类方法返回值类型应比父类方法返回值类型更小或相等，子类方法声明抛出的异常类型比父类方法声明抛出的异常类型更小或相等；“一大”指的是子类方法的访问权限应该比父类方法的访问权限更大或相等。



***
## 包
### package
Java包机制需要两个方面的保证
1. 源文件里使用package语句指定包名
2. class文件必须放在对应的路径下

同一个包中的类不必位于相同的目录下，只要让CLASSPATH环境变量里包含这两个路径即可。jvm会自动搜索CLASSPATH下的子路径，把它们当成用一个包下的类来处理。

### import
import可以向某个Java文件中导入指定包层次下某个类或者全部类：
* import package.subpackage...Classname
* import package.subpackage...*

### import static 静态导入
import static用于导入指定类的某个静态成员变量、方法或全部的静态成员变量、方法：
* import static package.subpackage...ClassName.fieldName|methodName
* import static package.subpackage...ClassName.*


***
## 接口
接口是从多个相似类中抽象出来的规范，使用interface关键字定义接口。  
接口定义的是多个类共同的行为规范，这些行为是与外部交流的通道，这意味着接口里通常是一组公用方法。

### 基本语法
```
[修饰符] interface 接口名 extends 父接口1, 父接口2...
{
    零到多个常量定义...
    零到多个抽象方法定义...
    零到多个内部类、接口、枚举定义...
    零到多个默认方法或类方法定义...
}
```
#### 详细说明
* 接口本身的修饰符可以是public或~~default~~
* 接口成员只能使用public访问控制修饰符
* 接口里的成员变量总是使用public static final这三个修饰符来修饰
* 使用default修饰默认方法

[具体用法](https://www.runoob.com/java/java8-default-methods.html)

### 接口和抽象类的区别
接口作为系统与外界交互的窗口，接口体现的是一种规范。对于接口的实现者而言，接口规定了实现者必须向外提供哪些服务（以方法的形式来提供）；对于接口的调用者而言，接口规定了调用者可以调用哪些服务，以及如何调用这些服务（就是如何来调用方法）。当在一个程序中使用接口时，接口是多个模块间的耦合标准；当在多个应用程序之间使用接口时，接口是多个程序之间的通信标准。

从某种程度上来看，接口类似于整个系统的“总纲”，它制定了系统各模块应该遵循的标准，因此一个系统中的接口不应该经常改变。一旦接口被改变，对整个系统甚至其他系统的影响将是辐射式的，导致系统中大部分类都需要改写。

抽象类则不一样，抽象类作为系统中多个子类的共同父类，它所体现的是一种模板式设计。抽象类作为多个子类的抽象父类，可以被当成系统实现过程中的中间产品，这个中间产品已经实现了系统的部分功能（那些已经提供实现的方法），但这个产品依然不能当成最终产品，必须有更进一步的完善，这种完善可能有几种不同方式。


***
## 内部类
内部类比外部类可以多使用三个修饰符：private、protected、static。  
内部类的上一级程序单元是外部类，它具有4个作用域：同一个类、同一个包、父子类和任何位置，因此可以使用4种访问控制权限。

内部类：
* 成员内部类
  * 静态内部类
  * 非静态内部类
* 局部内部类
* 匿名内部类

### 非静态内部类
在非静态内部类对象里，保存了一个它所寄生的外部类对象的引用，因此在非静态内部类里可以直接访问外部类的private成员，但是反过来就不成立了。非静态内部类只在非静态内部类范围内是可知的，并不能被外部类直接使用。

如果外部类的成员变量、内部类成员变量与内部类里方法的局部变量重名，可通过使用this、外部类类名.this作为限定来区分。

Java不允许在非静态内部类里定义静态成员。

### 静态内部类
使用static修饰的内部类被称为类内部类，也称为静态内部类。静态内部类可以包含静态成员，也可以包含非静态成员。

静态内部类是外部类的*类相关*的，而不是外部类的*对象相关*的。也就是说，静态内部类对象不是寄生在外部类的实例中，而是寄生在外部类的类本身中。静态内部类对象只持有外部类的类引用，没有持有外部类对象的引用。

外部类不能直接访问静态内部类的成员，但可以使用静态内部类的类名作用调用者来访问静态内部类的类成员，也可以使用静态内部类对象作为调用者来访问静态内部类的实例成员。

#### 关于接口
Java还允许在接口里定义内部类，接口里定义的内部类默认且只能使用public static修饰。

接口里还能定义内部接口，同上默认且只能添加public static修饰符，但这么做的意义不大。

### 使用成员内部类
#### 定义变量
在外部类以外的地方定义内部类（包括静态和非静态两种）变量的语法格式如下：
```Java
OuterClass.InnerClass varName
```
在外部类以外的任何地方使用内部类时，内部类完整的类名应该是`OuterClass.InnerClass`。如果外部类有包名，则还应该增加包名前缀。

#### 非静态内部类
*仅叙述在外部类以外使用内部类的场景*

由于非静态内部类的对象必须寄生在外部类的对象里，因此创建非静态内部类对象之前，必须先创建其外部类对象。在外部类以外的地方创建非静态内部类实例的语法如下：
```Java
OuterInstance.new InnerConstructor()
```
在外部类以外的地方创建非静态内部类实例必须用外部类实例和new来调用非静态内部类的构造器。

当创建一个子类时，子类构造器总会调用父类的构造器，因此在创建非静态内部类的子类时，必须保证让子类的构造器可以调用非静态内部类的构造器，调用非静态内部类的构造器时，必须存在一个外部类对象。下面程序定义了一个子类继承了Out类的非静态内部类In类：
```Java
public class SubClass extends Out.In
{
   // 显式定义SubClass的构造器
   public SubClass(Out out)
   {
      // 通过传入的Out对象显式调用In的构造器
      out.super("实参～～～");
   }
}
```
**切记如果一个非静态内部类的子类的对象存在，则一定存在与之对应的外部类对象！！！**

#### 静态内部类
因为静态内部类是外部类*类相关*的，因此创建静态内部类对象时，无需创建外部类对象。在外部类以外的地方创建静态内部类实例的如下：
```Java
new OuterClass.InnerConstructor()
```

调用静态内部类的构造器时无须使用外部类对象，所以创建静态内部类的子类也比较简单：
```Java
public class StaticSubClass extends StaticOut.StaticIn{}
```

### 局部内部类（鸡肋）
如果把一个类放在方法里定义，则这个内部类就是一个局部内部类，局部内部类仅在该方法里有效。由于局部内部类不能在外部类的方法以外的地方使用，因此局部内部类也不能使用访问控制符和static修饰符修饰。如果需要用局部类定义变量、创建实例或者派生子类，那么都只能在局部类虽在的方法类进行。


### 匿名内部类
定义匿名内部类的格式如下：
```Java
// 必须且只能 ➡️ 继承一个父类或者实现一个接口
// 不能是抽象类，不能定义构造器
new 实现接口() | 父类构造器(实参列表)
{
   //匿名内部类的类体部分
}
```
创建匿名内部类时会立刻创建一个该类的实例，这个类定义立即消失，匿名内部类不能重复使用，所以它适合创建那种只需要使用一次的类。

如果局部变量被匿名内部类访问，那么该局部变量相当于自动使用了final修饰。


***
## Lambda表达式

### 基本概念
Lambda表达式支持将代码块作为方法参数。  
Lambda表达式就相当于一个匿名方法。  
**Lambda表达式的主要作用就是代替匿名内部类的烦琐语法。**  
Lambda表达式允许使用更简洁的代码来创建只有一个抽象方法的接口的实例。  
Lambda表达式由三部分构成：
* 形参列表
* 箭头（->）
* 代码块

### 函数式接口
函数式接口代表只包含一个抽象方法的接口。函数式接口可以包含多个默认方法、类方法，但只能声明一个抽象方法。

Lambda表达式实际上将会被当成一个“任意类型”的对象，到底需要当成何种类型的对象，这将取决于运行环境。Lambda表达式的类型，也被称为“目标类型”。

如果采用匿名内部类语法来创建函数式接口的实例，则只需要实现一个抽象方法，在这种情况下即可采用Lambda表达式来创建对象，该表达式创建出来的对象的目标类型就是这个函数式接口。

#### 注意事项
Lambda表达式的目标类型必须明确是函数式接口，否则代码会报错:
```Java
Object obj1 = () -> System.out.println("Hello world"); 
}
```
将Lambda表达式赋值给Object变量，编译器只能确定该Lambda表达式的类型为Object，而Object并不是函数式接口。可进行如下修改：
```Java
Object obj2 = (Runnable) () -> System.out.println("Hello world");
```
使用函数式接口对Lambda表达式进行强制类型转换后，这样就可以确定该表达式的目标类型为Runable函数式接口。

Lambda表达式的目标类型完全可能是变化的，唯一的要求是Lambda表达式实现的匿名方法与目标类型（函数式接口）中唯一的抽象方法有相同的形参列表。
```Java
@FunctionalInterface
interface FKTest {
    void run();
}

Object obj3 = (FKTest) () -> System.out.println("Hello world");
```

### 方法引用与构造器引用
方法引用和构造器引用可以让Lambda表达式的代码块更加简洁。
方法引用和构造器引用都需要使用两个英文冒号。
![object-oriented+20210525181706](https://raw.githubusercontent.com/loli0con/picgo/master/images/object-oriented%2B20210525181706.png%2B2021-05-25-18-17-08)

```Java
import javax.swing.*;

@FunctionalInterface
interface Converter {
    Integer convert(String from);
}

@FunctionalInterface
interface MyTest {
    String test(String a, int b, int c);
}

@FunctionalInterface
interface YourTest{
    JFrame win(String title);
}

public class Test {
    public static void main(String[] args) {
        // 引用类方法
        // Converter converter1 = from -> Integer.valueOf(from);
        Converter converter1 = Integer::valueOf;
        Integer val = converter1.convert("99");
        System.out.println(val);

        // 引用特定对象的实例方法
        // Converter converter2 = from -> "Hello world".indexOf(from);
        Converter converter2 = "Hello world"::indexOf;
        Integer value = converter2.convert("world");
        System.out.println(value);

        // 引用某类对象的实例方法
        // MyTest mt = (a,b,c) -> a.substring(b,c);
        MyTest mt = String::substring;
        String str = mt.test("Java I Love tou",2,9);
        System.out.println(str);

        // 引用构造器
        // YourTest yt = (String a) -> new JFrame(a);
        YourTest yt = JFrame::new;
        JFrame jf = yt.win("我的窗口");
        System.out.println(jf);
    }
}
```



***
## 枚举类
枚举类使用enum关键字定义，是一个特殊的Java类，和普通的类有一些区别：
* 枚举类默认继承了java.lang.Enum类
* 非抽象的枚举类默认会使用final修饰
* 枚举类默认且只能使用private修饰
* 枚举类的所有实例必须在枚举类的第一行显式列出，系统会自动添加public static final修饰

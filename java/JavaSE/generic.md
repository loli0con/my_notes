# 泛型

## 参考链接
https://blog.csdn.net/s10461/article/details/53941091  
https://blog.csdn.net/qq_27093465/article/details/73229016  
https://www.cnblogs.com/jingmoxukong/p/12049160.html  
https://juejin.cn/post/6844903650788114439

## 背景
在没有泛型之前，一旦把一个对象“丢进”Java集合中，集合就会忘记对象的类型，把所有的对象当成Object类型处理。当程序从集合中取出对象后，就需要进行强制类型转换，这种强制类型转换不仅使代码臃肿，而且容易引起ClassCastExeception异常。

增加了泛型支持后的集合，完全可以记住集合中元素的类型，并可以在编译时检查集合中元素的类型，如果试图向集合中添加不满足类型要求的对象，编译器就会提示错误。增加泛型后的集合，可以让代码更加简洁，程序更加健壮（Java泛型可以保证如果程序在编译时没有发出警告，运行时就不会产生ClassCastException异常）。

## 泛型的定义
泛型，即“**参数化类型（parameterized type）**”，就是允许在定义类、接口、方法时使用**类型形参**，这个**类型形参**（或叫泛型）将在声明变量、创建对象、调用方法时动态地指定（即传入实际的类型参数，也可称为**类型实参**）。

如果没有为泛型指定实际的类型（实参），此时该泛型被称为raw type（原始类型），默认是声明该泛型形参时（形参）指定的第一个上限类型。

## 泛型的意义

### 代码复用
泛型适用于多种数据类型执行相同的代码（代码复用）
```Java
// 冗余的代码
private static int add(int a, int b) {
    System.out.println(a + "+" + b + "=" + (a + b));
    return a + b;
}

private static float add(float a, float b) {
    System.out.println(a + "+" + b + "=" + (a + b));
    return a + b;
}

private static double add(double a, double b) {
    System.out.println(a + "+" + b + "=" + (a + b));
    return a + b;
}

// 通过泛型实现代码复用
private static <T extends Number> double add(T a, T b) {
    System.out.println(a + "+" + b + "=" + (a.doubleValue() + b.doubleValue()));
    return a.doubleValue() + b.doubleValue();
}
```

### 类型安全
泛型中的类型在使用时指定，不需要强制类型转换（编译器会在**编译时**检查类型），可以避免繁琐的**强制转换**以及随之而来的`ClassCastException`

```Java
// 不安全
List list = new ArrayList();
list.add("xxString");
list.add(100d);
list.add(new Person());
/*
list中的元素都是Object类型（无法约束其中的类型），所以在取出集合元素时需要人为的强制类型转化到具体的目标类型，且很容易出现java.lang.ClassCastException异常
*/

// 类型安全
List<String> list = new ArrayList<String>();
/*
引入泛型，提供编译前的检查，使list中只能放String
*/
```

## 泛型的使用

### 泛型类
```Java
class Point<T>{         // 此处可以随便写标识符号，T是type的简称  
    private T var ;     // var的类型由T指定，即：由外部指定  
    public T getVar(){  // 返回值的类型由外部决定  
        return var ;  
    }  
    public void setVar(T var){  // 设置的类型也由外部决定  
        this.var = var ;  
    }  
}  
public class GenericsDemo06{  
    public static void main(String args[]){  
        Point<String> p = new Point<String>() ;     // 里面的var类型为String类型  
        p.setVar("it") ;                            // 设置字符串  
        System.out.println(p.getVar().length()) ;   // 取得字符串的长度  
    }  
}
```

### 泛型接口
```Java
interface Info<T>{        // 在接口上定义泛型
    public T getValue();  // 定义抽象方法，抽象方法的返回值就是泛型类型
}

// 如果在继承泛型接口时未传入泛型实参，那么该过程与定义泛型类相似，在声明类的时候，需将泛型的声明也一起加到类中。
class InfoImpl<T> implements Info<T>{
    private T value;
    public InfoImpl(T value){
        this.setValue(value);
    }
    public void setValue(T value){
        this.value = value;
    }
    public T getValue(){
        return this.value;
    }
}

// 如果在继承泛型接口时传入类型参数，那么该实现类在所有使用了泛型的地方都要替换成传入的实参类型。
class StringInfoImpl implements Info<String>{
    private String value;
    public StringInfoImpl(String value){
        this.setValue(value);
    }
    public void setValue(String value){
        this.value = value;
    }
    public String getValue(){
        return this.value;
    }
}

public class GenericsDemo24{  
    public static void main(String arsg[]){  
        Info<String> i = null;        // 声明接口对象  
        i = new InfoImpl<String>("汤姆") ;  // 通过子类实例化对象  
        System.out.println("内容：" + i.getVar()) ;  
    }  
}
```

### 泛型方法
泛型方法，是在调用方法的时候指明泛型的具体类型。

#### 定义
定义泛型方法语法格式：
![generic+20220317110604](https://raw.githubusercontent.com/loli0con/picgo/master/images/generic%2B20220317110604.png%2B2022-03-17-11-06-05)

定义泛型方法时，必须在返回值前边加一个<T>，来声明这是一个泛型方法，持有一个泛型T，然后才可以用泛型T作为方法的返回值。

调用泛型方法语法格式：
![generic+20220317110621](https://raw.githubusercontent.com/loli0con/picgo/master/images/generic%2B20220317110621.png%2B2022-03-17-11-06-21)

#### 调用
当调用一个泛型方法时，在方法名前的尖括号中放人具体的类型：

```java
class ArrayAlg{
    public static <T> T getMiddle(T... a){
        return a[a.length / 2];
    }
}

String middle = ArrayAlg.<String>getMiddle("]ohnM", "Q.n", "Public");
```

大多数情况下，方法调用中可以省略类型参数，编译器有足够的信息能够推断出所调用的方法。


#### 优点
为什么要使用泛型方法呢？因为泛型类要在实例化的时候就指明类型，如果想换一种类型，不得不重新new一次，可能不够灵活；而泛型方法可以在调用的时候指明类型，更加灵活。


## 类型变量的限定
同一种泛型可以对应多个版本（因为参数类型是不确定的），不同版本的泛型类实例是不兼容的。

举个例子：定义`List<Object>`，传入`List<Number>`，将会编译报错。

为了表示各种泛型List的父类，可以使用类型通配符，**类型通配符**是一个问号（?），它可以匹配任何元素类型。`List<?>`在**逻辑上**是`List<String>`、`List<Integer>`等所有`List<具体类型实参>`的父类。

一个类型变量或通配符可以有多个限定，限定类型用“ & ”分隔，而逗号用来分隔类型变量。在Java的继承中，可以根据需要拥有多个接口超类型，但限定中至多有一个类。如果用一个类作为限定，它必须是限定列表中的第一个。

## 上下界
可以使用上/下界通配符来缩小类型参数的类型范围，例如：
`<? extends Number>` 或 `<T super Integer>`。

注意：
* 上界通配符和下界通配符不能同时使用。
* 上界可设置多个，`<T extends B1 & B2 & B3>`，第一个类型参数（B1）可以是类或接口，其他类型参数（B2、B3···）只能是接口。

### 进&出
指定通配符的上下界，就是为了支持类型型变，[协变只出不进，逆变只进不出！](https://juejin.cn/post/6844903650788114439#heading-11)
* 比如Foo是Bar的子类，这样的`A<Bar>`就相当于`A<? extends Foo>`的子类，可以将`A<Bar>`赋值给`A<? extends Foo>`类型的变量，这种型变方式被称为协变。
  * **出**：假定A类型是一个容器，此处的问号（?）代表一个未知的类型，但是这个未知类型一定是Foo的子类型，取出来的对象一定可以隐式转换为其基类Foo。
  * **进**：假定A类型是一个容器，程序无法确定容器能容纳的对象的真实类型，所以程序无法将任何对象“丢进”。唯一的例外是null，它是所有引用类型的实例。
* 比如Foo是Bar的子类，当程序需要一个`A<? super Bar>`的变量时，程序可以将`A<Foo>`赋值给`A<? super Bar>`类型的变量，这种方式称为逆变。
  * **进**：假定A类型是一个容器，编译器知道容器的下限的父类型Bar，根据继承链（隐式转换）可知，可以用Bar类及其父类型的变量去引用Foo类型及其子类型的对象。
  * **出**：假定A类型是一个容器，编译器无法确定从容器中取出的到底是哪个父类的对象，因此取出来的对象只能被当成Object类型处理。



## 编译和运行
### 类型擦除
Java语言泛型在设计的时候为了兼容原来的旧代码，Java的泛型机制使用了“擦除”机制。

无论何时定义一个泛型类型，都自动提供了一个相应的**原始类型**(raw type)。**原始类型**的名字就是删去类型参数后的泛型类型名。擦除(erased)类型变量，并替换为限定类型：用第一个限定类型来替换，如果没有给定限定就用 Object 替换。当有多个限定时，编译器在必要时会执行类型转换（即切换限定），因此为了提高效率，应该将标签接口(即没有方法的接口)放在边界列表的末尾。

Java 泛型是使用类型擦除来实现的，任何具体的类型信息都会被擦除，因此在运行时JVM是不知道泛型信息的，即**JVM里没有泛型**。编译器虽然会在编译过程中移除参数的类型信息，但是会保证类或方法内部参数类型的一致性。当把一个具有泛型信息的对象赋予给另一个没有泛型信息的变量时，所有在尖括号之间的类型信息都将被扔掉。

### 推论
不管为泛型形参传入哪一种类型实参，对于Java来说，它们依然被当成同一个类处理，在内存中也只占用一块内存空间，因此在静态方法、静态初始化快或者静态变量的声明和初始化中不允许使用泛型形参。

泛型不能用于显式地引用运行时类型的操作之中，例如：转型、`instanceof`操作和`new`表达式，因为所有关于参数的类型信息都丢失了。


## 泛型方法和类型通配符
大多数时候都可以使用泛型方法来代替类型通配符。

泛型方法允许形参用来表示方法的一个或多个参数之间的类型依赖关系，或者方法返回值与参数之间的类型依赖关系。如果没有这样的**类型依赖关系**，就不应该使用泛型方法。

类型通配符被设计出来的目的就是为了**支持灵活的子类化**。如果一个泛型形参仅使用一次，其他参数的类型、方法返回值的类型都不依赖它，那么这个泛型形参就没有存在的必要，这时应该用通配符来代替它。

综上所述，默认使用通配符。

## 类型推断
使用泛型方法的时候，通常不必指明类型参数，因为编译器会为我们找出具体的类型。这称为类型参数推断（type argument inference）。类型推断主要有两个方面：
* 可通过调用方法的上下文来推断泛型的目标类型。
* 可在方法调用链中，将推断得到的泛型传递到最后一个方法。

泛型推断也不是万能的，因此也可以显式指定泛型的实际类型。

## 字母意义
* E — Element，常用在java Collection里，如：List<E>,Iterator<E>,Set<E>
* K,V — Key，Value，代表Map的键值对
* N — Number，数字
* T — Type，类型，如String，Integer等等

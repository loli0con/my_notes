# 序列化

* 对象的序列化（Serialize）指将一个Java对象写入IO流中：Java对象 --> byte[]
* 对象的反序列化（Deserialize）则指从IO流中恢复该Java对象：byte[] --> Java对象

## 可序列化
如果要让某个对象支持序列化机制，则它的类必须是可序列化的。为了让某个类是可序列化的，则该类必须实现如下两个接口之一：
* Serializable：Serializable是一个“标记接口”——它是一个空接口，没有定义任何方法。
* Externalizable

Serializable接口是一个标记接口（Marker Interface），实现了标记接口的类仅仅是给自身贴了个“标记”，并没有增加任何方法。

## 序列化
一旦某个类实现了Serializable接口，该类的对象就是可序列化的，程序可通过如下两个步骤序列化该对象：
1. 创建一个ObjectOutputStream，这个输出流是一个处理流，必须建立在其他节点流的基础之上。
2. 调用ObjectOutputStream对象的writeObject()方法输出可序列化对象。

## 反序列化
如果希望从二进制的内容中恢复对象，则需要使用反序列化，步骤如下：
1. 创建一个ObjectInputStream输入流，这个输入流是一个处理流，必须建立在其他节点流的基础之上。
2. 调用ObjectInputStream对象的readObject()方法读取流中的对象，该方法返回一个Object类型的Java对象。如果知道该Java对象的类型，则可以将该对象强制类型转换成其真实的类型。
3. 如果使用序列化机制向文件中写入了多个Java对象，使用反序列化机制恢复对象时必须按实际写入的顺序读取。

### 异常
readObject()可能抛出的异常有：
* ClassNotFoundException：没有找到对应的Class
* InvalidClassException：Class不匹配

#### ClassNotFoundException
反序列化读取的仅仅是Java对象的数据，而不是Java类，因此采用反序列化恢复Java对象时，必须提供该Java对象所属类的class文件，否则将会引发ClassNotFoundException异常。 

#### InvalidClassException
InvalidClassException，这种情况常见于序列化的`Person`对象定义了一个int类型的age字段，但是反序列化时，`Person`类定义的age字段被改成了long类型，所以导致class不兼容。

当一个可序列化类有多个父类时（包括直接父类和间接父类），这些父类要么有无参数的构造器，要么也是可序列化的——否则反序列化时将抛出InvalidClassException异常。如果父类是不可序列化的，只是带有无参数的构造器，则该父类中定义的成员变量值不会序列化到二进制流中。

### 注意
反序列化时，由JVM直接构造出Java对象，不调用构造方法，构造方法内部的代码，在反序列化时根本不可能执行。因此，它存在一定的安全隐患。


## 对象引用

### 递归序列化
当对某个对象进行序列化时，系统会自动把该对象的所有实例变量依次进行序列化，如果某个实例变量引用到另一个对象，则被引用的对象也会被序列化；如果被引用的对象的实例变量也引用了其他对象，则被引用的对象也会被序列化，这种情况被称为递归序列化。

如果某个类的成员变量的类型不是基本类型或String类型，而是另一个引用类型，那么这个引用类型必须是可序列化的，否则拥有该引用类型成员变量的类也是不可序列化的，无论是否实现Serializable、Externalizable接口。

### 序列化算法
Java序列化机制采用了一种特殊的序列化算法：
1. 所有保存到磁盘中的对象都有一个序列化编号
2. 当程序试图序列化一个对象时，程序将先检查该对象是否已经被序列化过，只有该对象从未（在本次虚拟机中）被序列化过，系统才会将该对象转换成字节序列并输出。
3. 如果某个对象已经被序列化过，程序将只是直接输出一个序列化编号，而不是再次重新序列化该对象。

当使用Java序列化机制序列化可变对象时一定要注意，只有第一次调用wirteObject()方法来输出对象时才会将对象转换成字节序列，并写入到ObjectOutputStream；在后面程序中即使该对象的实例变量发生了改变，再次调用writeObject()方法输出该对象时，改变后的实例变量也不会被输出。

## 自定义序列化
通过在实例变量前面使用transient关键字修饰，被transient修饰的实例变量将被完全隔离在序列化机制之外。

Java还提供了一种自定义序列化机制，通过这种自定义序列化机制，可以让程序控制如何实例化各实例变量。在序列化和反序列化过程中需要特殊处理的类应提供如下特殊签名的方法，这些特殊的方法用以实现自定义序列化：
* `private void writeObject(java.io.ObjectOutputStream out) throws IOException;`方法负责写入特定类的实例状态，以便相应的readObject()方法可以恢复它。通过重写该方法，程序员可以完全获得对序列化机制的控制，可以自主决定哪些实例变量需要序列化，需要怎样序列化。在默认情况下，该方法会调用out.defaultWriteObject来保存Java对象的各实例变量，从而可以实现序列化Java对象状态的目的。
* `private void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException;`方法负责从流中读取并恢复对象实例变量，通过重写该方法，程序员可以完全获得对反序列化机制的控制，可以自主决定需要反序列化哪些实例变量，以及如何进行反序列化。在默认情况下，该方法会调用in.defaultReadObject来恢复Java对象的非瞬态实例变量。在通常情况下，readObject()方法与writeObject()方法对应，如果writeObject()方法中对Java对象的实例变量进行了一些处理，则应该在readObject()方法中对其实例变量进行相应的反处理，以便正确恢复该对象。
* 当序列化流不完整时，`private void readObjectNoData() throws ObjectStreamException;`方法可以用来正确地初始化反序列化的对象。例如，接收方使用的反序列化类的版本不同于发送方，或者接收方版本扩展的类不是发送方版本扩展的类，或者序列化流被篡改时，系统都会调用readObjectNoData()方法来初始化反序列化的对象。

writeObject()方法存储实例变量的顺序应该和readObject()方法中恢复实例变量的顺序一致，否则将不能正常恢复该Java对象。

还有一种更彻底的自定义机制，它甚至可以在序列化对象时将该对象替换成其他对象。如果需要实现序列化某个对象时替换该对象，则应为序列化类提供如下特殊方法：`ANY-ACCESS-MODIFIER Object writeReplace() throws ObjectStreamException;`。此writeReplace()方法将由序列化机制调用，只要该方法存在。因为该方法可以拥有私有（private）、受保护的（protected）和包私有（package-private）等访问权限，所以其子类有可能获得该方法。Java的序列化机制保证在序列化某个对象之前，先调用该对象的writeReplace()方法，如果该方法返回另一个Java对象，则系统转为序列化另一个对象。根据上面的介绍，可以知道系统在序列化某个对象之前，会先调用该对象的writeReplace()和writeObject()两个方法，系统总是先调用被序列化对象的writeReplace()方法，如果该方法返回另一个对象，系统将再次调用另一个对象的writeReplace()方法……直到该方法不再返回另一个对象为止，程序最后将调用该对象的writeObject()方法来保存该对象的状态。

与writeReplace()方法相对的是，序列化机制里还有一个特殊的方法，它可以实现保护性复制整个对象。这个方法就是：`ANY-ACCESS-MODIFIER Object readResolve() throws ObjectStreamException;`。这个方法会紧接着readObject()之后被调用，该方法的返回值将会代替原来反序列化的对象，而原来readObject()反序列化的对象将会被立即丢弃。与writeReplace()方法类似的是，readResolve()方法也可以使用任意的访问控制符，因此父类的readResolve()方法可能被其子类继承。这样利用readResolve()方法时就会存在一个明显的缺点，就是当父类已经实现了readResolve()方法后，子类将变得无从下手。如果父类包含一个protected或public的readResolve()方法，而且子类也没有重写该方法，将会使得子类反序列化时得到一个父类的对象——这显然不是程序要的结果，而且也不容易发现这种错误。总是让子类重写readResolve()方法无疑是一个负担，因此对于要被作为父类继承的类而言，实现readResolve()方法可能有一些潜在的危险。通常的建议是，对于final类重写readResolve()方法不会有任何问题；否则，重写readResolve()方法时应尽量使用private修饰该方法。

## Externalizable —— 另一种自定义序列化机制
Java还提供了另一种序列化机制，这种序列化方式完全由程序员决定存储和恢复对象数据。要实现该目标，Java类必须实现Externalizable接口，该接口里定义了如下两个方法：
* void readExternal(ObjectInput in)：需要序列化的类实现readExternal()方法来实现反序列化。该方法调用DataInput（它是ObjectInput的父接口）的方法来恢复基本类型的实例变量值，调用ObjectInput的readObject()方法来恢复引用类型的实例变量值。
* void writeExternal(ObjectOutput out)：需要序列化的类实现writeExternal()方法来保存对象的状态。该方法调用DataOutput（它是ObjectOutput的父接口）的方法来保存基本类型的实例变量值，调用ObjectOutput的writeObject()方法来保存引用类型的实例变量值。

如果程序需要序列化实现Externalizable接口的对象，一样调用ObjectOutputStream的writeObject()方法输出该对象即可；反序列化该对象，则调用ObjectInputStream的readObject()方法。

当使用Externalizable机制反序列化对象时，程序会先使用public的无参数构造器创建实例，然后才执行readExternal()方法进行反序列化，因此实现Externalizable的序列化类必须提供public的无参数构造器。

两种序列化机制的对比：
|实现Serializable接口|实现Externalizable接口|
|---|---|
|系统自动存储必要信息|程序员决定存储哪些信息|
|Java内建支持，易于实现，只需要实现该接口即可，无须任何代码支持|仅仅提供两个空方法，实现该接口必须为两个空方法提供实现|
|性能略差|性能略好|

### 版本
Java序列化机制允许为序列化类提供一个private static final的serialVersionUID值，该类变量的值用于标识该Java类的序列化版本。如果一个类升级后，只要它的serialVersionUID类变量值保持不变，序列化机制也会把它们当成同一个序列化版本。

为了在反序列化时确保序列化版本的兼容性，最好在每个要序列化的类中加入`private static final long serialVersionUID`这个类变量，具体数值自己定义。这样，即使在某个对象被序列化之后，它所对应的类被修改了，该对象也依然可以被正确地反序列化。如果增加或修改了字段，可以改变serialVersionUID的值，这样就能自动阻止不匹配的class版本：
```Java
public class Person implements Serializable {
    private static final long serialVersionUID = 2709425275741743919L;
}
```

如果不显式定义serialVersionUID类变量的值，该类变量的值将由JVM根据类的相关信息计算，而修改后的类的计算结果与修改前的类的计算结果往往不同，从而造成对象的反序列化因为类版本不兼容而失败。可以通过JDK安装路径的bin目录下的serialver.exe工具来获得该类的serialVersionUID类变量的值。命令如下：`serialver Person`（Person为自定义的类名）。如果在运行serialver命令时指定-show选项（不要跟类名参数），即可启动序列版本检查器的图形用户界面。

不显式指定serialVersionUID类变量的值的另一个坏处是，不利于程序在不同的JVM之间移植。因为不同的编译器对该类变量的计算策略可能不同，从而造成虽然类完全没有改变，但是因为JVM不同，也会出现序列化版本不兼容而无法正确反序列化的现象。

如果类的修改确实会导致该类反序列化失败，则应该为该类的serialVersionUID类变量重新分配值。那么对类的哪些修改可能导致该类实例的反序列化失败呢？下面分三种情况来具体讨论：
1. 如果修改类时仅仅修改了方法，则反序列化不受任何影响，类定义无须修改serialVersionUID类变量的值。
2. 如果修改类时仅仅修改了静态变量或瞬态实例变量，则反序列化不受任何影响，类定义无须修改serialVersionUID类变量的值。
3. 如果修改类时修改了非瞬态的实例变量，则可能导致序列化版本不兼容。如果对象流中的对象和新类中包含同名的实例变量，而实例变量类型不同，则反序列化失败，类定义应该更新serialVersionUID类变量的值。如果对象流中的对象比新类中包含更多的实例变量，则多出的实例变量值被忽略，序列化版本可以兼容，类定义可以不更新serialVersionUID类变量的值；如果新类比对象流中的对象包含更多的实例变量，则序列化版本也可以兼容，类定义可“以不更新serialVersionUID类变量的值；但反序列化得到的新对象中多出的实例变量值都是null（引用类型实例变量）或0（基本类型实例变量）。
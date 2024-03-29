# 类图

## 定义
类class的定义：
* 具有相同属性、操作、方法、关系或者行为的一组对象的描述符
* 类是真实世界事物的抽象
* 问题领域的类:在对系统建模时，将会涉及到如何识别业务系统中的事物，这些事物构成了整个业务系统。在UML中，把所有的这些事物都建模为类 (class)

对象object的定义：
* 当这些事物存在于真实世界中时，它们是类的实例，并被称为对象
* 同一个类的各对象具有
  * 相同的属性，但属性的取值可以不同 
  * 提供相同的操作、有相同的语义
  
把类相关的元素画在一起，即为类图。

## 类图中常用的UML元素

### 实体

#### 具体类
具体类在类图中用矩形框表示，矩形框分为三层：第一层是类名字。第二层是类的成员变量；第三层是类的方法。成员变量以及方法前的访问修饰符用符号来表示：
* “+”表示 public；
* “-”表示 private；
* “#”表示 protected；
* 不带符号表示 default。

![class+20220220132603](https://raw.githubusercontent.com/loli0con/picgo/master/images/class%2B20220220132603.png%2B2022-02-20-13-26-04)

![class+20220220133354](https://raw.githubusercontent.com/loli0con/picgo/master/images/class%2B20220220133354.png%2B2022-02-20-13-33-54)

#### 抽象类
抽象类在UML类图中同样用矩形框表示，但是抽象类的类名以及抽象方法的名字都用斜体字表示。

![class+20220220132631](https://raw.githubusercontent.com/loli0con/picgo/master/images/class%2B20220220132631.png%2B2022-02-20-13-26-33)

#### 接口
接口在类图中也是用矩形框表示，但是与类的表示法不同的是，接口在类图中的第一层顶端用构造型`<<interface>>`表示，下面是接口的名字，第二层是方法。

![class+20220220132730](https://raw.githubusercontent.com/loli0con/picgo/master/images/class%2B20220220132730.png%2B2022-02-20-13-27-31)

#### 包
类和接口一般都出现在包。

![class+20220220133156](https://raw.githubusercontent.com/loli0con/picgo/master/images/class%2B20220220133156.png%2B2022-02-20-13-31-57)

### 关系
![class+20220220133210](https://raw.githubusercontent.com/loli0con/picgo/master/images/class%2B20220220133210.png%2B2022-02-20-13-32-11)

#### 实现关系
实现关系是指接口及其实现类之间的关系。在UML类图中，实现关系用空心三角和虚线组成的箭头来表示，从实现类指向接口。在Java代码中，实现关系可以直接翻译为关键字`implements`。

![class+20220220133455](https://raw.githubusercontent.com/loli0con/picgo/master/images/class%2B20220220133455.png%2B2022-02-20-13-34-55)

#### 泛化关系
泛化关系（Generalization）是指对象与对象之间的继承关系。如果对象A和对象B之间的“is a”关系成立，那么二者之间就存在继承关系，对象B是父对象，对象A是子对象。

在UML类图中，泛化关系用空心三角和实线组成的箭头表示，从子类指向父类。在Java代码中，对象之间的泛化关系可以直接翻译为关键字`extends`。

![class+20220220133556](https://raw.githubusercontent.com/loli0con/picgo/master/images/class%2B20220220133556.png%2B2022-02-20-13-35-56)


#### 关联关系
关联关系（Association）是指对象和对象之间的连接，它使一个对象知道另一个对象的属性和方法。在Java中，关联关系的代码表现形式为一个对象含有另一个对象的引用。也就是说，如果一个对象的类代码中，包含有另一个对象的引用，那么这两个对象之间就是关联关系。

关联关系有单向关联和双向关联。如果两个对象都知道（即可以调用）对方的公共属性和操作，那么二者就是双向关联。如果只有一个对象知道（即可以调用）另一个对象的公共属性和操作，那么就是单向关联。大多数关联都是单向关联，单向关联关系更容易建立和维护，有助于寻找可重用的类。

在UML图中，双向关联关系用带双箭头的实线或者无箭头的实线双线表示。单向关联用一个带箭头的实线表示，箭头指向被关联的对象。这就是导航性（Navigatity）。

![class+20220220133639](https://raw.githubusercontent.com/loli0con/picgo/master/images/class%2B20220220133639.png%2B2022-02-20-13-36-40)

![class+20220220134525](https://raw.githubusercontent.com/loli0con/picgo/master/images/class%2B20220220134525.png%2B2022-02-20-13-45-27)

一个对象可以持有其它对象的数组或者集合。在UML中，通过放置多重性（multipicity）表达式在关联线的末端来表示。多重性表达式可以是一个数字、一段范围或者是它们的组合。多重性允许的表达式示例如下：

数字：精确的数量
* \*或者0..*：表示0到多个
* 0..1：表示0或者1个，在Java中经常用一个空引用来实现
* 1..*：表示1到多个

关联关系又分为依赖关联、聚合关联和组合关联三种类型。

#### 依赖关系
依赖（Dependency）关系是一种弱关联关系。如果对象A用到对象B，但是和B的关系不是太明显的时候，就可以把这种关系看作是依赖关系。如果对象A依赖于对象B，则 A “use a” B。

在UML类图中，依赖关系用一个带虚线的箭头表示，由使用方指向被使用方，表示使用方对象持有被使用方对象的引用。

![class+20220220133822](https://raw.githubusercontent.com/loli0con/picgo/master/images/class%2B20220220133822.png%2B2022-02-20-13-38-23)

依赖关系在Java中的具体代码表现形式为B为A的构造器或方法中的局部变量、方法或构造器的参数、方法的返回值，或者A调用B的静态方法。

#### 聚合关系与组合关系
聚合（Aggregation）是关联关系的一种特例，它体现的是整体与部分的拥有关系，即 “has a” 的关系。此时整体与部分之间是可分离的，它们可以具有各自的生命周期，部分可以属于多个整体对象，也可以为多个整体对象共享，所以聚合关系也常称为共享关系。

在UML图中，聚合关系用空心菱形加实线箭头表示，空心菱形在整体一方，箭头指向部分一方。

![class+20220220134121](https://raw.githubusercontent.com/loli0con/picgo/master/images/class%2B20220220134121.png%2B2022-02-20-13-41-22)

组合（Composition）也是关联关系的一种特例，它同样体现整体与部分间的包含关系，即 “contains a” 的关系。但此时整体与部分是不可分的，部分也不能给其它整体共享，作为整体的对象负责部分的对象的生命周期。这种关系比聚合更强，也称为强聚合。如果A组合B，则A需要知道B的生存周期，即可能A负责生成或者释放B，或者A通过某种途径知道B的生成和释放。

在UML图中，组合关系用实心菱形加实线箭头表示，实心菱形在整体一方，箭头指向部分一方。

![class+20220220134148](https://raw.githubusercontent.com/loli0con/picgo/master/images/class%2B20220220134148.png%2B2022-02-20-13-41-49)

在Java代码形式上，聚合和组合关系中的部分对象是整体对象的一个成员变量。但是，在实际应用开发时，两个对象之间的关系到底是聚合还是组合，有时候很难区别。在Java中，仅从类代码本身是区分不了聚合和组合的。如果一定要区分，那么如果在删除整体对象的时候，必须删掉部分对象，那么就是组合关系，否则可能就是聚合关系。从业务角度上来看，如果作为整体的对象必须要部分对象的参与，才能完成自己的职责，那么二者之间就是组合关系，否则就是聚合关系。
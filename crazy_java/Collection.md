# 集合
![Collection+Collection](https://raw.githubusercontent.com/loli0con/picgo/master/images/Collection%2BCollection.png%2B2021-06-01-17-54-35)
## 遍历
有如下方式去遍历集合：
* Iterable.forEach方法
* Iterable.iterator方法，获取一个迭代器（*往后&能删*）
  * 使用迭代器的hasNext、next、remove等方法
  * 使用迭代器的forEachRemaining方法
* for/forEach循环
* Collection.stream方法，StreamAPI

***
## Set
Set的两个特征：
* 不允许包含相同的元素，即**不重复**
* 集合中的元素**无序**

### HashSet
HashSet是Set接口的典型实现，它按Hash算法来存储集合中的元素，有很好的存取和查找性能。

HashSet的特点：无序、非同步、值可为null。

#### hashCode 和 equals
有两个重要的方法，`hashCode`和`equals`，它们在使用HashSet时非常关键。

##### 存储规则
HashSet通过调用对象的hashCode()方法来得到该对象的hashCode值，然后**根据hashCode值来决定该对象在HashSet中的存储位置**。

##### 相同判定
HashSet集合判断两个元素相等的标准是**两个对象通过equals()方法比较相等，并且两个对象的hashCode()方法的返回值也相等**。

##### 结论
如果需要把某个类的对象保存到HashSet集合中，重写这个类的equals()方法和hashCode()方法时，应该尽量保证两个对象通过equals()方法比较返回true时，它们的hashCode()方法返回值也相等。

当把可变对象添加到HashSet中之后，尽量不要去修改该元素中参与计算hashCode()、equals()的实例变量，否则将会导致HashSet无法正确操作这些集合元素。

#### 桶
HashSet中每个能存储元素的“槽位（slot）”通常称为”桶（bucket）“，如果有多个元素的hashCode值相同，但它们通过equals()方法比较返回false，就需要在一个“桶”里放多个元素，这样会导致性能下降。

### LinkedHashSet
HashSet还有一个子类LinkedHashSet，它使用了*双向*链表记录集合元素的的添加顺序。

LinkedHashSet需要维护链表，所以性能略低于HashSet，但在迭代访问Set里的全部元素时将有很好的性能。

### TreeSet
TreeSet是SortedSet接口的实现类，可以确保集合**元素处于排序状态**。
TreeSet采用红黑树的数据结构来存储集合元素，它支持两种排序方式：自然排序和定制排序。默认采用自然排序。

#### 自然排序
TreeSet会调用集合元素的compareTo(Object obj)方法来比较元素之间的大小，然后将集合元素按升序排列，这种方式就是自然排序。

#### compareTo 和 equals
##### compareTo
Java提供了一个Comparable接口，该接口里定义了一个compareTo(Object obj)方法，该方法返回一个整数值，实现该接口的类必须实现该方法，实现了该接口的类的对象就可以比较大小。当一个对象调用该方法与另一个对象进行比较时，例如obj.compareTo(obj2)，如果该方法返回0，则表明这两个对象相等；如果该方法返回一个正整数，则表明obj1大于obj2；如果该方法返回一个负整数，则表明obj1小于obj2。

当把一个对象加入TreeSet集合中时，TreeSet调用该对象的compareTo(Object obj)方法与容器中的其他对象比较大小，然后根据红黑树结构找到它的存储位置。如果两个对象通过compareTo(Object obj)方法比较相等，新对象将无法添加到TreeSet集合中。

##### equals
当需要把一个对象放入TreeSet中，重写该对象对应类的equals()方法时，应保证该方法与compareTo(Object obj)方法有一致的结果。

##### 注意事项
同HashSet一样，推荐不要修改放入集合中元素的关键实例变量。

#### 定制排序
在创建TreeSet集合对象时，提供一个Comparator对象与该TreeSet集合关联，由该Comparator对象负责集合元素的排序逻辑，这样就可以实现定制排序。

Comparator是一个函数式接口，里面包含一个int compare(T o1,T o2)方法，该方法用于比较o1和o2的大小；如果方法返回正整数，则表明o1大于o2；如果该方法返回0，则表明o1等于o2；如果该方法返回负整数，则表明o1小于o2。

***
## List
List接口继承了Collection接口，还**增加了一些根据索引来操作集合元素的方法**。  
List额外提供了一个listIterator方法，该方法返回一个ListIterator对象，ListIterator接口继承了Iterator接口，提供了专门操作List的方法（*往前&能增*）。


List判断两个对象相等只要**通过equals()方法比较**返回true即可。

### ArrayList和Vector
Vector和它的子类Stack已经过时，不推荐使用，保留它仅为了向后兼容。Vector和ArrayList在用法上几乎完全相同，Vector是线程安全的，性能低。

ArrayList类封装了一个动态的、允许再分配的Object[]数组，ArrayList对象使用initialCapacity参数来设置该数组长度，当向ArrayList中添加元素超出该数组的长度时，他们的initialCapacity会自动增加。创建对象时可指定initialCapacity。

有两个方法可以重新分配Object[]：ensureCapacity 和 trmToSize。

ArrayList默认是线程不安全的，可以通过工具类将它变成线程安全的。

#### 注意
此*java.util.ArrayList*非彼*java.util.Arrays.ArrayList*

***
## Queue
Queue用于模拟**队列**这种数据结构——“**先进先出**（FIFO）”的容器。

### Priority
**优先队列**，根据队列元素的大小进行重新排序，**最小的元素先出队**。不可插入null元素，toString()方法返回值不可直接利用。

有两种排序方式（同TreeSet）：自然排序 和 定制排序。

## Deque
Deque接口是Queue接口的子接口，它代表一个**双端队列**，它里面定义了一些双端队列的方法。

Deque可以当成**栈**来使用。

### ArrayDeque
ArrayDeque是Deque接口典型的实现类，它是基于数组实现的双端队列，创建Deque时可指定一个numElement参数，用于指定Object[]的长度，默认为16。

### LinkedList
LinkedList实现了List接口（可以根据索引来随机访问集合中的元素），实现了Deque接口（可以当成双端队列来使用）。

LinkedList内部以链表的形式来保存集合中的元素。

### Array 和 Linked
[数组和链表的区别](https://zhuanlan.zhihu.com/p/52440208)

|区别|数组|链表|
|-----|-----|-----|
|优点|随机访问性强 & 查找速度快|插入删除速度快 & 内存利用率高，不会浪费内存 & 大小没有固定，拓展很灵活|
|缺点|插入和删除效率低 & 可能浪费内存 & 内存空间要求高，必须有足够的连续内存空间 & 数组大小固定，不能动态拓展|不能随机查找，必须从第一个开始遍历，查找效率低|
|典型类|ArrayList|LinkedList|
|保存形式|ArrayList和ArrayDeque内部以数组的形式来保存集合中的元素|LinkedList内部以链表的形式来保存集合中的元素|
|遍历方式|应该使用随机访问方法（get）|应该采用迭代器（Iterator）|
|如何选择|默认选择|需要经常插入和删除|

**注意表格中的“数组”指的是原生的数组，是ArrayXXX内部维护的Object[]，不是这个ArrayXXX本身，ArrayXXX实现了重新分配内部数组大小的功能**




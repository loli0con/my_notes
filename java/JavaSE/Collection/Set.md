# Set

## 特征
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

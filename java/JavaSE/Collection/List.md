# List
List接口继承了Collection接口，还**增加了一些根据索引来操作集合元素的方法**。

## listIterator
List额外提供了一个listIterator方法，该方法返回一个ListIterator对象，ListIterator接口继承了Iterator接口，提供了专门操作List的方法（*往前&能增*）。

## 方法
List\<E>接口的主要方法：
* 在末尾添加一个元素：boolean add(E e)
* 在指定索引添加一个元素：boolean add(int index, E e)
* 删除指定索引的元素：E remove(int index)
* 删除某个元素：boolean remove(Object e)
* 获取指定索引的元素：E get(int index)
* 获取链表大小（包含元素的个数）：int size()

## 实现类
List接口的两种实现：
* ArrayList（通过数组）
* LinkedList（通过链表）

ArrayList和LinkedList的区别（数组和链表的区别）：
|行为|ArrayList|LinkedList|
|---|---|---|
|获取指定位置的元素|速度很快|需要从头开始查找元素|
|添加元素到末尾|速度很快|速度很快|
|在指定位置添加/删除|需要移动元素|不需要移动元素|
|内存占用|少|较大|

通常情况下，总是优先使用ArrayList。

### ArrayList
ArrayList类封装了一个动态的、允许再分配的Object\[]数组，ArrayList对象使用initialCapacity参数来设置该数组长度，当向ArrayList中添加元素超出该数组的长度时，他们的initialCapacity会自动增加。创建对象时可指定initialCapacity。有两个方法可以改变Object[]数组的长度：ensureCapacity方法和trmToSize方法。

扩容操作需要调用 Arrays.copyOf() 把原数组整个复制到新数组中，这个操作代价很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。

## List和Array

### List转Array
把List变为Array的方法：
1. 调用toArray()方法直接返回一个Object\[]数组：`Object[] array = list.toArray();`
2. 调用toArray(T\[])传入一个类型相同的Array：`T[] array =list.toArray(new T[list.size()]);`
   1. 如果传入的数组不够大，那么List内部会创建一个新的刚好够大的数组，填充后返回
   2. 如果传入的数组比List元素还要多，那么填充完元素后，剩下的数组元素一律填充null
   3. 最好是传入一个“恰好”大小的数组
3. 调用toArray(IntFunction\<T[]> generator)方法：`T[] array = list.toArray(T[]::new);`

### Array转List
把Array变为List的方法：
1. 使用List.of(T...)方法把数组转换成List
2. 使用Arrays.asList(T...)方法把数组转换成List

要注意的是，如果方法返回的是一个**只读List**，对它调用add()、remove()方法会抛出UnsupportedOperationException。


## equals()
List内部并不是通过==判断两个元素是否相等，而是使用equals()方法判断两个元素是否相等。因此，要正确使用List的contains()、indexOf()这些方法，放入的实例必须正确覆写equals()方法。
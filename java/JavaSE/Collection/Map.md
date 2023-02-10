# Map
![Map+Map](https://raw.githubusercontent.com/loli0con/picgo/master/images/Map%2BMap.png%2B2021-06-01-23-22-13)



## 映射
Map是存储**键值（key-value）映射表**的数据结构，能高效通过key快速查找value。key和value可以是任何引用类型的数据，key不允许重复，即同一个Map对象的任何两个key通过equals方法比较总是返回flase。key和value之间存在单向一对一关系，即通过指定的key，总能找到唯一的、确定的value。


## Entry
Map提供了一个Entry内部类来封装key-value对，计算Entry存储规则时则只考虑Entry封装的key。

```Java
public interface Map<K,V> {
    interface Entry<K,V> {}
}
```



## equals() 和 hashCode()
在Map的内部，**对key做比较是通过equals()实现的**，正确使用Map必须保证：作为key的对象必须正确覆写equals()方法。**key计算索引的方式是调用key对象的hashCode()方法**，它返回一个int整数。

正确使用Map必须保证：
* 作为key的对象必须正确覆写equals()方法，相等的两个key实例调用equals()必须返回true；
* 作为key的对象必须正确覆写hashCode()方法，且hashCode()方法要严格遵循以下规范：
  * 如果两个对象相等，则两个对象的hashCode()必须相等；
  * 如果两个对象不相等，则两个对象的hashCode()尽量不要相等。




## HashMap
HashMap内部包含了一个Entry类型的数组`Node<K,V>[] table`（Node为实现类），Node包含了4个字段：
```Java
Static class Node<K,V> implements Map.Entry<K,V>{
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
    ...
}
```

[HashMap 源码详细分析](https://segmentfault.com/a/1190000012926722)

## LinkedHashMap
LinkedHashMap使用双向链表来维护key-value对的次序，该链表负责维护Map的迭代顺序，迭代顺序与key-value对的插入顺序保存一致。因为使用链表来维护内部顺序，所以在迭代访问Map的全部元素时将有较好的性能。

## SortedMap接口 和 TreeMap实现类
Map接口派生出了一个SortedMap子接口，SortedMap接口有一个TreeMap实现类。

TreeMap就是一个红黑树数据结构，每个key-value对即作为红黑树的一个节点。TreeMap存储key-value对（节点）时，需要根据key对节点进行排序，Key必须实现Comparable接口，或者在创建TreeMap时传入Comparator。